#  多 Agent 协作扫描 - AI代码审计与漏洞挖掘平台  
youki
                    youki  C4安全   2026-11-23 01:05  
  
工具介绍  
  
**ZeroHunter**  
 是一个面向代码安全审计的 AI 辅助漏洞挖掘平台。  
  
它通过多个专用 Agent 协同分析项目目录、调用链、跨请求数据流和潜在漏洞，并提供一个本地 Web 控制台，用于管理项目、运行扫描、查看审计结果和维护规则。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CORQ8wxZunWe0XFHv1xMDL8nTqWAlPdsCRwH93mooWs3n4Viah2a8yX5c3aK5V2p4guukiatXWx6ibJQ1dDp6iaILLUXLlnKKx6pQ8/640?wx_fmt=png&from=appmsg "")  
  
核心能力  
  
多 Agent 协同扫描：**TreeScan、CallScan、DataFlowScan**  
 和 Auditor 分阶段完成代码审计。  
  
项目与版本管理：按来源、所有者、仓库和版本组织待审计代码。  
  
  
  
全仓库与局部扫描：支持完整项目扫描，也支持针对指定目录创建 specific run。  
  
  
  
漏洞审计与复核：查看漏洞详情、证据链、调用链、数据流和审计结论，并支持人工标注。  
  
  
  
可配置的扫描知识库：在控制台维护 Prompt、Skill 和 Sink 规则。  
  
  
  
LLM 配置切换：支持维护多个模型配置，按需要切换扫描模型和参数。  
  
  
  
大文件分页浏览：扫描产物、Markdown、JSONL、日志和对话记录按需加载，降低内存占用。  
  
  
  
报告导出：支持编写和导出漏洞复现报告。  
  
  
控制台预览  
  
扫描工作台  
  
控制台提供项目列表、扫描流程、目录树、Agent 产物和审计结果等统一入口。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/niasx7fyic9CMibqYfXcWVcUxITDFic2psmypRb8Sbx2hoUP7h5eI7l34hWGRibWEuUdtxySvKefbR5S8TFFpWFlE9qomVNK3qYd1W1gOpTOWQOg/640?wx_fmt=png&from=appmsg "")  
  
项目信息与版本项目支持自定义名称、描述、标签、审计状态以及多个代码版本。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/niasx7fyic9CN2DXwUiayIEqGCRSbicFKwtibfjVZbBp74ndAkfuvfq1eeZAIlicI2spRSXt9sf1F7XAanXzExojJu1hKQ8o1CSPsWLJDCviajDuDk/640?wx_fmt=png&from=appmsg "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CN1sPSVicjbJmP5WfyDE1FvR6c7v2IsyJqFq5O8hSLSbFKaLylEicHUKwhibSLGW0ibNMRQhwH7HGMssu3lHR9WWoQNEENeuREfZ50/640?wx_fmt=png&from=appmsg "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CMmLaXES1x4BtplwOFwKibwPiaTLQ68rPMFutfwxGPuafqfg5H2QyNEhJ2TzWVFALA1Bm3JPmNhgiaTDWlqwUusc9hNKRFIESXfp4/640?wx_fmt=png&from=appmsg "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/niasx7fyic9COwib2hOVrhdSmf44h5dM8kSaRnicDvB1lib3iaTZRSUF1t2pick7cib4lDr7qTSJdLPXKv3hVDPwSWBX30VFuZZZNPZ89IibyUzia2bUA/640?wx_fmt=png&from=appmsg "")  
  
环境要求  
  
  
```
Python 3.10+
Node.js 18+
npm
可访问的 OpenAI 兼容或项目支持的 LLM 服务（运行实际 AI 扫描时需要）
```  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/niasx7fyic9CMdYY8DY2sz8kH1Ffgn71YyF6Bp5qIPuFdttlCKvChQzvBUxFV0kJcSult0ia4r9BxQQ6gY8Q3eE8o7VPrtutsWjvEwxibduuHGc/640?wx_fmt=png&from=appmsg "")  
  
下载地址  
  
https://github.com/vs-limits/ZeroHunter  
  
  
内部社区已有770+师傅加入  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CMSHBvYMO76JqnXwWAE7uS87f4MXPBia5BXSKiapQevic1Yt2cTYxiaRMUOnicCoerjrwf5aGFYosuicQy6OsX2VS5nu5kib5qPWh0e1M/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1#imgIndex=18 "")  
  
加入价格已涨价至**79.9**  
，后续人数增加还会上涨价格！先到先得！  
  
**最新漏洞挖掘工具、AI工具第一时间分享！**  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CN5PDQib1SoicbAFDHz6cosm1A2zSQb7iavgzq13zgX1L3vPpoQHHib2licIjxovdaiaQ4EWebOAc1v0l5oKqpnkuknEO6ytXThbZLQ8/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1#imgIndex=19 "")  
  
  
**漏洞挖掘快人一步！**  
  
内容框架（持续新增中）  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CPwGujPBmYz9ibfgs7wRQ5S7Ik5qNukbibPa4uxhB9ksVnCia3m2fYM8J5IibY9IyfeKLmgYiadaHbIp5bJGdvdr7rKk2JPebcaSNUo/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1#imgIndex=20 "")  
  
**欢迎各位小伙伴加入了圈子，有意向的师傅们可以扫码加入我们，共同进步。**  
  
  
