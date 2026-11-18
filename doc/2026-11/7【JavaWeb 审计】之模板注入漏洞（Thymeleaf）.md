#  7【JavaWeb 审计】之模板注入漏洞（Thymeleaf）  
 初梦的安全小屋   2026-11-17 13:50  
  
> 阅读指引：如果你是第一次接触"模板引擎"和"模板注入"，建议从头按顺序读。本文先用生活化的例子把基础概念讲清楚，再逐步过渡到 Thymeleaf 这个具体框架，最后才是漏洞本身。文中的代码块和链接都保留完整，遇到看不懂的英文缩写不必慌，第一次出现时都会给出解释。  
## 一、Java 模板引擎与漏洞  
  
这一章先回答两个最基础的问题：模板引擎是做什么用的？它为什么会和安全漏洞扯上关系？把这两点弄明白，后面的内容就容易理解了。  
### 1、模板引擎介绍  
  
目前  
 Java 常见的模板引擎有 FreeMarker、Velocity 和 Thymeleaf。  
  
模板引擎，其实就是一个"根据模板和数据生成最终文本"的工具，目的是让页面显示  
和数据内容  
分开。可以用一个生活化的例子来理解：模板就像一张填空试卷，数据就是要填进去的答案，模板引擎则负责把答案自动填到对应的空格里，最后生成一张完整的网页。我们都了解  
 JSP，它其实也是一种模板引擎，通过后端向前端插入数据来实现动态网页的效果。  
  
