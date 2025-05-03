# Deepseek+本地文件=私有化本地大脑！





![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524681)













**deepseek出来之后，很多朋友都在想怎么做一个自己的本地知识库系统，今天就给大家一个教程，教大家用cherrystudio+RAG来做一个简单的本地大模型知识库。**







------



▌什么是RAG技术？
检索增强生成（Retrieval-Augmented Generation）是一种让AI模型结合本地数据库生成回答的技术，就像给ChatGPT连接了一个专属知识库。本文演示的正是这种技术的本地化实现方案。



------



**01**

**找到合适的LLM源**

------



Deepseek以及其他类似的大模型，目前在很多地方都有部署。**你既可以通过Ollama实现本地化的调用，也可以通过硅基流动等平台实现远程API的调用。**具体的就不在这里详细描述了，有需要的朋友圈可以参见之前的文章：



------



- [DeepSeek官网瘫痪？这几个平台也可以“白嫖R1”，建议收藏！](https://mp.weixin.qq.com/s?__biz=MzA4MTA3MDQ2MA==&mid=2650743706&idx=1&sn=d2ecc60bdc1782a7f0a3bfaa3c2faab6&scene=21#wechat_redirect)
- [AI工具党！DeepSeek-R1 5分钟本地部署指南](https://mp.weixin.qq.com/s?__biz=MzA4MTA3MDQ2MA==&mid=2650743630&idx=1&sn=18b5d3df97decb63254111a40af2cf3f&scene=21#wechat_redirect)



------



但是值得注意的是，因为做本地向量知识库，除了需要基础的大模型之外还需要**embedding的模型**，典型的embedding模型有BAA/bge-m3等，清华智谱也有类似的模型，也可以去他们官网调用。这点后面会有详细介绍，另外ollama和硅基流动本身也有这部分，不需要重新注册和配置新平台了。

![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524651)







**02**

**选择最合适的大模型和数据可视化工具**

------



目前可供选择可视化的大模型的工具有很多，而且都是开源的，比较典型的分别是：



- **chatbox：**开源的。支持多终端使用，但是不支持本地知识库微调。![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524619)![img]()

	

- 
- 

```
解释



GitHub: https://github.com/Bin-HuangWebsite: https://chatboxai.app/zh
```

**
**



- **AnythingLLM：**开源的应用。支持对话和微调。欧美风格，比较全面，但是使用习惯偏向欧美。有些复杂，我个人使用不喜欢。不支持多终端。

![anything-llm/locales/README.zh-CN.md at master · Mintplex-Labs/anything-llm  · GitHub](https://mmbiz.qpic.cn/sz_mmbiz_png/zvStibNiazLDL0rEQ4BReG6IM9BylQJo06iakHjrrsEjFWJIaVdibDXElImiauFaaOZfYfZfwEEia9MyngVta707fzBA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

- 
- 

```
解释



GitHub: https://github.com/Mintplex-Labs/anything-llm Website: https://anythingllm.com/desktop
```





**cherrystudio：**开源的。比较适合中国人的习惯，用起来比较舒服。不过功能相比anythingllm确实没有特别强大。不支持多终端。

![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524623)

- 
- 

```
解释



GitHub: https://github.com/CherryHQ/cherry-studio/anything-llm Website: https://cherry-ai.com/
```





- **其他：fastgpt，dify，**

![FastGPT - 企业级智能AI模型问答知识库](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524616)



这俩其实比以上几个都好。但是我们这里讨论的是本地化部署，这俩本地化部署都需要安装docker，我就不做讨论了。我个人建议有条件的，使用dify或者fastgpt做rag效果更好。



**这里我选择使用cherrystudio，作为本次微调的方式**







**03**

**上手操作**

------



**Step 1:下载cherry studio到本地**

官网地址：https://cherry-ai.com/





**step2:做好Model的配置**

点击设置——>model provider。然后选择对应的provide供应商，填好前面的信息（API Key和API host等），就可以点击manage，来管理模型。

- 如果是硅基流动就按照如下配置，记住要多配置一个embedding model，比如**BAAI/bge-large-zh-v1.5和****Pro/BAAI/bge-m3**等。

![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524612)



- 如果是ollam就按照如下配置，另外除了pull下基础模型外，比如deepseek r1或者llma，记住要多pull一个embedding的模型，具体命令是：

	- 

	```
	解释
	
	
	
	ollama pull mxbai-embed-large
	```

	- 

- 

- 

![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524712)





**Step3:部署数据库**

点击左边的kownledge base（倒数第二个按钮）在新建的时候会选择一个知识库名，再加一个embedding的model。这时候你就选择之前前面配置过的model。（怎么选好呢？简单道理，越贵越好）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/zvStibNiazLDL0rEQ4BReG6IM9BylQJo06YPSmY9CTwMibPriapM7bhJQnpyibTNMMZl45sia9QzXISZjcibUp1DbGicew/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)



![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524812)



然后上传对应的知识文件，当你看到上传的文件打了绿色的对号就完成了。同时，你也可以通过url，文件夹，和网页等补充信息传输。这里就不做多赘述了

![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524721)





**step4:提问**

在提问的时候，你在选择：

- **模型：**选择模型，比如r1，就是@的形状的小button
- **知识库：**选择你希望关联的知识库系统，就是那个小放大镜的button。（你如果选择了知识库，他就只会针对数据库问题进行query）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/zvStibNiazLDL0rEQ4BReG6IM9BylQJo069q8WghC8DibtODrmAFr16INufufkevBw8GL3f1Z6N5Llkic2tfsjLGUw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)



这样提问最后就全面针对知识库的文件去做回答了。







**04**

**Rag的小技巧**

------



因为embeedding的模型在文档读取和拆分的时候能力不一定强，尤其是针对pdf或者ppt的拆分，中间的逻辑比较难，最好你先清晰的梳理好。



同时针对一些图片形的pdf，或者加密的pdf，一般是无法做有效切分的，这里要注意。尽量转成文本类的格式。所以你**尽量采用excel，markdown，json这三种标准的格式，**其实是最好的。比如你看fastgpt给的标准格式就是这样的（下图），**类似于QA**，这样拆分起来更好理解。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/zvStibNiazLDL0rEQ4BReG6IM9BylQJo06R9DIZg3icPyGeU3NHFRafQBGEHCv47cibgg6uXOtSnTj75oOhHjqw9pA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)







**05**

**更好的实践？**

------



切分后的数据其实照这样。是类似于关键词索引 + 最终数据导向的结果，如下图。所以有时候会弄错。但是cherrystudio等不支持直观结果的展示，所以不好调整。

![图片](./Deepseek+%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6=%E7%A7%81%E6%9C%89%E5%8C%96%E6%9C%AC%E5%9C%B0%E5%A4%A7%E8%84%91%EF%BC%81.assets/640-20250503222524781)



我对比了下，dify和fastgpt对于知识库支持更好。他们都可以针对切分后的模块进行编辑，尤其是fastgpt还支持自己去在和模型沟通的时候，顺便做数据embedding之后的微调。下次我会在写一个如何用fastgpt做RAG的教程，个人觉得这个更加practical。 







