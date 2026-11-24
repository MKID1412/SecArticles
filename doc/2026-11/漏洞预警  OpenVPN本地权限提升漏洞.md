#  漏洞预警 | OpenVPN本地权限提升漏洞  
浅安
                    浅安  浅安安全   2026-11-23 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-121720  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
OpenVPN是一款开源的虚拟私人网络软件，利用SSL/TLS协议实现加密通信，支持点对点和站点到站点的安全连接，广泛应用于远程访问和企业网络。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SUq88uQTsWXticMSxiakqdwBia2Z1ao2f5jfrXmMiagU0icibjWe65HXyunQiaVx4bMv5tt6LCicAxqpA1tBQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-121720**  
  
**漏洞类型：**  
本地权限提升  
  
**影响：**  
获取权限****  
  
**简述：**  
OpenVPN存在本地权限提升漏洞，由于openvpnserv的RunOpenvpn()例程在启动openvpn.exe时，未能充分过滤用户定义的环境变量。本地已认证用户可以通过设置环境变量来影响服务的执行环境，进而使服务以高权限执行非预期的操作。  
  
**0x04 影响版本**  
- 2.4.0 <= OpenVPN Windows <= 2.6.23  
  
- 2.7_alpha1 <= OpenVPN Windows <= 2.7.8  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://github.com/OpenVPN/openvpn  
  
  
  
