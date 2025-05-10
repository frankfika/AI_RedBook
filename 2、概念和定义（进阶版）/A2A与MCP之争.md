# 大模型的「Type-C接口之争」：MCP VS A2A（谷歌），谁才是Agent的未来事实标准？

> 作者 | 龙鹏飞-腾讯云开发者
>
> 原文 | 《打起来了！MCP VS A2A，谁才是Agent的未来事实标准？》


谷歌在MCP协议快速发展之际推出A2A协议，定位为智能体Agent间的协调协议。本文通过具体的案例介绍了MCP和A2A的细节，通过同一案例在MCP与A2A两种模式下的实现差异，认为A2A模式下的 Agent 能够通过与大模型深度交互，交付更具价值的功能特性，从而更有效地吸引开发者群体。此外，A2A架构赋予每个 Agent 自主选择底层大模型的权利，这一开放性设计也将进一步吸引大模型供应商参与生态构建。 与行业普遍认为两种协议具有互补性的共识不同，笔者认为MCP和A2A协同发展仍面临显著挑战。文中还列举了 K8s 与Docker 的历史协同案例作为类比，将技术演进的想象空间留给读者。 限于笔者水平，本文部分观点可能存在错误，恳请大家不吝赐教。

注：作者4.18整理发布于内部系统，文中观点仅代表个人看法。

![Image](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649039.png)

## MCP

MCP（Model Context Protocol）由Anthropic推出，标准化AI与外部工具/资源的交互，如数据库、API调用，增强智能体能力。

介绍了 MCP 的总体架构和核心概念，并用一个例生成金融报告的例子来介绍 MCP 的运行流程。熟悉 MCP 的读者可以直接跳过本章。

### 1.1 MCP 总体架构

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649171.png)

我们来回顾下 MCP 的架构，是一个典型的 Client-Server 的架构，其中作为交互主体的MCP Host（如Claude Desktop、IDE环境或近期广受关注的Cursor编辑器），是具备大模型交互能力的应用程序载体， 能理解用户的需求，能选择合适的 Client 调用 Server 去访问资源，Server 部署在本地或者远程，可以本地或远程的文件系统，数据源，API 打交道，专注实现某些功能。

这里以 "去系统查询消费在50-100元的深圳地区客户" 的例子，Host 将收到用户的需求，通过大模型分析出要调用 DataQueryClient（MCP Client），并且准备好调用 DataQueryServer(MCP Server) 的参数（查询哪个表，语句是什么）， MCP Server 收到需求后，调用数据库返回数据。更详细的交互参考1.3 节的案例。

初看左侧架构图似乎逻辑自洽，仔细看有会不少疑问：Host 和 MCP Client是什么关系？MCP Client 和 MCP Server是什么关系？MCP Server一定要部署在本地吗？这里画的是本地数据源，那远程数据源不可以吗？这里画的是远程的服务，本地服务不可以吗。

最新版技术规范对架构图示进行了调整（右侧版本），主要优化体现在：明确Host可同时调度多个MCP Client，确立Client与Server 1:1对应原则，且Server部署位置不受限制，用统一的Resource 抽象层替代原有数据源与服务分离式标注。

原始图没有画出一种场景，就是通过本地的Server去调用 internet的资源。也就是右图红框部分。

### 1.2 MCP 核心概念

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649133.png)

#### 资源(Resources)

MCP Client可通过标准化接口对Server端资源进行查询、修改及订阅操作，支持接入API接口、文件系统、数据库等异构数据源。

#### 提示(Prompts)

作为Server端能力的操作指南，提示词模板包含参数配置规则与交互协议。开发者需通过规范流程获取结构化描述，为大语言模型提供精确的接口调用参数生成依据。

#### 工具(Tools)

Server端注册的可执行操作需包含明确的功能描述。大语言模型基于用户请求上下文，通过语义解析匹配最佳工具组合。

#### 采样(Sampling)

