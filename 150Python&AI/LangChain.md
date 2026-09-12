# 基础知识
写提示词、调大模型、处理输出格式、提供上下文、调相关接口函数。
LangChain作用就是把这几个模块设计好后，串在一起完成用户需求。
为什么要用LangChain？

LangChain：主要是agent，RAG的api
`LangChain-Community`：`Model IO、Agent Tooling、Retrieval`
LangChain-Core：LangChain表达式语言。
`LangGraph`：agent与agent之间的交互行为实现的话就可以使用
`LangSmith`：主要做链路追踪，起到运维和监控作用。
LangServe：将LangChain的可运行项和链部署提供REST API

---
基于RAG（检索增强开发）架构的开发，解决：
- 大模型的知识冻结
- 大模型幻觉
基于Agent架构的开发，解决：
Agent = LLM（大语言）+Memory（记忆）+Tools（工具）+Planning（规划决策）+行动（Action）
记忆：
- 短期记忆：存储单次对话周期的上下文，但受限于模型的上下文窗口长度。
- 长期记忆：可以通过模型参数微调（固化知识）、知识图谱（结构化语义网络）、向量数据库（相似性检索）方式实现。
工具：通过调用外部工具（如API、数据库）扩展能力边界。
规划决策：通过任务分解、反思与自省框架实现复杂任务处理。
行动：实现执行决策的模块，涵盖软件接口操作和物理交互。
RAG和Fine-tuned model的区别在于，前者简单但通过网络传播、后者难还有可能不对但基于本地。

---
没有最好的大模型。
为什么不要依赖LLM排行榜单？
- 榜单已被应试教育污染，榜单体现的是整体能力，放在局部上排名低的或许更好
主要使用OpenAI为例展开后续课程。

按照输出区分对话或非对话模型，输出str是非对话模型、输出msg（AIMessage）是对话模型

**提示词模板**
重点是：`Prompt~;ChatPrompt~;FewShotPrompt~;了解XxxMessagePrompt`

---
**优化幻觉**：优化提示词（模板）、使用RAG、进行大模型微调等。
优化提示词模板，详细点就是 提供少量样本示例的方式。

**`IOModel、Chain、Memory、Tools`**
- Memory：新的记录作为完成的记录存储，早期的作为摘要。
- Tools:特定的方法调用，也就是`Function Calling`，`python`中有两种方式@Tool装饰器 和 `Structured from_function`.
- Agent有两种模式Function Call 和 ReAct模式。前者成本低且需要相应的支持，后者成本高但无需支持

对于大模型 与 Agent的核心区别：是否涉及到工具的使用：
- 针对大模型：仅能分析出要调用的工具，但是此工具（或函数）不能真正的执行
- 针对Agent：除了分析出要调用的工具之外，还可以执行具体的工具（函数）


---
`ctrl+n`

---
**Retrieval**
![[Pasted image 20260424221008.png]]
文档嵌入模型（Text Embedding Models）负责将 **文本** 转换为 向量 **表示**，即模型**赋予了文本计算机可理解的数值表示**
注意Load的编码 和 解码 的字符集要一致

短期记忆，memory；长期记忆，RAG
切分的向量，相似的词在向量空间中的距离越近。

**文档切分器**
当拿到统一的一个Document对象后，接下来需要切分成Chunks。
拆分策略：
- 方法1：根据句子切分
- 方法2：按照固定字符数来切分
- 方法3：按固定字符数来切分，结合重叠窗口（overlapping windows）
- 方法4：递归字符切分方法
- 方法5：根据语义内容切分

若必须禁用分隔符（如处理无空格文本），需容忍实际块长略小于 `chunk_size`,系统默认推荐4000

---
**文档嵌入模型**
在向量数据库中，不仅存储了数据（或文档）的向量，而且还存储了数据（或文档本身）