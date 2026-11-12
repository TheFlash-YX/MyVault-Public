AI 应用开发 / AI Agent 研发工程师
秋招面试问题库
基于个人简历定制 · 仅问题版
已删除：LLM 原理类问题、科研论文相关问题
使用建议：优先准备标注为【最高优先级】和【高优先级】的模块。针对简历中出现的每一个技术名词，至少准备“是什么 → 为什么这样设计 → 怎么实现 → 异常场景 → 替代方案/扩展方案”五层追问。
## 目录与优先级

一、自我介绍与项目总览（15 题）
二、KindPartner：Agent Runtime 核心设计【高优先级】（31 题）
三、ask_user 内容澄清机制【最高优先级】（49 题）
四、Agent 流式输出 / SSE【高优先级】（21 题）
五、Function Calling / Tool System【高优先级】（30 题）
六、MCP【高优先级】（31 题）
七、Memory 系统【最高优先级】（43 题）
八、Partner / Workspace / 多租户隔离【高优先级】（25 题）
九、RAG 基础【最高优先级】（45 题）
十、LlamaIndex / LightRAG / RAG Provider【高优先级】（32 题）
十一、Prompt Engineering / Agent 安全（20 题）
十二、Agent Evaluation【最高优先级】（27 题）
十三、KindVideo：标注探针池（27 题）
十四、困难样本 / 状态机 / 并发（27 题）
十五、数据库 / MySQL（24 题）
十六、Spring Boot / Java（20 题）
十七、FastAPI / Python / asyncio【高优先级】（22 题）
十八、Redis / Kafka / 中间件（18 题）
十九、Docker / 部署 / Production（23 题）
二十、AI Coding / Claude Code / Codex / SDD / TDD（25 题）
二十一、系统设计题：Agent 岗高频【最高优先级】（29 题）
二十二、场景题 / 故障排查题【最高优先级】（27 题）
二十三、简历数据真实性追问【最高优先级】（19 题）
二十四、行为面 / 实习经历（19 题）
二十五、求职动机 / 岗位匹配（16 题）
二十六、简历技术细节“卡真实性”连续追问【最高优先级】（33 题）
二十七、AI Agent 开放题【高优先级】（27 题）
## 一、自我介绍与项目总览

1. 请用 2～3 分钟介绍一下自己，以及为什么想做 AI 应用 / AI Agent 方向？
2. 你的本科是信息管理与信息系统，硕士是网络空间安全，为什么最后选择 AI Agent 开发？
3. 你认为自己相比纯后端开发或者纯算法同学，做 AI 应用开发的优势是什么？
4. 你在视源股份实习期间主要负责什么工作？
5. KindVideo 和 KindPartner 两个项目分别解决什么问题？
6. 这两个项目中，你个人负责的部分分别有哪些？
7. 哪些功能是你完全独立设计和开发的？
8. 哪个项目最能体现你的 AI Agent 开发能力？
9. 你认为自己在这段实习中技术上最大的成长是什么？
10. 你做过的最复杂的技术设计是什么？
11. 遇到过最棘手的线上问题是什么？
12. 如果让我只看你简历中的一个项目，你希望我重点看哪个？为什么？
13. 你简历里很多地方用了“设计”，你具体承担的是架构设计、模块设计，还是需求实现？
14. 你的项目有没有真正上线？目前有多少用户或者业务量？
15. 项目中你做的工作是 PoC、Demo，还是生产级系统？
## 二、KindPartner：Agent Runtime 核心设计【高优先级】

16. 介绍一下 KindPartner 整体的 Agent 架构。
17. 一个用户请求进入系统之后，完整执行链路是什么？
18. 你的 Agent Loop 是怎么实现的？
19. 一轮 Agent Loop 包含哪些阶段？
20. Agent 在什么情况下继续调用工具，什么情况下结束？
21. 如何防止 Agent 无限循环？
22. 最大迭代次数如何设置？
23. 如果模型一直重复调用同一个工具怎么办？
24. Tool Calling 的状态如何维护？
25. Tool Result 是如何重新传递给模型的？
26. 多次 Tool Call 是串行还是并行？
27. 如果模型一次返回多个 Tool Call，你们怎么处理？
28. Agent Runtime 和业务层之间如何解耦？
29. 为什么设计 UnifiedContext？
30. UnifiedContext 里面包含哪些信息？
31. Context 太大会不会导致 Prompt 过长？
32. 你如何管理 Agent 的上下文窗口？
33. Agent 的 System Prompt 是怎么组织的？
34. 不同 Partner 如何共享 Runtime 又保证行为不同？
35. 为什么不直接为每个 Partner 单独创建一套 Agent？
36. 如果未来有几百种 Partner，当前架构能扩展吗？
37. Agent Runtime 有没有状态机？
38. 你的 Runtime 和 LangGraph 这类框架有什么区别？
39. 为什么没有直接使用 LangGraph？
40. 如果现在让你重构，你会不会换成 LangGraph / OpenAI Agents SDK / Claude Agent SDK 一类框架？
41. Agent Runtime 中哪些状态必须持久化，哪些可以只存在内存？
42. Agent 的可恢复执行需要保存哪些信息？
43. 如果某个 Tool 有副作用，Agent 重试时如何避免重复执行？
44. 你如何区分模型生成阶段、工具执行阶段和结果汇总阶段？
45. 如果 Tool 调用耗时很长，Runtime 如何支持取消？
46. Runtime 中如何统一处理异常？
## 三、ask_user 内容澄清机制【最高优先级】