当Server端需触发模型推理时，通过标准化流程发起协同计算请求。该机制包含用户授权确认、输入数据格式化等子流程，最终将结果返回至调用方。

建议结合图1.3的案例进行验证性学习，通过对照时序图与代码实现掌握交互机制

### 1.3 MCP案例说明

如果前面看完还是一头雾水的话，没关系，这节我们构造一个案例，来更加直白地解释 MCP 的整个运作流程。

#### 1.3.1 总体流程

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649167.png)

这里用户的需求是：获取 2021-2025 年人民币兑美元的最优汇率变动报告，要求图文并茂。

首先我们快速在大脑里面轮动下，如果要完成这个任务，大概可能涉及的动作：

- 查询有没有现成的报告，如果没有或者不完整，就要想办法生成缺的部分。
- 补齐缺失的报告，可以先调用多家银行的 API 或者数据，然后将数据绘制成图，接着自动生成报告的文字，和图一起生成相关的报告。
- 如果某些月份的数据缺失，例如2025 剩余月份的数据，可以通过预测的方式补齐，并在报告中指出这些是预测数据。

详细的 MCP 协议操作的流程已经在图中列出来了，需要说明的是，这个流程图只是一个大致的示意图，有些流程其实是涉及多个步骤的，没有完整标示出来，不影响理解。

系统开始运行后，Client 和 Server 之间就会通过 MCP 协议交换信息，每个 MCP Server 能干哪些事情 Client 都非常清楚。

HOST 在接收到用户的请求后，通过大模型对用户的输入进行分析，确定所有的流程和调用的工具，后面的步骤都是按部就班，对着每个流程和图上的序号了解即可。

#### 1.3.2 Resources

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649153.png)

Resources，MCP Client 可以通过 Resources 去读取 MCP Server 上面的文本资源，二进制资源，还可以订阅资源的变动。

这里 MCP Client 可以过 Resources 去报告管理 MCP Server 查询某个文件目录的情况，可以把相关的报告下载下来。

为了大家更直白地理解，这里我通过元宝生成了相关的 json 返回形式，不过这个仅仅用来示例，可能和官方的接口有差异，具体的还是要以官方文档为准。

#### 1.3.3 Prompts

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649130.png)

Prompts，初学者或者没开发过大模型应用的人或许有点难理解，以这个绘图 MCP Server 举例，绘图 MCP Server 提供的 prompts 实际上就是一个预置的模板，告诉调用的 MCP Client，应该按照这个 prompts 的模板去和大模型交互，由大模型去理解这里面的参数，这些参数用来干什么的，后面调用 tools 就知道了。

比如这里，MCP Server 提供的 prompts 模板实际上就是表达：如果有人想画图，那就需要知道图表类型 char_type，配色方案 color_template 等。

#### 1.3.4 Tools

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649130.png)

Tools，代表 MCP Server 能支持的操作，这里绘图的 MCP Server 支持两种工具能力，plot_mermaid_flowchart 可以绘制时序图，plot_python_data 可以绘制统计图。Server 会在 tools 相关的接口里面详细描述工具的详细信息。

**chart_type 参数如何确定**

以 plot_python_data 为例，会表明这个工具需要哪些参数，比如 chart_type 这些参数，前面我们在介绍 prompts 的时候说过，chart_type 这些参数是 Server 提供的 prompts 模板中给出的，一旦给大模型输入了这些上下文，大模型就能理解 char_type 的类型，比如用户是要统计报告，大模型就会自动分析出 char_type 应该用 line 类型。

**x_column 参数如何确定**

这个参数实际上是不在 prompts 模板里面的，那究竟该如何确定呢？不用担心，成熟的大模型会自己解决！大模型根据查询到的汇率数据，自然就会理解到 x_column 应该是填入月份数据。这也是大模型应用的优势点，就是对需求的理解能力。

**如何确定选择哪个tool**

