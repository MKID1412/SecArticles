#  t00ls.com 注册接口邀请码参数 SQL 注入导致邀请码校验绕过漏洞复盘  
原创 aqwzaa 万知安全
                    aqwzaa 万知安全  安全wz啊   2026-11-23 14:36  
  
前言:闲来无事刷到 0xshe 师傅安全社公众号发布的帖子[T00ls论坛 注册邀请码校验绕过复现](https://mp.weixin.qq.com/s?__biz=Mzg3NzYzODU5NQ==&mid=2247485942&idx=1&sn=805600ca12ecea3eefec2105fb4ccf3e&scene=21#wechat_redirect)  
。后续自己进行了复现，发现已经被官方修复；后又通过 SQL 注入绕过邀请码实现账号注册。该文章发布已取得论坛主授权，且漏洞均已修复。  
# 一个单引号，绕过 T00ls 的邀请码：注册接口 SQL 注入  
> **一句话总结**  
：注册接口把邀请码原样拼进了 SQL，一个单引号就能把查询打断；换成不带注释符的恒真表达式   
' OR '1'='1  
，邀请码校验直接失效，账号注册成功。  
  
## 一、背景：邀请码是唯一门槛  
  
T00ls 是邀请制社区，注册页只有一个硬门槛——邀请码。  
  
整个注册流程很朴素：  
- GET /register.php  
 打开注册表单；  
  
- 表单只有   
invitecode  
、  
username  
、  
password  
、  
password2  
、  
email  
，外加一道固定文字问答；  
  
- POST /register.html?regsubmit=yes  
 提交。  
  
页面里没有图形验证码，没有滑块，也没有人机验证组件。唯一的“防机器人”是一道固定问答——题干「T00ls 的宗旨是什么？（5 个汉字）」，答案就明明白白写在页面上：「低调求发展」。  
  
换句话说，  
**注册流程的全部安全边界，就是那串邀请码**  
。邀请码一旦可绕，注册就完全敞开。  
## 二、漏洞与影响  
  
漏洞出在邀请码查询这一步。接口把   
invitecode  
 直接拼接进 SQL，没有做参数化，于是出现了标准的三态：  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n44" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 240px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">邀请码输入</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n45" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 191.333px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">响应体</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n46" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 452px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">页面原文</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n48" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 240px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">空</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n49" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 191.333px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">11103 B</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n50" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 452px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">没有填写邀请码不能注册</span></code></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n52" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 240px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">&#39;</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n53" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 191.333px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">128 B</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n54" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 452px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">DB ERROR!</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 报错页</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n56" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 240px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">&#39;&#39;</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n57" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 191.333px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">11088 B</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n58" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 452px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">邀请码不正确</span></code></span></span></td></tr></tbody></table>  
一个单引号把 SQL 打断成数据库报错页，成对引号又恢复成正常业务提示。  
**一报一恢复**  
，这就不是随机故障，是引号闭合型注入。  
  
再往下走，用不含注释符的恒真表达式   
' OR '1'='1  
 改变 WHERE 的语义，邀请码查询恒真，校验通过，流程一路走到注册成功。  
  
影响主要有三点：  
1. **绕过社区唯一的准入控制**  
。任何人不需要邀请码就能注册会员账号，可被用于批量注册、养号、灌水，直接破坏邀请制的意义。  
  
1. **注册即登录**  
，且没有邮件激活环节，注册完成那一刻账号就已经可用。  
  
1. **根因是注入而非单纯逻辑缺陷**  
。理论上可以继续做布尔/报错盲注；本次测试严格遵守边界，只验证到“绕过校验”为止，没有读取任何一张表的数据。  
  
## 三、复现过程  
### 1 拉取注册页  
```
```  
  
返回约 13KB 的正常注册表单，几个关键字段：  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n78" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 342px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">字段</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n79" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 568px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">说明</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n81" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 342px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">formhash</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n82" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 568px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">游客态固定 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">412b9f6e</span></code></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n84" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 342px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">invitecode</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n85" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 568px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">邀请码，必填，</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">注入点</span></span></strong></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n87" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 342px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">tusisecqaaid</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n88" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 568px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">本次问答编号，0~9 随机</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n90" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 342px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">secanswer</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n91" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 568px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">问答题答案，填 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">低调求发展</span></code></span></span></td></tr></tbody></table>  
响应会下发一个   
UTH_secq  
 Cookie，后续提交带着它即可通过问答校验。  
### 2 负对照：空邀请码会被拦  
  
先确认“门是好的”。只留空   
invitecode  
，其余字段正常：  
```
```  
  
返回 200，响应体 11103 字节：  
```
```  
  
空串在业务层被正常拦住。这是基线。  
### 3 单引号：SQL 被打断  
  
只改一个字节，  
invitecode=  
 换成   
invitecode=%27  
：  
```
```  
  
返回 200，响应体   
**128 字节**  
：  
```
```  
### 4 成对引号：业务页恢复  
  
invitecode=%27%27  
，其余不变，返回 200，响应体   
**11088 字节**  
：  
```
```  
  
单引号 128 字节报错、成对引号 11088 字节正常业务页——报错和恢复成对出现，排除了“服务器随机报错”，注入确认成立。  
### 5 站点自带的过滤层：只挡注释符  
  
站点其实有一层自研的 SQL 安全检查。同样的位置换不同写法：  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n114" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 210.667px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">邀请码输入</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n115" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 124px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">响应体</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n116" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 548.667px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">结果</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n118" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 210.667px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">&#39; OR 1=1-- -</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n119" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 124px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">277 B</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n120" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 548.667px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">SQL 安全检查未通过, 操作已被拦截 SEC--4</span></code></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n122" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 210.667px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">&#39; OR 1=1#</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n123" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 124px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">277 B</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n124" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 548.667px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">同上</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n126" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 210.667px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">&#39; OR &#39;1&#39;=&#39;1</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n127" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 124px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">11088 B</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n128" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 548.667px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">正常业务页，</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未被拦截</span></span></strong></span></span></td></tr></tbody></table>  
结论很清楚：这层过滤只匹配   
--  
、  
#  
 这类注释符，对纯布尔恒真写法完全无效。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XwvIIOgh4ZGIdD9zq2ibmRBDM1vibtOVNG4bXYqGOricibKMdhloIRlz9jySlibJSmaZxzdLib4RqGUzLZ33JJuDsdNKLv48szcnUwfdXrZhNJazQ/640?wx_fmt=png&from=appmsg "")  
