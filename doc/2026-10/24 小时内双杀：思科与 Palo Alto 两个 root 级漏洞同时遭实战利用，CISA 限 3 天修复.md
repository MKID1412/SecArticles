#  24 小时内"双杀"：思科与 Palo Alto 两个 root 级漏洞同时遭实战利用，CISA 限 3 天修复  
原创 承影
                    承影  兰花豆说网络安全   2026-10-18 01:04  
  
# 做安全的都知道一句话：漏洞不可怕，可怕的是"正在被利用"的漏洞。而这一周，两家网络设备的"顶梁柱"厂商，几乎在同一时间被爆出正在被黑客实战利用的 root 级漏洞。一个在你收邮件的网关，一个在你守边界的防火墙。  
## 一、开场：你的"第一道防线"和"最后一道理财邮差"，同时中招  
  
先交代背景，免得外行看懵。  
  
企业网络里有两块"命门"设备：**邮件安全网关**  
，负责筛掉带毒邮件、拦钓鱼；**下一代防火墙（NGFW）**  
，负责在网络边界上拦攻击、做访问控制。思科（Cisco）和 Palo Alto，是全球企业这两块设备最大的两家供应商。  
  
偏偏就在 10 月 14—15 日这两天，两家先后确认：自家的核心产品里，有漏洞**正在被黑客在真实世界里利用**  
，而且利用成功的结果，都是拿到设备的 **root（最高管理员）权限**  
。  
  
这意味着什么？拿到 root，就等于拿到了这台设备的"万能钥匙"——它能读走设备上的一切凭据、悄悄改配置、抹掉作案痕迹，甚至把攻击延伸到整张网络。  
  
这不是演习。CISA（美国网络与基础设施安全局）直接把 Palo Alto 的漏洞塞进了 KEV（已知被利用漏洞）目录，并下令联邦机构在 **10 月 17 日前**  
必须修复并做取证排查。三天，是底线。  
## 二、事件还原：两记闷棍，各有各的打法  
  
**第一记：思科 Secure Email Gateway（CVE-2026-76519）**  
  
10 月 15 日，思科发布通告：Secure Email Gateway 的**高级恶意软件防护（AMP）**  
模块在处理邮件附件时出了岔子。当"文件信誉服务（File Reputation）"开启并绑定到邮件策略、且 Content Scanner Tools 版本低于 25.3.0.5525 时，攻击者只需发一封**精心伪造附件的邮件**  
经过这台网关，就能在设备底层操作系统上**写入任意文件**  
，进而以 root 身份执行命令。  
  
最要命的是：思科明确说"**没有可用的绕过方案**  
"。也就是说，除了打补丁，你没别的临时挡箭牌。一旦设备被拿下，攻击者能拿到设备集群之间互相认证的**私钥 SSH 密钥**  
，借此把集群里其他成员也一锅端，还能删除、隐藏自己的活动痕迹。思科建议查日志 grep "System Security" system_logs  
，并交叉比对防火墙/外联日志里的异常上传下载。  
  
**第二记：Palo Alto PAN-OS（CVE-2026-0322）**  
  
同一天稍早，Palo Alto 披露：PAN-OS 防火墙的**管理接口**  
存在代码注入缺陷（CWE-94），一个已认证的攻击者只要能网络访问到管理接口，就能以 root 执行任意代码。CVSS 4.0 评分 **8.6（高危）**  
，厂商给了最高紧急级别。Palo Alto 说已观察到"非常有限"的实战利用，目标直指管理接口。  
  
CISA 当天就把它加进 KEV，并按 BOD 26-04 指令要求做"取证分类"——也就是说，联邦机构不仅要打补丁，还得先确认"在被攻破前系统到底有没有已经被拿下"。  
  
受影响的是 PA-Series、VM-Series 防火墙和 Panorama 管理平台的 PAN-OS；Cloud NGFW 与 Prisma Access 不受影响。同样，**没有已知的临时绕过方案**  
。  
  
把两件事放在一起看，会发现一个让人不安的共性：**它们都是"拿到设备最高权限"的漏洞，而且都"没有绕过方案"**  
。这意味着，在补丁落地前，防守方几乎只能靠"收敛暴露面"这一招缓冲。对企业来说，这周的加班清单上，这两台设备必须排在最前面——因为它们一旦失守，丢的不只是自己，还有整张信任网络的钥匙。  
## 三、深度拆解：两个漏洞，一张对比表看明白  
  