那可能到这里，读者还会有疑问：既然有多个工具，那 Host 怎么知道调用哪个工具？这个其实是Host 把所有的工具信息和用户需求加上一些系统的 prompts 一起扔给 LLM，由LLM去判断的（后面有代码分析）。

比如这里，LLM 理解到用户是要画趋势图而不是时序图。因此会去调用 plot_python_data。

题外话，resource 也是可以用 tool 来实现，比如查询文件的 tool，下载文件的 tool，好像也没有什么问题。A2A 在这里就没有分开定义 resource 和 tool 对象

#### 1.3.5 Sampling

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe972Atk9kUtRXbNLicS94JcmEAXV9etSoYCIbgRLAc38ialEgCiaBx4QyjJ8zFic6E419Urjq7EftL7R5A/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=wxpic)

前面的概念都比较好理解，这里来讲一下 sampling，为了讲清楚这个例子，特意构造了一个不知是否恰当的例子，大家看看就好。

在绘图的过程中，绘图 MCP Server 发现，2025 年的数据并不是很完整，于是想让大模型预测下剩下月份的数据。但是 MCP Server 又没法直接和大模型交互，于是只能给 MCP Client 发起 sampling 的需求。

MCP Client 收到请求以后反馈给用户，用户鉴权同意以后，Host 调用 LLM 进行数据预测，最后把预测的数据发给绘图 MCP Server，绘图 MCP Server 继续完成绘图。这实际上就是 Server 主动去请求 Client 的一个行为，本质上是希望通过 Client 去调用大模型，但sampling 这个功能导致 MCP Client 和 MCP Server 会严重耦合。

#### 1.3.6 MCP Client 如何选择工具

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508165741786.png)

前面我们提了一下 MCP Client 是如何选择 MCP Server 的工具的，这里从MCP源码出发，其实很简单，就是关键的几步：获取全部的工具信息 → 嵌入到预定义的模板中 → 拼接用户的请求 → 一起发给 LLM 去决策。

这里我们可以看到，嵌入模板，用户请求，工具信息，以及大模型的能力每个环节都可能影响最终的结果。因此要求不管是系统模板，还是用户需求描述，MCP Server工具描述，都要尽可能的清晰准确。

到这里，MCP 差不多就讲完了。

![Image](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## A2A

A2A（Agent-to-Agent）是谷歌开发的智能体协作协议，支持跨平台任务分配与通信，实现多智能体动态协作。

介绍了A2A 协议的定位，通过官网的招聘全流程例子来说明 A2A 交互的关键功能，最后通过官方的 LangGraph 例子来说明 A2A 的运作流程。

### 2.1 A2A 定位

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250507231649130.png)

这是官方提供的 A2A 协议定位，大体就是 A2A 负责 Agent 和 Agent 之间交互，MCP 负责具体工具的交互。

A2a Agent 之间交互主要涉及四个关键的功能，接下来我们通过官方的招聘例子来介绍这四大关键功能。

### 2.2 A2A 交互的关键的四个功能

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508165813863.png)

#### （1）Capability Discovery 能力发现

首先，用户发布一个招聘需求到 Client Agent，可以认为就是当前这个系统。

大模型思考，需要通过A2A协议找到具有 candidate sourcing 能力的 Agent，于是在已知的 AgentCard（可以认为是Agent 的名片，记录了 Agent 的能力） 中发现 sourcing Agent可能具有这种能力。

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508165859404.png)

#### （2）Collaboration 协作能力

接下来，Sourcing Agent (Server Agent)开始思考，发现需要向用户询问要招聘的区域。用户回复了需要找和美国时差在三小时之内的。这就是 Collaboration 协作能力。Collaboration 协作和人与人的对话很像，不懂就问，互惠双赢。

#### （3）UXNegotiation 用户体验协商机制

Sourcing Agent找到候选人以后，思考通过什么样的方式传回给 Client Agent更好，在能力发现阶段 Client Agent 支持 iframe框架，因此选择通过 iframe 格式回传。

