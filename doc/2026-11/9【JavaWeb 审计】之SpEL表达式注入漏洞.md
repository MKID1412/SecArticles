#  9【JavaWeb 审计】之SpEL表达式注入漏洞  
 初梦的安全小屋   2026-11-23 15:00  
  
> 阅读指引：如果你是第一次接触 SpEL 和表达式注入，建议从头按顺序读这一章。本章先用生活化的例子把"表达式"这个核心概念讲清楚，再一步步介绍 SpEL 的常见写法和几个关键接口。文中的代码块、链接和标题编号都原样保留，遇到看不懂的英文缩写或类名不必慌，第一次出现时都会给出解释。把本章读懂了，下一章理解 SpEL 注入漏洞就会水到渠成。  
## 一、SpEL  
  
本章介绍  
 SpEL 的基本概念、基础语法、用法与应用场景。你可以把这一章当成一份"入门说明书"：先认识 SpEL 是什么、能做什么，再学习它最常用的几种写法，最后看看它在实际项目里都用在哪。  
### 1、SpEL 简介  
  
官方文档：https://docs.spring.io/spring-framework/docs/3.0.x/reference/expressions.html  
  
SpEL，全称为 Spring 表达式语言（Spring Expression Language），是一个由 Spring 框架提供的表达式语言。它是一种基于字符串的表达式语言，可以在运行时对对象进行查询和操作。  
  
别被"表达式语言"这个名词吓到。所谓"表达式"，其实就是一段能算出结果的字符串，比如   
2 + 3  
、  
'Hello ' + 'World'  
 都算表达式——前者算出 5，后者算出 "Hello World"。SpEL 就是把这种"写字符串、算结果"的能力做成了 Spring 里的一套通用工具，让你用简短的一段文字，就能读写对象的属性、调用方法、做各种运算。  
  
举个生活化的例子：如果把程序里的对象比作一台机器，SpEL 就像这台机器的"遥控器"。你不需要拆开机器去写很多行 Java 代码，只要按几个按钮（写一小段表达式字符串），就能让机器告诉你某个属性的值，或者帮你完成一次计算。  
  
SpEL 支持在 XML 和注解配置中使用，它可以在 Spring 框架的各种组件中使用，如 Spring MVC 控制器、Spring Security 安全框架、Spring Data 数据访问框架等。它可以方便地访问对象的属性、调用对象的方法、进行数学运算、逻辑运算、正则表达式匹配等操作。  
  
第一次见到这些名词，先记住大意就行：Spring MVC 是 Spring 用来开发 Web 接口的模块；Spring Security 负责登录和权限控制；Spring Data 负责访问数据库；而 bean 指的是由 Spring 容器帮我们创建和管理的对象。  
  
使用  
 SpEL 可以大大简化编程工作，使得配置和编写代码更加灵活和易于维护。例如，可以使用 SpEL 在 XML 或注解中配置 Spring bean 的属性，或在 Spring MVC 控制器中动态地计算请求参数。  
  
总而言之，SpEL 是 Spring 框架中一个非常有用的特性，它提供了一种灵活、简洁的方式来操作和查询对象。  
  
SpEL 表达式语言支持以下功能：  
  
l  
   
Literal expressions（字面量表达式）  
  
l  
   
Boolean and relational operators（布尔和关系运算符）  
  
l  
   
Regular expressions（正则表达式）  
  
l  
   
Class expressions（类表达式）  
  
l  
   
Accessing properties, arrays, lists, maps（访问属性、数组、列表、映射）  
  
l  
   
Method invocation（调用方法）  
  
l  
   
Relational operators（关系运算符）  
  
l  
   
Assignment（赋值运算符）  
  
l  
   
Calling constructors（调用构造函数）  
  
l  
   
Bean references（Bean 引用）  
  
l  
   
Array construction（数组构造）  
  
l  
   
Inline lists（内联列表）  
  
l  
   
Ternary operator（三目运算符）  
  
l  
   
Variables（变量）  
  
l  
   
