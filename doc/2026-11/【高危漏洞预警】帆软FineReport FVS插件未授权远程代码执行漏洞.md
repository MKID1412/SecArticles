#  【高危漏洞预警】帆软FineReport FVS插件未授权远程代码执行漏洞  
 飓风网络安全   2026-11-23 15:51  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Yd9HAo0qc3pFJBFDNiaBfc1tTezaBWmJXcKAdEibjK2ToyiclKk8uYP06pRmoufSoDEzBKPic0rSiaqMAWJh9CHeXicaAFCSG0QuobHicUeZWz9zXw/640?wx_fmt=jpeg "")  
  
一、预警摘要  
- **情报披露**  
：帆软 FineReport 的**FVS（FineVis 数据可视化）插件**  
存在**未授权远程代码执行漏洞**  
；  
- **功能环节（情报标题指向）**  
：与 FVS**复用资源导入**  
相关——即 FVS 资源中心中共享/复用组件、模板等资源的导入环节，鉴权或资源校验存在缺陷；  
- **初步风险定性**  
：未授权（无需登录）触发，若情报属实，攻击者可在未登录状态下构造请求触达导入/资源处理逻辑，存在服务器端代码执行风险；  
- **当前缺口**  
：漏洞编号、受影响插件/工程版本、触发接口参数、修复版本、官方缓解措施、公开 PoC/EXP**均未公开**  
；帆软官方安全漏洞声明页截至本文编写时**尚未收录**  
该条目。  
> 结论先行：**先按"高优先级待核实情报"纳入资产排查，不要等待完整 PoC；但在厂商通告或授权复现确认前，不要对外宣称具体接口、版本或攻击链路。**  
  
## 二、信息核实状态表  
<table><thead><tr><th><span leaf="">字段</span></th><th><span leaf="">状态</span></th><th><span leaf="">说明</span></th></tr></thead><tbody><tr><td><span leaf="">漏洞编号（CVE/CNVD）</span></td><td><strong><span leaf="">待补充</span></strong></td><td><span leaf="">公开渠道未检索到分配编号</span></td></tr><tr><td><span leaf="">厂商</span></td><td><span leaf="">帆软软件（FanRuan）</span></td><td><span leaf="">已核实</span></td></tr><tr><td><span leaf="">产品</span></td><td><span leaf="">FineReport（集成 FineVis / FVS 数据可视化插件）</span></td><td><span leaf="">已核实；FineBI 同插件是否受影响待确认</span></td></tr><tr><td><span leaf="">漏洞类型</span></td><td><span leaf="">未授权远程代码执行（情报口径）</span></td><td><span leaf="">已核实情报定性；具体根因待官方/研究方披露</span></td></tr><tr><td><span leaf="">关联功能</span></td><td><span leaf="">FVS 复用资源导入 / 资源中心</span></td><td><span leaf="">依据情报标题与 FVS 功能模块推断，</span><strong><span leaf="">待官方确认具体接口</span></strong></td></tr><tr><td><span leaf="">受影响版本区间</span></td><td><strong><span leaf="">待补充</span></strong></td><td><span leaf="">未公开；需以插件版本号 + 工程 JAR 包日期为准</span></td></tr><tr><td><span leaf="">修复版本 / 补丁</span></td><td><strong><span leaf="">待补充</span></strong></td><td><span leaf="">未公开；参考历史同类问题，通常为升级 FineVis 插件与工程版本</span></td></tr><tr><td><span leaf="">公开 PoC / EXP</span></td><td><strong><span leaf="">未公开</span></strong></td><td><span leaf="">暂无公开可利用代码；本文不提供、也不臆造利用脚本</span></td></tr><tr><td><span leaf="">在野利用</span></td><td><strong><span leaf="">待确认</span></strong></td><td><span leaf="">情报未明确是否已被在野利用</span></td></tr><tr><td><span leaf="">官方安全声明</span></td><td><span leaf="">尚未收录</span></td><td><span leaf="">帆软官方漏洞声明页目前最新条目为 2025-10-20 export/excel API SQL 注入</span></td></tr></tbody></table>## 三、背景：FVS 与"复用资源"是什么  
- **FineReport**  
：帆软企业级报表平台，广泛部署于政企、金融、制造、零售等行业的内网与 DMZ 环境；  
- **FVS（FineVis 数据可视化插件）**  
：FineReport 中用于搭建可视化大屏（战情室/看板）的插件模块，提供画布、组件、模板与资源中心；  
- **复用资源**  
：FVS 资源中心内可被多个大屏/模板共享引用的组件、图表、主题、模型等资源；支持导入、上传、分组与复用。历史文档中与"复用资源"相关的错误码为 72300001/72300002/72300003/72300004，对应资源不存在、加载失败等场景；  
- **风险相关性**  
：资源导入类功能天然涉及**文件/包解析、服务端写入、组件加载**  
，一旦鉴权缺失或解析校验不严，容易演化为任意文件写入、反序列化或代码执行——这也是帆软历史多次漏洞集中出现的功能区域。  
> 以上为产品功能背景；"复用资源导入环节鉴权缺失导致 RCE"的具体技术路径，**需等厂商或研究方正式披露后确认**  
。  
  
