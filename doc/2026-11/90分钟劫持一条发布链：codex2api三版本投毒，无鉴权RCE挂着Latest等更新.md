#  90分钟劫持一条发布链：codex2api三版本投毒，无鉴权RCE挂着Latest等更新  
原创 赤霄攻防实验室
                    赤霄攻防实验室  赤霄攻防实验室   2026-11-24 01:20  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8adIETb91Nicm58ziafcUIliaZC88l6IwP9Sr7ceGriarRGSibLEibwHiaiaHA3yq1C7EDQpN9M57ozBruRSPHrRUbs7ESzltkECNcqPqb8/640?wx_fmt=png "")  
  
如果你在用 codex2api——把 OpenAI Codex 包装成 API 网关的 2400 星开源项目——昨天 12 点到 15 点之间点过「在线更新」，你的服务器现在可能正在替别人执行命令。11 月 23 日 14:38，安全研究员 azure9 发出紧急报告 #916：v3.1.10、v3.1.11、v3.1.12 三个版本的发布链被劫持，全部内置后门，其中 v3.1.11 **删掉了全部鉴权**  
，知道路径就能远程执行任意命令。当天晚上官方紧急发布 v3.2.0 确认投毒，并要求运行过这三个版本的用户「按实例已失陷处理」。  
  
整个攻击只用 90 分钟，全程自动化，main 分支一行没碰。这篇拆解它怎么做到的。  
## 被劫持的不是代码，是「发布」这个动作  
  
codex2api 的 main 分支从头到尾是干净的——这是事件最反直觉的地方。攻击者拿到仓库所有者的 GitHub 凭据后，没往 main 推一行代码，而是在 main 之外构造孤立提交，直接打上 v3.1.10 的 tag。GitHub Actions 看到 tag 自动开跑 Release 工作流，从恶意提交构建全平台二进制，发布、标记 Latest 一气呵成。维护者回头查 main，什么都没有。  
  
这正是「发布链劫持」和传统代码投毒的区别：你 review 的是 main，用户下载的是 release，而 release 的原料是一个不在任何分支上的幽灵提交。所有恶意提交的作者都是同一个合成身份 build@codex2api.local  
，无签名、无真实账号——事后只能凭 Actions 运行记录确认：每一次 tag 推送，用的都是仓库所有者本人的凭据。  
## 三段演进：从「带钥匙的后门」到「不锁门」  
  
三个版本一步比一步大胆。v3.1.10 先试探：注入带硬编码 token 门控的「诊断路由」——看似有锁，钥匙就刻在公开源码里。19 分钟后的 v3.1.11 干脆删掉 token 校验，成为完全无鉴权的命令执行接口，并被刻意留作 Latest，内置「在线更新」会把它推送给所有点检查更新的实例。8 分钟后的 v3.1.12 扩展成四合一：执行命令、导出数据库和 Redis 密码、任意 SQL、任意 TCP 连接——最后一项意味着可以当内网跳板。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8acpaHCG8xIcPUAbUibnYTDOlNGR9QNa5n157LS3wNT6ZicgHGztQUgVUBJfH5RFcIyl3M5iaLvZ0UaO0UbkR938LvQXWJnq3fibeho/640?wx_fmt=png "")  
  
三个恶意版本：只有 v3.1.11 被刻意保留为 Latest  
  
本号从 GitHub API 直接拉了关键提交 a2e7745c 的完整 diff——正是删掉鉴权的那次变更。改动两处：main.go 删 8 行 token 校验、换新路由名；同一提交顺手把 Docker 构建工作流换成空壳。diff 至今仍可按 commit SHA 读取，铁证钉死。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8acdv1XqFh3g7mKyVbDicR3BiacCedkSWDf2dKBUB31OlrFYMBrKqWe86ATjnSTymlpNiaRTdgDjfEqIk5iadyyAnDpVJQYz4hONvibQ/640?wx_fmt=png "")  
  
提交 a2e7745c：删除 X-QA-Token 校验，换成无鉴权路由（本号自 GitHub API 存档）  
## 更狠的在后面：偷 CI 密钥，扫了 21 个仓库  
  
如果只是 release 后门，还只是一次普通供应链事件。独立复盘（WormW 与 azure9 联合，报告发出两小时后发布）发现了原始报告都没覆盖的一步：攻击者把名为 zz-qa67489-exfil 的 workflow 推进仓库并触发，把 16 个 Actions secrets——部署 SSH key、数据库连接串、DockerHub/Render/Telegram/DeepSeek 凭据——外传到 C2 服务器。随后同一套工具以 15 秒到 1 分钟一个的节奏，把同一个窃密 workflow 跑遍该账号至少 21 个仓库，全部成功。  
  