User defined functions（用户定义的函数）  
  
l  
   
Collection projection（集合投影）  
  
l  
   
Collection selection（集合选择）  
  
l  
   
Templated expressions（模板表达式）  
  
上面这张能力清单不用逐条背，先扫一眼知道  
 SpEL"能干这些事"就够了。后面用到哪一种，再回来对照。  
### 2、SpEL 基础  
  
更多基础知识大家可以学习这份文档：http://itmyhome.com/spring/expressions.html  
  
下面几小节讲  
 SpEL 具体怎么用。建议边看边对照代码示例，重点理解"每一类写法解决什么问题"，语法细节记不住可以回头查。  
#### 2.1、表达式符号  
  
在  
 SpEL 中，使用   
#{<表达式>}  
 界定符被认为是  
 SpEL 表达式，可以使用相关变量、属性和方法等操作。  
  
这里的  
   
#{...}  
 相当于一个"标记框"：只要把内容放进这对符号里，Spring 就知道"这里面是一段 SpEL 表达式，需要算一算"，而不是把它当成普通文本。  
#### 2.2、类型表达式  
  
T(type) 用于获取一个类型的 Class 对象。它的作用是让 SpEL 表达式在运行时获取指定的类型的 Class 对象，以便在表达式中可以使用该类的方法和属性。  
  
这里的  
 T(...) 可以理解成一个"取类型"的开关。所谓 Class 对象，就是 Java 里用来描述"某个类本身"的对象——通过它你能拿到这个类的名字、静态方法、静态属性等。写 T(java.lang.String)，就等于告诉程序"把 String 这个类型拿过来"。  
  
使用  
 T(type) 的语法格式为：T(package.ClassName)，其中 package 是指类所在的包名，ClassName 是指类名。例如，要获取 java.lang.String 类的 Class 对象，可以使用表达式 T(java.lang.String)。  
  
在  
 SpEL 中，同样可以使用 T(type) 来调用静态方法和属性，例如：T(java.lang.Math).PI 表示获取 Math 类中的 PI 静态属性，T(java.lang.Math).random() 表示调用 Math 类中的 random() 静态方法。"静态方法 / 属性"是指不用创建对象、直接通过类名就能调用的方法和属性。  
  
需要注意的是，在使用  
 T(type) 操作符时，要确保指定的类型已经被加载，否则会抛出 ClassNotFoundException 异常。  
  
