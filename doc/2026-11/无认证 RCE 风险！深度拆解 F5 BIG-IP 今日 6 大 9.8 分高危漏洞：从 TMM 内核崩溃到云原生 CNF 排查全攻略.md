#  无认证 RCE 风险！深度拆解 F5 BIG-IP 今日 6 大 9.8 分高危漏洞：从 TMM 内核崩溃到云原生 CNF 排查全攻略  
原创 Kit Chung
                    Kit Chung  安全圈动向   2026-11-23 00:00  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bHeteOibsicdhEsj6kGePRJ3oDl8lE3EFZqXXaZsxKCGnhLmibHicHDyHdFXo2ORkmmWZr6IciaOueQfOATiaA7n0TKGJiaicHq3icOibzU9IBic2u3uSs/640?wx_fmt=png&from=appmsg "")  
  
刚刷完 F5 最新一波安全通告，F5 一次性甩出了 **22 份安全公告**  
（其中 10 份涉及 NGINX），而真正让网安和运维圈炸锅的，是 **BIG-IP 里的 6 个 CVSS v3.1 高达 9.8 分（v4.0 为 9.3 分）的重磅级漏洞**  
。  
  
先说最要命的结论：**攻击者不需要任何登录凭证，就能直接把流量管理微内核打崩，导致业务流量全面中断（直到 TMM 重启），而且 F5 官方明确表示，这些漏洞极有可能被利用实现远程代码执行。**  
  
更让人头大的是，这 6 个洞目前**没有任何临时缓解方案**  
，官方也没排进常规发布版本，只给了工程热补丁。趁着目前还没爆出大规模在野利用，我赶紧把里面的技术原理、触发路径和落地踩坑点扒了一遍，分享给各位做排查参考。  
## 底层原理拆解：为什么这次的「数据平面栈溢出」这么棘手？  
  
以往很多 F5 高危洞出在管理平面，咱们靠内网隔离、ACL 限制管理口访问还能先顶一阵子。但这次完全不一样。  
> 核心痛点：  
> 这 6 个漏洞全部位于 BIG-IP 的数据平面，本质都是**栈缓冲区溢出**  
。  
  
  
熟悉 BIG-IP 架构的朋友都知道，数据平面跑着核心的 **TMM（Traffic Management Microkernel）**  
，负责处理设备上几乎所有的业务网络流量。管理平面这次虽然幸免于难，但数据平面本身就是暴露在业务流量面前的——只要虚拟服务器配置撞上了触发条件，恶意构造的流量就能直接把栈打穿。  
  
从底层利用路径来看，这类栈溢出的危害分两层：  
- **保底效果是 DoS：**  
畸形输入覆盖了栈上的关键数据或返回地址，直接触发 TMM 进程崩溃重启，期间经过该设备的业务流量瞬间断流。  
  
- **上限效果是无认证 RCE：**  
如果攻击者精准控制了溢出覆盖的返回地址和 ROP 链，就能劫持控制流，在无需认证的前提下直接拿下执行权限。  
  
这里顺便提一嘴，别把今天这批洞和 9 月 22 日披露的 **CVE-2026-94127**  
 混为一谈。那个洞已经被在野利用，当天就进了美国 CISA 的已知被利用漏洞清单，但它属于 APM 作为 OAuth 授权服务器时的**堆缓冲区溢出**  
；而今天这 6 个（其中 4 个由 WatchTowr 研究员 Sina Kheirkhah 报告，2 个由 F5 内部挖出）清一色是**栈溢出**  
，触发前置条件也完全不同。  
## 触发路径与复现条件：你的虚拟服务器中招了吗？  
  
好消息是，这 6 个 9.8 分的洞并不是“开箱即炸”，每一个都要求虚拟服务器上启用了**特定的 iRules 命令或 APM 策略组合**  
。其中 5 个栽在管理员最常用的流量调度神器 **iRules**  
 上，1 个栽在 **APM**  
 模块里。  
  
