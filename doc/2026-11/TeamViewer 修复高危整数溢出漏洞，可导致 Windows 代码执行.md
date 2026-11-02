#  TeamViewer 修复高危整数溢出漏洞，可导致 Windows 代码执行  
Swati Khandelwal
                    Swati Khandelwal  代码卫士   2026-11-02 05:49  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**TeamViewer****已修复一个高危漏洞（****CVE-2026-111584****，****CVSS 8.8****分），它可导致远程会话中被控制机器攻击控制它的计算机。在****TeamViewer****会话中，连接并取得控制权的计算机是查看端，而被控制的机器是主机端。**  
  
恶意或被入侵的主机可利用该漏洞使查看端的  
TeamViewer  
崩溃，并且根据平台不同，还可能在查看端计算机上运行代码，不过这仅在用户开始与该主机的远程控制会话后才会生效。修复程序已在  
TeamViewer 15.83  
版本发布。  
  
该漏洞影响  
Windows  
、  
Linux  
和  
macOS  
上的  
TeamViewer  
完整客户端、  
Windows  
便携版以及  
15.83  
之前的所有  
Android  
远程控制应用版本。具体如下：  
  
（  
1  
）  
TeamViewer  
完整客户端（含便携版）  
  
- 15.83  
之前版本（  
Windows  
、  
Linux  
、  
macOS  
）  
  
- 旧版  
/  
维护版：  
V15.64  
（  
Windows  
）、  
V14.7  
（  
Windows  
、  
Linux  
、  
macOS  
）、  
V13.2  
（  
Windows  
、  
Linux  
、  
macOS  
）  
  
  
  
（  
2  
）  
TeamViewer  
远程控制（  
Android  
）  
15.83  
之前版本  
  
TeamViewer  
在公告中列出了完整客户端、便携版和  
Android  
应用，但未列出主机端。  
15.83  
版本更新还修复了该公告中另外两个确实影响主机端的漏洞，因此运行主机端的用户也应更新。安全公司  
Remedio  
的研究员  
Omri Dar  
于  
10  
月  
29  
日发布了该漏洞的技术说明。他写道，在这个漏洞中，  
“  
攻击者是主机端，受害者是查看端。  
”  
  
在远程会话中，主机端告诉查看端其屏幕图像有多大。查看端使用该尺寸来预留内存，但计算可能溢出并预留过少空间。随后到达的屏幕数据便会被写入该空间末尾之外。这至少会使查看端的  
TeamViewer  
会话崩溃，而且主机端可以随时再次触发崩溃。  
  
在  
Windows  
上，  
Remedio  
表示他们将漏洞利用推进得更远，绕过了操作系统内置的漏洞利用防御，并在查看端运行了他们选择的代码。该演示在主机端使用了替代程序来提供部分攻击数据，因此尚不是完全通过网络执行的攻击。该结果仅在  
Windows  
上展示。  
macOS  
、  
Linux  
或  
Android  
上是否同样可行，文中未涉及。  
TeamViewer  
仅表示根据具体平台的情况，有可能实施代码执行。  
  
Dar  
写道，该攻击不需要密码，除了接受会话外不需要额外点击，并且在  
TeamViewer  
的默认设置下即可生效。该漏洞影响  
15.83  
之前的所有版本，包括  
TeamViewer  
在  
9  
月发布的安全更新  
15.82  
。上个月更新并未清除该漏洞。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfV4Sucx1AHPfmyOKavwFLOcKBGRQqkaHwH9zr9klkicibKbCHNf8iaIVAYwqc2rM4yvkDwJnBPn2Rr4ECIxslp5Wib67wOK5ciaDo7M/640?wx_fmt=gif&from=appmsg "")  
  
**应采取措施**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUII09hbuP3QicUQ1WaoVp8icibsLSXnxFTj7AOQAXo1mqAlST00Y7Yt2lOsQnAoEBol4LMeXK4jlpiaibVuW2LSia00Ap6PNlib7MzLU/640?wx_fmt=gif&from=appmsg "")  
  
  
  
TeamViewer  
提供的唯一修复方法是更新，且未列出任何应变措施。当前用户应升级到  
15.83  
或更高版本。受支持的旧分支的修复版本可参见  
 TeamViewer   
发布的公告。  
  
15.83  
版本还修复了该公告中的另外两个高危漏洞，其中包括一个影响  
Linux  
和  
macOS  
的单独代码执行漏洞。同日发布的配套公告  
TV-2026-1012  
修复了  
TeamViewer  
通过自动更新分发的远程管理附加组件中的另外六个漏洞。  
  
TeamViewer  
表示在发布修复程序时，并未发现该漏洞的相关利用攻击。截止目前，该漏洞并未被列入  
 CISA KEV  
清单。  
Remedio  
报告称未发现在野攻击，且也未发布漏洞利用代码。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[TeamViewer 多个漏洞可导致RCE攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526988&idx=2&sn=d9963aa394ca4b8b36a63f40fabffe3a&scene=21#wechat_redirect)  
  
  
[TeamViewer DEX高危漏洞可导致 Nomad 服务遭劫持](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524747&idx=2&sn=51ac4b38d86a59e2a6d418263e3e6405&scene=21#wechat_redirect)  
  
  
[TeamViewer内网IT系统疑遭APT组织攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247519913&idx=2&sn=e88e554be7efec8feabdf988c77630c8&scene=21#wechat_redirect)  
  
  
[UEFI 严重漏洞潜伏16年，可绕过 Secure Boot 在网络启动服务器上执行代码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527570&idx=2&sn=e714ade0a06e22b539cd1e9792db58d3&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/10/teamviewer-flaw-lets-remote-computers.html  
  
  
题图：Pixa  
b  
ay Licens  
e  
  
  
**本文由奇安信编译，不代表奇安信观点。转载请注明“转自奇安信代码卫士 https://codesafe.qianxin.com”。**  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSf7nNLWrJL6dkJp7RB8Kl4zxU9ibnQjuvo4VoZ5ic9Q91K3WshWzqEybcroVEOQpgYfx1uYgwJhlFQ/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSN5sfviaCuvYQccJZlrr64sRlvcbdWjDic9mPQ8mBBFDCKP6VibiaNE1kDVuoIOiaIVRoTjSsSftGC8gw/640?wx_fmt=jpeg "")  
  
**奇安信代码卫士 (codesafe)**  
  
国内首个专注于软件开发安全的产品线。  
  
   ![](https://mmbiz.qpic.cn/mmbiz_gif/oBANLWYScMQ5iciaeKS21icDIWSVd0M9zEhicFK0rbCJOrgpc09iaH6nvqvsIdckDfxH2K4tu9CvPJgSf7XhGHJwVyQ/640?wx_fmt=gif "")  
![]( "")  
![]( "")  
  
   
觉得不错，就点个 “  
在看  
” 或 "  
赞  
”   
  