47. 为什么需要设计 ask_user 工具？
48. 普通 Agent 遇到信息不足时一般怎么处理？
49. 你这里的 ask_user 和“模型直接回复用户一个问题”有什么区别？
50. 为什么一定要保持在同一个 Turn 中？
51. 什么叫 Turn？
52. 为什么重新创建 Turn 会丢失调用上下文？
53. ask_user 被调用之后，Agent Loop 发生了什么？
54. “挂起 Agent Loop”具体是怎么实现的？
55. 是阻塞线程吗？
56. 为什么使用异步队列？
57. Turn 级异步队列的数据结构是什么？
58. 每个 Turn 都维护一个 Queue 吗？
59. Queue 的生命周期怎么管理？
60. 用户回复后，你怎么找到对应的 Turn？
61. 如果用户同时打开两个对话，会不会回复错 Turn？
62. 如果同一个 Turn 中连续调用两次 ask_user 怎么办？
63. 如果用户一直不回复怎么办？
64. 是否有 timeout？
65. timeout 以后 Agent 状态怎么处理？
66. 用户关闭网页后怎么办？
67. 服务重启以后挂起的 Agent 怎么恢复？
68. 当前实现是内存队列还是持久化队列？
69. 如果是内存队列，多实例部署会不会出问题？
70. 如果部署三个 FastAPI 实例，用户请求到了不同实例怎么办？
71. 如果升级成分布式系统，你准备怎么改？
72. Redis Stream、Kafka、RabbitMQ 和 asyncio.Queue，你会怎么选？
73. Tool Result 是怎么重新插入原调用链的？
74. 重新恢复之后，模型看到的 messages 是什么结构？
75. 为什么可以保证不会重复执行之前的 Tool？
76. 你写“重复执行率 0%”，这个指标怎么测出来的？
77. 98% 的恢复成功率是怎么统计的？
78. 剩下 2% 失败的原因是什么？
79. ask_user 有没有 race condition？
80. 用户在 Agent 挂起之前就提交补充信息会怎么样？
81. 前端和后端如何同步“等待用户输入”的状态？
82. StreamEvent 是什么？
83. StreamEvent 有哪些事件类型？
84. SSE 中如何区分 token、tool call、ask_user、done、error？
85. 如果 SSE 断线了怎么办？
86. SSE 重连后如何恢复之前的事件？
87. 为什么用 SSE 而不是 WebSocket？
88. WebSocket 相比 SSE 有什么优缺点？
89. ask_user 本质上是不是 Human-in-the-loop？
90. 和 LangGraph interrupt / resume 机制相比有什么区别？
91. 如果用户修改之前的问题，而不是回答澄清问题，怎么处理？
92. 挂起期间如何支持用户主动取消当前任务？
93. 恢复时如何确认用户回复属于当前 ask_user，而不是普通新消息？
94. 如果用户重复提交同一条澄清回复，如何保证幂等？
95. 如果前端收到 ask_user 事件后刷新页面，如何恢复 UI 状态？
## 四、Agent 流式输出 / SSE【高优先级】

96. SSE 的底层原理是什么？
97. SSE 和普通 HTTP 请求有什么区别？
98. SSE 使用什么 Content-Type？
99. SSE 为什么天然适合大模型流式输出？
100. SSE 是单向还是双向通信？
101. 浏览器断开连接后后端如何感知？
102. FastAPI 怎么实现 StreamingResponse？
103. 模型 token 是如何一路传到 React 前端的？
104. 如何解决 token 输出和工具调用事件混杂的问题？
105. 一个 Agent 回答期间发生多个工具调用，你怎么组织事件流？
106. 如何保证事件顺序？
107. SSE 传输过程中发生异常怎么处理？
108. 如果模型生成速度比前端消费速度快怎么办？
109. 有没有 backpressure 问题？
110. nginx 等反向代理会不会缓冲 SSE？
111. SSE 心跳怎么设计？
112. 为什么不用 WebSocket？
113. 如果后续加入语音实时对话，你还会选择 SSE 吗？
114. 如何给每个 StreamEvent 分配 event id？
115. 前端如何避免重复消费重连后的事件？
116. 如何记录一次完整 Agent 流式请求的 trace？
## 五、Function Calling / Tool System【高优先级】

117. Function Calling 的基本原理是什么？
118. 模型为什么知道有哪些工具可以调用？
119. Tool Schema 如何传给模型？
120. JSON Schema 在 Function Calling 中起什么作用？
121. 如果模型生成非法参数怎么办？
122. 你们是否使用 Pydantic 做参数校验？
123. Tool 调用失败以后怎么处理？
124. Tool exception 会直接返回给模型吗？
125. Tool Result 应该包含多少信息？
126. Tool Result 太长怎么办？
127. Tool description 怎么写会更容易被模型正确调用？
128. 工具数量过多会发生什么？
129. 如果有 100 个工具，是否全部放入 Prompt？
130. 如何做工具动态加载？
131. 如何做工具检索？
132. 你简历里写“工具白名单”，具体如何实现？
133. 如何防止 Partner 调用没有权限的工具？
134. 权限检查应该在 Prompt 层还是代码层？
135. 为什么不能只通过 Prompt 告诉模型“不要调用某工具”？
136. 如果模型绕过 Prompt 直接构造 Tool Call 怎么办？
137. Tool 权限系统如何设计？
138. Tool 是否应该支持版本管理？
139. 两个 Tool 名字或者语义非常接近时怎么办？
140. 如何评价一个 Tool Calling 系统的效果？
141. Tool Calling 的准确率怎么测试？
142. 如何构建 Agent Tool Benchmark？
143. 有副作用的 Tool 和只读 Tool 在执行策略上有什么区别？
144. 发送邮件、删除数据、支付这类 Tool 是否需要人工确认？
145. 如何设计 Tool timeout、retry 和 circuit breaker？
146. 如果 Tool 已执行成功但响应丢失，Agent 应该怎么处理？
## 六、MCP【高优先级】