## 四、历史同类漏洞脉络（风险研判参考，非本次漏洞结论）  
  
帆软报表/FVS 插件近年已公开的同类问题，可作为本次排查的参照：  
<table><thead><tr><th><span leaf="">时间</span></th><th><span leaf="">漏洞</span></th><th><span leaf="">类型</span></th><th><span leaf="">影响范围（历史口径）</span></th><th><span leaf="">处置</span></th></tr></thead><tbody><tr><td><span leaf="">2024-08</span></td><td><span leaf="">CNVD-2024-45455</span></td><td><span leaf="">授权用户远程命令执行</span></td><td><span leaf="">FineVis 插件 V2.6.1~V2.9.0.4</span></td><td><span leaf="">升级插件至 V2.9.1.1 及以上</span></td></tr><tr><td><span leaf="">2024-08-04</span></td><td><span leaf="">FVS 目录穿越</span></td><td><span leaf="">路径穿越</span></td><td><span leaf="">FineVis 插件 V2.7.1~V2.9.0.2</span></td><td><span leaf="">升级插件至 V2.9.1.1 及以上</span></td></tr><tr><td><span leaf="">2024-07-25</span></td><td><span leaf="">FVS 多个中危漏洞</span></td><td><span leaf="">组合缺陷</span></td><td><span leaf="">FineVis 插件 V2.6.1~V2.9.0.1</span></td><td><span leaf="">升级工程与插件版本</span></td></tr><tr><td><span leaf="">2024-07</span></td><td><span leaf="">ReportServer/channel 接口</span></td><td><span leaf="">命令执行/SQL 注入/任意文件下载</span></td><td><span leaf="">JAR 包 2024-07-23 之前版本</span></td><td><span leaf="">升级工程；禁用老引擎/远程设计接口；清理 sqlite 驱动</span></td></tr><tr><td><span leaf="">2025-03</span></td><td><span leaf="">HSQL 命令执行、任意文件读取</span></td><td><span leaf="">命令执行/文件读取</span></td><td><span leaf="">FR 11.0.32 及以下等</span></td><td><span leaf="">升级至 11.0.33 及以上</span></td></tr><tr><td><span leaf="">2025-10</span></td><td><span leaf="">export/excel API SQL 注入</span></td><td><span leaf="">SQL 注入</span></td><td><span leaf="">FR 11.5.4 及以下等</span></td><td><span leaf="">升级工程版本</span></td></tr></tbody></table>  
**规律**  
：帆软风险长期集中在三类——① 插件/模板/资源解析；② 远程设计与 channel 类接口；③ 数据库驱动（sqlite/MySQL/HSQL）相关功能。本次 FVS 插件未授权 RCE 落在第一类。  
## 五、风险研判  
1. **平台暴露面广**  
：FineReport 在国内报表/BI 市场装机量大，部分实例将报表门户、预览入口直接暴露在 DMZ 或办公网，内网横向也可达；  
1. **未授权意味着门槛低**  
：若无需登录即可触达导入/资源处理接口，相比"授权后 RCE"，可被批量扫描与自动化利用；  
1. **报表平台失陷后的连带危害**  
：RCE 后可读取平台配置、数据连接凭据（数据库账号口令）、模板与数据集，进而横向到业务数据库；插件/模板加载路径也可能被用于持久化；  
1. **版本不确定期的应对原则**  
：在官方版本区间公布前，以"资产是否暴露、是否启用 FVS、插件与工程版本是否偏旧"作为风险分级依据，而不是仅凭版本号下结论。  
## 六、自查与缓解建议（不依赖未公开细节即可先执行）  
### 6.1 资产盘点（立即做）  
1. 梳理全部 FineReport / FineBI 实例，记录：工程版本、JAR 包日期、部署方式（传统/Tomcat/运维平台/容器）、是否安装 FineVis（FVS）插件及插件版本；  
1. 标注网络位置：公网/DMZ/办公内网/生产内网；记录访问入口域名、端口与反向代理路径；  
1. 核查是否启用 FVS 大屏与资源中心、是否开放资源导入/上传能力。  
### 6.2 网络与访问收敛  
1. 报表门户、设计/资源管理类入口限制来源 IP，仅允许运维与可信网段访问；  
1. 公网暴露的实例优先下线或置于 VPN/堡垒机之后；  
1. 按帆软《产品安全加固手册》禁用不必要的远程设计、单点登录、老引擎等接口。  
### 6.3 账号与配置加固  
1. 清理内置测试账号与弱口令，管理员账号强制强口令与 MFA；  
1. 核查插件列表，删除不明来源/历史遗留插件（参考历史恶意插件事件）；  
1. 核查数据连接中是否存在 sqlite 等高危驱动及演示数据连接；  
1. 检查资源中心是否存在非本团队上传的模板、组件或资源包。  
### 6.4 日志与狩猎（待细节公开后可细化规则）  
- 审计访问日志中对 FVS/资源导入、模板上传、插件管理相关路径的**未登录/匿名访问**  
；  
- 关注异常的资源包上传、模板导入、服务端新增脚本/文件；  
- 关注报表进程拉起的异常子进程、外联与数据连接变更；  
- 回溯近期公网来源的可疑请求记录。  
### 6.5 升级与跟踪  
1. 持续跟踪帆软官方安全漏洞声明页与 360 漏洞研究院后续通告，等待：受影响版本、修复版本、接口细节；  
1. 历史版本（FR 8/9/10、FineBI 4/5、FDL 4.0/4.1）已停止维护，纳入大版本升级计划；  
1. 官方补丁发布后，按"工程版本 + FineVis 插件版本"双维度升级，避免只升其一。  
## 七、PoC 与验证边界  
### 7.1 当前状态  
- **公开 PoC/EXP：无**  
。截至本文编写，未检索到公开可复现的利用代码、报文样本或技术细节；  
- 本文**不提供**  
利用脚本、报文构造或 webshell 模板；在未取得授权、未确认受影响版本前，任何主动探测/利用均属违规。  
### 7.2 授权复测前置条件（待细节公开后）  
1. 仅在隔离实验环境、使用自建测试实例进行；  
1. 确认工程版本、FineVis 插件版本与 JAR 包日期；  
1. 对照官方通告确认受影响版本区间与触发接口；  
1. 复测以"是否存在未授权资源导入/代码执行"为判据，不植入持久化或外联载荷；  
1. 生产环境不得直接投递测试样本。  
### 7.3 待官方/情报方补充后再完善的内容  
- 触发接口与请求参数、鉴权缺失的具体环节；  
- 受影响工程与插件版本区间、修复版本号；  
- 根因归类（文件写入/反序列化/模板表达式注入/路径穿越等）；  
- 可用于 SIEM/WAF 的检测特征与 IOC。  
