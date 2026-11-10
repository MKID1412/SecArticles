#  漏洞预警 | OpenSSH/musl密码认证绕过漏洞  
浅安
                    浅安  浅安安全   2026-11-09 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-123158  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
OpenSSH是OpenBSD项目维护的广泛使用的开源SSH远程登录与加密传输服务组件，被广泛应用于Linux、Unix及嵌入式系统。musl libc是面向嵌入式与容器场景的轻量级C标准库，被Alpine Linux、OpenWrt、Buildroot、Yocto及大量Docker镜像采用。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SX1BVL2Le6Wj5Xfwibo44DWzQHHurrhvfKlYnxuJZgypTQDuWRibYYIMTP8JTcWhf4VIPkYbjnFvicUg/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-123158**  
  
**漏洞类型：**  
密码认证绕过  
  
**影响：**  
执行  
远程  
命令  
  
**简述：**  
OpenSSH/musl存在密码认证绕过漏洞，由于musl libc的crypt()函数在失败时返回单字符"*"，而OpenSSH在未启用PAM的认证路径中，对密码字段为"*"的账户借用其他账户的真实盐值执行crypt()校验。当攻击者提交257～1024字节的超长密码时，musl的crypt()因KEY_MAX(256)长度限制返回"*"，与影子密码字段"*"比较相等，strcmp判定通过，导致攻击者无需任何有效凭据即可通过SSH密码认证登录仅允许密钥登录或已锁定密码的账户。成功绕过认证后，若目标账户具备免密sudo/doas权限，攻击者可直接提权至root实现完全控制。  
  
**0x04 影响版本**  
- OpenSSH < 10.6p2  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.openssh.com/  
  
  
  