147. MCP 是什么？
148. MCP 解决了什么问题？
149. MCP 和 Function Calling 有什么区别？
150. MCP Server 是什么？
151. MCP Client 是什么？
152. Host、Client、Server 三者分别是什么？
153. MCP 中 Tool、Resource、Prompt 分别是什么？
154. MCP Tool 是如何被 Agent 发现的？
155. MCP 的 capability negotiation 是什么？
156. MCP 支持哪些 Transport？
157. stdio 和 HTTP 模式分别适合什么场景？
158. 一个本地 MCP Server 怎么启动？
159. MCP Server 如何定义 Tool Schema？
160. 如果一个 MCP Server 挂掉怎么办？
161. 多个 MCP Server 的工具重名怎么办？
162. MCP Tool 的鉴权怎么处理？
163. MCP 是否存在安全风险？
164. 如果 MCP Server 是第三方提供的，你会如何隔离？
165. MCP Tool 能否访问用户本地文件？
166. 如何防止恶意 MCP Server 获取敏感信息？
167. MCP 和传统 OpenAPI Plugin 有什么区别？
168. MCP 和 REST API 有什么区别？
169. 为什么现在 Agent 领域越来越多使用 MCP？
170. 如果让你设计一个企业内部 MCP 平台，你会怎么设计？
171. MCP Server 如何动态注册和卸载？
172. MCP Tool 是否需要做权限控制和审计？
173. 你实际接过哪些 MCP Server？
174. 你使用 MCP 的真实业务场景是什么？
175. 如何对 MCP Server 做健康检查？
176. 如何限制第三方 MCP Server 的网络和文件系统权限？
177. 企业内部多个团队都提供 MCP Server 时，如何做治理？
## 七、Memory 系统【最高优先级】

178. 为什么 Agent 需要 Memory？
179. Context 和 Memory 有什么区别？
180. Short-term Memory 和 Long-term Memory 有什么区别？
181. 为什么你们设计成 L1、L2、L3 三层？
182. L1 Trace 保存什么？
183. 为什么不能直接把聊天记录全部当记忆？
184. L2 是如何从 L1 提取的？
185. Memory Consolidator 是模型还是规则？
186. Memory Consolidator 的 Prompt 怎么设计？
187. 哪些内容应该进入长期记忆？
188. 哪些内容绝对不应该进入长期记忆？
189. L3 画像是如何形成的？
190. 为什么 L3 要引用 L2，L2 再引用 L1？
191. 引用链有什么价值？
192. 如果 L3 画像是错误的怎么办？
193. 用户修改 L2 以后 L3 是否重新生成？
194. Memory 是否会出现冲突？
195. 用户之前说“喜欢数学”，后来又说“不喜欢数学”，怎么处理？
196. Memory 是否需要时间衰减？
197. Memory 是否需要置信度？
198. Memory 如何做去重？
199. Memory 如何做合并？
200. Memory 怎么检索？
201. 是语义检索还是结构化查询？
202. 每轮对话需要把所有 Memory 放进 Prompt 吗？
203. 如何选择当前 Turn 相关的 Memory？
204. Memory 很多以后，检索性能怎么处理？
205. Memory 会不会污染模型输出？
206. 你们为什么强调“后台静默沉淀率 0%”？
207. 为什么让用户主动决定是否写入 Memory？
208. 这种设计相比自动 Memory 有什么优劣？
209. 隐私方面怎么处理？
210. 如何支持用户删除记忆？
211. 删除 L1 后，L2/L3 怎么办？
212. 删除 L2 后引用它的 L3 怎么办？
213. Memory 是否应该做软删除？
214. Memory 数据是否需要加密？
215. 你写“引用覆盖率 100%”，怎么定义这个指标？
216. 记忆系统如何做测试？
217. 如何判断某条 Memory 已经失效？
218. 不同 Partner 是否应该共享 Memory？
219. 如何防止 Memory Retrieval 把其他用户的信息召回？
220. 如果 Memory 数量达到百万级，你会如何设计存储和检索？
## 八、Partner / Workspace / 多租户隔离【高优先级】

221. Partner 是什么概念？
222. Workspace 是什么？
223. 为什么要为不同 Partner 创建独立 Workspace？
224. Soul 是什么？
225. Soul 和 System Prompt 有什么区别？
226. Skill 是什么？
227. Notebook 是什么？
228. Partner 的课程知识库怎么挂载？
229. Partner 配置是在创建时加载还是每轮动态加载？
230. UnifiedContext 如何装配这些组件？
231. 如何保证 Partner A 读不到 Partner B 的数据？
232. “跨作用域泄漏 0%”怎么测？
233. “越权调用拦截率 100%”怎么测？
234. 隔离是在数据库层、Service 层还是 Agent Runtime 层？
235. 如果模型明确要求读取另一个 Partner 的 Memory，会发生什么？
236. 为什么“学生记忆只读、关系记忆独立可写”？
237. 关系记忆是什么？
238. Partner 删除以后，其关系记忆怎么处理？
239. 多个 Partner 能否共享课程知识库？
240. 如果共享知识库，权限怎么控制？
241. 这个系统本质上是否是一个多租户系统？
242. 多租户系统有哪些常见隔离方案？
243. 如果未来服务十万学生，这套架构有哪些瓶颈？
244. Workspace 的配置如何版本化？
245. Partner 升级 Skill 后，历史会话怎么兼容？
## 九、RAG 基础【最高优先级】

