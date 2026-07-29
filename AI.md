## AI

RAG：知识库/联网搜索...

### 知识库

将文本切分，通过嵌入模型将文本块转换为向量嵌入到向量数据库里

用户提问文本通过嵌入模型转换为向量去数据库里进行相似度检索

### vue2react

### AICR

**实现**

**v1**:远端GitLab,通过GitHub action实现提mr时ai对本次mr修改代码进行审查并回写评论

**产生幻觉原因**：Transformer 架构中直接导致幻觉，

自回归生成：逐词预测，无法回头修正，每一步预测都是**局部最优**，一旦早期步骤出错，后续只能在这个错误基础上"自圆其说"，无法全局回溯修正

Transformer 的核心是**自注意力机制**，它计算的是**词语之间的统计关联强度**，而非**真实的因果关系**，模型会把高频共现误认为事实关联，把相关当作因果

**v2**:由于远端CR会出现幻觉，问题发现晚，根据前端开发场景梳理了diff获取策略的优先级，让审查能在写代码时而不是提MR后才触发，在本地通过mcp暴露能力，实现了一个 **MCP Server**，在 `.cursor/mcp.json` 中以 stdio 方式npm命令启动，把代码审查的能力封装成两个MCP Tool 注册上去，Cursor 里的 Agent 就能像调用本地函数一样触发审查。整体通过**"规则引擎 + 大模型"组合**，针对前端层面框架、类型检测和公司组件库写对应的CursorRule（.cursorrules 声明项目背景、用例模板、API模式、命名规范等->全局生效，团队共享，强约束，适合标准化项目），对ESLint，Prompt进行调优，减少噪音和误报。规则层管命名规范、敏感信息这些静态检查，模型层做逻辑合理性和最佳实践的语义审查。鉴权通过 **GitLab Personal Access Token**实现，检测Token是否拥有项目访问权限，提交PR后再通过CI/CD Pipeline中的Agent调用MCP，通过MCP工具拉取 MR diff、回写评论。通过分批CR的方式避免AI偷懒

**两个MCP Tool**（Agent loop循环判断用哪个工具）

- **`cr_shift_left_batch_review(session_id)`**：入参是 `session_id`（审查会话标识），服务端根据当前上下文自动拉取 diff（走 GitLab API 或本地 git diff），不需要手动传代码进来。发起一次批量代码审查会话，返回结构化问题列表
- **`cr_shift_left_post_batch_results(session_id, batch_results)`**：入参是 `session_id` + `batch_results`（结构化审查结果数组），每条结果包含文件路径、行号、严重程度评分（1–5）、问题分类、描述、原有代码片段、建议修改代码。，返回一个网页包括所有可修改的问题

**优化**：通过Skill实现CR，在输入aicr时，命中skill执行skill中的脚本，将mcp工具逻辑集成到脚本中，可以本地运行，不需要重复提交MR。

**未解决问题**：

- 单次 MR 或工作区变更很大时，需要合理切分（按文件/按模块）、控制单次审查 token 与耗时

  按文件类型/模块分桶，桶互不依赖，可并发审查，通过mcp确认分桶情况，通过workflow启动 N 个 Agent 并发审查，最终主 Agent 去重拿结果格式化输出报告

### 心得

- 提示词：给上下文、定约束、要完整
- Debug：给完整报错信息 + 相关代码 + 已尝试的解决方法
- 新项目搭建：先让输出目录结构，画流程图，写伪代码，确认后再生成具体文件
- ai结合：业务方面 研发提效 质量自测
- 用AI的前提：设计与实现的标准化 -> 生产的自动化 要先有一套规范并且组内遵循再植入ai辅助提效

### 与项目结合

端侧和工程侧(自动化)

样板间 组件库 低代码 figma2code

### MCP

mcp使用方式

```js
"github": {//stdio
   	"command": "npx",
     "args": ["-y", "@modelcontextprotocol/server-github"],
},
//WebSocket 方式
{
  "getTime": {
    "url": "ws://localhost:3000"
  }
}
```



| 方式     | 适用场景               |
| -------- | ---------------------- |
| `npx -y` | 快速启动，每次下载最新 |
| 全局安装 | 常用工具，减少启动时间 |
| 本地路径 | 自定义修改或离线环境   |
| Docker   | 隔离环境、跨平台       |
| Python   | Python 生态的工具      |
| SSE      | 远程服务、微服务架构   |
| 内嵌代码 | 深度集成、自定义逻辑   |

### Skill

### Ollama

```
irm https://ollama.com/install.ps1 | iex
# 下载模型c
ollama pull llama3.2
# 直接对话
ollama run llama3.2
```

```
API http://127.0.0.1:11434/
```



| 接口                   | 用途                 |
| ---------------------- | -------------------- |
| `GET /api/tags`        | 列出本地已下载的模型 |
| `POST /api/generate`   | 生成文本（单次问答） |
| `POST /api/chat`       | 多轮对话（带上下文） |
| `POST /api/embeddings` | 获取文本的向量表示   |
| `POST /api/pull`       | 下载模型             |
| `POST /api/delete`     | 删除模型             |

**ONNX = AI 模型的"通用语言"，让模型脱离训练框架，在任何设备上高效运行，都转成 `.onnx` 格式，一次转换，到处运行**

## AI

