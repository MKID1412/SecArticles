#  【一天一个nacos漏洞·第三天】【严重】Nacos的MCP和A2A  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-02 05:47  
  
   
  
# 【一天一个nacos漏洞·第三天】【严重】名字是拼出来的，权限只看半截  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天两份都是 **9.1 分**  
，一个在 MCP，一个在 A2A。  
  
共同点不只是"鉴权对象和操作对象不一致"——更狠的是**那个 key 本身是拼出来的**  
：把名字和版本用一个连字符粘在一起，而这个连字符**本来就是名字的合法字符**  
。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址、账号与口令均未写入本文。内容仅用于安全学习与防御加固。  
  
## 一、这两样东西怎么存的  
  
Nacos 3.x 的 AI 模块里，有两类对象要"登记"：  
  
**MCP 服务器**  
——把 AI Agent 要用的工具服务登记进来，含端点、工具清单、版本。  
  
**A2A Agent**  
——Agent 之间互相调用的入口，登记的是一张 AgentCard  
：名字、版本、地址、协议绑定、技能。别人按"名字 + 版本"来查你的卡片，然后照着上面写的地址发请求。  
> 💡 今天两个洞的落点，都在**“这条记录用什么 key 存”**上。给 A2A 的卡片算 key 的那行代码是：编码(名字) + "-" + 版本  
。**请记住这个连字符。**  
  
## 二、MCP：还是那个错位，但这次够 9.1  
  
第二天讲过 MCP 上的"鉴权看 mcpName  
、干活看 mcpId  
"。今天这份是同一个位置、换了个靶机，但有两处新东西值得看。  
### 2.1 权限被收到最小，攻击反而更成立  
  
这次没有用那种"public:*:*  
 通配"的普通账号来测——那样容易被说成"权限本来就大"。测试者临时建了一个**只持有一个 MCP 名字的读写权限**  
的账号，其它什么都没有：  
  
```
permission: public:*:ai/demo-mcp  →  rw
（只有这一个名字，别人的 MCP 一个都不在授权范围内）

```  
  
  
然后是决定性的对照：**直接用受害者名字访问，403；用自己的名字访问、把操作对象换成受害者 id，200。**  
  
<table><thead><tr><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">动作</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">请求要点</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">结果</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">对照</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">mcpName=受害者</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">403</span></strong><span style="display: inline;"><span leaf=""> authorization failed</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">读</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">mcpName=demo-mcp&amp;mcpId=受害者</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">200，完整定义（含 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">localServerConfig</span></code><span style="display: inline;"><span leaf="">）</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">改</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">body 里 id 指向受害者，追加版本 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">6.6.6-evil</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">200，受害者多出一个版本</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">删</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">DELETE ?mcpName=demo-mcp&amp;mcpId=受害者</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">200，整条 MCP 消失</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">建</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">在未授权的名字下创建资源</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">200，抢注成功</span></span></td></tr></tbody></table>  
  
三个入口（Console 8080、Admin 8848、gRPC 9848）走的是同一段业务逻辑，全通。  
### 2.2 那 3 秒，是这条链最诚实的证词  
  
删完之后出现一个很有意思的现象：用受害者的 id 去查，**立刻查还是 200，大约 3 秒后才变成 not found**  
。  
  
为什么？因为**缓存失效也是按攻击者自报的那个名字去失效的**  
——被清掉的是 demo-mcp  
 的缓存，而受害者那台 MCP 的缓存在旁边安安静静地活着，直到自然过期。  
> 🔍 这 3 秒比任何日志都能说明问题：**“鉴权对象”“失效对象”“实际被删对象”，三者可以是三个不同的东西**  
。 如果它们真的一致，删除的瞬间受害者就该查不到了。  
  
## 三、A2A：这次不是两个字段，是一个字段被劈成两半  
  
A2A 这个洞的形态不太一样，我觉得比 MCP 那个更值得写。  
### 3.1 key 是怎么拼的  
  