246. 什么是 RAG？
247. 为什么需要 RAG？
248. RAG 和 Fine-tuning 有什么区别？
249. 什么情况下应该用 RAG，什么情况下应该微调？
250. 一个完整的 RAG Pipeline 包含哪些环节？
251. 文档上传之后到最终回答经历哪些步骤？
252. Chunking 为什么重要？
253. Chunk size 怎么选？
254. Chunk overlap 为什么存在？
255. Chunk 太大会有什么问题？
256. Chunk 太小会有什么问题？
257. Semantic Chunking 和固定长度 Chunk 有什么区别？
258. Embedding 是什么？
259. Embedding 模型怎么选？
260. 中文场景下你会选择什么 Embedding 模型？
261. Dense Retrieval 是什么？
262. Sparse Retrieval 是什么？
263. BM25 原理是什么？
264. Dense Retrieval 和 BM25 怎么结合？
265. Hybrid Search 是什么？
266. Reranker 有什么作用？
267. Cross Encoder 和 Bi Encoder 有什么区别？
268. Top-K 怎么选？
269. RAG 中 Recall 和 Precision 哪个更重要？
270. 如果召回内容正确但模型回答错了，问题在哪里？
271. 如果召回不到答案，怎么排查？
272. 如何降低 RAG 幻觉？
273. 怎么让回答可以引用来源？
274. Citation 怎么和生成内容对应？
275. 如何避免模型伪造 Citation？
276. Query Rewrite 有什么作用？
277. Multi-query Retrieval 是什么？
278. HyDE 是什么？
279. Parent-Child Retrieval 是什么？
280. Metadata Filtering 是什么？
281. RAG 如何做权限过滤？
282. ACL Filter 应该放在召回前还是召回后？
283. RAG 的 Context 太长怎么办？
284. Long Context 能否完全替代 RAG？
285. Knowledge Graph RAG 和普通 Vector RAG 有什么区别？
286. 文档更新后如何处理旧索引？
287. 同一文档不同版本如何避免重复召回？
288. PDF 中表格、公式、图片如何进入 RAG？
289. 多模态 RAG 和文本 RAG 的主要差异是什么？
290. 如何做查询缓存和检索缓存？
## 十、LlamaIndex / LightRAG / RAG Provider【高优先级】

291. 为什么同时使用 LlamaIndex 和 LightRAG？
292. LlamaIndex 的核心抽象有哪些？
293. LlamaIndex 中 Index、Retriever、Query Engine 分别是什么？
294. LightRAG 和传统向量 RAG 有什么不同？
295. LightRAG 为什么需要图结构？
296. 什么类型的数据适合 LightRAG？
297. 什么类型的数据更适合普通 Vector RAG？
298. 你们为什么设计 RAGProvider？
299. Provider Pattern 是什么？
300. RAGPipeline 协议里定义了哪些方法？
301. RAGService 如何路由不同 Provider？
302. 一个知识库为什么需要绑定 Provider？
303. 能不能同一个知识库同时跑两个 Provider？
304. 如果以后新增 GraphRAG，需要改多少代码？
305. 如何做到 Provider 插拔？
306. 不同 RAG Provider 返回格式不一致怎么处理？
307. 你们统一返回的数据结构是什么？
308. 来源 Citation 如何归一化？
309. Provider 异常时是否支持降级？
310. LlamaIndex 挂了是否自动切 LightRAG？
311. RAG Provider 是否使用工厂模式 / 策略模式？
312. 这部分为什么不用简单 if-else？
313. F1 78% 是怎么计算出来的？
314. Recall@5 91% 是怎么计算的？
315. 测试集有多少条数据？
316. Ground Truth 从哪里来？
317. Recall@5 中的“相关文档”如何定义？
318. 你们如何评价生成答案而不仅仅是 Retrieval？
319. 有没有使用 RAGAS？
320. 有没有使用 LLM-as-a-Judge？
321. LLM-as-a-Judge 有什么问题？
322. RAG 评测如何避免数据污染？
## 十一、Prompt Engineering / Agent 安全

323. 你如何理解 Prompt Engineering？
324. System Prompt、User Prompt、Tool Description 分别有什么作用？
325. Prompt 太长会产生什么问题？
326. Instruction hierarchy 是什么？
327. 什么是 Prompt Injection？
328. RAG 场景中的 Prompt Injection 怎么产生？
329. 用户上传的 PDF 中包含“忽略之前所有指令”怎么办？
330. 如何防止间接 Prompt Injection？
331. System Prompt 能完全防住 Prompt Injection 吗？
332. Tool Description 是否可能成为攻击入口？
333. 如何防止 Agent 被诱导调用危险工具？
334. Prompt 如何版本管理？
335. Prompt 改动如何做回归测试？
336. 如何评价两个 Prompt 哪个更好？
337. 有没有做过 A/B Test？
338. Few-shot 示例什么时候有效？
339. Structured Output 和自由文本各适合什么场景？
340. 如何防止模型把检索内容中的指令当作系统指令执行？
341. Agent 是否应该有 Tool Allowlist / Denylist？
342. 如何做高风险 Tool 的二次确认？
## 十二、Agent Evaluation【最高优先级】

