#  Rancher Fleet 严重漏洞已在更新中修复  
sec随谈
                    sec随谈  sec随谈   2026-11-04 01:10  
  
**内容摘要**  
  
SUSE 于 2026 年 10 月 29 日发布了安全公告，涉及 GitOps 管理引擎 Rancher Fleet 中的两个严重漏洞。其中最危险的是   
CVE-2026-94290  
，它允许本地权限提升至 cluster-admin（集群管理员）。因此，系统管理员应立即安装最新的软件更新，防止集群被接管。  
  
**为何重要**  
  
Fleet 是一款面向多集群 Kubernetes 管理的开源 GitOps 引擎，可在大规模集群群组中自动化部署。据行业估计，有数千家企业组织通过 SUSE Rancher 使用 Fleet，各团队用它来大规模管理容器集群。因此，Fleet 控制平面中的弱点会给企业工作负载带来重大风险，攻击者可以扰乱部署、窃取敏感业务数据。  
  
好在 SUSE 确认这两个漏洞尚未在现实中被利用，研究人员也没有公开任何一个漏洞的概念验证（PoC）利用代码。  
  
**攻击原理**  
  
安全公告详细说明了代理管理（agent management）和集群注册（cluster registration）方面的两个不同漏洞。  
  
**代理管理控制器权限提升**  
  
首先，CVE-2026-94290 是 Fleet 代理管理控制器中的一个严重权限提升漏洞。  
GHSA-qfmw-gw96-ffcr 安全公告  
说明了其中的风险：“该控制器没有充分限制一个 Fleet Cluster 对象可以引用哪些 kubeconfig secret。”  
  
该控制器在管理集群上以 cluster-admin 权限运行。由于这个漏洞，普通用户可以定义一个引用管理员凭据的自定义集群（custom cluster），结果控制器便会以 root 级别的权限执行该用户的载荷。实际上，“一个仅持有默认 user 全局角色、在本地集群上完全没有任何角色绑定的已认证 Rancher 用户，就能在 Rancher 管理集群上获得完整的 cluster-admin 权限。”  
  
**通过注册令牌实现集群身份接管**  
  
与此同时，第二个漏洞涉及集群身份接管，编号为   
CVE-2026-104615  
。  
GHSA-gx8v-2929-9q95 安全公告  
警告，注册过程中存在授权不当的问题。报告指出：“注册请求所声称的集群身份被直接接受，而未与发起该请求所用的注册令牌进行绑定。”持有有效工作区（workspace）令牌的攻击者，可以冒认另一个集群的身份，进而访问 secret、扰乱管理服务。  
  
**受影响版本**  
  
这些 Rancher Fleet 漏洞影响低于 0.16.3、0.15.8、0.14.12 和 0.13.17 的独立版（standalone）Fleet，也影响低于 Rancher 2.15.3、2.14.7、2.13.11 和 2.12.15 版本中捆绑的 Fleet。  
  
**补丁与缓解措施**  
  
SUSE 已在最新版本中修复这两个问题。管理员应立即升级到 Fleet 0.16.3 或已打补丁的 Rancher 版本。此外，各团队应收回不受信任账户的工作区自助服务（workspace self-service）权限，并审查集群是否存在异常的角色绑定（role binding）。  
  
参考链接：  
  
https://github.com/rancher/fleet/security/advisories/GHSA-qfmw-gw96-ffcr  
  
https://github.com/rancher/fleet/security/advisories/GHSA-gx8v-2929-9q95  
  
  
