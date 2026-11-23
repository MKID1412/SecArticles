#  【已复现】nginx 多个远程代码执行漏洞（nginx2root）  
 乌雲安全   2026-11-23 02:38  
  
nginx 是广泛部署的高性能 Web 服务器与反向代理，常用于 HTTP/HTTPS 服务、反向代理、负载均衡、HTTP/2 与 HTTP/3 接入以及邮件代理等场景，大量部署在互联网暴露面、云环境和容器集群入口。  
  
2026 年 11 月，长亭安全应急响应中心监测到 nginx 多个远程代码执行漏洞（合称 nginx2root）风险。攻击者在无需认证的情况下，可针对启用 HTTP/2、HTTP/3 或特定 resolver、mirror、邮件代理配置的 nginx 服务触发内存破坏，并以 nginx worker 进程权限执行任意命令。F5 已于 2026 年 11 月 18 日发布 nginx 1.31.9（mainline）与 1.30.7（stable）修复版本，建议受影响用户尽快排查暴露面并完成升级。  
  
**漏洞描述**  
 Description   
**01**  
  
### 漏洞成因  
  
五个 nginx 漏洞均为内存破坏类缺陷，可各自独立触发，不需要组合利用，差异在于受影响的配置路径。  
- **CVE-2026-123007（HTTP/2 流控窗口整数溢出）**  
：nginx 在排队响应 DATA 帧时从连接级发送窗口扣减额度，客户端可通过 WINDOW_UPDATE 再次增加该窗口；若流在 DATA 发出前被取消，清理逻辑会把已扣减额度重复加回连接窗口且不做上限校验，使窗口越过 31 位上限。后续窗口检查发生无符号下溢，异常窗口值转换为有符号长度后变为负值，最终在 TLS 输出路径形成堆越界读写，可泄露地址并劫持清理回调。  
  
- **CVE-2026-123634（resolver 释放后使用）**  
：主请求与其 mirror 子请求等待同一 DNS 解析结果时，若主请求因连接异常终止，会释放 mirror 仍在等待队列中的解析上下文；解析器随后继续使用该已释放上下文并调用其 handler。  
  
- **CVE-2026-123633（HTTP/2 连接释放后使用）**  
：仍有活动流的 HTTP/2 连接可被留在可回收队列中；请求建立上游连接时触发连接槽回收，当前请求所在的 HTTP/2 连接被关闭并释放，上游处理代码继续访问已释放的请求结构。  
  
- **CVE-2026-121810（HTTP/3 连接释放后使用）**  
：QUIC 连接在新流创建过程中被重新标记为可回收；上游连接建立再次触发回收并关闭父 QUIC 连接，导致请求释放后继续被使用。  
  
- **CVE-2026-113425（邮件代理认证释放后使用）**  
：mail proxy 使用 auth_http 认证时，客户端连接关闭会释放会话及其内存池，但认证连接仍持有该会话指针；认证连接的后续事件使用已释放会话，并可劫持认证上下文中的回调。  
  
Wiz Research、Ikotas Labs、Metnir、Rapid7 与 Xint 各自独立发现了 CVE-2026-123633 与 CVE-2026-123007，并向 NGINX 背后的 F5 报告。两者均为无需身份验证的严重远程代码执行漏洞。  
### 漏洞影响  
  
未认证攻击者可从网络侧触发上述 nginx 漏洞，以 nginx worker 进程权限执行任意命令。实际影响取决于 nginx 版本、HTTP/2 与 HTTP/3 及代理配置、worker 权限等条件。披露方引用 Shodan 统计称，在可识别版本的 nginx 主机中约 98.9% 落在 CVE-2026-123007 受影响范围；该统计基于版本字符串，不能直接等同于可利用资产数量。  
  
Wiz 研究团队将 CVE-2026-123007 与 CVE-2026-123633 评为 Critical（CVSS 4.0 9.2），并确认两者均可实现未认证远程代码执行。其中 CVE-2026-123007 可在仅提供静态文件的 nginx 上触发，CVE-2026-123633 更常见于反向代理配置；利用需先完成内存地址泄露，ASLR 不构成有效防护。  
### 处置优先级：高  
  
**漏洞类型：**  
 远程代码执行  
  
**漏洞危害等级：**  
 高  
  
**触发方式：**  
 网络远程  
  
**权限认证要求：**  
 无需权限  
  
**系统配置要求：**  
 需要特定配置  
  
**用户交互要求：**  
 无需用户交互  
  
**利用成熟度：**  
 POC 已公开  
  
**修复复杂度：**  
 中，官方已提供临时缓解方案  
  
  
  
**影响版本**  
 Affects   
**02**  
  
