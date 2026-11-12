---
type: Note
domain: python
title: Python 可变默认参数陷阱与 dataclass 的 default_factory
aliases: [可变默认值, 默认参数共享, mutable default argument, default_factory]
description: 默认参数在 def 执行时只求值一次、存进 __defaults__ 被所有调用共享；None 哨兵与 field(default_factory=...) 两种解法；dataclass 只拦 list/dict/set，自定义可变对象照样漏
tags: [python, 默认参数, dataclass, 易错点]
status: learning
created: 2026-08-21
updated: 2026-08-21
---

# 概述

- Python 的默认参数值在 `def` 执行时**只求值一次**，对象挂在函数上被后续所有缺省调用共享——可变默认值（list/dict/set/自定义实例）会被跨调用污染；函数里用 None 哨兵解，dataclass 里用 `field(default_factory=工厂)` 解

# 详细内容

## 根源：`def` 是可执行语句，默认值只求值一次

- 解释器执行到 `def` 那一行时：求值所有默认值表达式 → 把**结果对象**存进函数对象的 `__defaults__` 元组
- 之后每次调用缺省该参数，拿到的都是**当初那个对象的引用**——求值一次，共享 N 次

## 最小复现与证据

```python
def add(item, bucket=[]):
    bucket.append(item)
    return bucket

add(1)   # [1]
add(2)   # [1, 2]      ← 不是 [2]:上一次调用的残留
add(3)   # [1, 2, 3]   ← 越调越脏

def f(x=[]): return x
f() is f()        # True —— 同一个对象
f.__defaults__    # ([1, 2, 3],) —— 默认值就存在函数对象上
```

## 为什么 `x=0`、`name=""` 从不出事

- 坑只对**可变对象**发作：不可变默认值（int/str/None/tuple）没法原地改，`count += 1` 是"造新对象 + 局部名重绑定"，函数里存的那个 `0` 不动
- `bucket.append(...)` 是**原地修改共享对象本身**——区别在"重绑定 vs 原地改"，不在语法

## 解法一：None 哨兵（普通函数）

```python
def add(item, bucket=None):
    if bucket is None:
        bucket = []   # 每次调用现场 new 一个
    bucket.append(item)
    return bucket
```

- `None` 不可变、共享无害；真正要的可变对象在函数体内**每次调用时新建**

## 解法二：dataclass 的 `field(default_factory=...)`

- dataclass 直接禁止内置可变默认值，类创建时即报错：
  ```python
  @dataclass
  class Turn:
      events: list = []
  # ValueError: mutable default <class 'list'> for field events
  # is not allowed: use default_factory
  ```
- **但检查是启发式的，只拦内置 list/dict/set**（新版含 bytearray）；自定义可变对象不拦：
  ```python
  @dataclass
  class Turn:
      lock: asyncio.Lock = asyncio.Lock()   # 不报错!但所有实例共享同一把锁
  ```
- 正确写法——传**工厂函数本身**，不是调用结果：
  ```python
  @dataclass
  class Turn:
      events: list = field(default_factory=list)          # ✔
      lock: asyncio.Lock = field(default_factory=asyncio.Lock)
  # default_factory=list()  ✘ 反而报错
  ```
- 语义：生成的 `__init__` 没收到该参数时**调用一次工厂**，每实例拿到崭新对象。大致等价于：
  ```python
  def __init__(self, events=None, ...):
      self.events = list() if events is None else events
  ```

## 真实出处（DeepTutor `_TurnExecution`）

- `persisted_events: list = field(default_factory=list)`——若不隔离，A turn 落库的事件混进 B turn 的列表，下次 flush 被当 B 的再镜像一遍：**跨 turn 数据污染**
- `flush_lock: asyncio.Lock = field(default_factory=asyncio.Lock)`——共享则所有 turn 的落库被一把锁全局串行，违背"每 turn 一把锁"的设计
- 配套知识：dataclass 字段声明 `名字: 类型 = 默认值`；不可变默认值可直接写（`flag: bool = False`)

# 相关链接

-
