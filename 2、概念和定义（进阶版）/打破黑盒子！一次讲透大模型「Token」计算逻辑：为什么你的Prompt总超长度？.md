# 打破黑盒子！一次讲透大模型「Token」计算逻辑：为什么你的Prompt总超长度？



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817910)











**本文用实测数据打假三大神话：**



**❌神话1：英文Token一定比中文少？**

**❌神话2：低价API入场就能省钱？**

**❌神话3：开源模型绝对透明没套路？**

**
**



![img](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/300-20250503221817776.png)

**弗兰克的AI日记**

AI博主，斜杠玩家——前AI独角兽高管/私募投资总监/程序员～

71篇原创内容



公众号









**01**

**Token是什么？**

------



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817833)



大模型（如GPT、Claude等）处理文本时，会先将文本拆解成一个个**「Token」——这是大模型理解世界的“基本单位”。**无论你输入的是问题、回答还是代码，最终都会被转换成数字化的Token（向量）。Token的数量和计算会直接影响大模型**使用的成本计算（****按Token收费的服务（如OpenAI API）直接影响使用成本）、上下文长度限制（****模型的上下文窗口，如GPT-4的8k、32k，本质上是Token的数量上限）、理解误差（****Token拆分方式可能影响模型对专业术语或特殊名称的理解）。**



Token化是将文本分割成Token的过程，中文称为“分词”，这个过程对于大模型理解和处理自然语言至关重要。不同的Token化方法会根据具体应用和需求选择不同的策略。常见的Token化方法包括： 

- **按单词分割：**将文本按空格分割成单词。优点是易于理解，但在处理未登录词（out-of-vocabulary）时会有问题。 
- **按字符分割：**将文本按字符分割。优点是处理灵活，但会生成大量Token，增加计算负担。 
- **按子词分割**：将文本分割成更小的子词单元，平衡了词级和字符级的优缺点。常用的子词分割算法有：Byte Pair Encoding（BPE） 、WordPiece、SentencePiece、Unigram。







**02**

**Token的历史由来**

------



要理解现代大模型的Token，**必须回到自然语言处理（NLP）的核心问题：****人类的文字？**



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817852)



传统时代的「词语分割」英文先天优势，依靠空格分隔单词，早期NLP只需按空格拆分即可（如“machine learning”=2个词）；但是对于中文，就是地狱难度，因为中文无显式分隔符，需要依赖字典或统计模型（如“人工智能”需判断是“人工/智能”还是“人/工/智能”）。



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817825)

2015年革命：Google提出「字节对编码」（BPE），将字符与子词结合，让模型自动学习最佳分割方式，核心逻辑是高频组合保留（如“ing”），低频拆分（如“anthropology”→“anthro”+“pology”）。



 **让token取代传统的分词成为计算机的语言的「数字化基因」，从此，无论是代码、表情符号或火星文，统统可被切割为Token。**



- ##### **典型Tokenizer的演进史**

![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817822)

##### ** **





**03**

**Token不等于Byte，两者的本质区别？**

------



正如上文所说，token是在传统的byte的演进基础上的一个新的分词方式。传统字符编码（Byte字节）与Token的本质差异**「字符编码」（如一个汉字占2字节）和「大模型的Token」是计算机处理文字的两个不同层面的概念**，二者无直接关联。举例如下：



------



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817872)



------



### **为什么人们会混淆这两个概念？**

------

### **早期程序员思维，\**“1个汉字=2字节”是针对二进制存储的比喻，但这与语义无关。但实际上，Token的根本逻辑是语义拆分，而非物理存储的“价格替身”。\****

### **计算成本的直觉联想用户易认为“占存储多的汉字理应付更多钱”。中文Token数普遍更高的“感性验证”。比如**

- 
- 
- 

```
解释



英文："Hello" → 1 Token中文：“你好” → 2 Tokens（直接联想：“2个汉字=2 Token=更贵”）
```





**为什么我们需要token而不是byte？**

------



你想想如果token消失了会怎样？**理想世界——模型直接理解“你好”的UTF-8编码 0x4F600x597D ，无需中间商赚差价；然而现实是：26万汉字×6万词组=156亿组合，当前算力无法承受如此高维向量计算。**除此之外，还有更好的数学规约，可以更好的把Token将文本映射到固定维度的向量空间，每个Token对应一个ID（如“你好”→ [101, 202]。还有 统一接口的原因：无论是Python代码、文言文还是emoji🌈，全部转换为Token序列。



Token的本质，是算力与语言复杂性之间的无奈妥协 —— 而**中文，正站在这场妥协的风暴中心。Token具体拆分取决于编程语言的常见组合（如“Python”是保留词为1 Token），而字符编码仅是物理存储的“管道工”。**



####   

**04**

**中文真的更消耗token吗？**

------



**不完全对！**



最早的BPE确实是为英文设置的，这也导致了很多高频字母组合的天然适配。**除了英文之外的包括中文在内的其他语言，会消耗相对更多的token，因为缺乏好的tokenizer的拆分，且最初的模型并不是为了设计的。**

