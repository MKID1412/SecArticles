#  漏洞预警 | Joomla访问控制不当漏洞  
浅安
                    浅安  浅安安全   2026-11-03 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-35223  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Joomla!是一款强大的在线内容管理系统。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30Mhw2afL7sAtYZbmslnCatLxPg8YpTQBnJFPVsv0fj1dHEwkwSqVY4xn7T4OklMF5BpHFYF32w9t08YlibmppKBkLX5Y6vS7ibphm8/640?wx_fmt=png&from=appmsg "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-35223**  
  
**漏洞类型：**  
访问控制不当  
  
**影响：**  
获取敏感信息  
  
**简述：**  
J  
oomla!存在访问控制不当漏洞，由于其核心组件com_config的Web服务端点中存在不当的访问检查，攻击者可利用此漏洞绕过正常身份验证，非法获取对配置设置的访问权限，进而导致修改或窃取敏感配置数据。  
  
**0x04 影响版本**  
- 4.0.0 <= Joomla! CMS <= 5.4.5  
  
- 6.0.0 <= Joomla! CMS <= 6.1.0  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.joomla.org/  
  
  
  
