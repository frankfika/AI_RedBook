# Mermaid语法 ｜在Notion上用代码实现“思维导图”绘制





![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551335)





   正文内容从这里开始（可直接省略，亦可配图说明）。Notion已经成了我的生活必需品，但是notion有很多功能欠缺，比如思维导图。。。但是今天我发现，其实只是之前忽略了，**notion的思维导图竟然是藏在code的模块里面，需要用代码的语法来书写，这个语法叫Mermaid，有点类似于JS，核心是可以借助markdown来动态完成图形的生成。**（这个其实在数据分析领域还是有很多库的，比如panda）。



但是我不得不说，我真的不大喜欢这种用户体验。这一看就是工程师的手笔，炫技型的产品设计，忽略了大部分用户的能力，莫名其妙的提高了很多学习成本。。。Anyway，人在屋檐下不得不低头，为了notion其他的完美设计，忍了，让我们花半小时来学习notion里面做脑图（当然***Mermaid***不止能干这个）的方法吧。







**01 流程图（可以为简易版脑图）**

------



![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551386)



其实Mermaid里面的脑图看起来并不像我们所要的脑图，反而简化版的流程图更像一点。为了方便，我就简单介绍下流程图先**。以下是一个简易版的示例：**



```
解释



---title: Node with text---flowchart BT    A[Christmas] -->|Get money| JOHN(Go shopping)    JOHN --> C{Let me think}    C -->|One| D[Laptop]    C ---|Two| E[iPhone]    C -->|Three| newLines["Line1    Line 2    Line 3"]    
```

![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551341)



- Title: 上下必须包含两个 --- 等分隔符，并且写明是 title

- 格式声明: 开头第一行的“flowchart”表示下面在写的是流程图，后面的字母“TB”表示自上而下，同样表示的还有LR（从左到右），RL（从右到左）等。

- 实体 (Entity): 每个固定的entity代表是以字母或者单词来代替，比如文中的“A”或“John”，同时每个entity里面的文字用括号来说明，但是不同的括号会代表不同的形状主体，比如 [] 代表正方形，() 代表椭圆形，{} 代表棱形等。

- 链接符: 在每个实体之间的关联可以用 - → 来表示，代表有剪头关联；- - - 代表无箭头关联；- →|as|, 其中 as 代表文字，同理如果是没有箭头的指向只需要把 > 改为 - 就可以，其他不变。

	![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551376)

- 新的分行: 使用 newlines 表示，里面每一行单独分行写就好。



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
解释



flowchart     id0[长方形]    id1(长方形，代圆角)    id2([椭圆形])    id3[[子程序形状中的节点]]    id4[(圆柱形)]    id5((圆形))    id6>不对称]    id7{棱形}    id1 --o id2    id3 --x id4    id5 <--> id6    id5 ----- id3    id5 --NO--- id3
```

![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551354)

**
**

**
**

**
**

**02 脑图**

------



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
解释



mindmap  root((mindmap title))    Origins      Long history      ::icon(fa fa-book)      Popularisation        British popular psychology author Tony Buzan    Research      On effectivness<br/>and features      On Automatic creation        Uses            Creative techniques            Strategic planning            Argument mapping    Tools      Pen and paper      Mermaid
```

![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551345)



其实仔细看看这个mindmap的逻辑非常简单，在声明origin（初始节点）后，基本每一行都代表同一层级的节点，只要保持在同一行，就是同一层级的节点。这个语法其实很像python。



当然这里面的entity图形其实是可以变化的，语法基本类似于flowchart的entity的图形，但是也有些变化，具体的可以参考官方文档。**但是该说不说，这个的写法确实比flowchart会简单很多。**

**
**

**
**

**
**

**03 类图**

------



- 
- 
- 
- 
- 

```
解释



classDiagram    Animal <|-- Duck    Animal <|-- Fish    Animal <|-- Zebra    class Animal{       +int age       +String gender       +isMammal()       +mate()    }
    class Duck{      +String beakColor      +swim()      +quack()    }    class Fish{      -int sizeInFeet      -canEat()    }    class Zebra{      +bool is_wild      +run()    }
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/zvStibNiazLDLSGvms95LLrF8B050cfP6d1dQ6a6l3KrwrVDiaPo9ght8CYT4DfZQO2aUkZvugJXOjfgmhCZnicHyw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)



这部分写法写实和在具体代码里面写 interface 几乎一样，不带括号就是普通的变量，带括号就是 function。<|-- 符号的左边一般表示是父类，右边是继承的子类。







**04 甘特图**

------



- 
- 
- 
- 
- 
- 
- 
- 
- 

```
解释