不仅传统 BIG-IP 受影响，云原生版本 **BIG-IP Next for Kubernetes和****BIG-IP CNF**  
 也未能幸免。我把 6 个漏洞的精确触发条件整理成了下面这张排查表，大家可以直接对着线上配置一条条核对：  
<table><thead><tr style="background-color: rgb(237, 242, 247);color: rgb(26, 54, 93);text-align: left;"><th style="padding: 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">CVE 编号</span></span></section></th><th style="padding: 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">所属模块</span></span></section></th><th style="padding: 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;"><section><span leaf=""><span textstyle="" style="font-size: 13px">虚拟服务器精确触发条件（复现前置路径）</span></span></section></th><th style="padding: 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;"><section><span leaf=""><span textstyle="" style="font-size: 13px">受影响产品范围</span></span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);"><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;color: rgb(197, 48, 48);white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">CVE-2026-123637</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">APM</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">配置了包含 </span></span><strong><span leaf=""><span textstyle="" style="font-size: 13px">Client Information</span></span></strong><span leaf=""><span textstyle="" style="font-size: 13px"> 和 </span></span><strong><span leaf=""><span textstyle="" style="font-size: 13px">Endpoint Security</span></span></strong><span leaf=""><span textstyle="" style="font-size: 13px"> 两种代理的 Per-Request Policy（单请求策略）</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">仅 BIG-IP APM</span></span></section></td></tr><tr style="background-color: rgb(248, 250, 252);"><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;color: rgb(197, 48, 48);white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">CVE-2026-121805</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">iRules (PEM)</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">针对客户端流量调用了 </span></span><code style="background-color: rgb(237, 242, 247);color: rgb(197, 48, 48);padding: 2px 5px;border-radius: 3px;font-family: Consolas, Monaco, monospace;font-size: 12.5px;"><span leaf=""><span textstyle="" style="font-size: 13px">PEM::subscriber</span></span></code><span leaf=""><span textstyle="" style="font-size: 13px"> 命令的 iRule</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">BIG-IP PEM、BNK、BIG-IP CNF</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);"><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;color: rgb(197, 48, 48);white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">CVE-2026-120292</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">iRules</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">绑定了 HTTP Profile，且 iRule 中使用了 </span></span><code style="background-color: rgb(237, 242, 247);color: rgb(197, 48, 48);padding: 2px 5px;border-radius: 3px;font-family: Consolas, Monaco, monospace;font-size: 12.5px;"><span leaf=""><span textstyle="" style="font-size: 13px">MR::message route</span></span></code></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">BIG-IP（全模块）、BNK</span></span></section></td></tr><tr style="background-color: rgb(248, 250, 252);"><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;color: rgb(197, 48, 48);white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">CVE-2026-120285</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">iRules</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">绑定了 HTTP Profile，且 iRule 中使用了 </span></span><code style="background-color: rgb(237, 242, 247);color: rgb(197, 48, 48);padding: 2px 5px;border-radius: 3px;font-family: Consolas, Monaco, monospace;font-size: 12.5px;"><span leaf=""><span textstyle="" style="font-size: 13px">PSC::ip_address</span></span></code></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">BIG-IP（全模块）、BNK、BIG-IP CNF</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);"><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;color: rgb(197, 48, 48);white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">CVE-2026-116953</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">iRules</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">针对客户端流量调用了 </span></span><code style="background-color: rgb(237, 242, 247);color: rgb(197, 48, 48);padding: 2px 5px;border-radius: 3px;font-family: Consolas, Monaco, monospace;font-size: 12.5px;"><span leaf=""><span textstyle="" style="font-size: 13px">GENERICMESSAGE::route</span></span></code><span leaf=""><span textstyle="" style="font-size: 13px"> 命令的 iRule</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">BIG-IP（全模块）、BNK</span></span></section></td></tr><tr style="background-color: rgb(248, 250, 252);"><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);font-weight: 700;color: rgb(197, 48, 48);white-space: nowrap;"><section><span leaf=""><span textstyle="" style="font-size: 13px">CVE-2026-120722</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">iRules (DNS)</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">针对客户端流量调用了 </span></span><code style="background-color: rgb(237, 242, 247);color: rgb(197, 48, 48);padding: 2px 5px;border-radius: 3px;font-family: Consolas, Monaco, monospace;font-size: 12.5px;"><span leaf=""><span textstyle="" style="font-size: 13px">DNS::scrape</span></span></code><span leaf=""><span textstyle="" style="font-size: 13px"> 命令的 iRule</span></span></section></td><td style="padding: 9px 10px;border: 1px solid rgb(203, 213, 224);"><section><span leaf=""><span textstyle="" style="font-size: 13px">BIG-IP DNS、BNK（</span></span><strong style="color: rgb(197, 48, 48);"><span leaf=""><span textstyle="" style="font-size: 13px">注：CNF 存疑</span></span></strong><span leaf=""><span textstyle="" style="font-size: 13px">）</span></span></section></td></tr></tbody></table>  
**划个重点避坑：**  
  