<table><tbody><tr><td data-colwidth="131"><p style="text-align: center;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv1owd"><span textstyle="" style="letter-spacing: 1px;">维度</span></span></p></td><td><p style="text-align: center;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv1peb"><span textstyle="" style="letter-spacing: 1px;">思科 SEG（CVE-2026-76519）</span></span></p></td><td><p style="text-align: center;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv1n39"><span textstyle="" style="letter-spacing: 1px;">Palo Alto PAN-OS（CVE-2026-0322）</span></span></p></td></tr><tr><td data-colwidth="131"><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv3j3"><span textstyle="" style="letter-spacing: 1px;">产品定位</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv1q80"><span textstyle="" style="letter-spacing: 1px;">邮件安全网关</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv18zo"><span textstyle="" style="letter-spacing: 1px;">防火墙/管理接口</span></span></p></td></tr><tr><td data-colwidth="131"><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv1hij"><span textstyle="" style="letter-spacing: 1px;">触发条件</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv7fd"><span textstyle="" style="letter-spacing: 1px;">伪造附件邮件过网关 + 文件信誉服务开启</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv18j"><span textstyle="" style="letter-spacing: 1px;">能访问管理接口 + 已认证</span></span></p></td></tr><tr><td data-colwidth="131"><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv3ti"><span textstyle="" style="letter-spacing: 1px;">后果</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv24eu"><span textstyle="" style="letter-spacing: 1px;">root 任意文件写入 + 命令执行</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmviq8"><span textstyle="" style="letter-spacing: 1px;">root 任意代码执行</span></span></p></td></tr><tr><td data-colwidth="131"><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv1jsz"><span textstyle="" style="letter-spacing: 1px;">CVSS/紧急度</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmvafi"><span textstyle="" style="letter-spacing: 1px;">严重（厂商最高关注）</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv15es"><span textstyle="" style="letter-spacing: 1px;">CVSS 4.0 8.6，最高紧急</span></span></p></td></tr><tr><td data-colwidth="131"><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv22ng"><span textstyle="" style="letter-spacing: 1px;">CISA 动作</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmvt1y"><span textstyle="" style="letter-spacing: 1px;">建议修复</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmvafp"><span textstyle="" style="letter-spacing: 1px;">已入 KEV，限 10/17 前修复+取证</span></span></p></td></tr><tr><td data-colwidth="131"><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmvvw8"><span textstyle="" style="letter-spacing: 1px;">绕过方案</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmvxxv"><span textstyle="" style="letter-spacing: 1px;">无</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv22nu"><span textstyle="" style="letter-spacing: 1px;">无（Threat ID 仅有限覆盖）</span></span></p></td></tr><tr><td data-colwidth="131"><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmvft1"><span textstyle="" style="letter-spacing: 1px;">最大风险面</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmvf1l"><span textstyle="" style="letter-spacing: 1px;">邮件策略配置不当</span></span></p></td><td><p style="text-align: justify;margin-left: 8px;margin-right: 8px;margin-bottom: 16px;line-height: 1.75em;"><span leaf="" style="color: rgba(0, 0, 0, 0.9);font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.034em;font-style: normal;font-weight: normal;" mpa-font-style="mvburhmv1e8c"><span textstyle="" style="letter-spacing: 1px;">管理接口暴露到公网</span></span></p></td></tr></tbody></table>  
  
两个漏洞有一个共同命门：**管理/暴露面**  
。Palo Alto 自己都坦言，入口"大概率不是漏洞本身，而是脆弱的管理访问卫生"——比如管理员凭据被钓鱼或复用、管理接口被挂在公网上、或者早先的入侵给了攻击者合法登录权限。一句话：**防火墙被攻破，往往不是因为墙不够厚，而是因为管理门没锁好。**  
  
在动手处置前，先把两件"外行容易懵"的事说清楚。**邮件安全网关**  
，你可以理解为公司邮箱的"安检机"，所有进出邮件都要过它，筛病毒、拦钓鱼；**防火墙管理接口**  
，则是管理员远程配置这台墙的"控制台"。两件事放在一起看就明白：一个在你收邮件的入口被撕开，一个在你管设备的后门被撬开，而且都拿到了 root。更要命的是，两家都明确说"没有绕过方案"——这意味着，在补丁打上之前，你连个临时挡箭牌都没有，唯一的缓冲，就是赶紧收敛暴露面。  
## 四、影响与启示：给运维和安全负责人的应急清单  
  
对用这两家设备的企业和机构，这不是"看看就好"的新闻，而是**本周必须排期的加班**  
。给你一份可落地的处置清单：  
  
**1.立刻盘点资产版本**  
  
查思科 SEG 的 AsyncOS 与 Content Scanner Tools 版本、Palo Alto 的 PAN-OS 版本，对照官方修复版本  
  
（思科 15.5.5-014 / 16.5.0-780、Content Scanner Tools ≥25.3.0.5525；PAN-OS 10.2/11.1/11.2/12.1/12.2 对应热修）。  
  
**2.先收敛暴露面，再谈补丁**  
  
Palo Alto 管理接口**绝不能**  
从公网 IP 可达，立刻限制到受信任内网 IP；这是没打补丁前最有效的"缓兵之计"。  
  
**3.做取证，别只打补丁**  
  
