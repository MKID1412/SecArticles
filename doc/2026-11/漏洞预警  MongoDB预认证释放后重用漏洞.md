#  漏洞预警 | MongoDB预认证释放后重用漏洞  
浅安
                    浅安  浅安安全   2026-11-22 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-118383  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
MongoDB是一个开源的NoSQL数据库管理系统，采用文档导向存储方式，以BSON格式存储数据。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SW6US95gq5OXwOKiaFNgP0Ohomqb6Rp9hibcu0n0PdX6rDN6JTq7sN1wWDxlbKQMWcGBzhcd9cPPONA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-118383**  
  
**漏洞类型：**  
释放后重用  
  
**影响：**  
进程崩溃  
  
**简述：**  
MongoDB存在预认证释放后重用漏洞，由于ReplicationCoordinatorImpl::CatchupState对象在catch-up完成后的清理逻辑中存在双重abort缺陷。注册在catch-up waiter上的完成回调捕获了裸this指针，且忽略了完成状态，在abort_inlock()内部再次调用abort_inlock(kSucceeded)。内层调用重置了_repl->_catchupState并销毁了对象，外层调用随后继续通过已释放的指针进行读写，形成该漏洞。未认证客户端可以通过发送两个包含readConcern.afterOpTime的ping请求，在std::multimap<OpTime, Waiter>中制造排序循环，使真正的catch-up waiter永远无法被满足，进而导致进程崩溃，甚至在服务器进程上下文执行任意代码。  
  
**0x04 影响版本**  
- 9.0.0 <= MongoDB Server < 9.0.7  
  
- 8.3.0 <= MongoDB Server < 8.3.16  
  
- 8.0.0 <= MongoDB Server < 8.0.37  
  
- 7.0.0 <= MongoDB Server < 7.0.48  
  
- 6.0.0 <= MongoDB Server < 6.0.30  
  
- 5.0.0 <= MongoDB Server < 5.0.35  
  
- 4.4.0 <= MongoDB Server < 4.4.32  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.mongodb.com/  
  
  
  