```
String actualDataId = encode(name) + "-" + version;   // 写
String versionDataId = encode(name) + "-" + version;  // 读、删

group = "agent-version"

```  
  
  
看起来没问题对吧？问题在下一行。  
### 3.2 那个连字符，本来就在名字里合法  
  
编码器允许名字里出现字母、-  
、.  
、:  
，纯字母的名字原样返回。而版本号**完全不校验格式**  
——只要非空就行。  
  
```
(name="a",     version="b-1.0.0")  →  a-b-1.0.0
(name="a-b",   version="1.0.0")    →  a-b-1.0.0
                                     ↑ 同一个 dataId

换成真实的：
(demo,         demo-x-1.0.0)  →  demo-demo-x-1.0.0
(demo-demo-x,  1.0.0)         →  demo-demo-x-1.0.0

```  
  
  
也就是说：**一个字符串拼出来之后，你没法反推它是哪个 (名字, 版本) 拼的。**  
 这不是"校验没写"，是"这个 key 从设计上就不唯一"。  
### 3.3 于是攻击只要两步  
  
第一步，找一个自己有权创建的名字（比如 demo  
）；第二步，把版本号填成**受害者名字的剩余部分**  
。  
  
```
GET /v3/console/ai/a2a?agentName=demo&version=victim-x-1.0.0
                      ↑ 鉴权看这里      ↑ 从不参与鉴权

→ 200 {"name":"demo-victim-x","version":"1.0.0",
        "url":"https://victim.example/a2a", ...}

```  
  
  
因为鉴权层只读 agentName  
 拼资源名 public:DEFAULT_GROUP:ai/<agentName>  
，**版本号从头到尾没有进过任何一次权限判断**  
。前半截合法就够了，后半截你想写什么都行——而业务层拿这两半拼出一个 key 去读写。  
### 3.4 决定性对照：直连被拒，撞键通过  
  
<table><thead><tr><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">同一个受限账号（只授权 ai/demo）</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">结果</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">直接读受害者：</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">agentName=demo-victim-x</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">403</span></strong></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">撞键读：</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">agentName=demo&amp;version=victim-x-1.0.0</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">200</span></strong><span style="display: inline;"><span leaf="">，拿到完整卡片</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">撞键改：PUT，卡片 url 换成我的地址</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">200</span></strong><span style="display: inline;"><span leaf="">，受害者读到的 url 变了</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">撞键删：DELETE 同一个 version</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">200</span></strong><span style="display: inline;"><span leaf="">，受害者侧 404</span></span></td></tr></tbody></table>  
  
只读账号也不是没事干——它能跨 agent 读卡片。**读到的 url 和 supportedInterfaces，就是下游 Agent 接下来要连的地方。**  
### 3.5 顺带的两个：别名撞车、半截注册  
  
**别名撞车。**  
 编码器有个短路逻辑——名字以 ____:  
 开头就认为"已经编码过了"，原样返回。于是 ____:xxx  
 这种名字，和真正带斜杠的名字编码后**落在同一个 meta 键上**  
——两个语义完全不同的公开名，共用一条记录。  
  
**半截注册。**  
 注册分两步写库：先写 meta，再写版本详情。第二步冲突了会返回 409，**但第一步不回滚**  
。结果库里留下一个"指向别人版本、却挂着自己名字"的影子 agent；而受害者那边重试注册，会一直 409。  
## 四、为什么这两个都是 9.1  
  
两份报告的向量一模一样：  
  
```
AV:N / AC:L / PR:L / UI:N / S:C / C:L / I:H / A:L  →  9.1（Critical）

```  
  
  
前面几项没什么争议：网络可达、构造确定、只要一个普通账号、不需要受害者做任何事。真正决定它跨过 9 分线的，是 S:C  
（作用域变更）这一项。  
  
理由其实很朴素：**我的授权范围只有"我自己的那一个名字"，而我实际动到的是别人的对象——这已经出了我原本被授权的边界。**  
 再加上被改掉的 url  
 / 端点会被下游客户端拿去用，影响继续往外传。  
> 📐 **说句公道话：**  
 如果按"同一个 Nacos 实例内的数据对象算同一作用域"来算（S:U  