盯紧最后那个 **CVE-2026-120722**  
！F5 的官方安全公告里写着 BIG-IP CNF 不受影响，但当天同步更新的 CVE 官方记录里却赫然把 CNF 列为了受影响产品。这种“左右互搏”在厂商应急响应初期挺常见的，如果你们生产环境跑了 CNF 且用到了 DNS::scrape  
，千万别心存侥幸，按受影响处理最稳妥。  
  
**怎么快速自查？**  
  
如果是传统 BIG-IP，直接登录后台导出一份 bigip.conf  
，全局 grep  
 一把那 5 个 iRules 关键字（PEM::subscriber  
、MR::message route  
、PSC::ip_address  
、GENERICMESSAGE::route  
、DNS::scrape  
），再扫一遍 APM 的 Per-Request Policy 里有没有同时挂那两个 Agent，十分钟就能摸清家底。  
## 修复方案与落地「深坑」：只给 EHF 热补丁，运维太难了  
  
查出中招之后，真正的考验才刚开始。  
  
如开头所说，针对这 6 个 9.8 分高危漏洞，**官方没有给出任何临时规避手段，也没有列出常规修复版本，唯一的解法就是打工程热补丁。**  
  
用过 F5 EHF 的老哥都懂，这玩意儿在生产环境上是把双刃剑：  
1. **缺少完整 QA 测试：**  
F5 在公告里把丑话说在了前头——EHF 是夹在常规版本之间紧急构建的，**没有经过标准发布版本的全面质量评估**  
，官方热补丁政策甚至明确写着“不提供担保，不保证一定能正常工作”。上生产之前，务必先在测试环境把业务流量和 iRules 逻辑跑通，别漏洞还没被黑客打，自己先打补丁把业务干挂了。  
  
1. **定制补丁不兼容：**  
如果你当前的系统本来就跑着之前申请的定制 EHF，不能直接叠加，必须重新找 F5 Support 团队定制一份合入新修复的 EHF。  
  
1. **云原生版本修复对应表：**  
1. **BIG-IP Next for Kubernetes：**  
2.4.0–2.4.2 升至 BNK-2.4.2-EHF1  
；2.3.0–2.3.4 升至 BNK-2.3.4-EHF1  
；2.2.0–2.2.5 升至 BNK-2.2.5-EHF1  
。  
  
1. **BIG-IP CNF**  
（修复 CVE-2026-121805 和 CVE-2026-120285）：同版本号区间分别升至 CNF-2.4.2-EHF1  
、CNF-2.3.4-EHF1  
 和 CNF-2.2.5-EHF1  
。  
  
1. **老版本别以为「没上榜就安全」：**  
F5 只检测仍在技术支持周期内的版本。如果你的老设备已经过保、不在受影响列表里，绝不代表它免疫，反而意味着你连补丁都没地方拿。  
  
## 别光盯着 9.8 分：另外 6 个中高危漏洞也有「暗雷」  
  
