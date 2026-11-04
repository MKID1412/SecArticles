#  漏洞预警 | Linux kernel释放后重用漏洞  
浅安
                    浅安  浅安安全   2026-11-03 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-98432  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Linux kernel是美国Linux基金会开源的操作系统Linux所使用的内核。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30MhxQXbwG1WSjsS1TMvUY3aIsCJ1ic2kxclUiaYic5gV2p9dmfrcTERVZZ5bEOnWFIvWxTRHhnpSrVwYdscfiaPjyuYZZadMxgIIH8Gg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-98432**  
  
**漏洞类型：释放后重用**  
  
**影响：**  
获取root权限  
  
**简述：**  
Linux Kernel IPv4 TCP存在释放后重用漏洞，由于IPv4 TCP握手处理中的竞态条件：tcp_v4_syn_recv_sock()将请求套接字的ireq->ireq_opt所有权直接转移给子套接字的inet_opt而未复制，且未清除原指针。请求套接字无需持有监听锁即可处理，允许另一CPU并发处理重传SYN并通过tcp_check_req()构建SYNACK。子套接字在inet_sock_destruct()中以普通kfree()释放inet_opt，导致ip_options_build()中发生释放后重用。攻击者可利用该漏洞通过网络触发内核内存损坏、信息泄露、拒绝服务，或在特定条件下实现远程任意代码执行并获取root权限。  
  
**0x04 影响版本**  
- Linux Kernel 5.10.* < 5.10.272  
  
- Linux Kernel 5.15.* < 5.15.223  
  
- Linux Kernel 6.1.* < 6.1.190  
  
- Linux Kernel 6.6.* < 6.6.159  
  
- Linux Kernel 6.12.* < 6.12.113  
  
- Linux Kernel 6.18.* < 6.18.56  
  
- Linux Kernel 7.2.* < 7.2.10  
  
- Linux Kernel 7.3-rc < 7.3-rc7  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.kernel.org/  
  
  
  
