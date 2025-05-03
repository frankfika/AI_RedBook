# 字节跳动开源 LangManus：不止是 Manus 平替，更是 AI 自动化新引擎



当 AI 自动化成为大势所趋，字节跳动悄然掷出一枚重磅开源项目——LangManus。它不仅被视为 Manus 的有力竞争者，更被寄予厚望，成为推动 AI 自动化普及的开源引擎。LangManus 的野心在于，它不仅整合了强大的 LLM 能力，还将网络搜索、网页爬取、Python 代码执行等工具融为一体，目标直指复杂任务的自动化解决。

那么，LangManus 如何重塑 AI 自动化格局？它又将如何赋能开发者？让我们一起揭开 LangManus 的神秘面纱，探寻其背后的技术逻辑与无限可能

![字节复刻了一个manus 并开源了Python 项目： - 即刻App](./%E5%AD%97%E8%8A%82%E8%B7%B3%E5%8A%A8%E5%BC%80%E6%BA%90%20LangManus%EF%BC%9A%E4%B8%8D%E6%AD%A2%E6%98%AF%20Manus%20%E5%B9%B3%E6%9B%BF%EF%BC%8C%E6%9B%B4%E6%98%AF%20AI%20%E8%87%AA%E5%8A%A8%E5%8C%96%E6%96%B0%E5%BC%95%E6%93%8E.assets/640-20250503221413076)





------

**AI 自动化的新变量：从 Manus 到 LangManus**

在 AI 自动化的浪潮中，Manus 以其智能体协作模式，为复杂任务的处理提供了新的思路。如今，字节跳动开源的 LangManus，正试图在 Manus 的基础上，构建一个更开放、更强大的 AI 自动化平台。

**LangManus 的诞生，不仅仅是技术的复刻，更是一次对 AI 自动化未来的探索。**它以开源为底色，以社区为驱动力，旨在将 LLM 的强大能力与各种实用工具相结合，实现任务的自动化处理，并回馈给整个开发者社区。





------

**LangManus 技术解构：多智能体协作，驱动自动化引擎**

**LangManus 的核心竞争力，在于其精巧的技术架构和丰富的功能特性。**

它是多智能体系统，LangManus 采用分层架构，由多个智能体协同工作，实现复杂任务的分解和处理。协调员、规划员、主管、研究员、程序员、浏览器和报告员等智能体各司其职，如同一个高效运转的团队。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

- 
- 
- 
- 
- 
- 
- 

```
*   *协调员 (Coordinator)*：任务的入口，负责接收用户指令并分配任务。*   *规划员 (Planner)*：当任务复杂时，负责制定详细的执行计划。*   *主管 (Supervisor)*：任务的指挥中心，监督任务执行并调用其他智能体。*   *研究员 (Researcher)*：信息的探索者，负责网络搜索和数据挖掘。*   *程序员 (Programmer)*：代码的创造者，负责编写和执行 Python 代码。*   *浏览器 (Browser)*：网页的操控者，模拟用户在浏览器中的操作。*   *报告员 (Reporter)*：结果的呈现者，负责整理任务执行结果并生成报告。
```



- **LLM 深度集成：** LangManus 兼容多种主流 LLM，如 Qwen、OpenAI 等，并采用三层 LLM 系统，以适应不同场景的需求：推理 LLM、基础 LLM 和视觉语言 LLM。
- **工具生态：** LangManus 集成了丰富的工具，包括网络搜索 (Tavily API)、神经搜索 (Jina)、Python REPL 和代码执行环境、浏览器控制等，为任务的执行提供强大的支持。
- **工作流管理：** LangManus 提供了可视化的工作流程图和任务分配监控功能，让用户可以清晰地了解任务的执行过程。
- **API 服务：** 基于 FastAPI 的 API 服务，支持流式传输，方便用户进行二次开发和集成。



值得一提的是，LangManus 还支持 AWS Graviton 和 Docker，进一步提升了其性能和易用性。





------

**快速上手：LangManus 的安装与配置**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**想要体验 LangManus 的强大功能？只需简单几步：**

- **安装**

- 
- 
- 
- 
- 

```
pip install uv  # 使用 uv 包管理器git clone https://github.com/byteplus/lang-manus.git  cd lang-manusuv pip install -r requirements.txtplaywright install
```



- **安装和启动**

	在.env文件里面讲对应的 API 密钥、模型信息等完成配置，然哦呼就可以使用下面命令直接启动了！

- 

```
    python webui.py
```

**更多详细步骤，可以参考github上的文档。** 





------

**快应用场景：LangManus 的无限可能**

LangManus 的应用场景非常广泛，无论是企业内部效率提升，还是个人效率工具的打造，都能发挥重要作用。**核心功能是在大模型的基础上完成任务自动化，即自动化处理多步骤任务，**例如计算 HuggingFace 模型的影响力指数，让繁琐的工作变得简单高效。典型的场景有

- **自动化周报生成：**告别手动整理数据，一键生成周报。
- **智能客服系统：**7x24 小时在线，快速响应用户问题。
- **企业级私密部署方案：**保障数据安全，满足企业内部需求。
- **人力资源：**智能简历筛选，提升招聘效率。
- **房产决策：**数据驱动分析，辅助投资决策。
- **旅行规划：**个性化行程推荐，一键搞定旅行计划。



虽然 LangManus 的目标是向 Manus 看齐，但开源赋予了它独特的优势。开源意味着更多的可能性，更多的创新，以及更强大的社区支持。**尽管 LangManus 在某些方面可能还有提升空间，但它所代表的开源力量，将推动 AI 自动化技术的快速发展。而且相信背后的字节在短期内还会有进一步的发力。**





------

**结语**

LangManus，作为字节跳动开源的 AI 自动化框架，为开发者提供了一个强大的工具，可以更加便捷地构建各种自动化应用。它的开源特性和多智能体协作的架构，使其具备了广阔的应用前景。随着社区的不断完善和技术的不断发展，LangManus 有望在 AI 自动化领域扮演越来越重要的角色。

如果你对 AI 自动化充满好奇，不妨亲自体验 LangManus，或许它将为你开启全新的工作方式。