我最开始看到的时候也是比较疑惑，因为我觉得跟前面两个能力比起来，这个更像是某个特定的功能，由于目前没有更多的资料了，这里先按照这个去理解。

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508165926404.png)

#### （4）Task and State Management 任务和状态管理

候选人面试完了，更新状态，接着开始背调，也是通过 Agentcard 去选择，这里是选择到了 Cymbal Background Agent。执行任务过程中发现这个 Agent 只能在美国用，没法背调国际人员，于是又找了其他的 Agent 帮忙，直到最后所有流程顺利结束，更新任务和状态。

### 2.3 A2A 核心概念

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508165937476.png)

通过招聘场景示例，我们已经直观地看到了A2A协议的实际应用的运作流程。

这里简单看下A2A协议的核心概念，AgentCard 能力发现，任务Task，消息Message，任务产出 Artifact，Server 推送通知 Notification，还有流式 Streaming。

这些概念本质上都是普通服务开发中常见的概念，这说明AI Agent开发与传统服务开发具有相同的底层逻辑。AI Agent开发中，这些服务交互环节都会与大模型深度结合，传统服务业务逻辑固定，在AI Agent中可能由大模型动态决策。 

### 2.4 A2A 官方示例 LangGraph Agent

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508165955275.png)

这是官方三个例子中的 LangGraph Agent，该 Agent 提供了查询两个币种之间汇率的能力，会根据用户的输入情况，对接大模型，执行相关的动作。代码非常简洁易运行，唯一就需要想办法弄到 gemini 的 token，样例代码只有 LangGraph Agent 依赖了 gemini，Client Agent 没有依赖 gemini。

这里在原来的时序图基础上，补充了 LangGraph Agent 和 Gemini LLM 的交互。实际运行的流程也非常简单：

用户输入的信息足够，比如美元换英镑，LLM 会判断信息足够了，Agent 会直接去调用汇率查询查询的接口，最终将结果返回给用户。

用户输入的信息不足，例如没有说要转换的目标币种，LLM 会判断信息完整，会提示状态为 input_required，引导用户继续输入。等到用户补充完整后，Agent 最终就去调用汇率查询的接口，最终将结果返回给用户。

这里还支持流式，Server Agent 会向Client Agent 实时推送任务进展。

### 2.5 A2A System Instruction

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508170031297.png)

这里主要是通过代码解释下，大模型在判断输入不够的情况下如何引导用户继续输入的的，看代码就会发现，关键就在于这里的 system instruction 提示词，限定了这个 Server Agent 的行为动作，就像是诸葛亮的锦囊妙计一样：

" 你是一个汇率转换助手，你只管去调用 get_exchange_rate 工具，话题无关的不要回答，发现用户信息不够，response 状态改成 input_required，执行过程中报错，输出 error，信息够了改成 complete。"

等到大模型发现用户输入数据不够，就返回 input_requeired 状态，Client 端就知道是需要继续补充信息了，除了状态，大模型还会提示该补充什么数据，示例是什么。

另外一个需要提一下的是 Agentcard，在标准定义的路径下可以直接获得。这里面的内容很重要，相当于 Agent 的对外名片，决定了其他 Client Agent 会不会选到你，更进一步，会不会正确的使用你。

### 2.6 A2A Client Agent如何选择 Server Agent

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508170050676.png)

到这里，似乎还有一个问题没有解决，Client Agent 在面对一堆的 Server 的 AgentCard 的时候，如何选择最合适的哪个。我们知道一般可以通过代码或者 LLM，例如 MCP 就是采用的 LLM 的方式，A2A 没有明确的定义，上面 demo 中里面 Client Agent 完全没有和 LLM 交互，通过代码选择的。

这里也是尝试了 gemini 2.5 pro 的回答。（如果 google 认为这个说得不对，那锅就他自家大模型背吧😄）。

