SpringAI要求最低使用JDK17.
# 认识AI
## AI发展过程
- 符号主义
- 机器学习
- 深度学习 --> 自然语言处理（NLP） --> 大语言模型(LLM)
NLP有一门核心的技术：**Transformer**
![[Pasted image 20251026184047.png]]
在自然语言处理（Natural Language Processing，**NLP**）中，
有一项关键技术叫Transformer，这是一种先进的神经网络模型，是现如今AI高速发展的最主要原因。
## 大模型底层原理
我们所熟知的大模型（`Large Language Models，LLM`），例如GPT、DeepSeek底层都是采用Transformer神经网络模型。
![[Pasted image 20251026182934.png]]
- T：基于Transformer的神经网络
- P：通过大量数据预训练，掌握自然语言规律
- G：基于上文计算概率，生成下一个token
# 大模型应用开发
## 模型部署

|  **云部署**  |        |        |       |      |         |
| :-------: | :----: | :----: | :---: | :--: | :-----: |
|    优点     | 前期成本低  | 部署维护简单 | 弹性扩展  | 全球访问 | 中小型企业使用 |
|    缺点     |  数据隐私  |  网络依赖  | 长期成本高 |      |         |
| **本地部署**  |        |        |       |      |         |
|    优点     |  数据安全  |  数据安全  | 长期成本低 | 高度定制 |         |
|    缺点     | 初始成本高  |  维护复杂  | 部署周期长 |      |         |
| **开放API** |        |        |       |      |         |
|    优点     | 前期成本极低 |  无需部署  | 无需维护  | 全球访问 |  快速调试   |
|    缺点     |  数据隐私  |  网络依赖  | 长期成本高 | 定制限制 |         |
云部署和本地部署（有定制化需求），两者的优劣势刚好反过来
开放API快速开发应用（个人or小型企业）
### 本地部署
本地部署最简单的一种方案就是使用ollama,因为显卡不好所以最好部署一些阉割版的或者是低级的模型（只能用于测试和demo使用）。

ollama安装完后在后台运行一个服务，用cmd用于与其交互.
7b,1.5b这个b是：当前这个模型支持参数的数量，支持的参数越多，代表模型的分析和推理能力越强。
## 调用大模型API
大模型最早出现在大众眼前，被大众所熟知就是OpenAI的ChatGPT，所以它的标志就成为了行业标准,成为了API规范

以下是DeepSeek官方给出的一段API示例代码：
```python
from openai import OpenAI

# 1.初始化OpenAI客户端  api_key就是申请的身份证密钥  base_url访问大模型地址
client = OpenAI(api_key="<DeepSeek API Key>", base_url="https://api.deepseek.com")

# 2.发送http请求到大模型
response = client.chat.completions.create(
    model="deepseek-r1", # 模型名称
    messages=[  # 也叫提示词（Prompt）
        {"role": "system", "content": "你是一个热心的AI助手，你的名字叫小团团"},
        {"role": "user", "content": "你好，你是谁？"},
    ],
    stream=False # 是否是流式输出，即 是否一次返回
)

# 3.打印返回结果
print(response.choices[0].message.content)
```
调用大模型，实际上就是给它发送HTTP请求。最主要的是给
- api_key就是申请的身份证密钥  
- base_url访问大模型地址
这两个参数赋值。
## 大模型应用
**大模型应用** 是基于大模型的推理、分析、生成能力，**结合传统编程** 能力，开发出的各种应用。
优劣势对比：
![[Pasted image 20251026194426.png]]
两者优势结合在一起激素大模型应用（Hybrid AI）