下面是几个例子：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">Class dateClass = parser.parseExpression(&#34;T(java.util.Date)&#34;).getValue(Class.class);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">Class stringClass = parser.parseExpression(&#34;T(String)&#34;).getValue(Class.class);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">boolean trueValue = parser.parseExpression(</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&#34;T(java.math.RoundingMode).CEILING &lt; T(java.math.RoundingMode).FLOOR&#34;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">).getValue(Boolean.class);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
   
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;property name=&#34;random&#34; value=&#34;#{T(java.lang.Math).random()}&#34;/&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;property name=&#34;message&#34; value=&#34;#{T(java.lang.Runtime).getRuntime().exec(&#39;calc&#39;)}&#34; /&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
顺便提一句：仔细看上面第二个代码块里的第二行，  
T(java.lang.Runtime).getRuntime().exec('calc')  
 这串东西会调用系统命令打开计算器——这正是后面要讲的 SpEL 注入漏洞的核心 payload，这里先留个印象。  
#### 2.3、SpEL 用法  
  
SpEL 的用法有三种形式：一是在注解中，二是在 XML 配置中，三是在代码中使用 Expression。下面是对应的基础例子。  
  
在注解中  
  
在  
 Spring 中，可以使用 @Value 注解将 SpEL 表达式应用于 Bean 属性、构造函数参数和集合元素等场景。@Value 注解可以将 SpEL 表达式注入到 Bean 的属性中，支持在表达式中引用其他 Bean、属性和环境变量等。  
  
注解可以简单理解为"写在代码里的配置标签"，@Value 这个标签的作用就是"把某个值赋给这个字段"。  
  
例如，在以下示例中，@Value 注解将 SpEL 表达式注入到 name 属性中：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@Component</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public class ExampleBean {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@Value(&#34;#{systemProperties[&#39;user.name&#39;]}&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">private String name;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">// Getter and setter methods</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
在 XML 配置中  
  
在  
 Spring 中，可以在 XML 配置中使用 SpEL 表达式来配置 Bean。SpEL 表达式可以在 XML 配置中用   
#{expression}  
 的形式引用。  
  
例如，在以下示例中，SpEL 表达式将被用于设置属性值：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;bean id=&#34;exampleBean&#34; class=&#34;com.example.ExampleBean&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;property name=&#34;name&#34; value=&#34;#{systemProperties[&#39;user.name&#39;]}&#34;/&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/bean&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
在代码块中使用 Expression  
  
在  
 Spring 中，可以使用 SpEL 的 Expression 接口在代码块中使用 SpEL 表达式。使用 Expression 接口可以在运行时编译和评估表达式，也可以设置变量和函数等。  
  
这里的  
 Expression 是 SpEL 里表示"一条已经解析好的表达式"的接口，而 ExpressionParser（表达式解析器）负责把字符串解析成 Expression。简单说：解析器先把字符串"翻译"成可执行的表达式，再调用 getValue() 得到结果。  
  
例如，在以下示例中，SpEL 表达式被用于计算两个数字的和：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">ExpressionParser parser = new SpelExpressionParser();</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">Expression exp = parser.parseExpression(&#34;2 + 3&#34;);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">Integer sum = (Integer) exp.getValue();</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">System.out.println(sum); // Output: 5</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>#### 2.4、SpEL 的 EvaluationContext 接口  
  
参考自：  
  
l  
   
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/expression/spel/support/StandardEvaluationContext.html  
  
l  
   
http://itmyhome.com/spring/expressions.html[#expressions]()  
-evaluation-context  
  
在使用  
 SpEL 时，我们需要将表达式与 EvaluationContext 进行绑定，然后将表达式交给 SpEL 引擎执行。EvaluationContext 为 SpEL 引擎提供了上下文信息，使得 SpEL 引擎能够正确地解析表达式中的变量、函数等信息，从而求出表达式的结果。  
  
EvaluationContext 翻译过来是"表达式上下文"，可以把它理解成表达式运行时的"环境说明书"——它告诉 SpEL 引擎：遇到变量去哪里找值、遇到函数去哪里找定义、遇到类型转换按什么规则来。  
  
SimpleEvaluationContext 和 StandardEvaluationContext 都是 SpEL 提供的 EvaluationContext 的实现类。它们都提供了表达式求值所需的上下文信息，但是在具体实现方面略有不同。  
  
这里对这两个实现类多说一句：SimpleEvaluationContext 像一把"只开了基本功能的锁"，只允许最简单的取值和类型转换；而 StandardEvaluationContext 像一把"全权限的万能钥匙"，功能齐全，但也正因为权限太大，下一章你会看到它正是漏洞的关键。先把这个区别记住。  
  
SimpleEvaluationContext 相对来说比较简单，它仅仅包含了变量解析器和类型转换器，不支持函数表达式。  
  
而  
 StandardEvaluationContext 提供了更丰富的上下文信息，包括变量解析器、类型转换器和函数表达式等。同时，StandardEvaluationContext 还支持自定义函数和类加载器等高级功能。  
  
通常情况下，如果只是简单的表达式求值，可以使用  
 SimpleEvaluationContext；如果需要使用函数表达式或自定义函数等高级功能，可以使用 StandardEvaluationContext。  
#### 2.5、SpEL 应用场景  
  
下面举例说明一些  
 SpEL 的应用场景。  
  
Spring 框架中的 Bean 定义，可以使用 SpEL 表达式来定义 Bean 的属性和构造函数参数。示例代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;bean id=&#34;exampleBean&#34; class=&#34;com.example.ExampleBean&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;property name=&#34;message&#34; value=&#34;#{&#39;Hello &#39; + worldBean.name}&#34; /&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/bean&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
Spring Security 框架中的权限控制，可以使用 SpEL 表达式来定义用户角色和资源权限。示例代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@PreAuthorize(&#34;hasRole(&#39;ROLE_ADMIN&#39;) and <a class="wx_topic_link" topic-id="mwkgs06o-7lz7a6" style="color: #576B95 !important;" data-topic="1" data-recommend="">#account</a>.enabled&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public void deleteUser(Account account) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">// delete user</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
Spring Data 框架中的查询条件，可以使用 SpEL 表达式来定义查询条件。示例代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@Query(&#34;select e from Employee e where e.salary &gt; :#{<a class="wx_topic_link" topic-id="mwkgs06o-izft5f" style="color: #576B95 !important;" data-topic="1" data-recommend="">#salary</a> + 1000}&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">List&lt;Employee&gt; findEmployeesWithSalaryGreaterThan(@Param(&#34;salary&#34;) int salary);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
Spring Batch 框架中的任务配置，可以使用 SpEL 表达式来定义任务的参数和条件。示例代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;batch:job id=&#34;exampleJob&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;batch:step id=&#34;step1&#34;&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;batch:tasklet&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;batch:chunk reader=&#34;itemReader&#34; writer=&#34;itemWriter&#34; commit-interval=&#34;#{jobParameters[&#39;commit.interval&#39;] ?: 100}&#34; /&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/batch:tasklet&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/batch:step&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">&lt;/batch:job&gt;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
Java 代码中的通用表达式计算，可以使用 SpEL 表达式来计算任意表达式。示例代码如下所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">ExpressionParser parser = new SpelExpressionParser();</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">Expression exp = parser.parseExpression(&#34;&#39;Hello &#39; + &#39;World&#39;&#34;);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">String message = (String) exp.getValue(); // message = &#34;Hello World&#34;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
以上列举的是一些常见场景，可以看出  
 SpEL 的身影遍布 Spring 的各个模块——这也意味着，只要项目用了 Spring，就有可能用到 SpEL，理解它的写法很有必要。  
## 二、SpEL 注入漏洞  
  
本章介绍  
 SpEL 注入漏洞的原理、环境部署与漏洞代码案例。  
### 1、SpEL 注入漏洞原理  
  
简单来说，当应用程序使用  
 SpEL 时，如果未正确处理用户输入数据，攻击者可以在表达式中注入任意的代码，并在应用程序的上下文中执行它，进而造成命令执行等漏洞。  
  
在上面提到了  
 SpEL 的 EvaluationContext，其中 StandardEvaluationContext 使用方法更加完整。  
  
因此，触发  
 SpEL 的漏洞的流程大致为：接收了用户的输入且未过滤等操作，将接收的参数使用 StandardEvaluationContext 去处理，并对表达式调用了 getValue() 或 setValue() 方法。  
  
如上流程，后端接收了用户输入且未过滤，而攻击者精心构造攻击  
 payload 即可实现命令执行等危险操作。  
### 2、环境部署  
  
老规矩，新建一个  
 SpringBoot 项目。  
  
①、创建一个 Spring Initializr，配置信息如下图所示：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicOajQLwwYEJWvgk7Fn0Er2ER9n8tC1IcXTJFUHsUSxHS8VuALKBIIFp3XKLjV8HEnyV5gFbNCsDKfvwJgmDENUj7IiaxfZYKOico/640?wx_fmt=png&from=appmsg "")  
  
   
  
创建  
 Spring Initializr 项目  
  
②、点击下一步后，选择 Spring Boot 版本和 Spring Web 依赖，如下图所示：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicOK2HTyHLCDJrawXP35EvsDeOkqQ1QDDzdCEWGvY6NndDgxcVD7kbUxawxxxAYiaic6hPwN29XCAjlIKRjxkVUgS1uPSgWIVpAvI/640?wx_fmt=png&from=appmsg "")  
  
   
  
选择版本与依赖  
  
③、最后点击创建即可，稍等片刻。  
### 3、漏洞代码案例  
  
在  
 src/main/java/com/example/speldemo 下新建一个名为 SpELdemo 的 Java Class，并键入以下代码，最终如下图所示：  
<table><tbody><tr><td data-colwidth="1254" width="1254" valign="center" style="padding: 7.5pt 9pt;border-width: 1pt;border-style: solid;border-color: rgb(216, 220, 227);background: rgb(244, 245, 247);"><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">package com.example.speldemo;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.expression.EvaluationContext;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.expression.Expression;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.expression.ExpressionParser;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.expression.spel.standard.SpelExpressionParser;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.expression.spel.support.StandardEvaluationContext;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.web.bind.annotation.GetMapping;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import org.springframework.web.bind.annotation.RestController;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">import java.util.Objects;</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@RestController</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public class SpELdemo {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">/**</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">* 127.0.0.1:8080/spel/vul/?ex=T(java.lang.Runtime).getRuntime().exec(&#34;calc&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">*/</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">@GetMapping(&#34;/spel/vuln&#34;)</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">public String vul1(String ex) {</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">ExpressionParser parser = new SpelExpressionParser();</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><font face="Consolas"><span leaf="">//StandardEvaluationContext权限过大，可以执行任意代码，默认使用可以不指定</span></font></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">EvaluationContext evaluationContext = new StandardEvaluationContext();</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">Expression exp = parser.parseExpression(ex);</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">return exp.getValue(evaluationContext).toString();</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p><p style="margin-top:2.6000pt;margin-bottom:2.6000pt;text-align:left;"><span style="font-family:Consolas;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">}</span></span><span style="font-family:&#39;Times New Roman&#39;;mso-fareast-font-family:黑体;color:rgb(36,41,47);font-size:10.5000pt;mso-font-kerning:1.0000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
代码参考自  
 https://github.com/j3ers3/Hello-JavaSec/blob/master/src/main/java/com/best/hello/controller/SpEL.java  
  
当然，这是理想状态下的漏洞代码案例。  
  
启动项目，浏览器访问  
 127.0.0.1:8080/spel/vul/?ex=T(java.lang.Runtime).getRuntime().exec("calc") ，即可触发在本地弹出计算器的操作。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CY1qhLA6JicMmNWFuubmy5dibTU9Ld7cLVYMbj1weBnZbK5Zcpd42ibICKn0YSj4kB2UOxLfcG6VDcYOaGsV4WrRn2OONAB7NxOrZbzHThn94I/640?wx_fmt=png&from=appmsg "")  
  
  
   
  
漏洞代码与弹窗效果  
  
上述代码简单释义如下。  
  
首先是  
   
ExpressionParser parser = new SpelExpressionParser();  
 ，创建  
 SpEL 解析器。  
  
然后是  
   
EvaluationContext evaluationContext = new StandardEvaluationContext();  
 ，创建  
 StandardEvaluationContext 对象。  
  
其次是  
   
Expression exp = parser.parseExpression(ex);  
 ，解析用户输入的传入值。  
  
最后是  
   
return exp.getValue(evaluationContext).toString();  
 ，使用  
 exp.getValue 获取值。  
  
上述代码表面操作下来，在传入恶意代码  
   
T(java.lang.Runtime).getRuntime().exec("calc")  
 后，弹出了计算器操作。  
  
经过调试发现，创建解析器，创建运行环境，解析传入值，最后获取值，上面每一步对于触发  
 SpEL 注入漏洞都很重要，少一步可能都无法触发。  
  
而关键点在于  
 parseExpression 解析步骤，我们传入的   
T(java.lang.Runtime).getRuntime().exec("calc")  
 ，被解析成了  
 SpEL 表达式，在获取值的时候触发了命令执行操作。  
  
可以在  
   
return exp.getValue(evaluationContext).toString();  
 处，打个断点进行  
 Debug。最终我打的断点跟踪在这，如下图所示：  
  
  
   
  
断点调试跟踪  
  
基础内容到这里就结束了  
  