343. 一个 Agent 应该怎么评测？
344. 普通 LLM Benchmark 为什么不能直接评价 Agent？
345. Agent Evaluation 可以拆成哪些维度？
346. Tool Selection Accuracy 怎么计算？
347. Tool Argument Accuracy 怎么计算？
348. Task Completion Rate 怎么定义？
349. Agent 的“成功”怎么定义？
350. 如何评估多步任务？
351. Agent trajectory 应该怎么评价？
352. 最终答案正确，但工具调用路径非常差，算成功吗？
353. 如何衡量 Token Cost？
354. 如何衡量 Latency？
355. 如何衡量 Tool Call 数量？
356. 如何评价 Agent 是否发生无效循环？
357. 如何搭建 Agent Regression Test？
358. 模型升级以后怎么保证 Agent 不退化？
359. Prompt 更新以后怎么回归？
360. Tool Schema 修改以后怎么回归？
361. 如何构建 Agent 测试数据集？
362. 测试集应该来自人工构造还是线上日志？
363. 如何处理线上日志中的隐私信息？
364. Deterministic Test 和 LLM Judge Test 怎么结合？
365. LLM-as-a-Judge 如何降低随机性？
366. Agent Eval 是否应该跑多次取平均值？
367. 如何评价 Human-in-the-loop Agent？
368. Agent 的恢复成功率应该如何测试？
369. 如何构建针对 Tool 重复调用、错误调用和死循环的回归集？
## 十三、KindVideo：标注探针池

370. 什么是标注探针？
371. 为什么需要探针？
372. “可信样本”是怎么产生的？
373. 为什么要隐蔽注入？
374. 如果标注人员知道哪些是探针，会发生什么？
375. 探针的注入比例怎么确定？
376. 探针是否会重复分配？
377. 你怎么避免探针被过度使用？
378. 什么叫“统一策略接口”？
379. 为什么不同 task_kind 需要不同匹配策略？
380. 策略模式在这里是怎么应用的？
381. 快照构建是什么？
382. 为什么审核前需要快照？
383. 如果原始数据在审核过程中被修改怎么办？
384. 匹配阈值怎么配置？
385. 不同任务的匹配算法分别是什么？
386. 自动审核逻辑是什么？
387. 什么情况下转人工审核？
388. 探针结果为什么要回流？
389. 为什么回流必须幂等？
390. 幂等是怎么实现的？
391. 使用数据库唯一索引还是业务 key？
392. 如果接口被重复调用 10 次，会发生什么？
393. 7000+ 探针是怎么累计的？
394. 探针池会不会出现错误样本污染？
395. 如何检测探针本身的质量？
396. 探针准确率如果下降怎么办？
## 十四、困难样本 / 状态机 / 并发

397. 什么叫困难样本？
398. 为什么单人无法裁定？
399. 多人投票机制如何设计？
400. 投票结果如何计算？
401. 为什么需要管理员最终裁定？
402. “派生状态机”是什么意思？
403. 和显式 status 字段有什么区别？
404. 状态有哪些？
405. 状态迁移条件有哪些？
406. 为什么采用可配置票数阈值？
407. 两个人同时提交最后一票，会发生什么？
408. 为什么需要乐观锁？
409. 乐观锁原理是什么？
410. 乐观锁和悲观锁区别是什么？
411. 你的乐观锁具体怎么实现？
412. version 字段更新失败以后怎么办？
413. 是否重试？
414. 重试多少次？
415. 如果高并发下大量冲突怎么办？
416. 什么情况下应该改用悲观锁？
417. 探针池“硬排除”是什么意思？
418. 为什么不能只在业务代码里过滤？
419. 投票之后自动切下一题怎么实现？
420. 用户连续双击提交会不会重复投票？
421. 如何保证一个人只能投一次？
422. 完成周期缩短 35% 是怎么算出来的？
423. 这个指标有没有其他因素影响？
## 十五、数据库 / MySQL

424. MySQL 的事务四大特性是什么？
425. 四种事务隔离级别分别是什么？
426. MySQL 默认隔离级别是什么？
427. 什么是脏读、不可重复读、幻读？
428. MVCC 是什么？
429. InnoDB 如何实现 MVCC？
430. undo log 是干什么的？
431. redo log 是干什么的？
432. binlog 是干什么的？
433. redo log 和 binlog 有什么区别？
434. 什么是聚簇索引？
435. 为什么 InnoDB 主键最好不要太长？
436. 联合索引最左匹配原则是什么？
437. 什么情况下索引会失效？
438. Explain 看哪些字段？
439. 如何判断一个 SQL 是否走索引？
440. 数据量从十万增长到一亿以后，你会怎么优化？
441. 分库分表什么时候需要做？
442. Offset Pagination 有什么问题？
443. Cursor Pagination 怎么实现？
444. 唯一索引如何实现幂等？
445. 数据库事务和分布式事务有什么区别？
446. 乐观锁更新 SQL 一般怎么写？
447. 为什么状态流转最好和版本号校验放在一个事务中？
## 十六、Spring Boot / Java

