---
type: Note
domain: python
title: Python asyncio、协程与异步生成器
aliases: [asyncio, coroutine, await, yield, async generator, event loop]
description: 区分进程、线程、协程和生成器，理解 await、create_task、gather、to_thread 与 yield 在异步流式系统中的职责
tags: [python, asyncio, coroutine, await, yield, concurrency]
status: learning
created: 2026-08-23
updated: 2026-09-21
---

# 概述

- 进程是资源隔离容器，线程是操作系统调度的执行单元，协程是线程内由 Event Loop 调度的可暂停任务，`yield` 则是生成器向调用方逐次产出数据的语法。
- `asyncio` 适合大模型 API、HTTP、WebSocket、数据库和队列等待等 I/O 密集场景；它依赖协程在 `await` 处主动让出执行权。
- `await` 只表示等待一个 Awaitable 并暂停当前协程，不自动产生并发；需要 `create_task()` 或 `gather()` 才能让多个协程同时推进。
- `async def` 与 `yield` 组合形成异步生成器，特别适合一边异步等待、一边把事件流式交给调用方。

# 详细内容

## 进程、线程与协程

- 一个进程拥有独立内存、Python 解释器和线程集合，进程之间默认不共享内存，适合 CPU 多核并行和故障隔离。
- 同一进程内的线程共享全局变量和文件资源，但各自有调用栈，需要锁处理数据竞争；线程适合包装同步阻塞 I/O 或会释放 GIL 的原生计算。
- Event Loop 通常运行在线程中，负责调度多个协程 Task；同一 Event Loop 线程在任意时刻只执行其中一个协程。
- 协程使用协作式调度，只有运行到 `await` 等挂起点才把控制权交回 Event Loop；长时间 CPU 计算且不让出的协程会卡住整个事件循环。
- 协程切换主要保存 Python 执行状态，通常比操作系统线程切换更轻量，因此适合大量并发等待任务。

## async、await 与 Task

- `async def` 定义协程函数；调用它只会得到协程对象，必须通过 `await`、`create_task()` 等方式驱动执行。
- `await operation()` 表示当前协程等待异步操作完成，等待期间 Event Loop 可以运行其他已经就绪的 Task。
- 连续写两个 `await` 仍然是串行执行：第二个操作要等第一个完成后才开始。
- `asyncio.create_task(coro)` 会立即把协程登记为独立 Task，使当前协程可以继续处理其他工作，再在需要结果时等待该 Task。
- `asyncio.gather(a(), b())` 同时推进多个协程并按参数顺序收集结果，适合相互独立的并行 I/O。
- `asyncio.to_thread(sync_fn)` 或 `run_in_executor()` 将同步阻塞函数放入线程池，避免文件解析、同步数据库或索引操作阻塞 Event Loop。

## yield、生成器与异步生成器

- `return` 返回最终结果并结束函数；`yield` 产出一个值后暂停函数，保留局部变量和执行位置，调用方请求下一个值时再恢复。
- 普通 `def` 中出现 `yield` 会得到同步生成器，通过 `for` 消费；它解决的是分批产出，不天然解决异步等待。
- `async def` 中出现 `yield` 会得到异步生成器，通过 `async for` 消费；它既能 `await` 外部数据，也能逐条 `yield` 给调用方。
- 在 `event = await queue.get(); yield event` 中，`await` 面向 Event Loop，解决“事件还没来”；`yield` 面向调用方，解决“事件来了以后逐条交付”。

## asyncio.Queue：协程间的生产者与消费者

- `asyncio.Queue` 是进程内的异步 FIFO 队列，连接生产数据与消费数据的协程；创建队列不会自动创建后台任务。
- `await queue.get()` 在队列为空时挂起当前协程，等生产者放入数据后恢复；事件循环仍可运行其他任务。有数据时可以立即返回，并非每次 await 都会实际让出执行权。
- `await queue.put(item)` 放入数据；设置正数 `maxsize` 后，队列满时会等待消费者取走数据，形成背压。默认 `maxsize=0` 表示不限制容量。
- `put_nowait(item)` 不等待，满时抛出 `asyncio.QueueFull`；`get_nowait()` 在空时抛出 `asyncio.QueueEmpty`。
- 一个队列中的每条数据只会被一次 `get()` 取走；多个消费者共享同一个队列是分担消费，不是广播。广播需要给每个订阅者独立队列，再逐个投递。
- `None` 可被应用约定为结束哨兵，但不是队列内置的关闭语义；业务数据本身可能为 `None` 时应使用独立哨兵对象。
- `task_done()` / `join()` 用于跟踪“取出的任务已处理完”，不等同于 `get()`；不使用完成计数时，不必机械加入这两个调用。
- 队列不是线程安全队列，也不是持久化消息中间件；跨线程提交需要事件循环的线程安全入口，进程退出后队列数据不会保留。

```python
import asyncio


async def main():
    queue = asyncio.Queue(maxsize=8)

    async def producer():
        await queue.put({"text": "hello"})
        await queue.put(None)

    async def consumer():
        while True:
            item = await queue.get()
            if item is None:
                break
            print(item["text"])

    await asyncio.gather(producer(), consumer())


asyncio.run(main())
```

## DeepTutor 中的对应关系

- `TurnRuntimeManager.start_turn()` 使用 `asyncio.create_task(self._run_turn(execution))` 后台启动 Turn，因此接口可以先返回 `turn_id`，前端再订阅事件。
- Orchestrator 将 `capability.run(context, bus)` 放入 Task，同时用 `async for` 消费 Bus，让 Agent 执行和事件输出并发推进。
- `StreamBus.subscribe()` 是异步生成器：没有事件时 `await queue.get()`，收到事件后 `yield event`，关闭时通过队列哨兵结束循环。
- 多个 KB 的预检索使用 `asyncio.gather()` 并发查询，整体耗时更接近最慢的单个查询，而不是所有 KB 延迟之和。
- LlamaIndex 的同步索引和检索会通过 `run_in_executor()` 转移到线程池，避免同步库阻塞 FastAPI 的主 Event Loop。
- `StreamEvent` 是被传递的数据对象，`StreamBus` 是负责广播这些对象的进程内异步通道；二者分别对应数据协议和通信机制。

## 常见误区

- `async def` 不代表函数已经后台运行；没有 `await` 或创建 Task，协程对象不会自行完成。
- `await` 不代表并行，它只提供可挂起点；并发来自多个已登记 Task 的交替推进。
- `asyncio` 不适合直接执行长时间纯 Python CPU 计算，这类任务应使用多进程、专门 Worker 或可释放 GIL 的实现。
- `create_task()` 后必须管理任务生命周期和异常，不能只创建引用后完全丢弃，否则异常可能无法被正常观察和持久化。
- Event Loop 并发通常不等于多核并行，它的优势是等待期间不闲置线程，而不是让 Python 计算自动加速。

# 相关链接

- [2026-09-21 学习日志](../../daily/2026-09-21.md)
- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [Human-in-the-loop 与暂停恢复面试题](../../interview/agents/human-in-the-loop.md)
- [KindPartner 项目答辩](../../interview/resume/kindpartner.md)
- [2026-08-23 学习日志](../../daily/2026-08-23.md)