两条线同用一套编号（i67425、qa67489），时间交错推进——release 后门打**运行实例**  
，Actions 窃密打**开发链**  
，一次行动两头通吃。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8adzSgo6VIR80yrHyEAbdSRW1bco12sSXt8K0t1eIDHN1P5iaG8F9HQmCrmHg81U2udbJs8vIgHDjTF8BD8dcicuskG7hFwh1p4Sw/640?wx_fmt=png "")  
  
90 分钟全自动：注入 → 发布 → 清理 → 窃密 → 扫仓库  
  
还有个细节：每次发布恶意版本后，攻击者都跑「清理工作流」——镜像指回干净版本、删 release 和 tag。其中一个清理步骤名直接写着 **"Best-effort delete backdoored package versions"**  
——他们明确知道自己在删后门。v3.1.10 和 v3.1.12 都删了，唯独留下 v3.1.11 当 Latest：不是失误，是刻意留一条能持续进人的通道。  
## 「在线更新」为什么成了帮凶  
  
codex2api 内置管理后台一键更新：检查最新版 → 下载 → 校验 SHA256 → 替换重启。听起来规范，问题在校验和的来源——它从**同一个 release**  
 拿 SHA256SUMS.txt 校验这个 release 的文件。发布物和校验值同源，等于自己给自己作证；攻击者控制了 release，自然也控制了校验文件。复盘称之为「自证校验」：防传输损坏，防不了投毒。正确做法是签名加独立信任根——校验值来源必须独立于被校验物。  
  
Docker 用户侥幸躲过：恶意提交把 Docker 构建换成了空壳，GHCR 镜像从未被污染，仅 05:20-05:31 间拉过 latest 的可能短暂拿到脏镜像。  
## 现在该怎么办  
  
官方已在当晚发布 v3.2.0，确认 #916 列出的七个恶意提交均不在新版提交历史中。处置分两种情况：  
  
v3.1.9 及更早的用户直接升级 v3.2.0 即可。运行过 v3.1.10/11/12 任何一个的——不要只是升级：先隔离旧实例保留日志，干净环境重部署，再轮换它读得到的所有凭据（管理密钥、数据库和 Redis 密码、API key、上游 OAuth 令牌）。只换程序不换凭据，等于换锁不换钥匙。  
  
检测侧三个抓手：访问日志查 /i67425、/i67464、/qa67181 开头的路径命中；出站连接查 104.223.21.19:38741；Actions 历史查 qa67489 相关 workflow。命中任何一条，按失陷走完整响应。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8afwOvNMKKTURmtqUA06xbkSBJVLx5sxWyKXyMwUvXyNXto0icicaUibafqtST1bOh3ibsH5MQknkRiaHVdghNwzCQMibUM0iaWia5lINNg/640?wx_fmt=png "")  
  
官方 v3.2.0 紧急安全更新公告：确认三版本投毒与凭据轮换清单  
## 比「谁被黑了」更值得记住的  
  
最该复盘的不是攻击者多聪明，而是三个信任假设被逐个击穿：**源码干净 ≠ 发布物干净**  
，幽灵提交绕过一切代码审计；**同源校验 ≠ 安全校验**  
，校验值必须来自独立信任根；**账号凭据 = 整条供应链**  
，一个 token 泄露，21 个仓库的 CI 密钥全部裸奔。给维护者的建议：release 资产上签名、设 tag 保护规则禁止 main 之外的提交打版本 tag、Actions secrets 定期轮换。给使用者的建议只有一条：更新通道本身就是攻击面，你在后台点的每一次「检查更新」，押上的是对方的整个账号安全。  
  
参考 · 来源  
  
原始安全报告 #916：github.com/james-6-23/codex2api/issues/916  
  
独立复盘（WormW/azure9）：github.com/WormW/codex2api-incident-2026-11  
  
官方 v3.2.0 安全公告：github.com/james-6-23/codex2api/releases/tag/v3.2.0  
  
本文仅为安全事件分析与防御参考，不构成任何攻击指引。 IOC 与时间线均来自公开可验证数据（GitHub API、Actions 运行记录、发布物哈希），本文标注为推断的内容（如攻击者身份）以原始报告的归因边界为准。  
  
