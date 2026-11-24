#  泛微 OA SQL 注入漏洞如何研判？从一条 HTTP 请求识别数据库攻击  
原创 小新
                    小新  小新的安全运营笔记   2026-11-24 00:00  
  
## 一、一个普通的 GET 请求，为什么可能导致数据库泄露？  
  
假设某天，安全设备突然告警：一台泛微 OA 服务器收到来自外部 IP 的异常 GET 请求。乍一看，这只是一个普通的页面访问：  
```
GET /pweb/careerapply/HrmCareerApplyPerView.jsp?id=1 HTTP/1.1
```  
  
但仔细检查后发现，id  
 参数后面出现了 UNION SELECT  
、db_name()  
 等 SQL 语句。这就不再是普通的页面访问，而可能是一次针对数据库的 SQL 注入攻击。  
  
本文分析的就是泛微 OA E-Cology 中 HrmCareerApplyPerView.jsp  
 页面存在的 SQL 注入漏洞，以及如何通过 HTTP 流量识别攻击行为。  
## 二、漏洞原理：一个 id 参数引发的 SQL 注入  
  
该漏洞影响泛微 OA E-Cology v8.0，涉及接口：  
```
/pweb/careerapply/HrmCareerApplyPerView.jsp
```  
  
问题出在接口的 id  
 参数。正常情况下，系统应该将用户传入的 id  
 当作普通数据，查询对应的招聘申请信息。但如果程序没有正确过滤输入，而是直接将参数拼接进 SQL 查询语句，攻击者就可能改变原本的 SQL 语句逻辑。这就是 SQL 注入。  
  
攻击者可以构造 UNION SELECT  
 语句，将额外的数据库查询结果拼接到原有查询结果中，从而尝试获取数据库版本、数据库名称等信息。更值得注意的是，根据漏洞资料，**该接口无需登录即可访问**  
，攻击者可能直接从外部发起探测。  
## 三、攻击流量有什么特征？  
  
这类漏洞的流量特征比较明显，主要集中在请求路径和 id  
 参数中。  
### 1. 请求路径固定  
  
重点关注：  
```
/pweb/careerapply/HrmCareerApplyPerView.jsp
```  
  
请求方法通常为 GET，请求参数为 id  
。  
> 注意：路径大小写可能存在变化，实际检索时应考虑大小写差异。  
  
### 2. SQL 注入关键字  
  
攻击者可能在 id  
 参数中拼接 SQL 语句，例如：  
```
GET /pweb/careerapply/HrmCareerApplyPerView.jsp?id=1+union+select+1,2,@@version,4,5,6,7 HTTP/1.1
```  
  
这里有几个值得关注的特征：  
<table><thead><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><th style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;text-align: center;font-weight: bold;color: rgb(72, 112, 172);background: rgb(247, 247, 247);scrollbar-width: thin !important;"><section><span leaf="">特征</span></section></th><th style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;text-align: center;font-weight: bold;color: rgb(72, 112, 172);background: rgb(247, 247, 247);scrollbar-width: thin !important;"><section><span leaf="">作用</span></section></th></tr></thead><tbody><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><code style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><span leaf="">union select</span></code></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">尝试联合查询</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><code style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><span leaf="">@@version</span></code></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">获取数据库版本</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><code style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><span leaf="">db_name()</span></code></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">获取数据库名称</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><code style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><span leaf="">HashBytes()</span></code></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">计算指定数据的哈希值</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><code style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><span leaf="">sys.fn_sqlvarbasetostr()</span></code></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">将二进制数据转换为字符串形式</span></section></td></tr></tbody></table>  
其中，UNION SELECT  
 是非常值得关注的关键字组合。  
### 3. 为什么经常出现 7 个字段？  
  
在这类攻击样本中，经常能看到类似：  
```
union select 1,2,3,4,5,6,7
```  
  
这是因为 UNION 联合查询要求两边查询结果的列数和数据类型能够匹配。攻击者会尝试填充字段，寻找能够成功执行的查询结构。因此，流量中反复出现固定数量的字段，也可以作为辅助识别特征。  
## 四、如何判断攻击是否成功？  
  
发现恶意请求，只能说明有人尝试攻击，并不能直接证明漏洞利用成功。还需要结合服务器响应进行判断。  
  
例如，攻击者可能发送以下 Payload：  
```
1 union select 1,2,sys.fn_sqlvarbasetostr(HashBytes('MD5','abc')),db_name(1),5,6,7
```  
  
这里做了两件事：使用 HashBytes('MD5','abc')  
 计算字符串 abc  
 的 MD5 值；使用 db_name(1)  
 尝试获取数据库名称。  
  
如果响应页面中出现：  
```
900150983cd24fb0d6963f7d28e17f72master
```  
  
