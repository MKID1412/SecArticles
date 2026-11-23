#  漏洞预警 | JeecgBoot SSRF漏洞  
浅安
                    浅安  浅安安全   2026-11-22 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-126997  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
JeecgBoot是一款基于代码生成器的低代码开发平台。前后端分离架构 SpringBoot2.x，SpringCloud，Ant Design&Vue，Mybatis-plus，Shiro，JWT，支持微服务。强大的代码生成器让前后端代码一键生成，实现低代码开发。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SW25cvic5s59KsEGyS6fTiaaogPXrsntzHQO87dEEoxdTnicwvibHLUm8wvIaRo5oQWGzJqicZPr2Q7G9w/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-126997**  
  
**漏洞类型：**  
SSRF  
  
**影响：**  
获取敏感信息  
  
**简述：**  
J  
eecgBoot存在SSRF漏洞，由于flow-run端点调用RestTemplate.exchange时未对目标地址做有效限制，未经验证的攻击者可构造恶意请求，让服务器代替其访问内网服务或云元数据接口，从而探测内网、读取敏感信息。  
  
**0x04 影响版本**  
- 3.9.0 <= JeecgBoot <= 3.9.6  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
http://www.jeecg.com/  
  
  
  