448. Spring IoC 是什么？
449. Dependency Injection 是什么？
450. Spring Bean 生命周期是什么？
451. Spring AOP 是什么？
452. @Transactional 原理是什么？
453. 为什么同类方法内部调用可能导致 @Transactional 失效？
454. Spring 事务什么时候会回滚？
455. Checked Exception 会不会默认回滚？
456. Spring Boot 自动配置原理是什么？
457. MyBatis 和 JPA 有什么区别？
458. MyBatis 的 #{} 和 ${} 有什么区别？
459. 如何防止 SQL Injection？
460. Java 中 HashMap 的底层结构是什么？
461. ConcurrentHashMap 怎么保证线程安全？
462. synchronized 和 ReentrantLock 有什么区别？
463. volatile 能保证原子性吗？
464. Java 线程池的核心参数有哪些？
465. 为什么不建议直接用 Executors 创建线程池？
466. CompletableFuture 用过吗？
467. 如果让 Java 服务调用模型 API，你如何设计并发控制？
## 十七、FastAPI / Python / asyncio【高优先级】

468. FastAPI 为什么性能比较高？
469. ASGI 和 WSGI 有什么区别？
470. async/await 的原理是什么？
471. Python 协程和线程有什么区别？
472. event loop 是什么？
473. 什么情况下 async 反而会变慢？
474. 如果 async 函数里面执行 CPU 密集任务会怎样？
475. 如果调用同步数据库驱动会怎样？
476. asyncio.Queue 是怎么工作的？
477. asyncio.create_task 有什么作用？
478. Task 抛异常没人 await 会怎样？
479. Python GIL 是什么？
480. GIL 为什么不会影响 asyncio I/O 并发？
481. 多进程和多线程怎么选？
482. FastAPI 多 worker 部署后，内存状态是否共享？
483. 这对你的 ask_user 机制有什么影响？
484. FastAPI 的 Dependency Injection 怎么用？
485. Pydantic 在项目中起什么作用？
486. 如何实现全局异常处理？
487. 如何实现请求 tracing？
488. asyncio.Event、Queue、Future 分别适合什么场景？
489. 如果一个挂起的协程需要跨进程恢复，该怎么做？
## 十八、Redis / Kafka / 中间件

490. Redis 常见数据类型有哪些？
491. Redis 为什么快？
492. Redis 是单线程的吗？
493. Redis 适合在哪些 Agent 场景使用？
494. Session 能不能放 Redis？
495. Agent Checkpoint 能不能放 Redis？
496. 分布式锁怎么实现？
497. SET NX EX 有什么问题？
498. Redlock 是什么？
499. Redis Stream 和 Pub/Sub 有什么区别？
500. Kafka 为什么吞吐量高？
501. Kafka 中 Topic、Partition、Consumer Group 分别是什么？
502. 如何保证消息不丢？
503. 如何保证消息不重复消费？
504. Kafka 能保证 exactly once 吗？
505. Agent 异步任务场景为什么可能需要消息队列？
506. 如果做长耗时 Agent Task，你会怎么设计？
507. ask_user 的恢复通知更适合 Pub/Sub、Stream 还是普通 KV？为什么？
## 十九、Docker / 部署 / Production

508. Docker 和虚拟机有什么区别？
509. Docker Image 和 Container 有什么区别？
510. Dockerfile 常见指令有哪些？
511. 如何减少 Docker Image 大小？
512. Multi-stage Build 是什么？
513. Agent 服务如何容器化？
514. API Key 如何安全注入容器？
515. 为什么不能直接写进 Dockerfile？
516. 容器日志怎么收集？
517. 如果服务发生 OOM 怎么排查？
518. 模型 API 出现限流怎么办？
519. 如何实现 exponential backoff？
520. 如何限制每个用户的 Agent 并发数？
521. 如何实现 Rate Limit？
522. 如果模型 API 挂了，你怎么降级？
523. 一个生产级 Agent 服务需要监控哪些指标？
524. Token 使用量怎么监控？
525. Tool Call 成功率怎么监控？
526. P95 / P99 Latency 是什么？
527. 如何做 Distributed Tracing？
528. 一次 Agent 请求跨模型、RAG、Tool、DB，怎么定位慢在哪里？
529. 如何做灰度发布和快速回滚？
530. 模型或 Prompt 版本变化时，线上如何做可观测和对比？
## 二十、AI Coding / Claude Code / Codex / SDD / TDD

531. 你日常如何使用 Claude Code？
532. Claude Code 和普通聊天式 AI 辅助编程有什么区别？
533. Codex 和 Claude Code 有什么区别？
534. OpenCode 是什么？
535. 什么是 Agentic Coding？
536. 你如何让 AI 理解一个大型代码库？
537. AI 修改代码之前，你会让它做什么？
538. 如何避免 AI 大范围误改代码？
539. 什么是 SDD？
540. 你理解的 Specification Driven Development 是什么？
541. SDD 和传统需求文档有什么区别？
542. TDD 是什么？
543. Red-Green-Refactor 分别是什么？
544. AI Coding 为什么适合 TDD？
545. 单元测试、集成测试、E2E 测试分别解决什么问题？
546. Mock 是什么？
547. 什么情况下不应该 Mock？
548. 如何让 AI 自动生成测试又保证测试不是“为了通过而通过”？
549. AI 生成代码以后，你怎么 Review？
550. 遇到 AI 连续修改失败怎么办？
551. 你是否认为未来工程师会主要通过 Agent 写代码？
552. AI Coding 最容易产生哪些工程风险？
553. Skill 和 Workflow 是什么？
554. 你是如何把 AI 融入自己的研发流程的？
555. 有没有实际因为 AI Coding 提升研发效率的数据？
## 二十一、系统设计题：Agent 岗高频【最高优先级】