代码调用和 LLM 调用的优缺点都很明显，用代码调用，速度快，结果确定，不足是针对复杂的逻辑编码复杂。LLM 优点是灵活，复杂的需求可以处理，缺点是慢，不好调试，且反复交互需要消耗大量的 token。没法说哪种方式一定好，还是要根据我们的业务场景来选择，相信未来这里应该还有很大的发展空间。



## A2A & MCP



介绍A2A 和 MCP 可能的协作模式，探讨一些开放性问题。对比同一案例在MCP与A2A两种模式下的实现差异，以及相关的个人思考。仅代表个人观点，部分观点可能存在错误，恳请大家不吝赐教。

### 3.1 A2A & MCP 可能的模式

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508170413661.png)![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

A2A 和MCP 究竟是个什么关系，官方在博客里面在A2A和MCP直接搞了个emoji 爱心 ❤，似乎是想表明 A2A 是和 MCP 协作的。前面 A2A LangGraph demo 的例子似乎和 MCP没有一点关系。在这里，A2A 举了一个和 MCP 协作的场景，也就是上面这张图，配上了一段非常让人费解的说法。

我尝试去了解了这里面的调用方式，看起来就像是途中的路径2，也就是：

a.MCP Server 通过 resources的方式注册其他 BlackBoxAgent 的 AgentCard（BlackBoxAgent 要支持MCP resource），

b.A2A Client Agent 通过 MCP 协议 找MCP Server 拿到 AgentCard （A2A Client Agent 要支持MCP resource）

c.A2A Client Agent 通过 A2A 去调用 BlackBoxAgent。

最终绕了一圈仅仅就是通过 MCP Server 拿了一个 remote Agent 的 card 而已，为什么不直接通过 A2A 发现呢，百思不得其解。难道为了合作而合作？

路径 1，其实就是一个标准的 A2A 调用，和 MCP 没有一点关系。

路径 3：A2A 只管Agent 之间调用，任何涉及工具操作，都需要通过 MCP Server 调用。看起来这个似乎是最符合 A2A 对 MCP 作为工具协议的定位，但是仔细思考，这里面问题也很大：

- a.权责不明，比如像官方例子中，remote Agent 直接调用一个 api 就能完成的事情，非得不让调用，重新找个调用这个 api 的 MCP Server？

- b.协议混跑，更杂乱了，A2A Agent 需要支持完整的 MCP。

- c.MCP Server 完全退化成一个调用工具的工具人，MCP Client 用不上，MCP Server 只做透传，存在的意义又在哪里。

因此，路径2，路径3 的合作方式都不是很理想。



### 3.2 A2A & MCP 开放性探讨

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508170449924.png)

#### 3.2.1 技术路径依赖差异

在当前都依赖和 LLM 对话结果的情况下，MCP选择工具强依赖LLM，sampling 需要 Server 请求 Client 再请求 LLM ，耦合严重，问题更加复杂。可以说模型优化优化不好，MCP demo 都跑不起来。相比而言，A2A Agent 选 Server 可以由代码选择，也可以和 LLM 混合，A2A 对于模型商，应用开发者都更友好。

MCP host基本就是绑定在在一个LLM上， A2A 则每个 Agent 可以按需 (准确率，性价比等)选择最合适自己的 LLM。

#### 3.2.2 社区运营能力

技术伙伴支持力度，A2A协议的开源生态已获50+企业支持，项目的活跃程度增长远超MCP。

技术把控能力上，目前看来 MCP 还有很多进步的空间：

MCP 官方 Java SDK 采用Spring框架开发导，经验显然不足。

MCP 引入 Streamable HTTP，稳定性和所有sdk的适配工作量不小，是否是当前 MCP最重要且紧急的事情？有没有成熟的方案可用？感觉社区在讨论这个问题上并不充分。

