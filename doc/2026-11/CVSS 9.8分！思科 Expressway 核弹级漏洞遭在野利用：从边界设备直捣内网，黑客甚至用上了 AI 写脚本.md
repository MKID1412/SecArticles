#  CVSS 9.8分！思科 Expressway 核弹级漏洞遭在野利用：从边界设备直捣内网，黑客甚至用上了 AI 写脚本  
原创 Hankzheng
                    Hankzheng  技术修道场   2026-11-23 23:59  
  
![](https://mmbiz.qpic.cn/mmbiz_png/kBd83kxr9I4LibN8AA0XzwUCkHgJjoMqufPLT3ZUhj525kibTBZTZB79cvnXf1x2w48KmGqZfO9hjttIWI67T7Y65WF2loSBbKlfiaiabez079Q/640?wx_fmt=png&from=appmsg "")  
  
做安全和运维的兄弟们，今天跟大家聊一个刚爆出来的硬核实战案例。  
  
前阵子思科刚给自家 **Cisco Expressway**  
 补了一个 **CVSS 评分高达 9.8**  
 的核弹级漏洞——CVE-2026-76466  
。结果大家也懂，PoC（概念验证代码）前脚刚公开，后脚黑客就端着枪进场了。  
  
俄罗斯安全公司 F6 刚刚披露了全球首例该漏洞的**在野利用（In-the-wild）**  
实录：一家俄罗斯企业的 4 台思科设备接连沦陷，黑客不仅直接拿下了底层操作系统的 **Root 权限**  
，还顺着集群一路摸进内网，甚至疑似用上了 **AI 工具**  
来辅助写横向移动脚本。  
  
今天我就结合 F6 的报告和思科的官方通告，带大家把这波攻击的技术路径、底层原理和排查思路彻底盘一遍。  
## 一、漏洞原理拆解：一个 Cookie 加证书校验失效，直接送出 Root  
  
很多同学可能好奇，Cisco Expressway 作为企业级边界通信网关，按理说防护应该挺严密的，怎么会被无认证远程打穿？  
  
我仔细看了下思科的漏洞公告，这个 **CVE-2026-76466**  
 其实是两个底层缺陷打出的“组合拳”，堪称逻辑漏洞里的教科书级翻车：  
1. **持久化 Cookie机制缺陷**  
  
设备在底层架构中使用了持久化的 Cookie（在 Erlang 分布式架构中，Cookie 本质上就是节点间互信的“通行证”）。一旦攻击者掌握了这个固定或可预测的 Cookie，就相当于拿到了内部通信的敲门砖。  
  
1. **集群 TLS 客户端证书校验“形同虚设”**  
  
更离谱的是，当 Expressway 集群开启了 TLS Verify Mode = Enabled  
（TLS 验证模式）时，系统**竟然没有对客户端证书进行充分校验**  
。  
  
> **技术路径还原：**  
> 正常情况下，外部请求连不到核心数据库服务，或者会被 mTLS 双向认证拦在门外。但在上述两个 Bug 的叠加下，攻击者只要网络可达，就能直接绕过证书校验，拿着 Cookie 连上目标设备的**数据库服务**  
，随后调用特定的 **RPC（远程过程调用）指令**  
。  
> 由于底层服务权限过高，这一步直接实现了**免认证的操作系统级任意命令执行**  
，且到手就是最高权限——**Root**  
。  
  
  
对于安全从业者来说，这里其实有一个很值得深思的架构痛点：**很多边界设备为了实现高可用集群，节点间的底层通信（如 Erlang 分布式的 Cookie 认证、内部 RPC）往往默认信任内网或依赖单一校验机制。一旦 TLS 校验层出现逻辑疏漏，内部高危接口就会直接暴露在公网炮火之下。**  
## 二、实战攻击链复盘：从边界失守到内网“丝滑”横向  
  
F6 捕获到的这次真实入侵，把“边界设备沦陷 = 内网门户大开”这句话演得淋漓尽致。整个攻击链路非常清晰：  
1. **突破口：公网暴露的未修复节点**  
  
目标企业有 **2 台暴露在公网的 Cisco Expressway X15.5.3 设备**  
没有及时打补丁。黑客利用公开的 PoC，直接打穿了这两台设备，拿到 Root 权限。  
  
1. **借力打力：顺着集群信任摸进内网**  
  
拿下边界只是第一步。由于 Expressway 采用集群部署，黑客利用其中一台已经控住的边界设备作为跳板，顺理成章地渗透进了位于**企业内网的另外 2 台集群节点**  
。总共 4 台设备，一锅端。  
  
1. **内网大扫荡：精准收割核心资产**  
  
站稳脚跟后，黑客并没有乱动，而是上传了定制的 Python 脚本，直接开启全自动内网探测与凭证窃取：  
  
1. **网络测绘与共享探测**  
扫描内网存活主机，寻找可访问的 **SMB**  
 共享资源；  
  
1. **核心服务交互**  
直接去碰内网的 **SQL Server**  
 数据库和 **WinRM**  
（Windows 远程管理服务）；  
  
1. **代码库收割**  
盯上了企业内部的 **GitLab**  
 仓库，目标非常明确——窃取明文凭证、密钥和其他高价值敏感数据。  
  
## 三、魔幻现实：黑客写横向脚本，居然也在用 AI？  
  
在分析黑客遗留在受害设备上的定制 Python 脚本时，安全研究员发现了一个特别有意思的细节：  
  
这些脚本里充满了工整的代码注释，甚至还有详细的“修复说明”。  
  
懂行的兄弟看到这儿估计已经会心一笑了。实战中，黑客手搓的临时工具通常都是能跑就行，极少有人会耐着性子写规范注释和 Bug 修复记录。这种画风，大概率是**用 AI 编程助手（大模型）辅助生成和调试出来的**  
。  
  
这也给咱们提了个醒：AI 正在大幅降低后渗透阶段的开发门槛。以前需要资深红队成员手写的内网定制化扫描、利用脚本，现在普通攻击者配合 AI，几分钟就能迭代出一个针对 SMB、WinRM、GitLab 的多合一工具。防守方的响应窗口，正在被进一步压缩。  
## 四、硬核自查与应急响应指南（建议收藏）  
  
目前这次攻击还没有归因到具体的已知 APT 组织，这意味着**任何暴露在公网的未修复 Cisco Expressway 设备都是潜在目标**  
。如果你司用到了这款产品，建议立刻按以下清单排查：  
### 1. 升级修复版本（最直接有效）  
  
思科早在 2026 年 10 月 21 日就发布了修复补丁，赶紧核对你的版本号：  
- **Cisco Expressway X14 及更早版本**  
直接迁移至受支持的修复版本；  
  
- **Cisco Expressway X15.5.3 及更早版本**  
立即升级至 **X15.5.20**  
 或更高版本。  
  
### 2. 失陷指标（IoCs）排查  
  
如果设备一直裸奔在公网且还没来得及更新，别光顾着打补丁，**先查日志和磁盘**  
，确认是不是已经被“光顾”过了：  
<table><thead><tr style="background-color: rgb(26, 54, 93);color: rgb(255, 255, 255);"><th style="padding: 10px 12px;border: 1px solid rgb(26, 54, 93);"><section style="text-align: center;"><span leaf=""><span textstyle="" style="color: rgb(0, 0, 0)">排查维度</span></span></section></th><th style="padding: 10px 12px;border: 1px solid rgb(26, 54, 93);"><section style="text-align: center;"><span leaf=""><span textstyle="" style="color: rgb(0, 0, 0)">具体特征（IoC）</span></span></section></th><th style="padding: 10px 12px;border: 1px solid rgb(26, 54, 93);"><section style="text-align: center;"><span leaf=""><span textstyle="" style="color: rgb(0, 0, 0)">说明</span></span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);"><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);font-weight: 600;color: rgb(45, 55, 72);"><section><span leaf="">恶意出口/来源 IP</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);font-family: Consolas, Monaco, monospace;color: rgb(197, 48, 48);word-break: break-all;"><section><span leaf="">66.179.31[.]32</span><span leaf=""><br/></span><span leaf="">18.130.47[.]116</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);color: rgb(74, 85, 104);"><section><span leaf="">在防火墙、流量审计设备中检索与这两个 IP 的通信记录</span></section></td></tr><tr style="background-color: rgb(248, 250, 252);"><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);font-weight: 600;color: rgb(45, 55, 72);"><section><span leaf="">系统日志异常</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);font-family: Consolas, Monaco, monospace;color: rgb(197, 48, 48);word-break: break-all;"><section><span leaf="">erlang.log 中出现</span><span leaf=""><br/></span><span leaf="">wombat@127.0.0.1</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);color: rgb(74, 85, 104);"><section><span leaf="">重点检查 Expressway 的 Erlang 系统日志，这是极强的入侵痕迹</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);"><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);font-weight: 600;color: rgb(45, 55, 72);"><section><span leaf="">恶意脚本落地路径</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);font-family: Consolas, Monaco, monospace;color: rgb(197, 48, 48);word-break: break-all;"><section><span leaf="">/mnt/harddisk/i{0-9}{4}[a-zA-Z]?/*.py</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(226, 232, 240);color: rgb(74, 85, 104);"><section><span leaf="">检查硬盘对应正则目录下是否存在可疑的 Python 脚本</span></section></td></tr></tbody></table>### 3. 架构层面的长效防御  
  
打完补丁、清完木马，还得从架构上补课。这次事件再次证明，**单靠“及时打补丁”是防不住 0-day/N-day 闪电战的**  
：  
- **严格做好网络隔离**  
DMZ 区的边界设备与内网核心区（尤其是数据库、GitLab、域控等）之间，必须实施最小权限访问控制，绝不能让边界节点直接拥有访问内网 SMB/WinRM 的通路；  
  
- **强化基础设施监控**  
对边界设备的异常外连、非业务时段的内网扫描行为（尤其是针对 445、1433、5985 等端口）配置高优告警。  
  
写在最后  
  
从 PoC 公开到全网在野利用，留给蓝队的黄金响应时间往往只有几十个小时。面对越来越“卷”、甚至开始用 AI 写工具的攻击者，咱们运维和安全团队能做的，就是把资产台账摸清、把网络隔离做死、把补丁响应拉满。  
  
你们公司有用到思科的边界设备吗？平时遇到这种 9.8 分的紧急漏洞，内部应急响应最快能做到多久？欢迎在评论区聊聊你的实战经验！  
  
[#网络安全]()  
    [#漏洞分析]()  
    [#思科]()  
    [#运维安全]()  
    [#应急响应]()  
  
  