gantt    title A Gantt Diagram    dateFormat  YYYY-MM-DD    section Section    task           :a1, 2014-01-01, 30d    Another task     :after a1  , 20d    section Another    Task name2      :2014-01-12  , 12d    another task      : 24d
```

![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551346)



- Section部分: 其实就是左边的横向坐标轴，写法就是 section + 名字。
- task部分: section 下面可以写一堆 task，格式就: 之前就是这个 task 的名字，后面的参数分别为“task 的代号 (optional)，开始的时间点 format (取决于你之前的定义的 format)，时间跨度 (d=天数)”。如果说一个 task 的开始时间在另一个之后，就写 after x (之前的 task 名字)

**
**

**
**

**
**

**05 饼图**

------



- 
- 
- 
- 
- 
- 
- 

```
解释



%%{init: {"pie": {"textPosition": 0.5}, "themeVariables": {"pieOuterStrokeWidth": "5px"}} }%%pie showData    title Key elements in Product X    "Calcium" : 42.96    "Potassium" : 50.05    "Magnesium" : 10.01    "Iron" :  5
```

![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551378)



饼状图其实很简单易懂的。

- title: 就写在第一行，后面写出 title 的名字
- init 的格式设置: 简单的配置，就是设置位置，主题的一些 css 的细节等，这里面如果不熟悉可以忽略。（可以试试，其实删了也无所谓）
- pie showData: 这里面的 showdata 的参数可以去掉，这样数据的具体数值就不会显示出来，只会显示百分比

**
**

**
**

**
**

**06 象限图**

------



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
解释



quadrantChart    title Reach and engagement of campaigns    x-axis Low Reach --> High Reach    y-axis Low Engagement --> High Engagement    quadrant-1 We should expand    quadrant-2 Need to promote    quadrant-3 Re-evaluate    quadrant-4 May be improved    Campaign A: [0.3, 0.6]    Campaign B: [0.45, 0.23]    Campaign C: [0.57, 0.69]    Campaign D: [0.78, 0.34]    Campaign E: [0.40, 0.34]    Campaign F: [0.35, 0.78]
```

![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551384)



- 主题：tilte之前有说过，和其他表类似的语法；
- 象限横纵轴名称：第二行定位 x 轴象限的两个区域的名称，语法为 x-axis Low Reach → High Reach (这里面“low reach”和“high reach”分别代表两个量)。同理第三行的 y-axie 也是如此。
- 每个象限的名称：quadrant-1，quadrant-2，quadrant-3，quadrant-4 . 定义每个象限的具体名称，语法简单，这里不做赘述。
- 象限上的点: 格式为 a:[x,y]，其中 a 为该点在图片显示的名称，x 和 y 为坐标轴的数值。比如 Campaign A: [0.3, 0.6]。







**07 时间线图**

------



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
解释



timeline    title Timeline of Industrial Revolution    section 17th-20th century        Industry 1.0 : Machinery, Water power, Steam <br>power        Industry 2.0 : Electricity, Internal combustion engine, Mass production        Industry 3.0 : Electronics, Computers, Automation    section 21st century        Industry 4.0 : Internet, Robotics, Internet of Things        Industry 5.0 : Artificial intelligence, Big data, 3D printing
```

![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551394)



基本语法和前面表格相似不做赘述。特别的是，每个 section 代表着垂直的 entity，然后在此之下每个层级再度精确，具体的见上述表格。如果太复杂，可以做一个简单的版本，如下：



- 
- 
- 
- 
- 
- 
- 

```
解释



timeline    title History of Social Media Platform    2002 : LinkedIn    2004 : Facebook : Google    2005 : Youtube    2006 : Twitter
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/zvStibNiazLDLSGvms95LLrF8B050cfP6dqmzHmHgBLricemczHd9zVRMKvu41WusN3fjiaSL8AicR0JLb1eia3iaWj0Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)







**总结 |**

------



![图片](./Mermaid%E8%AF%AD%E6%B3%95%20%EF%BD%9C%E5%9C%A8Notion%E4%B8%8A%E7%94%A8%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E2%80%9C%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE%E2%80%9D%E7%BB%98%E5%88%B6.assets/640-20250503223551386)



以上就是基本的教程，其实不要写的太复杂（我其实已经写复杂了），只用最简单的版本绝大多数时间就够用了。要是觉得还不够，就百度搜吧，，相关资料还是蛮多的。**不论是主题风格，颜色，还是一些其他的都可以个性化，而且可以直接部署在网站上**。

  

