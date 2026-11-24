#  安全工具丨AI 驱动的自动化代码审计与漏洞挖掘平台，支持多 Agent 协作扫描、数据流分析、漏洞管理和可视化审计  
原创 蓝星安全
                    蓝星安全  蓝星安全   2026-11-23 16:01  
  
         
**点击上方****蓝星安全****关注我**  
  
****  
  
**免责声明：本公众号分享的任何资料仅限用于安全学习，严禁用于其他用途，请严格遵守中华人民共和国法律法规，对因不遵守国家法律法规而产生的任何后果，均由个人自行承担，本公众号不承担任何责任！**  
  
****  
**获取资料，请扫码下方二维码加入知识星球**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcmxg12rU33pwcICNRkiaof5YSUAGfWPApU7M1BfdsTTOyvREw0gKD42g4U8eefG4n0XuYETEtbxSIN2drACnNVz3UeicCcldM5AA/640?wx_fmt=png&from=appmsg "")  
## 一、简介  
  
         ZeroHunter 是一个 **AI 辅助的代码安全审计与漏洞挖掘平台**  
。它通过多个专用 Agent 协同分析项目目录、调用链、跨请求数据流和潜在漏洞，并提供一个本地 Web 控制台，用于管理项目、运行扫描、查看审计结果和维护规则。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/opKHaHXLxcms64icELLwOqiaiclqBu01jl0WbxR9lnricS9zNnofww0K3mB1HP8s7UyWxWUVLzbIFCiaVfIv6HGqAn8NhXRUaT3ZglrFwZibKASOM/640?wx_fmt=png&from=appmsg "")  
## 二、核心功能  
### 2.1. TreeScan：先把仓库“看全”  
  
代码审计的第一步是建立地图，而不是找漏洞：这个项目有哪些目录、哪些入口、哪些高风险区域。  
  
TreeScan 负责这一层：  
- 对仓库做结构化扫描与切片；  
  
- 产出后续阶段可消费的扫描范围与目录视图；  
  
- 让“从哪里看起”有据可依，而不是凭感觉挑文件。  
  
它的价值在于**避免“抓着一个文件看到底”**  
。大型项目里，先有全局视图，后面的调用链和数据流才不会迷路。  
### 2.2. CallScan：把入口和危险操作连起来  
  
单个函数看起来人畜无害，漏洞往往出在“入口 → 中间层 → 危险操作”的链路上。  
  
CallScan 负责调用链分析：  
- 从入口点出发，沿调用关系向下追；  
  
- 识别中途的校验、过滤、权限判断；  
  
- 把“可能走到危险函数”的路径标出来。  
  
这一步的产出，是后续 Auditor 判断“是否真的可达”的关键证据。没有调用链，漏洞结论就只能靠猜。  
### 2.3. DataFlowScan：跨请求追数据  
  
很多真实漏洞往往需要跨请求、跨会话才能触发；“一个请求里参数直接进 SQL”反而是少数情况：登录后拿 token、后续请求带 token、服务端用 token 查数据。  
  
DataFlowScan 的重点是**数据流**  
：  
- 参数从哪里来；  
  
- 经过哪些函数、哪些存储；  
  
- 有没有被校验/编码/白名单约束；  
  
- 最终有没有到达 Sink。  
  
README 特别强调“**跨请求数据流**  
”，这是它和普通静态扫描器的重要区别：只做单函数污点分析，很容易漏掉需要多步请求才能触发的漏洞。  
### 2.4. Auditor：汇总证据，下结论  
  
前三个阶段产出的是“材料”，Auditor 负责“判决”：  
- 汇总 TreeScan 的范围、CallScan 的路径、DataFlowScan 的数据流；  
  
- 判断漏洞是否成立；  
  
- 给出审计结论与证据链；  
  
- 写入漏洞库，供人工复核。  
  
它并非“再问一遍模型”的步骤；它把前序证据组织成可读、可查的结论。对审计员来说，这比“模型说这里有问题”更有用。  
### 2.5. 项目与版本管理：一次审计，多个版本  
  
平台按“来源 / 所有者 / 仓库 / 版本”组织待审计代码：  
- 同一个项目可以维护多个版本；  
  
- 每个版本有独立的扫描记录与产物；  
  
- 支持完整项目扫描，也支持针对指定目录创建 **specific run**  
。  
  
这对“版本迭代后复扫”很实用：老版本的结论、新版本的差异，都能在同一个项目下对照，而不是每次从零开始。  
### 2.6. 漏洞审计与复核：证据链、调用链、数据流  
  
漏洞详情页不只给一条结论，而是把证据展开：  
- 漏洞证据；  
  
- 关联扫描链；  
  
- 调用链；  
  
- 数据流；  
  
- 审计结论；  
  
- 人工标注。  
  
这意味着审计员可以沿着证据逐层核对：入口在哪、路径通不通、参数可不可控、校验缺没缺。**结论可以被质疑，也可以被证伪**  
——这是代码审计平台最重要的品质。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxckK7MgRpxesp0N6IXrq7ysId3Y0xY2icRxgp6yTNhwKd5FpptTYEeTV8fX3NIibgfibWia2y8OWy7v8vXfRFfhcKyHVjaHibQSrD4PE/640?wx_fmt=png&from=appmsg "")  
### 2.7. 可配置知识库：Prompt、Skill、Sink 规则  
  
平台把“审计经验”做成了可维护的配置：  
- **Prompt**  
：不同 Agent 的提示词；  
  
- **Skill**  
：按语言组织的漏洞知识模块，仓库里能看到 PHP、Java、JavaScript、Python 等目录；  
  
- **Sink 规则**  
：按语言定义的危险操作，覆盖 SQL 注入、命令执行、反序列化、文件包含、SSRF、XXE、XSS 等类型，规则目录里也能看到 C/C++ 等语言。  
  
这套设计的价值是**可调**  
：不同项目、不同语言、不同团队经验，可以在控制台里改规则、改提示词，而不是等官方发版。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcnqlKucsrAq4QhYfJicgSmgj3Z29jibBKHtO3DRoZnjezNEypBYC4pGw4wjQ7c1SdjcXw4EnTkSbQA5GrXJNuvWsoXxAia4ImUGzY/640?wx_fmt=png&from=appmsg "")  
  
### 2.8. LLM 配置切换：效果与成本的平衡  
  
平台支持维护多个模型配置，按需切换扫描模型与参数。  
  
这对代码审计很实际：  
- 面对大仓库、复杂调用链，使用强模型；  
  
- 对批量初筛、简单规则匹配，使用成本较低的模型；  
  
- 不同项目可以绑定不同配置，避免“一套参数打天下”。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcmPZNW9j7ibDWB8DuiaRAib1BHgdzx2kJH9SEOkGGE65RqcHDHaYqaKppIn3XfDqN40upA7PR8iaUYl2ZO7AVyp2B5ichmRpMvUNtYE/640?wx_fmt=png&from=appmsg "")  
  
三、项目地址  
  
https://github.com/vs-limits/ZeroHunter  
  