未来 A2A会不会直接引入 gRPC，其高效的压缩能力（上量后必须要压缩）、原生流式支持、完善的多语言SDK、成熟的生产环境支撑以及广泛的开发者认可，都让人想不到不采用gRPC 的理由！

只能说水很深，就看 MCP 能不能把握得住了！

#### 3.2.3 LLM参与度平衡策略

对于稳定型任务（指定特定的调用路径）或时延敏感型任务（如数据库查询）或有降本诉求的任务，是否支持bypass LLM直接调用的方案，或者有相关的 cache 方案。

#### 3.2.4 系统可维护性

A2A 和 MCP都在关键的节点上依赖 LLM 模型，不确定性导致定位问题复杂，模型升级影响难以判断，如何保证任何的变更已有的工作流程不受影响，如何回归测试

MCP 的 sampling 导致 Server 和 Client 耦合（必须要去调用Client 用的 LLM），想独立测试Server变得更复杂。

当前代码框架还比较初级，需要手动处理的异步编程代码不少，后续定位问题复杂。

#### 3.2.5 生态价值

MCP：MCP Server没有直接和 LLM交互的能力，专注某一方向的功能，更像是工具人，商业模式不清晰，后续MCP Server的开发者是否有动力继续推进，值得观察。

A2A：Agent可提供更加丰富的能力，更具备商业价值，不仅可以与 LLM 交互，Server 还可以选择最合适的 LLM，Agent 和 Agent之间的边界更加清晰。

#### 3.2.6 权限 &安全

没有安全就没有一切，这是决定 MCP A2A能否成功的关键，是一个非常大的议题，限于篇幅，本次不展开，后续有机会专门来讲。

### 3.3 同一案例的 MCP 和 A2A 模式对比

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508170501665.png)

这里我们将之前的 MCP 例子通过 A2A 来实现以便，那可能就会有个汇率报告的 Agent，其 AgentCard 表明它具有直接输出图文报告的能力，不足的数据也会预测。

Client Agent 通过分析用户的需求后，发现这个 Agent 是满足需求的，于是通过 A2A 发起相关的任务。汇率报告 remote Agent 也有大模型的能力，可以能理解用户的诉求直接生成报告了，实在搞不定，还可以去请其他 Agent 帮忙。

这里我们很容易发现 A2A 模式的两个显著特点：

A2A 的Agent 有大模型的能力，能提供更有价值的结果，而不是仅仅只当一个工具人，更具有生态价值，对于开发者来说更有吸引力。

A2A 的Agent 可以按需选择（基于成本，延迟，效果等因素）适合自己的LLM，Agent之间解耦，边界清晰，更容易定位问题。A2A 模式不只围绕一种 LLM 运作，可以同时让多个 LLM 参与进来，对于 LLM 提供商来说更有吸引力。

### 3.4 从 Kubernetes 和 Docker 来看 AI 协议的未来

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508170513214.png)

接下来我们谈谈另外一个事情，看看能不能从中得到一些启发，现在我们都知道当前 K8s已经不支持 Docker 了，准确来说是在 Kubernetes v1.24版本以后弃用了Docker。以下的时间点不一定准确，大家权当故事看看就可以。

- 从 2013年 K8s 开始出现的时候，就一直围绕着 Docker 进行调度框架的开发，与此同时，K8s推动标准接口 cni, cri, csi 的发展，目的也很明确，不想被特定的产品绑定。
- K8s 一度扶持过 coreos 的 rkt 容器，但是效果很一般。本着自己xxx，别人也别想xxx（共同进步😄）的心态 ，coreos 在支持 cni, cri, csi 这些标准接口的时候非常积极，一直冲锋在最前面。
- 2014 年左右 Docker意识到了危机，提出自己的容器编排，Docker compose , Docker swarm，但最终都没有在生产环境大规模应用。
- 2016年左右，K8s 逐渐成熟了，Docker 被迫剥离 containerd 捐给云原生基金会，实际上这也是一个无奈之举，当时不捐，rkt 也会去做这个事情。此时 K8s 还没把事情做绝，还整出了 Dockershim 这种适配层去兼容 Docker，与此同时也再不断完善 CRI。
- 最终到了2020，正如我们所知道的，K8s 发布了备受关注的"讨伐檄文"，弃用了Docker，并在两年后的 v1.24版本移除了Dockershim，从此不再支持Docker。