在  
 JSP 之前，人们使用 servlet 来实现动态网页效果。servlet 是 Java Web 最底层的组件，但直接用它生成网页非常繁琐——想把一个页面返回给浏览器，得用 out.write() 一行一行地把 HTML 代码拼接出来，比如：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">out.write(&#34;&lt;html&gt;\n&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">out.write(&#34;&lt;head&gt;&#34;\n)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">out.write(&#34;&lt;h1&gt;hello servlet&lt;/h1&gt;\n&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">out.write(&#34;&lt;/head&gt;\n&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">out.write(&#34;&lt;/html&gt;\n&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
可以看到，连一个最简单的页面都要写这么多行输出语句，既累赘又容易出错。虽然  
 JSP 也是模板引擎的一种，但因为它本质上是 servlet，可以直接、无阻碍地访问底层的 servlet API，还能在页面里编写后端代码逻辑，导致前端和后端纠缠在一起，违背了面向对象"低耦合、高内聚"的设计思想，维护起来也越来越繁琐。因此经过演变，加上优秀的模板引擎不断出现，JSP 逐渐被替代，这其中就包括 FreeMarker、Velocity 和 Thymeleaf 等。  
  
（延伸阅读：JSP 为什么被淘汰了？https://www.zhihu.com/question/328713931）  
  
言归正传，用大白话再解释一遍模板引擎：比如前端做好一个展示人员信息的模板页面，在姓名、性别、年龄这些位置，用模板引擎规定的"变量符 / 指令 / 插值"做占位标记，后端查询回来的数据就能动态地填进这些占位处。模板引擎的核心价值就在于——同一个模板，填入不同的数据，就能生成不同的页面。  
### 2、模板注入漏洞  
  
SSTI（Server-Side Template Injection，服务器端模板注入），通俗地说，就是模板引擎在解析模板时，如果代码写得不够严谨，可能把攻击者提交的恶意内容当成了模板的一部分来执行。攻击者可以借此在服务器上执行任意代码，造成严重的安全威胁。  
  
简单讲：模板引擎渲染模板时，如果模板里混进了恶意代码，这些代码就会在渲染过程中被执行。不同的模板引擎触发漏洞的场景各不相同，下面我们会针对  
 FreeMarker、Velocity 和 Thymeleaf 这三款逐一讲解。  
  
这里要先建立一个关键认知：模板注入漏洞是依附于模板引擎存在的  
。项目里如果没有引入相关的模板引擎组件，基本就不会有这类漏洞；只有引入了组件，同时又用了不安全的编码方式，风险才会出现。  
  
本期我们聚焦  
 Thymeleaf 模板引擎。本节先从 Web 场景切入讲解 Thymeleaf 模板注入漏洞，漏洞原理分析部分将放在后续阶段讲解。  
  
现在请大家先把视角放在  
 Web 层。换个角度说，模板注入漏洞是发生在框架层面的漏洞——Web 项目里到底存不存在这个问题，还得看项目有没有引入相关模板引擎框架，以及有没有使用不安全的编码方式。  
## 二、Thymeleaf 模板引擎  
  
这一章先认识  
 Thymeleaf 是什么、怎么用，再通过一次完整的 SpringBoot 整合实操建立直观感受。不用急着记语法，先理解它解决了什么问题即可。  
### 1、什么是 Thymeleaf  
  
官方网站：https://www.thymeleaf.org/  
  
Thymeleaf 是一种用于 Web 环境和独立环境中的现代化服务器端 Java 模板引擎，能够处理 HTML、XML、JavaScript、CSS 甚至纯文本。它的特点是使用"自然模板"语法——也就是说，模板文件本身就是可以直接在浏览器里预览的规范 HTML，美工和前端能直接看懂、修改和维护，这是它区别于传统模板引擎的一大优势。此外 Thymeleaf 还支持扩展，你可以添加自定义标签和扩展函数来满足自己的需求。  
  
Thymeleaf 的主要特点包括：  
  
l  
   
可以处理  
 HTML，XML，JavaScript，CSS 等各种文本格式  
  
l  
   
可以在不同的  
 Web 框架中使用，如 Spring，Spring Boot，Play Framework 等  
  
l  
   
自然的模板语法，易于学习和使用  
  
l  
   
可以与其他  
 Java 技术集成，如 Spring Security，Spring Data 等  
  
l  
   
支持模板片段，使得模板的重用性更高  
  
l  
   
可以在运行时或者静态地执行模板，方便模板的测试和调试  
  
正因为  
 Thymeleaf 易学易用，又能与许多 Java 技术无缝集成，它成了 Java Web 应用程序中最受欢迎的模板引擎之一。下面是一个简单的 Thymeleaf 样例，先大致看一下它的写法，不用逐行深究：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;table&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;thead&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;tr&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;th th:text=&#34;#{msgs.headers.name}&#34;&gt;Name&lt;/th&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;th th:text=&#34;#{msgs.headers.price}&#34;&gt;Price&lt;/th&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/tr&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/thead&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;tbody&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;tr th:each=&#34;prod: ${allProducts}&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;td th:text=&#34;${prod.name}&#34;&gt;Oranges&lt;/td&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;td th:text=&#34;${<a class="wx_topic_link" topic-id="mwkf091x-wlwg6b" style="color: #576B95 !important;" data-topic="1" data-recommend="">#numbers</a>.formatDecimal(prod.price, 1, 2)}&#34;&gt;0.99&lt;/td&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/tr&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/tbody&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/table&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
像  
   
th:text  
、  
th:each  
 这种以  
   
th:  
 开头的属性，就是  
 Thymeleaf 的"指令"，用来告诉引擎"这里要填入数据"或"这里要循环生成"。后面会逐一用到。  
### 2、Thymeleaf 基础  
  
以下内容整理自官方文档，文档是英文版的，根据自己的理解做了部分整理。官方文档地址：  
  
l  
   
https://www.thymeleaf.org/documentation.html  
  
l  
   
https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html  
  
l  
   
https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html  
  
Thymeleaf 提供了多种标准表达式，可以先把它理解成"写在模板里、用来取数据和做运算的小语法"。包括：  
  
简单表达式：  
（最核心的  
 5 种，记住符号含义即可）  
  
l  
   
Variable expressions（变量表达式）  
${...}  
 —— 取变量的值，最常用  
  
l  
   
Selection expressions（选择表达式）  
*{...}  
 —— 在已选定的对象上取属性  
  
l  
   
Message (i18n) expressions（消息表达式）  
#{...}  
 —— 取国际化文案，i18n 是"国际化"的缩写  
  
l  
   
Link (URL) expressions（链接表达式）  
@{...}  
 —— 生成链接地址  
  
l  
   
Fragment expressions（分段表达式）  
~{...}  
 —— 引用模板片段  
  
字面量：  
（就是固定不变的值）  
  
l  
   
文本：'one text'、'Another one!' 等；  
  
l  
   
数值：0、34、3.0、12.3 等；  
  
l  
   
布尔：true、false  
  
l  
   
Null：null  
  
l  
   
Literal token（字面标记）：one、sometext、main 等；  
  
文本操作：  
  
l  
   
字符串拼接：  
+  
  
l  
   
文本替换：  
|The name is ${name}|  
  
算术操作：  
  
l  
   
二元运算符：  
+  
、  
-  
、  
*  
、  
/  
、  
%  
  
l  
   
减号（单目运算符）：  
-  
  
布尔操作：  
  
l  
   
二元运算符：  
and  
、  
or  
  
l  
   
布尔否定（一元运算符）：  
!  
、  
not  
  
比较和等价：  
  
l  
   
比较：  
>  
、  
<  
、  
>=  
、  
<=  
（gt、lt、ge、le）  
  
l  
   
等价：  
==  
、  
!=  
（eq、ne）  
  
条件运算符：  
  
l  
   
If-then：  
(if) ? (then)  
  
l  
   
If-then-else：  
(if) ? (then) : (else)  
  
l  
   
Default：  
(value) ?: (defaultvalue)  
  
特殊标记：  
  
l  
   
No-Operation（无操作）：  
_  
  
上面列出的这些表达式看起来很多，但不用死记——用到时回来查即可。我们进一步了解下最常用的简单表达式：大多数的 Thymeleaf 属性都允许把值设置或包含简单表达式。  
#### 2.1、条件表达式  
  
先看变量表达式。它是在上下文变量（Spring 里也叫"模型属性"，就是后端传给页面的数据）上执行的 OGNL 表达式；如果你把 Thymeleaf 和 Spring 集成使用，则是 Spring EL 表达式。这里 OGNL、SpEL 都是"表达式语言"的名字，作用类似——都是用来在模板里访问对象、调用方法的。它们的形式如下：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">${session.user.name}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
你可以在属性值或者属性值的一部分中找到它们，具体取决于属性的类型：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;span th:text=&#34;${book.author.name}&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
以上表达式在  
 OGNL 和 SpringEL 中等效于（右边这段 Java 代码帮你理解它到底做了什么）：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">((Book)context.getVariable(&#34;book&#34;)).getAuthor().getName()</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
变量表达式不只能用于输出，还能参与更复杂的处理，比如条件判断、循环迭代等：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;li th:each=&#34;book : ${books}&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
在这个例子中，  
${books}  
 从上下文中取出名为  
 books 的变量，把它当作可迭代的集合，供 th:each 循环使用。  
#### 2.2、选择表达式  
  
选择表达式与变量表达式非常相似，区别是：它是在"先前选好的对象"上取属性，而不是在整个上下文变量里查找。它的形式如下：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">*{customer.name}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
这个"选好的对象"由 th:object 属性指定：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;div th:object=&#34;${book}&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">...</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;span th:text=&#34;*{title}&#34;&gt;...&lt;/span&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">...</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/div&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
这里先用  
   
th:object="${book}"  
 选定  
 book 这个对象，之后   
*{title}  
 就直接取它的  
 title 属性，不必每次都写完整的 book 路径。上面的写法等同于：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">{</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">// th:object=&#34;${book}&#34;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">final Book selection = (Book) context.getVariable(&#34;book&#34;);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">// th:text=&#34;*{title}&#34;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">output(selection.getTitle());</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>#### 2.3、消息表达式  
  
消息表达式（通常称为"文本外部化"、国际化或 i18n）允许我们从外部源文件（.properties 文件）中，按一个键名 retrieving 对应语言的文案，还可以附带参数。这么做的好处是：页面文案集中管理，切换语言时不用改模板。  
  
在  
 Spring 应用程序中，它会自动与 Spring 的 MessageSource 机制集成。你可以在模板中这样找到它们：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">#{main.title}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">#{message.entrycreated(${entryId})}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
请注意，消息表达式里还可以嵌套变量表达式——比如你希望"消息键名"由上下文变量的值来决定，或者想把变量当作参数传进去。例如：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">#{${config.adminWelcomeKey}(${session.user.name})}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>#### 2.4、链接表达式  
  
链接表达式专门用来构建  
 URL，并为其添加有用的上下文和会话信息（这个过程通常叫"URL 重写"）。举个例子：应用部署在 Web 服务器的 /myapp 上下文下，模板里这样写：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a th:href=&#34;@{/order/list}&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
会被自动转换为（自动帮你补上应用名  
 /myapp 前缀）：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a href=&#34;/myapp/order/list&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
或者，如果需要保持会话、且未启用  
 cookie（或者服务器还不知道客户端支持 cookie），还可以转换为下面这样，把会话 id 拼进 URL：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a href=&#34;/myapp/order/list;jsessionid=23fa31abd41ea093&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
URL 还可以带参数：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a th:href=&#34;@{/order/details(id=${orderId},type=${orderType})}&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
结果如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a href=&#34;/myapp/order/details?id=23&amp;type=online&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
注意：在标签属性中应该对  
 & 符号进行 HTML 转义（所以这里写成了   
&  
）。  
  
链接表达式也可以是相对的，这种情况下不会添加应用程序上下文前缀到  
 URL 中：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a th:href=&#34;@{../documents/report}&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
也可以是服务器相对的（同样没有应用程序上下文前缀）：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a th:href=&#34;@{~/contents/main}&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
还可以是协议相对的（与绝对  
 URL 类似，但浏览器会使用当前显示页面所用的 HTTP 或 HTTPS 协议）：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a th:href=&#34;@{//static.mycompany.com/res/initial}&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
当然，链接表达式也可以是绝对的：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;a th:href=&#34;@{http://www.mycompany.com/main}&#34;&gt;...&lt;/a&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>#### 2.5、分段表达式  
  
分段表达式是  
 3.x 版本新增的内容。它提供了一种简单的方式来表示一段标记片段，并在模板周围移动、复用它。有了分段表达式，片段可以被复制，或者像参数一样传给其他模板。  
  
最常见的用法是使用  
 th:insert 或 th:replace 插入片段：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;div th:insert=&#34;~{commons :: main}&#34;&gt;...&lt;/div&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
不过它们可以出现在任何地方，就像使用其他任何变量一样：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;div th:with=&#34;frag=~{footer :: <a class="wx_topic_link" topic-id="mwkf091x-iyenmx" style="color: #576B95 !important;" data-topic="1" data-recommend="">#main</a>/text()}&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;p th:insert=&#34;${frag}&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/div&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
分段表达式还可以带参数。  
#### 2.6、表达式预处理  
  
官方文档：https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#preprocessing  
  
除了上面这些表达式处理功能，Thymeleaf 还有一个"预处理"能力。所谓预处理，就是在正常表达式执行之前先执行一遍的表达式，允许你先修改表达式本身，再去执行最终的结果。这个特性看着抽象，但正是后面漏洞利用的关键，需要留意。  
  
预处理的写法与普通表达式完全一样，只是要用双下划线把它包起来，比如  
   
__${expression}__  
。也就是说，表达式预处理被定义在两个下划线  
   
__  
 之间：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">#{selection.__${sel.code}__}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
我们看到的变量表达式  
   
${sel.code}  
 会先被执行。假如它的结果是  
 "ALL"，那么   
__  
 之间的值就会被替换成  
 "ALL"，并作为表达式的一部分再次执行，在这里最终会变成 selection.ALL。  
### 3、SpringBoot 整合 Thymeleaf  
  
老规矩，还是使用  
 IDEA 新建个名为 thymeleafdemo 的 SpringBoot 项目。  
  
①、第一步，使用 IDEA（我的版本是 2022.2.3）新建项目，配置如下图所示：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicMc2NrbKdbOxXjwyfIxYQZUslhCiaf66sZeCiabwgTfBRAqTRHEfpgNspy8BeTyicPgwJIU6ZgbKozvPGDIKPicxAfmHuQBfogHNIY/640?wx_fmt=png&from=appmsg "")  
  
  
   
  
第一步  
 新建项目配置  
  
