#  【一天一个nacos漏洞·第十六天】【高危】数据完整性破坏：注册中心实例元数据投毒  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-23 03:33  
  
   
  
# 【一天一个nacos漏洞·第十六天】【高危】数据完整性破坏：注册中心实例元数据投毒  
> "  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天讲**数据完整性**  
——不是"谁能读"，而是"**谁能让数据变成假的**  
"。  
  
Nacos 的服务注册表里，一个实例不只有 ip 和 port，还挂着一层**元数据**  
：权重、上下线状态、路由标签、心跳超时。  
  
这一层决定了**流量去哪、去多少、什么时候把它摘掉**  
。  
  
今天这两个洞说：**这层数据，谁都能写，写什么都算。**  
  
一个能给"**还没注册的实例**  
"提前下毒，等它上线时已经是个废的。  
  
一个把"**只改我给的字段**  
"实现成了"**只保留我给的字段**  
"——一句话之差，整张表被抹平。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址与账号口令均未写入本文。内容仅用于学习与防御加固。  
  
  
01  
  
一、元数据这一层，为什么值钱  
  
注册表里一条实例记录，最核心的是地址：  
  
```
192.0.2.10:8080

```  
  
  
但真正决定"它怎么被用"的，是挂在上面的一层东西：  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">字段</span></span></th><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">作用</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">weight</span></code></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">权重。负载均衡按它分流量，调低就慢慢饿死</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">enabled</span></code></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">上下线开关。false = 不接流量</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">metadata</span></code></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">自定义标签：版本、机房、灰度分组、归属团队</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">preserved.*</span></code></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">心跳间隔、不健康超时、IP 删除超时</span></span></td></tr></tbody></table>  
  
这一层有个专门的名字：**实例元数据（InstanceMetadata）**  
，按 ip:port:cluster  
 索引，走 Raft 持久化——**重启还在**  
。  
  
所以它的价值不在"能读"，在"**能改**  
"。改一条 weight，能让一台机器慢慢饿死；改一条 enabled，能让它从服务发现里消失。  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><section><span leaf=""><br/></span></section></th><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">VE-0546 · 全量更新</span></span></th><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">VE-0547 · 部分更新</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">7.1</span></strong></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">7.1</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">一句话</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">不校验实例是否存在，可给未注册坐标预置元数据</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">说好局部更新，实际整表替换</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">攻击时机</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">实例上线之前</span></strong></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">实例在线时</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">命令执行</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">否</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">否</span></span></td></tr></tbody></table>  
  
02  
  
二、VE-0546：给"还没出生"的实例下毒  
### 2.1 同一个文件里，三个方法两种待遇  
  
InstanceOperatorClientImpl  
 里有三个改实例的方法：  
  
```
removeInstance(...)   → clientManager.contains(clientId)，不存在直接 return
patchInstance(...)    → getInstance0(...)，找不到抛 RESOURCE_NOT_FOUND
updateInstance(...)   → 只校验 service 存在 ❗

```  
  
  
前两个都在问"**这个实例真的在吗**  
"，第三个不问。  
  
看它实际做了什么：  
  
```
public void updateInstance(String namespaceId, String groupName, String serviceName,                           Instance instance) {
    NamingUtils.checkInstanceIsLegal(instance);              // 只校验格式

    Service service = Service.newService(...);
    if (!ServiceManager.getInstance().containSingleton(service)) {  // 只要求 service 存在
        throw ... "service not found";
    }
    String metadataId = InstancePublishInfo.genMetadataId(
            instance.getIp(), instance.getPort(), instance.getClusterName());   // ip:port:cluster
    metadataOperateService.updateInstanceMetadata(service, metadataId, buildMetadata(instance));  // 直写
}

```  
  
  
**它校验的是"服务存不存在"，不是"实例存不存在"。**  
  
而同一个服务下可以有任何地址——包括一个**从未注册过、也不存在**  
的地址。  
### 2.2 于是可以给"未来"下毒  
  
