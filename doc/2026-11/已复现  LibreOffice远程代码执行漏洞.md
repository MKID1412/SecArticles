#  已复现 | LibreOffice远程代码执行漏洞  
原创 陆安予
                    陆安予  白帽子安全笔记2.0   2026-11-17 15:32  
  
   
  
## 一、这是什么  
  
11月17日，LibreOffice远程代码执行漏洞利用细节在外网公开，涉及2个漏洞的利用链（CVE-2026-104692与CVE-2026-104007），经过研究，完成复现。存在漏洞的版本≤26.2.6.x和或≤26.8.1，现版本为26.8.3，目标需具备java环境。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibXL9MCj2GqN9hwBe1WwvEvu75ZkRJ4elEicyDUP8rGOhp0UjESBesOHwYtEOGjq78eciczHZTxkCL3YzsMdSeVkOu64KCBJazwPSZAicyC6ibTI/640?wx_fmt=png&from=appmsg "null")  
  
## 二、漏洞复现  
  
本次使用版本为：  
  
https://downloadarchive.documentfoundation.org/libreoffice/old/26.2.6.3/win/x86_64/LibreOffice_26.2.6.3_Win_x86-64.msi  
### 1.编译载荷  
```
12cd .\evil\javac .\Driver.java
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibXL9MCj2GqM53RdLSXLe2GQtiaf16EezI1ibojcg2YImrsaaZvAODN7DowgGK0MyjjAp5BalQ6pRaQd7RNzfa5R7zLWD4MxZIIibAMKBUaVkEc/640?wx_fmt=png&from=appmsg "null")  
  
### 2.生成利用文件并监听  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibXL9MCj2GqNbJSc0rIIFcicTHVGKKttwxMnXgC8L4fLWdFb9ApgeOI0vBFMPFTCwfn9ny1wCaFTaia8tMCiaSDMpc4jaXOkjYnFia7W8WfrtlR8/640?wx_fmt=png&from=appmsg "null")  
  
### 3.打开特制的表格文档  
  
poc.obs  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ibXL9MCj2GqNQnQdXr2zJtJPOVahw7fibPgLNGVD1iam7LePbkhOHgYxjyl0KE19ckAFW26XHywqzvy4rxBJ6wrc89whyeL8vFHX1ebiacZic6ok/640?wx_fmt=gif&from=appmsg "null")  
  
  
注意文档需要在这些目录打开  
  
$SYSUSERHOME/../Downloads/poc.ods  
  
$SYSUSERHOME/../Desktop/poc.ods  
  
$SYSUSERHOME/poc.ods  
  
打开表格直接弹出计算器！  
## 三、POC获取  
  
回复20261117  
获取。  
## 四、使用场景  
1. 1. 在企业授权的攻防演习中，验证防守方的检测与响应能力  
  
1. 2. 理解各类加载技术的实现机制与运行过程  
  
1. 3. 在受控环境中测试安全设备的检出情况，用于优化检测策略  
  
## 五、免责声明  
  
本文涉及方案仅限合法授权的安全研究、渗透测试用途，使用者须确保符合《网络安全法》及相关法规。具体条款如下：  
1. 1. 仅可用于已获得书面授权的目标系统测试；  
  
1. 2. 遵守法律法规，不得用于侵犯他人隐私或数据窃取；  
  
1. 3. 本人不承担因用户滥用本软件导致的任何后果。  
  
使用即视为同意并接受上述条款。  
  
   
  
  