![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817867)



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817880)



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817814)



但是随着各种中文模型的崛起，模型“学”会合并高频词（如“中国”视为1 Token），因此中文会消耗更多token的问题也有了新的答案！这也导致了很多中文模型实际的token消耗量可能远小于英文的token，也会导致**同一句话，不同的模型消耗的token会产生较大的差异。**



![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817893)



虽然经过中文优化的模型（如ChatGLM、通义千问）确实对中文Token压缩能力更强，但总体而言，对于通用模型（如GPT-4）等，同样的语意，中文所消耗的token远多于英文，仍在追赶。





**“库”是死的，但规则是活的**

------



- **冻结的规则库：模型训练完成后，Tokenizer的合并规则就固定了（例如GPT-4的词表不会随时间变化）；****
	**

- **数据的魔法：训练时的语料决定高频组合。如果训练数据中“人工智能”反复出现，合并概率暴增。**

	**
	**



**小众语言toien的灾难：用科技霸权解释「语言贫富差距」** 

------



***\*相比中文和英文，小众语言会在token消耗上面显得更加无力。\**其实本质上，token从侧面印证了嫌贫爱富的，科技霸权来解释语音贫富差距。比如下面藏文的“你好“拆解”。**

- 
- 

```
解释



text_en = "Hello, world!" → 3 Tokens  text_ti = "ཀུན་གཟིགས་བདེ་ལེགས།"（藏语“你好”）→ 拆为单字 → 7 Tokens
```

**
**

**核心原因还是**

- ***\*数据贫瘠 ：\**训练语料不足 → 模型学不会词合并；** 
- ***\*拆分粗暴 ：\**单字Token导致冗余，成本陡增；** 
- ***\*商业忽视 ：\**大厂优先优化高使用率语言（中/英/西/法）。其实本质上，token从侧面印证了嫌贫爱富的，科技霸权来解释语音贫富差距。**

***\*
\**\**
\**\**\*\*典型token在不同「语言」和不同「模型」下的案例：\*\**\***

------

### ***\*案例一：\****

- 

```
解释



近日，国家卫健委宣布将人工智能辅助诊断系统应用于基层医疗，该系统能自动识别CT影像中的早期癌症病变，准确率达95%以上。专家表示，此举有望缓解医疗资源不均衡问题。  
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/zvStibNiazLDJsRvZmpsHbY5mnvNy2nrfl1lrk6Ly0y2dzO3OEXq0LoLGes2oyXu0080TDfjOq5TBD0iajkPJz3yg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817946)

### ** **

### ***\*案例二：\****

- 

```
解释



Content: Recently, the National Health Commission announced the application of AI-assisted diagnosis systems in primary healthcare. The system can automatically detect early-stage cancer lesions in CT scans with an accuracy rate exceeding 95%. Experts say this move will help alleviate the imbalance in medical resources.  
```

![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817949)

### ** **

![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221817953)

### ** **

```
解释
```

**
**

**05**

**灵魂问题**

------





**Q1：用英文提问真的比中文便宜吗？**
→ A: **不一定！** 例如短句“你好”→2 Token，对应英文“Hello”→1 Token；但长文本可能因中文的词语合并优于英文的长单词拆分。





**Q2：怎么计算你的token的费用？**

→ A: 每个大模型都提供自己的API，可以测试某一段文字的token的使用多少。当然你可以去一些开源的工具或者网站测试下你的token用量，比如这个：https://tiktokenizer.vercel.app/





**Q3：为什么同样的内容，不同模型Token数不同？**
→ A: 因各家模型使用不同的Tokenizer，同一段文本在GPT-3.5和Claude-2中的Token数可能有10%~30%差异！





**Q4：可以定制token吗？如何省钱？**

→ A:

- **选对模型：**国产模型优先，通义千问、ChatGLM对中文Token有深度优化，“区块链”可能直接合并为1 Token。

- **删冗余词：**去掉“的”“了”“呢”，Token数立减15%。

- **空格诈骗术：**在中文中插入空格（如“人工 智能”），可能“骗过”BPE合并！

- **缩写大法：**“概率图模型”→缩写“PGM”（英文缩写大概率合并为1 Token）。

- **行业黑话库：**金融、医疗等行业预定义专业术语合并表（如“非小细胞肺癌”→“1 Token”）；

	

![图片](./%E6%89%93%E7%A0%B4%E9%BB%91%E7%9B%92%E5%AD%90%EF%BC%81%E4%B8%80%E6%AC%A1%E8%AE%B2%E9%80%8F%E5%A4%A7%E6%A8%A1%E5%9E%8B%E3%80%8CToken%E3%80%8D%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E7%9A%84Prompt%E6%80%BB%E8%B6%85%E9%95%BF%E5%BA%A6%EF%BC%9F.assets/640-20250503221818004.png)



------