```
① 挑一个还不存在的坐标：192.0.2.12:8848:DEFAULT
② PUT 预置元数据：weight=0, enabled=false, metadata={team:xxx}
   → 200 ok
③ 过一会儿，真实客户端以同一坐标注册，上报 weight=100, enabled=true
④ 读列表：
   weight  = 0.0        ← 被预置值覆盖
   enabled = false      ← 被预置值覆盖
   metadata = {owner:real, team:xxx}   ← 真实的和伪造的混在一起

```  
  
  
**真实客户端上报的 weight=100 / enabled=true 完全没用。**  
 它注册成功的那一刻，就已经是一个"权重 0、已下线"的实例。  
  
这个思路很反直觉：**你不需要攻击那台机器，你只需要在它上线之前，先替它写好"我是个不可用的实例"。**  
### 2.3 最狠的是心跳参数  
  
元数据里有一组前缀特殊的键：  
  
```
preserved.heart.beat.interval    心跳间隔
preserved.heart.beat.timeout     多久没心跳算不健康
preserved.ip.delete.timeout      不健康多久后删除

```  
  
  
这些键**本来由服务端根据配置算出来**  
，但通过这条路径可以覆盖：  
  
```
PUT /v3/console/ns/instance
metadata=team=x,preserved.heart.beat.interval=1000
        ,preserved.heart.beat.timeout=1000
        ,preserved.ip.delete.timeout=1000

```  
  
  
实测结果（临时实例）：  
  
```
t0      注册，元数据已带上 1000/1000/1000
t0+8s   实例从列表消失
日志：  INFO [AUTO-DELETE-IP] ... 192.0.2.21:8848

```  
  
  
**8 秒，实例被服务端自己删掉了。**  
 而且它可以重复——客户端重连、再删一次。  
### 2.4 还有一点：它是持久的  
  
这条路径写的是 CP 元数据（Raft），不是内存缓存。  
  
报告里特意指出：**从未注册过的地址不会进入过期清理集合**  
，所以这些伪造的元数据会跟着 Raft 快照一直留着，**重启也在**  
。  
  
你可以批量制造一堆"未来会中毒"的坐标，等它们自己上线。  
  
03  
  
三、VE-0547：说好只改一个字段，它把整张表换了  
### 3.1 接口自己的承诺  
  
Nacos v3 有个"部分更新"接口：  
  
```
PUT /v3/admin/ns/instance/partial

```  
  
  
它的接口注释写得很清楚：  
> “Patch update will only update variables in requests, the others will keep original value.”  
  
  
规范文档也写了：  
> “Partial update only changes fields explicitly present in the request.”  
  
  
**“只改你给的，其它的保持原样”**  
 —— 这就是 partial 的全部意义。  
### 3.2 实现里的一个词差  
  
```
private void mergeMetadata(InstanceMetadata newMetadata, InstancePatchObject patchObject) {
    if (null != patchObject.getMetadata()) {
        newMetadata.setExtendData(new HashMap<>(patchObject.getMetadata()));   // ← 整表替换
    }
    ...
}

```  
  
  
**setExtendData(new HashMap<>(...)) —— 把整张表换成请求里这一份。**  
  
而它应该是 putAll  
 —— 按键合并。  
  
```
契约："只改你给的，其它保留"      （putAll）
实现："只保留你给的，其它全没了"  （setExtendData）

```  
  
  
**同一个意思，两种实现，方向完全相反。**  
### 3.3 默认值把它放大  
  
更糟的是 controller 这一层：  
  
```
instanceForm.validate();    // → fillDefaultValue()：weight=1.0, enabled=true

```  
  
  
**只要没传 weight / enabled，就自动补成 1.0 和 true。**  
  
于是：  
  
```
请求：partial 只传 metadata={"x":"1"}
→ validate 补上 weight=1.0, enabled=true
→ 一并写进 patchObject

```  
  
  
实测（基线是 weight=2.5、enabled=false）：  
  
```
改之前：weight=2.5   enabled=false   metadata={owner:team-payments, zone:zone-a}
改之后：weight=1.0   enabled=true    metadata={x:"1"}

```  
  
  
三个后果一起发生：  
1. **owner / zone 没了**  
（整表替换）  
1. **weight 从 2.5 回到默认 1.0**  
（默认值填充）  
1. **enabled 从 false 变 true**  
——**一个已经被运维下线的实例，被静默重新上线了**  
第 3 条最危险：运维点了"下线"，本以为流量不会再来到这台机器；结果一次"只改一个标签"的调用，把它悄悄放回流量池。  
### 3.4 顺带绕过了一道校验  
  