主要应用领域：
- **文本分析** : 数据提取和格式化、坐席质检、舆情分析、文本摘要、知识库
- **多模态** : 图片识别、音频识别、视频识别、音频生成、图片生成、视频生成等
- **机器人应用** : AI智能客服机器人开发、对话管理、情感分析、个性化回复等
- **智能体** : AI金融分析、自动化办公、智慧医疗、工业/制造智能体、运维智能体
- **自动驾驶** : 计算机视觉处理，车辆自动驾驶
## AI 应用开发技术架构
![[Pasted image 20251026195209.png]]
技术选型顺序：PARF
### Prompt问答
特征： 利用大模型推理能力完成应用的核心功能
![[Pasted image 20251026195305.png]]
应用场景：文本摘要分析、舆情分析、坐席检查、AI对话
### `Agent+Function Calling`
特征： 将应用端业务能力与AI大模型推理能力结合，简化复杂业务功能开发
![[Pasted image 20251026200106.png]]
应用场景： 旅行指南、数据提取、数据聚合分析、课程顾问
### RAG Embeddings（外挂知识库）
离线步骤： 文档加载、文档切分、文档编码、写入知识库
在线步骤：
1. 获得用户问题
2. 检索知识库中相关知识片段
3. 将检索结果和用户问题填入Prompt模版
4. 用最终获得的Prompt调用LLM
5. 由LLM生成回复
![[Pasted image 20251026200423.png]]
应用场景：个人知识库、AI客服助手
# `SpringAI`
使用哪一个？建议SpringAI
![[Pasted image 20251026200923.png]]
## 快速入门
OpenAI调用大模型的规范：
```python
from openai import OpenAI

# 1.初始化OpenAI客户端
client = OpenAI(
    api_key="<DeepSeek API Key>",
    base_url="https://api.deepseek.com"
)

# 2.发送http请求到大模型
response = client.chat.completions.create(
    model="deepseek-r1",
    temperature=0.7,
    messages=[
        {"role": "system", "content": "你是一个热心的AI助手，你的名字叫小团团"},
        {"role": "user", "content": "你好，你是谁？"},
    ],
    stream=False
)

# 3.打印返回结果
print(response.choices[0].message.content)
```
这个temperature参数范围0-1，其代表着模型生成文本的特性，接近于0生成文本更保守和精确，但易出现机械性重复文本；接近于1生成文本更具有创造性，但过于冒险，生成文本易发生不连贯的问题。

配置客户端：
```java
@Bean
public ChatClient chatClient(OllamaChatModel model) {
    return ChatClient.builder(model)
        .defaultSystem("你是可爱的助手，名字叫团团")
        .build();
}
```
用户发消息：
```java
String content = chatClient.prompt() //构建提示词
    .user("你是谁？") //用户的问题说的话
    .call()  //发给大模型请求,call返会响应结果 
    .content();  //解析响应中的内容
    
Flux<String> content = chatClient.prompt() 
	.user("你是谁？") 
	.stream()  //注意这行
	.content();
```
注意：
- call是批处理形式发送，用stream则是流式地处理
- 流式编程时需要设定对应的响应编码，负责会出现问题
SpringAI利用AOP原理提供了AI会话时的拦截、增强等功能，也就是  **Advisor**。
## 会话日志
SpringAI利用AOP原理提供了AI会话时的拦截、增强等功能，也就是Advisor。
![[Pasted image 20251028145844.png]]
```java
@Bean
public ChatClient chatClient(OllamaChatModel model) {
    return ChatClient.builder(model) // 创建ChatClient工厂实例
        .defaultSystem("你是可爱的小助手，名字叫小团团。")
        .defaultAdvisors(new SimpleLoggerAdvisor()) // 配置日志Advisor
        .build(); // 构建ChatClient实例
}
```
默认的是debug级别，需要在yml文件中进行配置
## 会话记忆
大模型是不具备记忆能力的，要想让大模型记住之前聊天的内容，唯一的办法就是把之前聊天的内容与新的提示词一起发给大模型
![[Pasted image 20251028152257.png]]
根据按需求自定义这个接口：
```java
public interface ChatMemory {
    void add(String conversationId, List<Message> messages);
    List<Message> get(String conversationId, int lastN);
    void clear(String conversationId);
}
```
具体实例：
![[Pasted image 20251028153002.png]]
添加会话id：
这里的话跟正式版的实现接口有些不同，可以去网上看spring ai的官方文档，在聊天记忆中的聊天客户端Memory有解决方案
通过不同的会话Id，使得不同的会话不会混淆彼此记忆。
![[Pasted image 20251028175202.png]]
服务端生成chatId，用布隆过滤器判断是否生成重复，然后放心add