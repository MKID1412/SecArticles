#  一个客户端管理员，把 Keycloak 变成自己的跳板（9.1 分 RCE）  
原创 红队安全圈
                    红队安全圈  红队安全圈   2026-11-09 11:00  
  
30 秒看懂  
  
影响范围 ·   
上游 Keycloak（开启 authorization services）与 Red Hat Build of Keycloak 26.4.18 之前；攻击者需要一个带 manage-clients 的账号  
  
利用状态 ·   
未在野利用，无公开 PoC，利用链已在官方漏洞页写清  
  
补丁状态 ·   
已修  
仅 Red Hat Build 26.4.18，上游尚未发版；先收回 manage-clients 权限  
  
建客户端时留下的一个内部标记没清掉，有客户端管理权限的人就能借它上传 JS 策略，在 Keycloak 的 JVM 里执行，跨出自己所在的 Realm。  
  
## 01  事情是什么  
  
CVE-2026-120984，CVSS 3.1 打 9.1 分，CWE-94。位置在 keycloak-services 的客户端管理组件。  
  
建一个开了 authorization services 的客户端，请求里同时带上 authorizationSettings，服务端为了让这个新客户端有一个默认策略，会在当前会话上打一个内部标记，临时放行脚本类策略的创建。放行完，标记没被清掉。  
  
Keycloak 默认禁止上传 JS 策略，因为策略求值本身就是服务端执行代码。但在那个标记还没散的会话里，导入客户端时可以塞进一段内联 JavaScript 策略，服务端的检查直接跳过。  
  
策略不会在上传时执行。它待在那里，等到任意一次 UMA 授权请求触发求值，代码才在 Keycloak 的 JVM 进程里跑起来。  
  
## 02  为什么危险  
  
判断这条值不值得花时间，看两个边界。  
  
第一个是 Realm。多租户 Keycloak 的隔离全靠 Realm，A 业务的管理员碰不到 B 业务的用户和配置。这段 JS 在 JVM 里执行，Realm 的边界就没了。官方描述里写得很直白：可以读宿主机上的敏感数据、改其他 Realm 的配置，甚至把整个服务停掉。  
  
第二个是权限语义。manage-clients 在很多部署里不是管理员权限，而是业务权限。业务团队自己管 OIDC 客户端、配 redirect URI、加 scope，这套下放在 Keycloak 里太常见了。下放的意思是：一个被钓鱼拿到的运维账号，或者一个外包账号，就够触发这条链。  
  
请求形状大概是这样，按公开描述还原，不是能直接用的 PoC：  
  
POST /admin/realms/acme/clients  
  
{  
  
 "clientId": "acme-portal",  
  
 "authorizationServicesEnabled":  
  
   true,  
  
 "authorizationSettings": {  
  
  "policies": [{  
  
   "type": "js",  
  
   "code": "// 你的 JS"  
  
  }]  
  
 }  
  
}  
  
后面再发一次授权请求让它求值，剩下就是 JVM 里的常规操作：起进程，或者直接读文件拼进响应。  
  
对红队来说这条链的价值不在单点，在于它是个转换器：把应用层权限换成基础设施权限。SSO 是信任根，打穿 IdP 等于所有下游应用的登录态都能伪造，而且日志里看是合法签发。  
  
## 03  谁受影响  
  
· 上游 Keycloak 开了 authorization services 的部署。截至发稿上游没列修复版本，26.8.2 的发布说明里没有这条 CVE  
  
· Red Hat Build of Keycloak 26.4.18 之前的版本  
  
· 前置三条：目标 Realm 开了授权服务、攻击者有一个带 manage-clients 的账号、能触发一次授权请求  
  
· 风险最高的是把客户端管理权限下放给业务方、外包或多租户运营团队，且管理端和业务同网暴露  
  
## 04  怎么修  
  
1. 跑 Red Hat Build of Keycloak 的升到 26.4.18，目前唯一有官方修复的分支（RHSA-2026:96037 包，96038 OpenShift 镜像）  
  
2. 跑上游 Keycloak 的先收权限：manage-clients 收回到最小集合，业务账号一律摘掉，等上游出补丁再升  
  
3. 审计现存的 JS 策略，看有没有人已经塞过东西：  
  
GET /admin/realms/{realm}/clients  
  
  /{id}/authz/resource-server  
  
  /policy?type=js  
  
Authorization: Bearer <token>  
  
4. 不用的 Realm 直接关掉 authorization services，这条链的前置条件就先少一个  
  
5. 翻事件日志，找客户端创建请求里 authorizationSettings 带 policies 的记录，正常业务不这么建客户端  
  
版本号在管理台页脚或容器镜像 tag 上看，别信资产表里登记的版本。  
  
## 05  一句判断  
  
这条不用今晚爬起来修，但别按需要管理员权限这个理由给它的优先级打折。真正的分界线是有没有把客户端管理权限发出去过。发过就是 P1，因为这个权限的语义已经从改配置变成了跑代码。没发过、Realm 也没开授权服务，那你属于不用管的多数。  
  
## 06  POC 链接  
  
https://access.redhat.com/security/cve/CVE-2026-120984  
  
https://bugzilla.redhat.com/show_bug.cgi?id=2565599  
  
https://access.redhat.com/errata/RHSA-2026:96037  
  
目前没有公开 PoC。上面三条是官方漏洞页、Bugzilla 技术描述和修复公告，技术细节与修复版本以这里为准。  
  
  
下期继续蹲厂商补丁和 PoC 动静，**关注**  
后漏洞先知道，修复不踩坑。  
  
— END —  
  
  
