#  Ingress-Nginx 漏洞允许攻击者执行任意代码  
原创 网络安全9527
                    网络安全9527  安全圈的那点事儿   2026-02-04 12:01  
  
流行的 Kubernetes 入口控制器ingress-nginx中发现了一个严重的安全漏洞，该漏洞可能允许经过身份验证的攻击者执行任意代码并访问敏感的集群密钥。  
  
该漏洞编号为 CVE-2026-24512，影响多个软件版本，需要管理员立即采取行动。  
  
Ingress 资源的 rules.http.paths.path 字段存在安全漏洞，攻击者可以利用该漏洞将恶意配置注入到 nginx Web 服务器中。  
  
此配置注入漏洞使攻击者能够在 ingress-nginx 控制器的上下文中执行任意代码。  
  
此外，攻击者还可以未经授权访问控制器有权读取的密钥。  
  
在默认安装中，ingress-nginx 控制器通常可以访问整个 Kubernetes 集群中的所有 Secret，从而大大增加潜在的影响。  
  
这表明该漏洞可以通过网络远程利用，攻击复杂度低，只需要低级权限，无需用户交互。  
## 受影响版本  
  
该漏洞会影响以下 ingress-nginx 版本：  
  
<table><thead style="box-sizing: border-box;border-bottom-width: 3px;border-bottom-style: solid;border-bottom-color: currentcolor;"><tr style="box-sizing: border-box;"><th class="has-text-align-left" data-align="left" style="box-sizing: border-box;padding: 2px 8px;text-align: left;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><strong style="box-sizing: border-box;font-weight: bold;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">产品</font></font></strong></th><th class="has-text-align-left" data-align="left" style="box-sizing: border-box;padding: 2px 8px;text-align: left;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><strong style="box-sizing: border-box;font-weight: bold;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">受影响版本</font></font></strong></th><th class="has-text-align-left" data-align="left" style="box-sizing: border-box;padding: 2px 8px;text-align: left;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><strong style="box-sizing: border-box;font-weight: bold;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">修复版本</font></font></strong></th></tr></thead><tbody style="box-sizing: border-box;"><tr style="box-sizing: border-box;background-color: rgb(240, 240, 240);"><td style="box-sizing: border-box;padding: 2px 8px;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">ingress-nginx</font></font></td><td style="box-sizing: border-box;padding: 2px 8px;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">所有版本 &lt; v1.13.7</font></font></td><td style="box-sizing: border-box;padding: 2px 8px;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">v1.13.7 或更高版本</font></font></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 2px 8px;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">ingress-nginx</font></font></td><td style="box-sizing: border-box;padding: 2px 8px;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">所有版本 &lt; v1.14.3</font></font></td><td style="box-sizing: border-box;padding: 2px 8px;border: 1px solid rgba(0, 0, 0, 0);word-break: break-word;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;"><font dir="auto" style="box-sizing: border-box;vertical-align: inherit;">v1.14.3 或更高版本</font></font></td></tr></tbody></table>  
使用 ingress-nginx 的组织必须立即采取措施保护其 Kubernetes 集群。  
  
Kubernetes安全响应委员会建议尽快升级到 ingress-nginx 版本 1.13.7、1.14.3 或任何更高版本。  
  
详细的升级说明请参阅官方的 Ingress-NGINX 升级文档。  
  
对于无法立即升级的环境，管理员可以通过部署验证准入控制器来实施临时缓解措施。  
  
应配置此控制器以拒绝任何使用 ImplementationSpecific 路径类型的 Ingress 资源，从而有效地阻止攻击向量，直到完成适当的升级。  
  
安全团队应监控其 Kubernetes 环境，以发现攻击迹象。Ingress资源的 rules.http.paths.path 字段中可疑或格式错误的数据可能表明存在攻击尝试。  
  
根据 Kubernetes安全公告，组织可以通过执行以下命令来验证他们是否正在运行易受攻击的版本：kubectl get pods --all-namespaces --selector app.kubernetes.io/name=ingress-nginx。  
  
如果发现利用漏洞的证据，管理员应立即联系 Kubernetes 安全团队，邮箱地址为 security@kubernetes.io。  
  
值得注意的是，正如 Kubernetes 项目宣布的那样，ingress-nginx 的维护即将停止，因此迁移到替代的入口解决方案是长期安全性的战略考虑。  
  
