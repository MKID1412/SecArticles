#  漏洞预警 | Synology DSM身份认证绕过漏洞  
浅安
                    浅安  浅安安全   2026-11-09 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-13639  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Synology DSM是群晖科技为其NAS存储设备开发的图形化操作系统，提供文件共享、备份容灾、虚拟化、监控、协作办公等功能，内置Web管理界面，支持PAM多因子认证、LDAP/AD域集成、File Station文件管理及丰富的套件扩展，广泛应用于企业数据存储与备份场景。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30MhyBjUkNN100TeeEGDqKgMN79UnWwxu8FgDGXsybFXHul8wZpX6gJYEpsDmCWhpLNWEjeqqEhay6MCxakuKRuI7RhFPoZuat1ic0/640?wx_fmt=png&from=appmsg "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-13639**  
  
**漏洞类型：**  
身份认证绕过  
  
**影响：**  
获取权限  
  
**简述：**  
Synology DSM存在身份认证绕过漏洞，由于其在生成会话JWT签名密钥时存在缓冲区错位缺陷：随机数生成函数randombytes_buf的输出被写入全局缓存，而实际写入密钥文件的却是刚被清零的32字节栈缓冲区，导致签名密钥固定为32个0x00字节，等同于硬编码常量密钥。未经验证的攻击者可使用全零密钥伪造aud="2FA"的1FA临时令牌JWT，并将其作为passwd参数提交登录接口，从而完全绕过密码与二次认证校验，免密登录任意用户，进一步通过File Station等接口实现任意文件读写、植入恶意载荷并接管设备。  
  
**0x04 影响版本**  
- DSM 7.4 < 7.4-90075  
  
- DSM 7.3.2 < 7.3.2-86009-4  
  
- DSM 7.2.2 < 7.2.2-72806-9  
  
- DSM 7.2.1 < 7.2.1-69057-12  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.synology.cn/  
  
  
  
