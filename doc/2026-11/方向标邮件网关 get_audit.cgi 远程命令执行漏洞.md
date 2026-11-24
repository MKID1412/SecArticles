#  方向标邮件网关 get_audit.cgi 远程命令执行漏洞  
Superhero
                    Superhero  Nday Poc   2026-11-24 02:18  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Melo944GVOJECe5vg2C5YWgpyo1D5bCkYN4sZibCVo6EFo0N9b7Kib4I4N6j6Y10tynLOdgov9ibUmaNwW5yeoCbQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=8n1b48rw&tp=webp#imgIndex=0 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Melo944GVOJECe5vg2C5YWgpyo1D5bCkhic5lbbPcpxTLtLccZ04WhwDotW7g2b3zBgZeS5uvFH4dxf0tj0Rutw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=vx7xykg3&tp=webp#imgIndex=1 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Melo944GVOJECe5vg2C5YWgpyo1D5bCk524CiapZejYicic1Hf8LPt8qR893A3IP38J3NMmskDZjyqNkShewpibEfA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=x3261qcu&tp=webp#imgIndex=2 "")  
  
内容仅用于学习交流自查使用，由于传播、利用本公众号所提供的  
POC  
信息及  
POC对应脚本  
而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号Nday Poc及作者不为此承担任何责任，一旦造成后果请自行承担！  
  
  
**01**  
  
**漏洞概述**  
  
  
方向标邮件网关 get_audit.cgi存在远程命令执行漏洞，未经身份验证的恶意攻击者可以利用该漏洞远程执行任意命令，获取系统权限。  
**02******  
  
**搜索引擎**  
  
  
fofa:  
```
header="fm_chk_cookie_ok"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1iaCb48GgukY1xe6DVDtlwHdicIZkYjp02aQm6XefeAk5PhDFTkfiacpp6OnP9ZaImHjhga74EqZU7LYdluAs7jYjAOhMP4JwoZhY/640?wx_fmt=png&from=appmsg "")  
  
  
**03******  
  
**漏洞复现**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1iagEIdf7Pt200xCTvfW7VaxXRVZRRetKPibdAS2Yt8ablCMeQ5nCmCV920CzeWcEBQtKRZ1L5sXk8JDRZFw8QYYRZ2c8bQcicFuI/640?wx_fmt=png&from=appmsg "")  
  
  
**04**  
  
**自查工具**  
  
  
nuclei  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/8o9dnzNIF1gSsLL7MKrS1JvS8wr1Hp76xZeBaTKiazwqwQIIrbTVyMaNx0wxuIKlu0qnZmVLMUl7ric3frTQB4WiaNia5Iq1sgZ5rp61J6n3o3o/640?wx_fmt=png&from=appmsg "")  
  
afrog  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1ia7CthtVc9X9dxd1v4iaiaMAx551hOsFicaF2VvyCss4ichuB9rjgFB5sA7rlc2sCmpFlvoZ0RKaWCYdrG4ounvecMnXNbGlgfpDRY/640?wx_fmt=png&from=appmsg "")  
  
  
**05******  
  
**修复建议**  
  
  
1、关闭互联网暴露面或接口设置访问权限  
  
2、升级至安全版本  
  
  
**06******  
  
**内部圈子介绍**  
  
### 【Nday漏洞实战圈】🛠️  
  
专注公开1day/Nday漏洞复现 · 工具链适配支持  
  
✧━━━━━━━━━━━━━━━━✧  
  
🔍 **资源内容**  
  
▫️ 整合全网公开1day/Nday漏洞POC详情  
  
▫️ 适配Afrog/Nuclei检测脚本  
  
▫️ 支持内置与自定义POC目录混合扫描  
  
🔄 **更新计划**  
  
▫️ 每周新增7-10个实用POC（来源公开平台）  
  
▫️ 所有脚本经过基础测试，降低调试成本  
  
🎯 **适用场景**  
  
▫️ 企业漏洞自查 ▫️ 渗透测试 ▫️ 红蓝对抗 ▫️ 安全运维  
  
✧━━━━━━━━━━━━━━━━✧  
  
⚠️ **重要声明**  
  
▫️  
仅限合法授权测试，严禁违规使用  
  
**▫️虚拟资源服务，购买后不接受任何形式退款**  
  
▫️  
付款  
前请评估需求，慎重考虑  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1iaK51NNicAtyia2ZG1MDk6V30LaaBgqvgmdBRamJOhzrzwbp6icoLhaEbNrYt4fUibHwRSZTIsJjb2IsbBj12EXyVFib73CeXMAicddA/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
  