556. 设计一个 ChatGPT 类 AI 对话系统。
557. 设计一个企业内部知识库问答系统。
558. 设计一个可以调用几十个 Tool 的 Agent 平台。
559. 设计一个支持 Human-in-the-loop 的 Agent。
560. 设计一个 Deep Research Agent。
561. 设计一个 Coding Agent。
562. 设计一个浏览器操作 Agent。
563. 设计一个支持 Memory 的个性化 Agent。
564. 设计一个多 Agent 协作系统。
565. 设计一个教学 AI Agent。
566. 设计一个可以服务 10 万用户的 RAG 系统。
567. 设计一个每天处理 100 万次 Tool Call 的 Agent Runtime。
568. 如果一个 Agent Task 可能运行 30 分钟，你怎么设计？
569. 如果服务中途重启，Agent 如何恢复？
570. Agent Checkpoint 应该保存什么？
571. Agent 状态存在哪里？
572. 如何保证重复恢复不会重复执行副作用工具？
573. 如何实现 Exactly-once Tool Execution？
574. 支付、发邮件这种有副作用的 Tool 怎么保证安全？
575. Agent 如何实现审批流程？
576. Agent 如何支持 Retry / Resume / Cancel？
577. 如何设计 Agent Observability？
578. Agent Trace 应该记录什么？
579. 如何设计 Agent Playground？
580. 如何设计 Agent Eval 平台？
581. 如何设计一个支持多租户、权限隔离的企业 Agent 平台？
582. 如何设计一个 Agent Checkpoint Store？
583. 如何设计 Tool Registry 和 Tool Gateway？
584. 如何设计 Prompt / Skill / Tool 的版本管理？
## 二十二、场景题 / 故障排查题【最高优先级】

585. 用户说 Agent 经常答非所问，你怎么排查？
586. 用户说 RAG 经常找不到文档，你怎么排查？
587. RAG Recall 很高，但最终回答准确率很低，原因可能是什么？
588. Agent 经常调用错工具，你怎么优化？
589. Agent Tool 参数经常填错怎么办？
590. Agent 调用 Tool 后经常重复调用同一个 Tool，怎么办？
591. Agent 经常陷入死循环怎么办？
592. Agent 延迟从 5 秒涨到 20 秒，你怎么定位？
593. Token Cost 突然上涨 3 倍，你怎么排查？
594. 模型 API 出现大量 429 怎么处理？
595. 用户上传一个 500 页 PDF，如何回答里面的问题？
596. 用户上传 10 万份文档，RAG 怎么设计？
597. 用户问的问题需要同时查询数据库和知识库，怎么设计？
598. 用户要求 Agent 操作公司数据库，如何保证安全？
599. Agent 执行 SQL 时如何防止删库？
600. Agent 调 MCP 工具时第三方服务异常怎么办？
601. 用户输入 Prompt Injection，让 Agent 泄漏 System Prompt 怎么办？
602. 知识库文档里存在 Prompt Injection 怎么办？
603. 用户要求 Agent 发送邮件，是否应该直接发送？
604. Tool 执行成功，但模型认为失败了，怎么办？
605. Tool 超时，但实际上服务端已经执行成功，重试会不会出问题？
606. 如何解决这种“未知执行状态”？
607. Agent 中最难解决的是模型问题还是工程问题？为什么？
608. 某个 Partner 偶发读取到不属于自己的知识库结果，你怎么排查？
609. 同一个 ask_user 回复偶发触发两次恢复，你怎么定位？
610. SSE 经常在公司网络中断开，你怎么优化？
611. Agent 某版本上线后成功率下降但没有明显报错，你怎么定位？
## 二十三、简历数据真实性追问【最高优先级】

612. 7000+ 探针数据从哪里来的？
613. 100+ 困难样本为什么只有 100 多？
614. 任务周期缩短 35% 的 baseline 是什么？
615. 1000+ 视频是多久积累的？
616. 同 Turn 恢复成功率 98% 如何统计？
617. 为什么不是 100%？
618. 重复执行率 0% 有多少测试样本？
619. 后台静默沉淀率 0% 是什么意思？
620. 引用覆盖率 100% 是业务指标还是技术指标？
621. 跨作用域泄漏 0% 测了多少场景？
622. 越权调用拦截率 100% 怎么证明？
623. F1 78% 是 answer F1 还是 retrieval F1？
624. Recall@5 91% 对应多少测试样本？
625. 这些指标是在真实生产流量还是测试环境得到的？
626. 有没有置信区间？
627. 有没有做过 A/B Test？
628. 如果我现在让你现场解释指标计算代码，你能解释吗？
629. 这些数据是否有日志、测试报告或数据库记录可以支撑？
630. 哪些指标是你亲自统计的，哪些来自团队已有数据？
## 二十四、行为面 / 实习经历