很多人一直说 Docker 公司太过闭塞才导致了今天的结果，但其实从商业角度来看，他们的做法并没有错，有时候时运很重要，如果当年 Docker的容器编排成功了，或者多一些伙伴的支持，今天鹿死谁手还真不一定。K8s 能最终干掉 Docker，与很多公司支持有很大关系，特别是那些和 Docker 存在利益冲突的伙伴，这也是为什么 A2A 眼下要重点提出已经获得了 50+ 技术伙伴的支持的原因。

分析下更深层次的原因，我认为是K8s站在编排视角，承接了用户的直接需求，通过协议的方式规定底层的运行方式，从一开始就不去绑定特定的产品，把主导权牢牢抓在了自己手上。而Docker 在编排领域的失败，意味着失去了获得用户需求的机会，只能当工具人，从这点来看，Docker 出局几乎就是注定的。

历史总是惊人的相似，A2A 非常像 Kubernetes，也还都是google 公司提出的，同样 A2A 是站在了一个更高的维度来考虑 ai Agent 的发展，至于如何去调用工具，A2A 目前还不关心。如果A2A 以这种方式同 MCP Server 合作，那随时都可以替换掉 MCP Server，或者要求 MCP Server 去做某些改动。MCP Server 没有 MCP Client，就像没有容器编排的 Docker 一样，没有了灵魂。

K8s 和 Docker 的爱恨情仇会不会重演，博客上 A2A 和 MCP 之间的❤能存在多久，有待观察。

### 3.5 个人对 A2A 和 MCP 的看法

![图片](./A2A%E4%B8%8EMCP%E4%B9%8B%E4%BA%89.assets/640-20250508170521104.png)

最后总结几点个人见解（仅代表个人观点，欢迎讨论）：

A2A 的 Agent 开发者有更大的操作空间，生态价值更高，多 LLM 参与的可能性更高，因此未来不论是 LLM 提供商还是开发者应该都很愿意接受A2A。

A2A架构下的Agent未必需要具备与LLM交互的能力，尤其在某些在规则明确的业务场景中，当基于确定逻辑的Agent在效率与成本维度超越大模型驱动的动态型Agent时，更值得我们去认真思考。但笔者对此持审慎乐观态度：这种技术竞争态势或将促使行业聚焦于真实业务痛点的模块化解决方案，而非延续对大模型交互范式的盲目追逐。

目前 A2A 和 MCP 的协同场景让人看起来很拧巴，未来如何发展有待进一步观察。

A2A和MCP的实现高度依赖大模型，需着重解决LLM在成本控制、响应速度及系统可靠性方面的挑战，这对商业化高要求场景尤为重要。

现阶段A2A与MCP框架仍处于初级阶段，提升开发效率和优化问题排查机制是未来发展的关键。

国内AI协议生态在现有世界格局、技术格局下面临重大发展机遇，有时间我们再来聊聊ANP。

这几年 AI 界可以说每天都有新概念，新成果，给人一种学不完，根本学不完的感觉。其间普通人的参与程度也慢慢变高，从全民预训练，全民微调，全民强化学习，全民 RAG。又在某个被 LangChain 折磨得苦不堪言的深夜，骂了几句以后，偷偷点开了 MCP 的 specification，到现在，A2A 横空出世。

人生没有白吃的苦，也没有白走的路，所有技术都是在磨砺中成长与进化，最终走向成熟， AI 走到今天更是验明了这点。

拥抱 AI，拥抱变化，拥抱未来。