全量 update 会调用 NamingUtils.checkInstanceIsLegal  
，它会校验心跳参数之间的关系：  
  
```
全量 update  metadata={"preserved.heart.beat.timeout":"1"}
→ 21004 instance error
  "Instance 'heart beat interval' must less than 'heart beat timeout' and 'ip delete timeout'."

partial 同一份 payload
→ 200 {"code":0,"data":"ok"}

```  
  
  
**同一个值，一个入口拒绝，另一个入口放行**  
——因为 patchInstance  
 压根没调那个校验函数。  
### 3.5 然后可以精确操纵"谁活着"  
  
注入不同的 preserved.heart.beat.timeout  
，健康判定的行为就变了。三实例同 t0 对比：  
  
```
       注入 1ms      对照（默认 15s）   注入 1h
t≈0s   healthy       healthy          healthy
t≈6s   UNHEALTHY     healthy          healthy     ← 1ms：6 秒就死
t≈14s  UNHEALTHY     UNHEALTHY        healthy     ← 对照正常下线
t≈18s  UNHEALTHY     UNHEALTHY        healthy     ← 1h：死了也"活着"

```  
  
  
**注入极小值 → 好实例 6 秒被摘掉；注入极大值 → 已经死掉的实例永远"健康"，流量继续打过去。**  
  
一头是拒绝服务，一头是把流量引向坟墓。  
  
04  
  
四、两个合起来  
  
```
VE-0546：给还不存在的实例预置元数据   →  攻击"未来"
VE-0547：把已存在的元数据整表覆盖     →  攻击"现在"

```  
  
  
一个是时间上的提前量，一个是语义上的反着做。但它们的根子是同一个：  
  
**在这套实现里，元数据只是"一堆挂在 ip:port:cluster 键上的值"，没有任何"这条记录属于谁、什么时候该允许谁改"的概念。**  
> 📌 顺带说一句：这已经是这个系列里**第二次**  
遇到"地址被当成键"了。 第十一天是 clientId = ip:port#ephemeral  
（用于认身份），今天是 metadataId = ip:port:cluster  
（用于定位数据）。**把地址当标识，本身没错；错的是把这个标识当成了"身份"或"归属"。**  
  
  
05  
  
五、边界  
  
两个都需要**对目标 service 有写权限**  
（未登录 403、只读 403）。跨 namespace 会被拦。  
  
所以它们不是"零权限打天下"，而是：  
> **“能往这个服务里写实例的人，也能改这个服务里任何实例的状态。”**  
  
  
而"服务写权限"在微服务团队里通常给得很宽——很多部署直接是 public:*:*  
。  
  
**在同一个服务里，"我的实例"和"你的实例"没有区别。**  
  
06  
  
六、修复  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">updateInstance 不校验存在性</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">与 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">patchInstance</span></code><span style="display: inline;"><span leaf=""> 对齐：先 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">getInstance0</span></code><span style="display: inline;"><span leaf=""> / </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">clientManager.contains</span></code><span style="display: inline;"><span leaf="">，不存在就 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">RESOURCE_NOT_FOUND</span></code></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">可给未注册坐标预置</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">拒绝为&#34;当前不存在&#34;的 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">ip:port:cluster</span></code><span style="display: inline;"><span leaf=""> 写元数据；或把元数据生命周期绑定到实例生命周期</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">partial 整表替换</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">mergeMetadata</span></code><span style="display: inline;"><span leaf=""> 改成 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">putAll</span></code><span style="display: inline;"><span leaf=""> 按键合并，别用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">setExtendData(new HashMap&lt;&gt;(...))</span></code></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">默认值放大</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">controller 只在请求</span></span><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">显式提供</span></strong><span style="display: inline;"><span leaf=""> weight/enabled 时才放进 patch；不要依赖 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">validate()</span></code><span style="display: inline;"><span leaf=""> 的 fillDefaultValue</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">patch 绕过校验</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">patchInstance</span></code><span style="display: inline;"><span leaf=""> 补 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">checkInstanceIsLegal</span></code><span style="display: inline;"><span leaf="">（心跳关系、权重范围、cluster 名）</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">保留键可注入</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">preserved.*</span></code><span style="display: inline;"><span leaf=""> 设白名单/只读，禁止通过管理面 metadata 覆盖；要改就做专用接口 + 值域校验</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">元数据不清理</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">给长期无对应真实实例的 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">InstanceMetadata</span></code><span style="display: inline;"><span leaf=""> 加过期清理，避免快照膨胀</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">单测固化了错误行为</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">InstanceOperatorClientImplTest</span></code><span style="display: inline;"><span leaf=""> 现在断言 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">extendData.size()==1</span></code><span style="display: inline;"><span leaf="">，等于把&#34;替换&#34;锁死了；改成断言&#34;未提供的键保留&#34;</span></span></td></tr></tbody></table>  
> 💡 一句可以写进接口设计规范的话：**partial 这个名字是一种承诺——它承诺"其它字段我不碰"。**  
如果实现做不到，就不该叫 partial；叫了，就得有人为那句注释负责。  
  
  
07  
  