②、第二步，选择 SpringBoot 版本，以及添加相关依赖，最后点击创建，如下图所示：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CY1qhLA6JicNLPTkAyhP3monHmwjBE64YYvlUdgMaBCNs2Y6866GqWpQkgwNM1hsYic3CTovCSvwEpia6ncW53vmeepiapp0PF6xHQwserJicWLs/640?wx_fmt=png&from=appmsg "")  
  
   
  
第二步  
 选择版本与依赖  
  
当然了，Spring 相关系列集成了 Thymeleaf，可以直接选择，我们也可以在 pom.xml 手动添加。  
  
③、第三步，在 pom.xml 文件中添加依赖，进入发现使用的是 Thymeleaf 的 3.0.15.RELEASE 版本，如下图所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;dependency&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;artifactId&gt;spring-boot-starter-thymeleaf&lt;/artifactId&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/dependency&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicP5V2ib9XaibMZC7FweAJwnSzibryia9XxnODpTUNsEt9KdkfeYbguFE9TygQJ5rEqib47MVU9DRiacEhHac8ibU5icLBGjfOSLSHjYZjQ/640?wx_fmt=png&from=appmsg "")  
  
   
  
第三步  
 pom.xml 依赖版本  
  
④、第四步，创建前端模板文件，在 src/main/resources/templates/ 下，新建一个名为 index.html 的前端模板文件，如下图所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;!DOCTYPE html&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;html lang=&#34;en&#34; xmlns:th=&#34;http://www.thymeleaf.org&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;head&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;meta charset=&#34;UTF-8&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><font face="Consolas"><span leaf="">&lt;title&gt;SpringBoot 整合 Thymeleaf&lt;/title&gt;</span></font></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/head&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;body &gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;span th:text=&#34;${data}&#34;&gt;&lt;/span&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/body&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/html&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicOAnoavOYgQO5aYuc6WicldYKyFiay6iaUEQCWSDO7SichsoN89yia6lZ1d4ardtXAVA3noBjmVTcWLuXQjj3To3fibGsADHdMriaquW0/640?wx_fmt=png&from=appmsg "")  
  
   
  