| 概念                   | 需要掌握的程度                                               |
| ---------------------- | ------------------------------------------------------------ |
| **LLM 基础**           | 了解 Transformer 架构、Token 机制、上下文窗口限制            |
| **Prompt Engineering** | 编写高效提示词、Few-shot/Chain-of-Thought 技巧               |
| **RAG**                | 检索增强生成原理、向量数据库（Pinecone/Milvus/Chroma）、Embedding 模型 |
| **Agent**              | ReAct 模式、工具调用（Function Calling）、多 Agent 协作框架（LangChain/LangGraph） |
| **模型微调**           | LoRA/QLoRA 原理、数据准备、训练流程（至少了解概念）          |
| **多模态**             | 文本+图像+音频的联合处理，了解 CLIP、GPT-4V 等               |

- **向量数据库**：Milvus、Weaviate、PgVector 的使用，Pinecone/Milvus/Chroma）、Embedding 模型

- **模型部署**：了解 ONNX、TensorRT、vLLM 等推理加速

  学习 LangChain.js 或 Vercel AI SDK

第3阶段（持续）：深度项目
  → 做一个 Agent 应用（能调用工具、执行多步骤任务）
  → 学习模型微调（至少跑通一个 LoRA 训练）
  → 关注 AI 前端新趋势（如 v0.dev、Bolt.new 等 AI 生成代码工具）

1. **关注产品**：多体验 AI 产品（Perplexity、Claude Artifacts、Cursor），思考前端如何承载 AI 能力
2. **展示热情**：JD 强调"对AI应用有强烈的热情与好奇心"，面试时准备你做过的小项目    "AI 智能表单填写助手"或"RAG 知识库问答前端"





Firebase 云信息传递 (FCM) 是一种跨平台消息传递解决方案，可供您免费、可靠地传递消息。
使用 FCM，可以通知客户端应用存在可同步的新电子邮件或其他数据。在即时消息传递等使用情形中，一条消息可将最大 4KB 的有效负载传送至客户端应用


翻译
获取当前系统的 cookie 拿到当前的语言环境
通过调用翻译平台提供的开放 API 拿到系统当前语言环境下的所有翻译内容
分析当前页面，拿到当前页面所有文案的节点getTextNodeList
与远程翻译平台的文案进行对比，如果文案没有被翻译，将文案包裹起来高亮，展示翻译内容
点击批量上传，并且用户鉴权成功后，根据获取到的 token 调用翻译平台的开放API进行上传
如果文案过于复杂，分辨不出来具体的翻译，则将该文案加入黑名单中缓存，翻译扩展会每天晚上8点定时请求一次接口，将数据同步到系统中人工翻译

创建二维数组
Array.from({ length: numRows },() => [])
new Array(numRows).fill(null).map(() => [])

typeof null 的结果为什么不是 null：
在 javascript 的最初版本中，使用的 32 位系统，并且底层都表示为2进制,为了性能考虑使用低位存储了变量的类型信息：
000：对象
1：整数
010：浮点数
100：字符串
110：布尔
全部为0: null
所以typeof就是利用这一机制去判断的,所以null全部为0就复合了对象的000,所以被判断为object


axios的cancelToken解决前端并发冲突
let pendingRequests = new Map()//存放请求信息，去判断是否上重复请求
// 请求拦截器
axios.interceptors.request.use((config) => {
  if (pendingRequests.has(requestKey)) {//map存在信息就取消请求
    config.cancelToken = new axios.CancelToken((cancel) => {
      // cancel 函数的参数会作为 promise 的 error 被捕获
      cancel(`重复的请求被主动拦截: ${requestKey}`);
    });
  } else {//存入信息，发送请求
    pendingRequests.set(requestKey, config);
    config.requestKey = requestKey;
  }
  return config;
},
(error) => {
  // 这里出现错误可能是网络波动造成的，清空 pendingRequests 对象
  pendingRequests.clear();
  return Promise.reject(error);
});
//响应拦截器
axios.interceptors.response.use((response) => {//说明请求结束，删掉map里的信息
   const requestKey = response.config.requestKey;
   pendingRequests.delete(requestKey);
   return Promise.resolve(response);
 }, (error) => {
   if (axios.isCancel(error)) {
     console.warn(error);
     return Promise.reject(error);
   }
   pendingRequests.clear();
   return Promise.reject(error);
 })

Get，Post区别
缓存：Get 请求能缓存，Post 请求不能
安全：Get 请求没有 Post 请求那么安全，因为请求都在 URL 中。且会被浏览器保存历史纪录。POST 放在请求体中，更加安全
限制：URL 有长度限制，会干预 Get 请求，这个是浏览器决定的
编码：GET 请求只能进行 URL 编码，只能接收 ASCII 字符，而 POST 没有限制。POST 支持更多的编码类型，而且不对数据类型做限制
从 TCP 的角度，GET 请求会把请求报文一次性发出去，而 POST 会分为两个 TCP 数据包，首先发 header 部分，如果服务器响应 100(continue)， 然后发 body 部分



将文件中的全局语句定义为 part ，并将 part 标记为是否有副作用。比如语句 let foo = 3 就没有副作用，如果没有用到 foo 就可以将这个语句删除。但是 let foo = bar() 就有副作用，即使没有用到 foo，无法保证移除函数调用 bar() 之后程序行为一致。
Tree shaking 通过从入口文件（这里是index.js）访问所有有副作用的 part，并沿着 import语句访问所有引用文件中有副作用的 part，只有这个过程中访问到的 part 才会被放进打包结果中