），同样的影响是 **8.8 High**  
——也就是第二天那个 MCP 漏洞的分数。**9.1 和 8.8 差的不是危害大小，而是"作用域怎么划"。**  
 这一点在评审时经常被吵，先把口径说清楚，比争分数有用。  
  
  
另外两份报告都写了 is_command_exec = false  
，这点很克制：它们并没有在本机落一个文件、也没有执行一条命令。但**二阶风险是实打实的**  
——  
- 被改写的 MCP localServerConfig  
（stdio 的 command / args / env），最后是被**别人的客户端**  
在本地启动时消费的；  
- 被改写的 AgentCard url  
，会让所有按版本查这张卡片的 Agent，把请求发到攻击者指定的地址。  
写报告时把这两层分开写（“我确实没执行” vs “但它会被执行”），比笼统标一个"可 RCE"要诚实得多。  
## 五、修复：别让 key 由字符串拼出来  
  
<table><thead><tr><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">key 有歧义</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">别用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">名字 + &#34;-&#34; + 版本</span></code><span style="display: inline;"><span leaf="">。要么加</span></span><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">长度前缀</span></strong><span style="display: inline;"><span leaf="">（</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">len:name:len:version</span></code><span style="display: inline;"><span leaf="">），要么干脆用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">(resource_id, version)</span></code><span style="display: inline;"><span leaf=""> 两列结构化存储——</span></span><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">两个值就别往一个字符串里塞</span></strong></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">version 不受管</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">复用同项目已有的版本格式白名单（Skill / AgentSpec 那边就有），别只判非空</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">没做归属校验</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">读写删之前校验 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">version ∈ meta.versionDetails</span></code><span style="display: inline;"><span leaf="">；读回来再核对一次 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">name/version</span></code><span style="display: inline;"><span leaf=""> 是否等于请求</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">鉴权只看半截</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">资源名要把 version 纳入，或者在 service 层再做一次对象级校验；MCP 那边同理——</span></span><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">先解析出对象，再用对象的规范名鉴权</span></strong></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">编码器不单射</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">取消 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">____:</span></code><span style="display: inline;"><span leaf=""> 前缀的短路放行；保证 encode 严格一对一</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">两步写库不回滚</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">meta + 版本详情放进同一事务；或第二步失败时补偿删除第一步</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">缓存与审计错位</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">失效和记账都用</span></span><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">解析后的真实对象</span></strong><span style="display: inline;"><span leaf="">，不用请求里自报的名字</span></span></td></tr></tbody></table>  
> 💡 一个可以套用的判据：**当两个不同来源的值被拼成一个标识符，先问一句"拼完还能不能还原"。**  
还原不了，就说明这段代码创造了一个可能指向别的东西  
的键。分隔符换成 |  
、\0  
 也未必安全——**最稳的是不拼。**  
  
## 写在最后  
  
连着三天，三个洞其实在讲同一类事——**都是"拼接"出的问题**  
，只是拼的东西不一样：  
  
**第一天**  
：把 namespace / group / dataId 拼成一条文件路径，拼完谁也没再确认它还在不在原地；  
  
**第二天**  
：两个字段各管一段（鉴权看 A、干活看 B），中间没人问"你们说的是同一个吗"；  
  
**第三天**  
：把名字和版本拼成一个 key，而这个 key 从数学上就不唯一。  
  
拼这件事本质是**把结构压平成字符串**  
。压平的那一刻，信息就丢了；而后面所有的权限判断、归属校验、缓存失效，都建立在那份被丢掉的信息上。  
  
所以修的时候也就一句话：**该是结构的地方，别用字符串表示。**  
 下一天见。  
> 📚 **系列回顾 · 【一天一个nacos漏洞】**  
> **第一天**  
：客户端快照路径穿越，一路退到 /etc/cron.d  
。**第二天**  
：MCP 与 Skill，鉴权看 A、干活看 B。**第三天**  
（本篇）：两个 9.1——A2A 的 key 拼接歧义，以及 MCP 那个错位的加强版。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