同一批公告里，BIG-IP 和 BIG-IQ 还附带了 6 个评分稍低（CVSS v3.1 为 7.5–8.8）的漏洞。虽然风头被那 6 个 9.8 分抢走了，但有两个权限和 Token 相关的洞，实战危害同样不小：  
### 1. CVE-2026-125274（iControl SOAP，8.8 分）—— 权限绕过与 HA 避坑  
  
只要拥有网络访问权限，哪怕只是最低的 **Guest 角色**  
，就能发送未授权的管理请求，进而执行系统命令、增删文件甚至禁用服务。  
- **修复版本：**  
BIG-IP 21.1.0.3  
、17.5.1.10  
、17.1.3.6  
；BIG-IQ 8.4.2.3  
。  
  
- **缓解注意：**  
升级前需把 iControl SOAP 访问限制在受信任网段。**但千万注意官方警告：如果在 Self-IP 地址上直接封禁 iControl SOAP，可能会直接把双机高可用同步搞崩！**  
动手改防火墙策略前一定要评估 HA 链路。  
  
### 2. CVE-2026-120718（iControl REST，8.1 分）—— 永不过期的 Token  
  
iControl REST 的认证 Token 可能不会按预期过期。攻击者一旦拿到一个有效 Token，就能无限给它“续命”，直到原持有者手动删除。  
- **修复版本：**  
BIG-IP 21.1.0.2  
、17.5.1.10  
、17.1.3.6  
。  
  
- **临时解法：**  
赶紧检查你们自己的自动化运维脚本，用完 Token 后立刻调用接口主动销毁，别等它自然过期。  
  
### 3. 其余 4 个 7.5 分无认证 TMM 崩溃漏洞（均涉及 APM / SAML / SSLO）  
- **CVE-2026-125272（APM）：**  
Per-Request Policy 中使用了基于表单的 Logon Page，可直接打崩 TMM。随上述 6 个高危漏洞的**同一批 EHF**  
 修复。  
  
- **CVE-2026-126822（APM SAML）：**  
访问策略配置为 SAML SP（服务提供商）时可打崩 TMM。同样走 **EHF**  
 修复；不需要 SAML SP 的直接删掉 SAML Auth Agent，并限制虚拟服务器来源 IP。  
  
- **CVE-2026-126336（APM SAML）：**  
SAML SP 代理开启了**加密断言**  
时可打崩 TMM。已在 BIG-IP 21.1.0.3  
、17.5.1.10  
、17.1.3.6  
 修复，升级前官方建议靠 HA 架构扛着。  
  
- **CVE-2026-121812（APM & SSL Orchestrator）：**  
Per-Request Policy 开启了 tmm.access.policytrace  
 数据库键（默认关闭）时可触发 TMM 崩溃。APM 走 EHF 修复，SSL Orchestrator 目前仅在 21.1  
 和 17.1  
 分支提供了修复。  
  
## 排查优先级  
  
面对这么大一堆 CVE，千万别慌着一上来就全网重启升级，按这个节奏来最稳：  
1. **立刻全网检索配置：**  
优先查 6 个 9.8 分对应的 iRules 命令和 APM 策略，以及 Guest  
 账号和 iControl SOAP 暴露面。  
  
1. **精准评估 EHF 风险：**  
确认命中 9.8 分条件的设备，立刻向 F5 开 Case 拿对应版本的 EHF，**务必先在测试环境或备机演练**  
。  
  
1. **顺手修补自动化脚本与常规版本：**  
对未命中 9.8 分但受 8.8 / 8.1 / 7.5 分影响的设备，规划升级到 21.1.0.3  
 / 17.5.1.10  
 / 17.1.3.6  
 等常规稳定版本，同时给调用 REST API 的脚本加上 Token 清理逻辑。  
  
你们线上的 F5 这次有没有踩中这几个 iRules 命令？或者以前打 EHF 踩过什么坑？欢迎在评论区聊聊，给后面的兄弟排排雷。  
#F5 BIG-IP 漏洞  #CVSS 9.8  #TMM 崩溃  #无认证 RCE  #栈缓冲区溢出  #iRules 漏洞  #BIG-IP APM  #工程热补丁 EHF  #CVE-2026-123637  #iControl SOAP  