尤其 PAN-OS，按 BOD 26-04 先排查是否已被入侵——检查管理员登录历史、设备配置有无异常改动、有无可疑外联。被拿下的设备要保留证据再重建。  
  
**4.查集群与私钥**  
  
思科 SEG 被攻破会牵连集群私钥，要轮换凭据与加密材料，评估同集群其他成员。  
  
**5.开好订阅与日志**  
  
有 Threat Prevention 订阅的 PAN-OS 客户启用 Threat ID 510046/510047（需 SSL 解密，覆盖有限）；对邮件网关异常外联保持监控。  
  
顺带科普两个名词，以后看到别再一脸懵。**KEV（已知被利用漏洞目录）**  
，是 CISA 列的"已经在真实世界被黑客用上了"的漏洞清单，进了这个清单，联邦机构就被强制要求限期修复——它比普通漏洞公告硬得多。**BOD 26-04**  
 则是美国发布的"约束性操作指令"，要求机构在打补丁前，先确认系统"被攻破前到底有没有已经被拿下"，也就是强制做取证。说白了：CISA 不光让你补洞，还让你先查"洞是不是已经被钻过了"。这不是 PAN-OS 第一次在管理接口上栽跟头，历史反复告诉我们，墙的管理门，永远是最先被敲的那扇。  
  
给国内的用户也提个醒：别以为这是"美国设备的事"。管理接口暴露到公网、设备版本长期不更新，是国内外都普遍存在的老毛病。无论你用的是哪家防火墙，先把"管理口能不能从公网敲开"这件事查清楚，比等一个 CVE 实在得多。  
## 五、收尾：金句与互动  
  
一句话总结：**最坚固的墙，往往是从没锁好的管理门被推倒的；而最该拦毒的网关，自己也可能成为投毒的入口。**  
  
这一周的"双杀"给所有甲方提了个醒：你的边界设备不是"设了就安全"，它们是攻击者眼里优先级最高的 trophy（战利品）。补丁要打，暴露面更要收。  
  
**互动时间**  
：你家的防火墙管理接口，现在是从公网能直接敲开的吗？敢不敢现在去查一眼？欢迎在评论区说说你们的网络设备暴露面治理经验——也提醒一句：查完记得把那扇门，关上。  
> 资料来源：CSO Online、Cisco 安全公告、Palo Alto 安全公告、CISA KEV 目录（2026 年 10 月 14—15 日）。修复版本与缓解措施以厂商官方公告为准。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/buQYO9JqQcqHibMSbuUlH1q4N0BqiaJT4A81RZWbGMibEU57wlC7W6ryRDOF3vnn8UEfjlTdxeyEovsOzIibw7B11Xp7353IVMKgugHfLB4YWic4/640?from=appmsg "")  
  
END  
  
推荐阅读  
  
[刚刚！微软端掉一个"中国钓鱼帝国"：10 亿封诈骗邮件，牵出 3 个中文工具包](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494059&idx=1&sn=75c990df8f1341e5ae7716c5d1b3def5&scene=21#wechat_redirect)  
  
  
2026-10-17  
[](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494059&idx=1&sn=75c990df8f1341e5ae7716c5d1b3def5&scene=21#wechat_redirect)  
  
  
[某公司员工将等保备案测评模板材料和测评脚本上传至境外某平台被罚款6万](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494054&idx=1&sn=b806ad5a4e55dd280de13962a5e03803&scene=21#wechat_redirect)  
  
  
2026-10-16  
[](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494054&idx=1&sn=b806ad5a4e55dd280de13962a5e03803&scene=21#wechat_redirect)  
  
  
[网络安全工程师必知：Function Calling 和 MCP，到底是什么？](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494049&idx=1&sn=d47aeba36a0fc4c546855621066e818a&scene=21#wechat_redirect)  
  
  
2026-10-14  
[](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494049&idx=1&sn=d47aeba36a0fc4c546855621066e818a&scene=21#wechat_redirect)  
  
  
[网络安全人士必知的AI Agent网络攻防能力基准测试框架BountyBench](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494039&idx=1&sn=87bb44dc7cad9b7270a3aee5bb2f4c92&scene=21#wechat_redirect)  
  
  
2026-10-13  
[](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494039&idx=1&sn=87bb44dc7cad9b7270a3aee5bb2f4c92&scene=21#wechat_redirect)  
  
  
[一文讲透 SCA 与 SBOM：软件供应链安全，为什么两者缺一不可？](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494031&idx=1&sn=6eeaf68397e78efa5250f925ca23c698&scene=21#wechat_redirect)  
  
  
2026-10-12  
[](https://mp.weixin.qq.com/s?__biz=MzI3NzM5NDA0NA==&mid=2247494031&idx=1&sn=6eeaf68397e78efa5250f925ca23c698&scene=21#wechat_redirect)  
  
  
  