第四步  
 index.html 模板文件  
  
⑤、第五步，在 src/main/java/com/example/thymeleafdemo/ 目录下新建一个名为 ThymeleafController 的 Java Class，并键入以下代码，最终如下图所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">package com.example.thymeleafdemo;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.stereotype.Controller;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.ui.Model;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.web.bind.annotation.RequestMapping;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">/**</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><font face="Consolas"><span leaf="">* 编号7089</span></font></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">*/</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@Controller</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public class ThymeleafController {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@RequestMapping(value = &#34;/&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public String index(Model model) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">model.addAttribute(&#34;data&#34;,&#34;hello thymeleaf!!!&#34;);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">return &#34;index&#34;;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicPVmMZJbPeC8nxUCOy8ibYGUvic8b7Jric4kal55hTqsy8F5fJgwUrETG9iag4ZO6S6K4icpz6verKIkial12FRmtYKPDysKlHCcwaX8/640?wx_fmt=png&from=appmsg "")  
  
   
  
第五步  
 ThymeleafController 代码  
  
⑥、第六步，在配置文件中键入以下配置内容，如下图所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">server.port=8090</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><font face="Consolas"><span leaf=""><a class="wx_topic_link" topic-id="mwkf091y-d88ieq" style="color: #576B95 !important;" data-topic="1" data-recommend="">#thymeleaf</a> 页面的缓存开关，默认 true 开启缓存</span></font></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">spring.thymeleaf.cache=false</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">spring.thymeleaf.prefix=classpath:/templates/</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">spring.thymeleaf.suffix=.html</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicPCqB28f89fyuicdkKRdp80LrPSGlI7LKhazicDpdiaf4lUqyq4UxwsmR6TO0ctMgdoXE6LyvnUGK1QkyG83G2Hu0J2nokLv083xE/640?wx_fmt=png&from=appmsg "")  
  
   
  
第六步  
 配置文件  
  
⑦、第七步，启动项目，浏览器访问 http://127.0.0.1:8090/ ，最终如下图所示：  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CY1qhLA6JicM4JiaMIr8yoRCtpuypGW8yI0kKbmXnDxs0uZmAwicTZP168W7NWQic1bxjBdoOfvBGMzkPeHcXic3iae0ic52hU1G4C9kyD5DzzicH7E/640?wx_fmt=png&from=appmsg "")  
  
   
  
第七步  
 启动访问效果  
  
至此，SpringBoot 整合 Thymeleaf 基础部分到此结束。此部分意在让大家通过基础的整合操作，来简单理解什么是 Thymeleaf。  
## 三、Thymeleaf 模板注入漏洞  
  
本章结合经典开源项目，讲解  
 Thymeleaf 模板注入漏洞的成因、漏洞源码示例，以及对应的修复方案。  
### 1、漏洞释义  
  
Thymeleaf SSTI 漏洞通常发生在使用动态输入来生成 Thymeleaf 模板的情况下。攻击者可以通过精心构造的输入向服务器发送恶意代码，这些代码将被 Thymeleaf 视为有效的模板表达式，并在服务器上执行。因此会导致服务器上的远程代码执行，从而使攻击者能够完全接管服务器并访问敏感数据。  
  
在  
 Thymeleaf 中，可以使用表达式来动态设置模板的值。例如   
${user.name}  
 将被替换为用户的名称。攻击者可以使用类似  
   
${T(java.lang.Runtime).getRuntime().exec('calc')}  
 的表达式来执行任意的系统命令。  
  
Thymeleaf 3.0.0 至 3.0.11 版本存在模板注入漏洞。该漏洞在 Thymeleaf 3.0.12 及以后版本已经得到修复，但还是存在一些 Bypass 的方式。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CY1qhLA6JicNbtGH2WaEnsiawEicSoDd38c9Yic1R54kZCZz2Xk4D82jICicvnWZjb8Fet06DuY8H3donxFYzEVuXeSJPomTZzc5hzQCh1hkenSw/640?wx_fmt=png&from=appmsg "")  
  
  
   
  
Thymeleaf 3.0.12 发行说明  
### 2、漏洞源码示例  
  
基础案例，这里我们以  
 https://github.com/veracode-research/spring-view-manipulation/ 这个项目进行讲解，比较经典。下载后，使用 IDEA 打开，进入 src/main/java/com/veracode/research/HelloController.java 文件中。  
  
通过本项目我们来认识  
 WEB 环境下存在 SSTI 漏洞的情形。第二阶段是从 WEB 角度出发讲解 JavaWeb 下常见的漏洞，故此节没有的 Thymeleaf SSTI 漏洞分析，具体这部分稍后会放到漏洞部分讲解，先从此节了解下前置基础知识吧。  
  
另外，从  
 WEB 漏洞视角来看，虽然 SSTI 漏洞发生在 WEB 漏洞场景下，但更多的是引入了相关模板引擎，并使用了不安全的版本或代码造成的。这么来说，也算是框架漏洞。  
  
spring-view-manipulation 项目中使用的 SpringBoot 版本为 2.2.0，Thymeleaf 版本为 3.0.11。  
#### 2.1、选择模板  
  
HelloController.java 第 24、25 行，代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@GetMapping(&#34;/path&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public String path(@RequestParam String lang) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">return &#34;user/&#34; + lang + &#34;/welcome&#34;; //template path is tainted</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
代码很简单，学过前面阶段基础的朋友肯定能看懂。应用场景比如说：国际化语言切换。定义  
 CN 模板和 EN 模板，通过修改 lang 的参数来实现中英文页面展示。  
  
而此时通过拼接路径名实现选择模板造成了模板注入，lang 的参数我们可控，即可传入攻击语句。代码注释作者也给出了攻击语句，大家启动项目自行运行下。  
  
（注意，一定要记得进行  
 URL 编码，否则会报错。）  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">http://127.0.0.1:8090/lang=__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22calc%22).getInputStream()).next()%7d__::.x</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>#### 2.2、片段选择器  
  
HelloController.java 第 31 到 34 行，代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@GetMapping(&#34;/fragment&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public String fragment(@RequestParam String section) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">return &#34;welcome :: &#34; + section; //fragment is tainted</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
在上面提到过，这是  
 3.x 版本新增的内容。在分段表达式中有一个"片段选择器"语法，用于选择模板中的某个片段或元素，并在页面中渲染该片段或元素。片段选择器通常以 th: 开头，例如 th:fragment 或 th:include。th:fragment 用于定义一个片段，th:include 用于在页面中包含一个片段。  
  
这段代码是片段选择器存在模板注入问题，在  
 section 参数中传入攻击语句。  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">http://127.0.0.1:8090/section=__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22calc%22).getInputStream()).next()%7d__::.x</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>#### 2.3、拼接路径  
  
HelloController.java 第 36 到 40 行，代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@GetMapping(&#34;/doc/{document}&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public void getDocument(@PathVariable String document) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">log.info(&#34;Retrieving &#34; + document);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">//returns void, so view name is taken from URI</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
由于返回为空，所以视图名字会从  
 URI 中获取，并且接收了 document 的传参，我们可以键入以下攻击语句。  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">http://127.0.0.1:8090/doc/__%24%7Bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22calc%22).getInputStream()).next()%7D__%3A%3A.x</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>### 3、Thymeleaf 模板注入漏洞修复  
  
在  
 spring-view-manipulation 中，作者针对上述情景给出了修复方案。  
#### 3.1、设置注解 @ResponseBody  
  
HelloController.java 第 42 到 46 行，代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@GetMapping(&#34;/safe/fragment&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@ResponseBody</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public String safeFragment(@RequestParam String section) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">return &#34;welcome :: &#34; + section; //FP, as @ResponseBody annotation tells Spring to process the return values as body, instead of view name</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
使用  
 @ResponseBody 注解告诉 Spring 将返回值作为响应体处理，而不是视图名称，因此无法进行模板注入攻击。  
#### 3.2、设置重定向 redirect  
  
HelloController.java 第 48 到 51 行，代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@GetMapping(&#34;/safe/redirect&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public String redirect(@RequestParam String url) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">return &#34;redirect:&#34; + url; //FP as redirects are not resolved as expressions</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
当视图名称以  
 redirect: 前缀开头时，Spring 不再使用 Spring ThymeleafView 解析，而是使用 RedirectView 解析，该视图不会执行表达式。但这个示例存在 URL 跳转漏洞。  
#### 3.3、设置 response 响应  
  
HelloController.java 第 53 到 56 行，代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@GetMapping(&#34;/safe/doc/{document}&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public void getDocument(@PathVariable String document, HttpServletResponse response) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">log.info(&#34;Retrieving &#34; + document); //FP</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
由于控制器在参数中具有  
 HttpServletResponse，Spring 认为已经处理了 HTTP 响应，因此视图名称解析就不会发生。这个检查存在于 ServletResponseMethodArgumentResolver 类中。  
### 4、进阶课外文章推荐  
  
分享一些关于  
 ThymeLeaf 模板注入漏洞优秀的文章，多多学习。  
  
l  
   
https://www.cnpanda.net/sec/1063.html  
  
l  
   
https://moonsec.top/articles/121  
  
l  
   
https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/  
  
l  
   
https://www.cnblogs.com/colo/p/15507738.html  
  
l  
   
https://xz.aliyun.com/t/9826  
  
l  
   
https://xz.aliyun.com/t/10514  
  