写在最后  
  
第十六天，第一次专门讲**完整性**  
。  
  
前面十五天的问题，大多能归到"谁能进"（认证、越权）；今天的两个，问的是另一件事：  
  
**“进来的这个人，能不能把数据改成我看不出来的样子。”**  
  
而这一次的答案尤其让人不舒服，因为：  
- **它不需要攻破任何东西**  
——用的都是正常接口、正常权限、返回 200 ok  
；  
- **它改的东西看起来很正常**  
——weight 从 2.5 变 1.0，谁会注意？enabled 从 false 变 true，看着像是"有人重新上线了"；  
- **它的效果延迟出现**  
——0546 是"等实例上线才生效"，0547 是"1-3 秒后缓存刷新才可见"。  
**没有报错、没有异常日志、没有告警。**  
 你只会看到"某台机器的流量莫名其妙变少了"，然后去查网络、查负载均衡、查应用——**唯独不会想到去查注册表里的一个字段**  
。  
  
所以这一篇想留的最后一句是：  
> **服务注册表里存的不是"地址列表"，是运维对"这台机器现在能不能接流量"的承诺。**  
这个承诺值不值钱，取决于**除了它自己，还有谁能改**  
。  
  
  
下一天见。  
> 📚 **系列回顾 · 【一天一个nacos漏洞】**  
> **第一天**  
：客户端快照路径穿越，一路退到 /etc/cron.d  
。**第二天**  
：MCP 与 Skill，鉴权看 A、干活看 B。**第三天**  
：MCP 与 A2A，名字和版本拼成的 key 不唯一。**第四天**  
：配置接口别名错配（9.9），Prompt/Skill 分隔符碰撞（9.1）。**第五天**  
：配置接口认错参数（9.6），AI 资源可见性被 Config 通道绕过（9.6）。**第六天**  
：MySQL 健康检查 SQL 注入（7.7），鉴权密钥进日志（7.8）。**第七天**  
：OIDC 信任链两端——不验来源（8.1），不认撤销（7.4）。**第八天**  
：两处 fail-open——插件缺失即全放行（7.2），空白主体即成管理员（8.1）。**第九天**  
：服务间共享身份——凭据被主动投递（7.4），静态值可重放（8.1）。**第十天**  
：发布审核被架空——审核后换内容（8.1），审核漏扫文件（7.7）。**第十一天**  
：注册表身份由 ip:port 推导——可覆盖注册（7.1），可删除实例（7.1）。**第十二天**  
：集群内部基础设施——锁接口无鉴权（8.2），同步接口退让鉴权（7.3）。**第十三天**  
：版本不可变性被破坏——已发布的能改回（7.1），未审核的能推上（7.7）。**第十四天**  
：凭证撤销失效——删号后令牌仍有效（8.1），改密不验旧口令（7.6）。**第十五天**  
：LDAP 接入边界——空口令匿名绑定（7.4），默认凭据是 password（7.4）。**第十六天**  
（本篇）：实例元数据投毒——未注册坐标可预置（7.1），partial 整表覆盖（7.1）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
