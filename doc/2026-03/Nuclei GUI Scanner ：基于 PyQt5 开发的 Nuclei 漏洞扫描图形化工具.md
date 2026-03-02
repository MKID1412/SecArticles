#  Nuclei GUI Scanner ：基于 PyQt5 开发的 Nuclei 漏洞扫描图形化工具  
原创 网安武器库
                    网安武器库  网安武器库   2026-03-01 03:33  
  
**更多干货  点击蓝字 关注我们**  
  
  
  
**注：本文仅供学习，坚决反对一切危害网络安全的行为。造成法律后果自行负责！**  
  
**往期回顾**  
  
  
  
  
  
  
·[Onyx：高效辅助的一站式渗透测试工具集](https://mp.weixin.qq.com/s?__biz=MzYzNTExNDYwMg==&mid=2247486547&idx=1&sn=2b2127c20251a5d01dbe9a5e0b7f4d18&scene=21#wechat_redirect)  
  
  
  
  
  
  
·[AI-Reverse-Engineering：ai逆向工具实战和CTF适用](https://mp.weixin.qq.com/s?__biz=MzYzNTExNDYwMg==&mid=2247486534&idx=1&sn=a516682db8b31e552d4879ea57676567&scene=21#wechat_redirect)  
  
  
  
  
  
  
·[Venom：全面的渗透测试利器](https://mp.weixin.qq.com/s?__biz=MzYzNTExNDYwMg==&mid=2247486518&idx=1&sn=08481bf21ca0caa11e1a60abc67f8e55&scene=21#wechat_redirect)  
  
  
  
  
  
  
·[猫头鹰 XSS 平台：一个针对XSS漏洞的测试平台](https://mp.weixin.qq.com/s?__biz=MzYzNTExNDYwMg==&mid=2247486503&idx=1&sn=b0abad58a1cdbebb8c1aec9e2e405dbd&scene=21#wechat_redirect)  
  
  
  
  
  
  
·[Donut+SGN 利用微软签名进行静态特征混淆与终端检测规避](https://mp.weixin.qq.com/s?__biz=MzYzNTExNDYwMg==&mid=2247486480&idx=1&sn=91080cabf64e334fb3151852df260b99&scene=21#wechat_redirect)  
  
  
  
  
  
  
·[ManSpider：一款黑客内网快速敏感信息搜集工具](https://mp.weixin.qq.com/s?__biz=MzYzNTExNDYwMg==&mid=2247486462&idx=1&sn=13d367de0d7867608564ca9827a012b9&scene=21#wechat_redirect)  
  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ibCZqSDX9ugSGKJibovaia9YxcaLfMJib6eFcsfYatVNptgRDr3kqeFwpGYKFziaX9s7BBcG8prEJFW1g1EickibFyug/640?wx_fmt=png&from=appmsg "")  
  
**背景分析**  
  
  
  
    Nuclei GUI Scanner 是一款基于 PyQt5 开发的 Nuclei 漏洞扫描图形化工具，提供友好的可视化界面，支持 POC 管理、资产搜索、AI 辅助分析等功能。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/x2ibBTFXYHicJvJE0icpyubr2INVBfhKH6XDQb3j2seCVuyick03KicTpyzPCHiafia6jsCrNTefzjPUmtqicynrZB0riaS7IVDd67s4wEcMgiaj9mic2g/640?wx_fmt=png&from=appmsg "")  
  
  
项目地址:  
```
https://github.com/ChenChen753/Nuclei_Gui
```  
  
  
核心功能  
  
1. 仪表盘  
```
扫描统计概览（总扫描次数、发现漏洞数、POC 数量）
漏洞趋势图表
最近扫描历史
漏洞严重程度分布
```  
  
2. 漏洞扫描  
```
支持单目标/批量目标扫描
可选择多个 POC 模板
实时显示扫描进度和结果
支持暂停/恢复/取消扫描
扫描结果可导出为 CSV/HTML
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/x2ibBTFXYHicJc4HZicGPWwpdTHpyk1x762PGgIqMeSx0C2PcBUbMsUmXCurJoFQAicCBeczibgSgicOVn4dUPTbuBEAbfOwSJ54yDI0z1vHopJe0/640?wx_fmt=png&from=appmsg "")  
  
  
3. 任务队列管理  
```
多任务排队执行
任务优先级设置（紧急/高/普通/低/后台）
断点续扫支持
任务状态实时监控
显示创建时间、开始时间、耗时
```  
  
  
4. POC 管理  
```
POC 列表浏览和搜索
按严重程度/标签筛选
POC 导入（文件/目录）
POC 在线同步（从 nuclei-templates 官方仓库）
POC 编辑器（语法高亮）
POC 快速测试
POC 收藏管理
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/x2ibBTFXYHicLxLQ4ytVGrUonCjIfKOJXmflLwtASlmBQaSW1ibR0AWX3LiaXB78WSFaCVTnQg4gtFXp9wMM12mNialHw6Mv3GxDetFKAv8KVXK8/640?wx_fmt=png&from=appmsg "")  
  
  
5. 资产搜索  
  
支持多个资产搜索引擎：  
```
FOFA（国内主流资产搜索引擎）
Hunter（奇安信鹰图平台）
Quake（360 网络空间测绘）
Shodan（国际知名搜索引擎）
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/x2ibBTFXYHicIZIaQEWoQzfBUarVXZEGBAFuyWRibCBqY7ZAIwOibjtNguDdBTIfmjSUFfo5U5HuNjxiaA8SctReNCSctbuGIkgL0Z2BlB5eKjro/640?wx_fmt=png&from=appmsg "")  
  
  
6. AI 助手  
  
支持 OpenAI 兼容接口（DeepSeek、通义千问、GPT 等）：  
```
FOFA 语法生成
POC 生成
漏洞分析
智能推荐
漏洞报告生成
```  
  
  
7. 漏洞报告生成  
```
补天/SRC 提交报告格式
详细技术分析报告
简要漏洞说明
修复建议报告
```  
  
  
8. 远程更新  
```
 启动时自动检查更新
 手动检查更新
 一键下载更新
 数据保留（扫描历史、自定义 POC、配置文件）
 更新日志显示
```  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ibCZqSDX9ugSGKJibovaia9YxcaLfMJib6eFcsfYatVNptgRDr3kqeFwpGYKFziaX9s7BBcG8prEJFW1g1EickibFyug/640?wx_fmt=png&from=appmsg "")  
  
如何上手使用  
  
  
  
快速开始  
  
  
环境要求  
  
- Python 3.8+  
  
- Windows 10/11 / macOS 10.14+ / Linux (Ubuntu 18.04+)  
  
  
安装依赖  
```
pip install -r requirements.txt
```  
  
  
安装 Nuclei 扫描引擎  
  
程序内置了多种下载方式，包括：  
```
主界面下载（点击"下载最新版本 Nuclei"按钮）
设置界面下载（在设置对话框中点击"下载 Nuclei"按钮）
命令行自动下载（ python download_nuclei_simple.py ）
手动安装（从 GitHub 下载适合您系统的版本）
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/x2ibBTFXYHicIxOQG1JqZBFO094FxsIdWtNkrQZ4nvS35bdxngQyl5cyxbyxPLXuc38LvpHduaDGmviaUS9hedRP4qdOP4OKyyqiae3GmK9HFGc/640?wx_fmt=png&from=appmsg "")  
  
  
启动程序  
  
- Windows ：   
  
python main.py 或双击 Run_Nuclei_GUI.bat  
  
  
- macOS/Linux ：   
  
python3 main.py  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/x2ibBTFXYHicLygUGQ0KiafKBezCgeVnF7vwZTkTibuErGObwY4sZiaou2wsV0VXobk9xWeCf3zg6p9GbOjCg1k5KWg2YxtbGmE44WN3fc3YC3Y8/640?wx_fmt=png&from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ibCZqSDX9ugSGKJibovaia9YxcaLfMJib6eRUtCzBCFbaMYy1c7utlweibCFXWsicmm9ebyvInBtdsD0QRlUDTdLib1g/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