就值得高度关注。其中 900150983cd24fb0d6963f7d28e17f72  
 是字符串 abc  
 的 MD5 值，master  
 是 SQL Server 中常见的系统数据库名称。如果请求中的测试内容与响应结果能够对应，就说明**数据库查询结果可能已经通过页面回显**  
。  
> 但如果只看到 HTTP 200，不能认定攻击成功。还需要检查响应体中是否真正出现了预期的数据。  
  
## 五、安全人员如何进行流量研判？  
  
实际工作中，可以按照以下步骤排查。  
### 第一步：定位可疑请求  
  
在 Wireshark 中可以先使用：  
```
http.request.method == "GET"
```  
  
然后重点检查请求 URI 是否包含：  
```
HrmCareerApplyPerView.jsp
```  
  
如果需要进一步筛选，可以关注：  
```
http.request.uri contains "HrmCareerApplyPerView.jsp"
```  
### 第二步：检查 id 参数  
  
重点搜索以下关键词：  
```
unionselectdb_name@@versionHashBytessys.fn_sqlvarbasetostr
```  
> 注意，攻击者可能使用 URL 编码、大小写变化或其他方式绕过简单匹配，因此需要结合解码后的参数内容判断。  
  
### 第三步：检查响应内容  
  
找到对应的 HTTP 响应，重点检查：是否返回数据库版本信息；是否出现数据库名称；是否出现特定 MD5 测试值；是否返回 SQL 错误信息。如果出现与请求中查询内容对应的结果，说明攻击可能已经成功利用。  
### 第四步：关联其他攻击行为  
  
继续检查同一来源 IP 是否存在：  
- 高频访问该接口；  
  
- 对其他 OA 接口进行漏洞扫描；  
  
- 文件上传或其他漏洞利用行为；  
  
- 登录尝试及后续异常访问。  
  
这样才能更完整地判断攻击者是在进行漏洞探测，还是已经进入后续利用阶段。  
## 六、给安全人员的快速研判 Checklist  
<table><thead><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><th style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;text-align: center;font-weight: bold;color: rgb(72, 112, 172);background: rgb(247, 247, 247);scrollbar-width: thin !important;"><section><span leaf="">检查项</span></section></th><th style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;text-align: center;font-weight: bold;color: rgb(72, 112, 172);background: rgb(247, 247, 247);scrollbar-width: thin !important;"><section><span leaf="">研判重点</span></section></th></tr></thead><tbody><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">请求路径</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">是否访问 HrmCareerApplyPerView.jsp</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">请求参数</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><code style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><span leaf="">id</span></code><section><span leaf=""> 是否包含 SQL 注入语句</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">SQL 关键字</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">是否出现 UNION SELECT、db_name、@@version</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">数据库函数</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">是否出现 HashBytes 等 MSSQL 函数</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">响应状态</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">是否返回正常页面或异常错误</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">响应内容</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">是否出现数据库版本、库名或测试 MD5 值</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">来源 IP</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">是否存在扫描器或异常访问行为</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;scrollbar-width: thin !important;"><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">后续行为</span></section></td><td style="margin: 0px;padding: 9px 12px;box-sizing: border-box;border: 1px solid rgb(217, 223, 228);font-size: 0.75em;line-height: 22px;vertical-align: top;scrollbar-width: thin !important;"><section><span leaf="">是否出现其他漏洞利用或异常操作</span></section></td></tr></tbody></table>## 七、如何防御和修复？  
  
对于已经部署泛微 OA 的企业，建议重点做好以下工作：  
1. **升级修复**  
：按照厂商官方安全公告和补丁要求升级至已修复版本，不要仅依赖 WAF 拦截。  
  
1. **限制访问**  
：对非必要的前台接口限制公网访问，OA 系统尽量通过 VPN 或可信网络访问。  
  
1. **WAF 防护**  
：针对目标路径和 SQL 注入特征配置防护规则。  
  
1. **代码修复**  
：对 id  
 等用户输入进行严格校验，并使用参数化查询，避免 SQL 拼接。  
  
1. **日志排查**  
：检查历史访问日志，确认是否存在成功回显、异常数据库访问及后续攻击行为。  
  
## 八、总结  
  
泛微 OA 这类 SQL 注入漏洞，研判重点并不复杂：  
- **看路径**  
：是否访问 HrmCareerApplyPerView.jsp  
；  
  
- **看参数**  
：id  
 中是否出现 UNION SELECT  
 等 SQL 语句；  
  
- **看响应**  
：是否返回数据库版本、库名或其他查询结果。  
  
最后要记住：**发现攻击 Payload，不等于漏洞利用成功；HTTP "200"，也不等于数据库已经被攻破。**  
 真正可靠的研判，需要把请求内容、响应结果和服务器日志结合起来，才能判断攻击处于探测、利用还是已经造成实际影响的阶段。  
  
  