631. 为什么选择视源股份这段实习？
632. 实习期间最有挑战的一件事是什么？
633. 你有没有和产品经理意见不一致的时候？
634. 和算法团队发生需求冲突时怎么处理？
635. 有没有需求一开始理解错了？
636. 有没有上线以后被打回重做的功能？
637. 有没有出现过线上 Bug？
638. 你造成过最严重的 Bug 是什么？
639. 怎么解决的？
640. 有没有因为自己的设计导致返工？
641. 你如何进行 Code Review？
642. 遇到陌生技术栈时怎么快速上手？
643. 需求不明确的时候怎么做？
644. 产品希望两天上线，但你认为需要一周，怎么沟通？
645. 如果算法团队说你的标注平台数据有问题，你怎么定位？
646. 如果你和同事对技术方案有不同意见怎么办？
647. 你更喜欢独立开发还是团队协作？
648. 你的技术短板是什么？
649. 你觉得自己目前离正式 AI Agent 工程师还有哪些差距？
## 二十五、求职动机 / 岗位匹配

650. 为什么不继续做传统后端？
651. 为什么不做纯算法？
652. 为什么选择 AI 应用开发？
653. AI 应用工程师和传统后端工程师最大的区别是什么？
654. AI Agent 工程师最核心的能力是什么？
655. 你认为 Agent 开发是“套 API”吗？
656. 如果未来模型能力越来越强，Agent 工程师还有价值吗？
657. 你怎么看“Agent Framework 最终会被模型能力吃掉”这个观点？
658. 你希望未来两三年重点发展什么方向？
659. 你更想做 Agent Runtime、RAG、Agent Infra，还是业务应用？
660. 为什么投我们公司？
661. 为什么投这个岗位？
662. 你对我们的 AI 产品有什么了解？
663. 如果入职，你认为自己最快能贡献什么？
664. 你希望第一份正式工作的技术成长是什么？
665. 你怎么看创业公司和大厂 AI Agent 岗的区别？
## 二十六、简历技术细节“卡真实性”连续追问【最高优先级】

666. UnifiedContext 是你自己设计的吗？
667. 为什么叫 UnifiedContext？
668. 代码里它具体是 class、dataclass 还是 dict？
669. 有哪些字段？
670. 谁负责构造？
671. 生命周期是什么？
672. 有没有序列化？
673. 有没有持久化？
674. 多 Turn 之间是否复用？
675. 并发请求是否线程安全？
676. 为什么不直接用 request context？
677. 有没有循环依赖问题？
678. 单元测试怎么写？
679. 如果现在让我画类图，你能画出来吗？
680. ask_user 的 Queue 存在哪里？
681. ask_user 的 key 是什么？
682. 谁创建 Queue？
683. 谁消费 Queue？
684. 谁销毁 Queue？
685. timeout 谁控制？
686. cancel 怎么处理？
687. SSE disconnect 怎么处理？
688. server restart 怎么处理？
689. multiple worker 怎么处理？
690. 为什么不用 Redis？
691. 线上真的部署过多 worker 吗？
692. 98% 是真实线上还是你的测试？
693. RAGProvider 是 interface、protocol 还是抽象类？
694. 新增一个 Provider 具体需要实现哪些方法？
695. 探针回流的幂等 key 到底是什么？
696. 困难样本状态是存储状态还是运行时派生？
697. 乐观锁的 version 字段具体在哪张表？
698. 如果现在打开你的代码仓库，你能在 5 分钟内指出这些实现的位置吗？
## 二十七、AI Agent 开放题【高优先级】

699. 你觉得当前 Agent 最大的工程瓶颈是什么？
700. Agent Framework 的核心价值是什么？
701. LangGraph 这种框架是否必要？
702. Workflow 和 Agent 有什么区别？
703. 什么场景根本不应该用 Agent？
704. 什么任务应该使用 deterministic workflow？
705. 什么任务适合 Agent？
706. Multi-Agent 是不是很多时候过度设计？
707. ReAct 是什么？
708. Plan-and-Execute 是什么？
709. Reflection 有什么作用？
710. Self-Consistency 是什么？
711. Tree of Thoughts 是什么？
712. Agent Search 和普通 Chain-of-Thought 有什么区别？
713. Agent 应该先 Planning 再 Execution 吗？
714. Plan 是否应该动态修改？
715. Agent 如何判断任务已经完成？
716. Agent 如何判断自己不知道？
717. Agent 如何请求人工介入？
718. Tool Use、Memory、Planning、Reflection 中哪个最重要？
719. 你怎么看 Computer Use Agent？
720. GUI Agent 和 API Agent 哪个更可靠？
721. Browser Agent 最大的问题是什么？
722. Coding Agent 为什么比普通聊天 Agent 更容易落地？
723. AI Agent 产品真正的护城河是什么？
724. 你认为未来 Agent Runtime 会向哪些方向发展？
725. 企业落地 Agent 时，可靠性、成本、速度三者如何权衡？
## 面试前最后自查

- 简历中所有量化指标都能解释：统计口径、样本量、baseline、计算方法。
- ask_user 能画出完整时序图：用户 → 前端 → SSE → Runtime → Queue → Tool Result → 恢复执行。
- UnifiedContext、RAGProvider、Memory 三层模型都能画出类图或数据流图。
- 能从代码级解释 asyncio.Queue、多 worker、服务重启、幂等、乐观锁。
- 能独立完成一个 Agent 系统设计题，并覆盖状态、持久化、Tool、安全、Eval、监控。
- RAG 能从 ingestion、chunk、embedding、retrieval、rerank、generation、citation、evaluation 全链路讲清楚。
- 每个简历项目至少准备 1 个失败案例、1 个技术取舍、1 个上线问题和 1 个重构方案。