> 💡 这也说明一件事：关键字黑名单只能提高攻击成本，永远不能当唯一防线。  
  
### 6 恒真条件绕过邀请码校验  
  
把   
invitecode  
 换成   
' OR '1'='1  
：  
```
```  
  
拼接后的查询语义变成：  
```
```  
  
返回 200，响应体   
**11766 字节**  
：  
```
```  
  
页面顶部导航已经变成登录态：用户名、提醒、短信、退出一应俱全。注册完成即登录，没有邮件激活这一步。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XwvIIOgh4ZFJlYYPk7l9KjqicVloiad0c150BrhLYDzqWO2LFQ5FCF7IsZc4snfHyc6ansibZNVoQdy9HL0bU88rGXdxnLAmo3oyyFdaWPcSog/640?wx_fmt=png&from=appmsg "")  
  
### 7 二次登录，确认账号真实可用  
```
```  
  
登录页会在前端把口令做一次 md5，  
password  
 字段填 md5 值。返回 200，响应体   
**11741 字节**  
：  
```
```  
  
账号口令都正常——这是一个真实可用、能正常登录的会员账号。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XwvIIOgh4ZFrebibNHdS4BlNYhAibWuWofTkj7Tc2alfnpoJPQbjJeKIlhiaMmazZQKpR1my15SJwQJC8Xm7N85ia86cJf2UMFeWotgxGxjUh7w/640?wx_fmt=png&from=appmsg "")  
## 四、几个容易误判的点  
  