<table><thead><tr><th style="padding:8px 10px;border:1px solid #dfe3e6;background-color:#f3f8f1;color:#333333;text-align:left;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy1g92" style="font-size: 16px;">漏洞</span></section></th><th style="padding:8px 10px;border:1px solid #dfe3e6;background-color:#f3f8f1;color:#333333;text-align:left;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy206b" style="font-size: 16px;">受影响版本</span></section></th><th style="padding:8px 10px;border:1px solid #dfe3e6;background-color:#f3f8f1;color:#333333;text-align:left;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy1fov" style="font-size: 16px;">受影响配置</span></section></th></tr></thead><tbody><tr><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmyiok" style="font-size: 16px;">CVE-2026-123007</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy1wjy" style="font-size: 16px;">nginx 1.9.5–1.31.8</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy363" style="font-size: 16px;">启用 HTTP/2</span></section></td></tr><tr><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmyji3" style="font-size: 16px;">CVE-2026-123634</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy1akz" style="font-size: 16px;">nginx 0.6.18–1.31.8</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmyug" style="font-size: 16px;">使用 resolver 与请求派生的 proxy_pass 目标，且 mirror 配置 mirror_request_body off</span></section></td></tr><tr><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy21qb" style="font-size: 16px;">CVE-2026-123633</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy5nl" style="font-size: 16px;">nginx 1.19.7–1.31.8</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy16c9" style="font-size: 16px;">启用 HTTP/2 且请求转发至上游</span></section></td></tr><tr><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy1dt9" style="font-size: 16px;">CVE-2026-121810</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy8j1" style="font-size: 16px;">nginx 1.25.0–1.31.8</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmyvkw" style="font-size: 16px;">启用 HTTP/3 且请求转发至上游</span></section></td></tr><tr><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy20eb" style="font-size: 16px;">CVE-2026-113425</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy1avq" style="font-size: 16px;">nginx 0.5.33–1.31.8</span></section></td><td style="padding:8px 10px;border:1px solid #dfe3e6;color:#333333;white-space:normal;"><section><span leaf="" mpa-font-style="mwsnmvmy1yq7" style="font-size: 16px;">邮件代理启用 proxy on 与 auth_http</span></section></td></tr></tbody></table>  
  
nginx 官方已在 1.31.9（mainline）和 1.30.7（stable）中修复上述五个漏洞。发行版常以补丁回移方式修复，nginx -v  
 显示的版本号可能不变，应以完整包版本比对发行版安全公告。CVE-2026-123634 除 mirror 外，SSI 等其他创建子请求的功能也可能到达同一缺陷，关闭 mirror 不能单独作为安全判定依据。  
  
**解决方案**  
 Solution   
**03**  
  
### 升级修复方案  
  
F5 已于 2026 年 11 月 18 日发布 nginx 1.31.9（mainline）和 1.30.7（stable），修复上述五个漏洞。建议升级至 1.31.9、1.30.7 或更高版本，并确认构建包含全部五个漏洞的修复；NGINX Plus 用户请按 F5 对应公告升级。安装后需重启 nginx 或执行受支持的二进制热升级，仅 reload 配置不会替换正在运行的二进制。  
  
官方安全公告（含受影响版本与修复版本）： https://nginx.org/en/security_advisories.html  
### 临时缓解方案  
  
以下措施可用于暂无法升级的环境，可能影响业务功能，升级后应及时移除。  
- **关闭 HTTP/2（CVE-2026-123007、CVE-2026-123633）**  
：nginx 1.25.1 及以上在每个受影响的 server 块设置 http2 off;  
，并移除 listen 指令中的 http2 参数；nginx 1.9.5 至 1.25.0 移除 listen ... http2  
 中的 http2 参数。依赖 HTTP/2 的业务（如原生 gRPC）会受到影响。  
  
- **消除请求时 DNS 解析（CVE-2026-123634）**  
：将使用变量的 proxy_pass 改为固定 upstream 组；不需要 mirror 时关闭 mirror。仅设置 mirror_request_body on 不能修复 resolver 缺陷，也不能只关闭 mirror 就认定安全。  
  
- **关闭 HTTP/3（CVE-2026-121810）**  
：移除包含 quic 的 listen 指令，设置 http3 off;  
，并将 Alt-Svc 改为 clear  
；同时用防火墙阻断 UDP 443，确认没有残留 QUIC 监听。  
  
- **限制邮件代理访问（CVE-2026-113425）**  
：将 IMAP、POP3、SMTP 监听绑定到回环地址，或仅允许可信来源 IP 访问；远程邮件客户端将无法直连。  
  
- **关闭既有连接**  
：reload 只影响新连接，已建立的 HTTP/2、HTTP/3 或邮件会话仍由旧 worker 处理；需重启 nginx 使旧 worker 退出，可能造成短时业务中断。  
  
CVE-2026-123007 的利用以 HTTP/2 控制帧为主，默认访问日志可能没有对应请求记录；排查时应结合连接字节统计、worker 进程启动的外部程序、异常 worker 退出与 core dump 综合判断，不能仅依赖访问日志。  
  
**漏洞复现**  
 Reproduction   
**04**  
  
长亭安全应急响应中心已在本地完成复现验证。本次复现针对 CVE-2026-123007，使用的是 Rapid7 公布的 PoC；该 PoC 需要发送上百G HTTP/2 流量，难以在真实攻击中隐蔽使用。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/EqS9GE77r0Mql2XMVPSUoOFhJJ5vsWZ2gpafKqlqIANuVqJibusdgia51DV02ArModhib03u89S3ACX9Cxe1STfpd4qXvGxhGvyg8fTH80Iibpc/640?wx_fmt=png&from=appmsg#imgIndex=1 "")  
  
Xint 研究团队称已在多个公有云环境利用成功，公开演示视频中的攻击过程仅需数秒，但其博客同样说明利用需要较大流量。观察视频可见，攻击者与目标均为 AWS ap-northeast-2（首尔）区域的 EC2 实例；同区域实例之间可获得数十至数百 Gbps 的带宽，这也解释了短视频演示为何能够完成。演示视频还结合 Linux 内核 SCTP 子系统缺陷，完成了从容器内 nginx worker 权限逃逸至宿主机 root 的完整链路。  
  
**时间线**  
 Timeline   
**05**  
  
2026 年 11 月 18 日 长亭安全应急响应中心发布通告  
  
### 参考资料  
  
[1]. https://xint.io/blog/nginx2root-remote-code-execution  
  
[2]. https://www.wiz.io/blog/nginx-http2-rce-vulnerabilities  
  
[3]. https://nginx.org/en/security_advisories.html  
  
[4]. https://nginx2root.com  
  
[5]. https://www.rapid7.com/blog/post/ve-cve-2026-123007-critical-nginx-http-2-integer-underflow-leads-to-rce-fixed/  
  
  
