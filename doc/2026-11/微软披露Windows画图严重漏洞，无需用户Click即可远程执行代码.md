#  微软披露Windows画图严重漏洞，无需用户Click即可远程执行代码  
 FreeBuf   2026-11-06 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX3MaR0xTMWibYYOibiciacFdicXVLiaZ52BG4azHCZF1P2SnQeuk4DjibR07IM5iaPJGe8TCic2PFtzlN2r0052F8EejCZjeyCGW0YMzZ6c/640?wx_fmt=gif "")  
  
  
![Windows画图漏洞](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0BXx11YHeG0iaPIrkVv5EMPmLaZIbYs78IGw7DfZG1GS6Rg3F2bzVYBPbNEOUyfVE1MCsYaAuEMIicdOEPkbutJjJpPZYbIfic4U/640?wx_fmt=png "")  
  
  
微软披露Windows画图存在一处严重漏洞，攻击者无需登录，也不需要用户Click任何内容，即可远程执行代码。  
  
  
该漏洞公告发布于2026年10月13日，属于越界读取漏洞，对应缺陷编号CWE-125。这类漏洞的成因是程序读取了预设内存边界之外的数据。微软表示，该漏洞可触发远程代码执行，可能严重影响数据隐私、数据完整性与系统可用性。  
  
  
本次漏洞随微软10月安全更新批次一同披露。Cybersecurity News针对2026年10月"补丁星期二（Patch Tuesday）"更新的专题报道，梳理了本次更新的整体背景与新增的风险分组规则。这类规则可帮助管理员统一评估关联漏洞，无需将每个漏洞作为孤立问题单独处理。  
  
  
Part  
01  
  
漏洞利用无需认证交互  
  
  
根据微软披露的信息，攻击者可向受影响服务发送特制网络包，成功利用后即可在目标设备上执行代码。目前公告未明确指出受影响的具体服务、对应网络端口或完整攻击链路，因此"仅恶意图片即可触发漏洞"的说法尚未得到证实。  
  
  
虽然漏洞利用不需要身份认证，也不需要用户交互，但实际利用复杂度较高。攻击者需要掌握目标系统的详细信息，利用成功率还会受部署环境、配置情况、现有安全控制措施的影响。  
  
  
微软将该漏洞的利用可能性评级为"较低"，同时表示目前该漏洞尚未被公开披露，也无已知的在野利用记录。需要注意的是，上述结论仅代表初始评估结果，并非永久有效的安全保证。  
  
  
Part  
02  
  
受影响版本记录存在差异  
  
  
当前微软安全公告的安全更新列表中，标注的受影响画图版本为build 11.2609.401.0。但NIST的漏洞记录显示，受影响版本为11.2605.91.0以下的版本。  
  
  
这一信息偏差十分关键。管理员应当遵循微软最新的更新指引开展修复，不能仅依据早前的版本阈值判断系统是否受影响。  
  
  
Part  
03  
  
主动核查补丁安装状态  
  
  
用户应当通过微软商店检查画图应用的更新，安装完成后需验证当前运行的版本号。有报告显示，部分设备即使执行了更新检查，仍会收到旧版本安装包。管理员可通过PowerShell执行命令Get-AppxPackage Microsoft.Paint | Select-Object Name, Version查看应用包版本，再与微软公告中的版本信息比对确认。  
  
  
Cybersecurity News此前针对PNG内存漏洞的单独报道，已经说明图像类软件为什么需要重点关注补丁安装，但那些漏洞与本次画图漏洞并无关联。针对本次漏洞，修复工作的核心是确认每台受影响设备都安装了修复版本的画图应用，不能直接假设自动更新已经成功完成部署。  
  
  
参考来源：  
  
Critical Windows Paint Vulnerability Enables Remote Code Execution Attacks  
  
https://cybersecuritynews.com/windows-paint-rce-vulnerability/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
