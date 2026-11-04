#  CNNVD | 人工智能重要漏洞通报（2026年第十五期）  
 中国信息安全   2026-11-04 02:13  
  
[](https://cisat.cn/all/14915419?from_tag=1)  
  
**漏洞情况**  
  
根据国家人工智能安全漏洞库（AIVD）统计，近期（2026年9月29日至2026年10月30日）共采集重要人工智能漏洞363个，AIVD对这些漏洞进行了收录。本周人工智能类漏洞主要涵盖了DeepSeek、Ollama、MLflow等多个厂商（项目）。AIVD对其危害等级进行了评价，其中超危漏洞41个，高危漏洞182个，中危漏洞140个。  
  
  
## 一人工智能漏洞增长数量情况  
  
  
近期AIVD采集人工智能漏洞363个。     
  
![](https://mmbiz.qpic.cn/mmbiz_png/uOZw5Efn8etImAlTJ0hSxkbYJ6UxraPmJ407bee8IQH77LW52KugQxlRHeePAFLr71qI5msdt2XzICQraoFw2OH7BluZ3DiaTicvUkEbjU1fU/640?wx_fmt=png&watermark=1#imgIndex=3 "")  
  
图1 近五周漏洞新增数量统计图  
  
  
## 二人工智能漏洞具体情况  
  
  
近期共采集重要人工智能漏洞363个，包括DeepSeek、Ollama、MLflow等多个厂商（项目）的漏洞。其中超危漏洞41个，高危漏洞182个，中危漏洞140个。具体如表1所示：  
  
表1 人工智能漏洞列表  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uOZw5Efn8evicdgarnywakRTCfVvVFQxrmmr09y03KklzglZc2Cx2XXymOhicH0WtMs65pda88EXeerbF22m3MCJgFl9m662YJ1Q2RwfWyNDo/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4 "")  
  
  
## 三重要人工智能漏洞实例  
  
  
近期重要漏洞实例如表2所示。  
  
表2 本期重要漏洞实例  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uOZw5Efn8es2cLMurPvYibckRWR5Ac8aDMf0d62p6eH59eAwKF7q7Qdlic5sXu7JMibMvXotjlYvh6aeULN7d6V3P9Pd9fjxS2RRkCxXYEhaic8/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=5 "")  
  
1. DeepSeek 信息泄露漏洞（CNNVD-2026-41538350）  
  
DeepSeek是中国DeepSeek公司的一个大语言模型。  
  
DeepSeek V4存在信息泄露漏洞，该漏洞源于访问控制不当，攻击者利用该漏洞可以获取敏感信息。  
  
目前厂商已发布升级补丁以修复漏洞，参考链接：  
  
[https://www.deepseek.com/](https://www.deepseek.com/)  
  
  
2. sgl-project SGLang 资源管理错误漏洞（CNNVD-2026-55699613）  
  
sgl-project SGLang是sgl-project组织开源的一个用于加速大模型推理的编程语言与运行时系统。  
  
sgl-project SGLang 0.5.22及之前版本存在资源管理错误漏洞，该漏洞源于内存分配不当，攻击者利用该漏洞可以耗尽内存导致系统终止运行。  
  
目前厂商已发布升级补丁以修复漏洞，参考链接：  
  
[https://github.com/sgl-project/sglang/releases](https://github.com/sgl-project/sglang/releases)  
  
  
3. Mervin Praison PraisonAI 输入验证错误漏洞（CNNVD-2026-64890438）      
  
Mervin Praison PraisonAI是Mervin Praison个人开发者开源的一个低代码多智能体协作框架。  
  
Mervin Praison PraisonAI 1.6.62之前版本存在安全漏洞，该漏洞源于未对主机、端口或解析地址进行验证，攻击者利用该漏洞可以获取敏感信息。  
  
目前厂商已发布升级补丁以修复漏洞，参考链接：  
  
[https://github.com/MervinPraison/PraisonAI/security/advisories/GHSA-4pcv-mg8v-vrgf    ](https://github.com/MervinPraison/PraisonAI/security/advisories/GHSA-4pcv-mg8v-vrgf)  
  
  
(来源：CNNVD)  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LJwWAbW20CgzSibQGWAGCQZmiczKibuibyTaeLj1AG46SqCTPFicibJfMZ7LkvufAonoangXAhkKylWbCia6FomhaPCZb0Eiaf3iczjfkUx8ddic9KdDE/640?wx_fmt=png&from=appmsg "")  
[](https://cisat.cn/)  
  
  