复现过程中，有几处差分很容易被误读，值得单独记一笔。  
  
**1. 128B 和 120B 是两种不同的错误**  
  
探测列名时，  
invitecode=x' OR used=0 AND '1'='1  
 返回的是   
**120 字节**  
的另一版报错文案（  
An error report has been dispatched to our administrator.  
）。同样是报错页，但文案不同：  
- 128B → SQL 语法被引号打断；  
  
- 120B → 语句能执行，但列名不存在。  
  
把同样位置换成   
status=0  
，返回的是 11088 字节的正常业务页，说明   
status  
 列存在。  
**报错文案的差异，本身就是列名探测的 oracle。**  
  
**2. “一次性”不代表修复**  
  
恒真条件命中的那条邀请码，在注册流程里会被标记为已使用。所以同一 payload 紧接着重放，会回到 11088 字节的「邀请码不正确」。  
  
注意：这里回的是  
**业务页，不是报错页**  
。说明 SQL 依然在正常执行，只是表里没有可用记录了——这恰恰反向证明了注入还在。  
  
**3. 差分与 Cookie 无关**  
  
步骤 2 和 3 的差分不依赖任何 Cookie：不带 Cookie 的全新会话，空邀请码仍是 11103B，单引号仍是 128B 报错。注入点本身不依赖登录态。  
  
**4. 全程只读，不碰数据**  
  
受测试边界约束，整场只验证到“绕过校验”，没有 union、没有 dump、没有读取任何业务数据。测试数据全部使用   
test_  
 前缀的自建账号与邮箱。  
## 五、根因：参数被拼进了 SQL  
  
从现象推断，后端大概率是这样取邀请码的：  
```
```  
  
参数原样拼进语句，单引号一闭合，后面的   
OR '1'='1  
 就成了 SQL 的一部分，WHERE 语义被改写，查询返回表里任意一条记录，校验自然通过。  
  
而那层自研的 SQL 过滤只匹配注释符和井号，  
' OR '1'='1  
 这种纯布尔写法不在它的字典里，直接被放行。  
## 六、修复建议  
  
核心只有一句：  
**参数化查询，并且只接受字符串**  
。  
```
```  
  
在此基础上补四层：  
1. **参数化查询**  
：  
invitecode  
 只作为绑定值，禁止出现在 SQL 字符串里；  
  
1. **输入白名单**  
：邀请码只允许固定字符集与长度（如   
^[A-Za-z0-9]{6,32}$  
），不匹配直接拒绝，不进入数据库；  
  
1. **类型校验**  
：只接受字符串标量，数组、对象直接拒绝——这一条同时能防住   
invitecode[]=  
 这类数组类型混淆；  
  
1. **过滤层不作唯一防线**  
：现有 SQL 安全检查可以保留，但它挡不住无注释的恒真表达式，真正的根因修复必须在参数化层面完成。  
  
回归时建议固定覆盖这五组：  
invitecode=  
 /   
'  
 /   
''  
 /   
' OR '1'='1  
 /   
invitecode[]=  
，应当返回统一拒绝，且不产生任何数据库写入。  
## 最后  
  
这次的问题不复杂，但很典型：  
- **“有过滤”不等于“安全”**  
。自研的 SQL 检查能挡住   
--  
，挡不住   
' OR '1'='1  
；  
  
- **参数化的价值就在这种地方**  
。只要参数进了 SQL 字符串，任何黑名单都只是时间问题；  
  
- **类型校验是最便宜的一层**  
。  
is_string  
 一行代码，同时解决数组绕过和对象注入。  
  
希望这篇复盘能给同样在做注册、登录、邀请码这类“入口参数”的师傅们提个醒：先看参数怎么进 SQL，再看过滤层，顺序不能反。  
> **声明**  
：本文仅用于技术交流与防御参考。所有测试均在站方修复前完成，仅使用自建   
test_  
前缀测试账号与邮箱，未读取、未修改、未删除任何既有业务数据；漏洞已于 2026-11-22 修复，请勿对任何未授权站点进行测试。  
  
  
