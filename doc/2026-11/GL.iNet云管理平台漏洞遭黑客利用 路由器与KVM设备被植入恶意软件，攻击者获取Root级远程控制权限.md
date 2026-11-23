#  GL.iNet云管理平台漏洞遭黑客利用: 路由器与KVM设备被植入恶意软件，攻击者获取Root级远程控制权限  
tuto
                    tuto  杂杂咱谈   2026-11-23 03:14  
  
## 一、事件概述  
  
2026 年 11 月 22 日，BleepingComputer 披露，网络设备制造商 **GL.iNet**  
 遭遇一起针对其云端远程管理服务的安全攻击。攻击者利用 GL.iNet 云管理平台中的安全漏洞，未经授权访问用户设备，并向部分 GL.iNet 路由器和 KVM 设备植入定制恶意软件。  
  
GL.iNet 已于 11 月 20 日发布安全公告，确认攻击者利用其云端远程管理服务中的漏洞，对部分客户设备实施未授权访问和恶意软件部署。  
  
此次事件涉及的设备主要包括：  
- **GL.iNet 路由器：包括家庭路由器、便携式旅行路由器等网络设备。**  
- **GL.iNet KVM 设备：用于远程访问和控制计算机、服务器的硬件设备。**  
- **GoodCloud 云管理平台：GL.iNet 提供的设备远程管理服务，允许用户通过云端管理受支持的路由器和 KVM 设备。**  
攻击者不仅能够在受影响设备上执行恶意命令，还在部分设备中部署了持久化后门和 Root 权限反向 Shell，使其能够远程控制已感染设备。  
  
GL.iNet 表示，已经修复云端漏洞并阻断此次攻击使用的访问路径，但需要特别注意：**云端漏洞修复并不意味着此前已经感染的设备会自动清除恶意软件。**  
  
目前，GL.iNet 尚未公布漏洞的具体技术细节、受影响设备总量，以及攻击者是否从受感染设备中窃取了用户数据。对于连接到受感染 KVM 设备的计算机是否遭到进一步入侵，厂商仍在调查中。  
## 二、攻击事件技术分析  
### 2.1 攻击入口：GoodCloud 云端远程管理服务  
  
此次攻击的核心入口是 GL.iNet 的 GoodCloud 云端远程管理服务。  
  
GoodCloud 允许用户通过云平台远程管理已注册的 GL.iNet 设备，包括：  
- 远程管理路由器及网络配置。  
  
- 远程访问和控制 KVM 连接的计算机。  
  
- 管理设备运行状态及相关网络功能。  
  
- 通过云端向设备传递管理指令。  
  
攻击者利用云端远程管理服务中的漏洞，绕过正常的安全控制机制，未经授权向目标设备下发恶意操作指令。  
  
GL.iNet 已确认该云端服务漏洞遭到实际利用，但没有公开具体漏洞成因，也没有确认攻击者使用的具体利用代码。  
  
安全研究人员 RemoteToHome Consulting 在分析过程中发现，GL.iNet GoodCloud 客户端中可能存在一个与 AstroWarp/SD-WAN 功能相关的命令注入问题。  
  
研究人员指出，相关处理程序可能将来自云端的参数直接传递给具有 Root 权限的 Shell 命令执行，而没有进行充分的输入验证。  
  
这意味着，如果攻击者能够控制相关云端参数，就可能构造恶意命令，在设备上实现高权限命令执行。  
  
不过，**GL.iNet 尚未确认这一具体命令注入问题就是此次攻击所利用的漏洞**  
，因此不能将研究人员发现的潜在漏洞直接等同于本次事件的已确认攻击入口。  
### 2.2 恶意软件部署与持久化  
  
在成功访问目标设备后，攻击者部署了专门针对 GL.iNet 路由器和 KVM 设备设计的恶意脚本及可执行文件。  
  
BleepingComputer 分析了攻击者使用的  
恶意脚本 ni.sh  
，发现其包含针对多种 GL.iNet 设备型号的适配逻辑。  
  
主要恶意行为包括：  
1. **设备识别：识别不同 GL.iNet 路由器及 KVM 设备型号。**  
1. **恶意载荷下载：从攻击者控制的服务器下载名为 gl-helper 和 gb 的恶意可执行文件。**  
1. **持久化部署：修改系统启动脚本和计划任务，使恶意程序在设备重启后仍能自动运行。**  
1. **进程伪装：将恶意进程伪装成 crond -i /tmp/.log，试图混淆管理员的排查工作。**  
1. **网络通信隐藏：替换系统原有的 netstat工具，通过修改后的脚本过滤特定恶意服务器的网络连接记录。**  
1. **远程访问控制：向设备写入 SSH 公钥，为攻击者提供额外的远程访问途径。**  
1. **远程命令执行：定期连接远程服务器，获取并执行攻击者下发的命令。**  
1. **痕迹修改：修改文件时间戳，增加恶意文件与正常系统文件之间的辨别难度。**  
其中，  
替换 netstat 工具的行为尤其值得关注。  
  
攻击者修改  
后的 netstat 会过滤指向 66.245.219[.]252 的连接信息  
，从而使管理员在使用常规网络排查工具时，无法直接观察到部分恶意网络通信。  
  
这表明攻击者不仅试图获得设备控制权，还采取了针对系统排查工具的反取证措施。  
### 2.3 Root 权限反向 Shell  
  
除常规恶意软件植入外，研究人员还发现部分受感染路由器建立了 Root 权限反向 Shell。  
  
攻击者利用 Linux 系统中  
的 nc 工具  
建立设备向外部服务器发起的连接，使远程攻击者能够通过反向连接执行系统命令。  
  
已发现的反向 Shell 通信地址：  
```
170.39.226[.]219
```  
  
此类反向连接能够帮助攻击者绕过部分网络环境中的入站连接限制。  
  
一旦设备成功建立反向 Shell，攻击者就可能获得持续的远程命令执行能力，进一步实施：  
- 修改路由器系统配置。  
  
- 执行任意系统命令。  
  
- 下载和运行其他恶意程序。  
  
- 修改系统启动配置。  
  
- 建立额外的远程访问后门。  
  
- 将受感染设备作为后续攻击的基础设施。  
  
由于反向 Shell 运行在 Root 权限下，其影响范围远超普通用户权限下的恶意程序执行。  
## 三、恶意软件运行机制  
### 3.1 恶意脚本 ni.sh  
  
BleepingComputer 对攻击者使  
用的 ni.sh 脚本  
进行了分析，发现其专门针对 GL.iNet 设备进行了定制。  
  
**恶意软件主要执行流程：**  
```
攻击者控制 GoodCloud 云端访问路径                |                v利用云端远程管理服务漏洞                |                v未经授权访问 GL.iNet 设备(路由器 / KVM 设备)                |                v下载并执行 ni.sh                |                v识别设备型号                |                v下载 gl-helper、gb 等恶意载荷                |                v+----------------------+----------------------+|                      |                      |v                      v                      |建立持久化             隐藏恶意行为             ||                      |                      |启动脚本               进程伪装                 |计划任务               netstat 替换             |SSH 公钥               文件时间戳修改           ||                      |                      |+----------------------+----------------------+                |                v建立远程控制通道                |                vRoot 权限反向 Shell                |                v远程命令获取与执行
```  
  
该脚本还具有以下值得关注的特征：  
- 包含针对不同 GL.iNet 硬件型号的识别逻辑。  
  
- 支持下载多个恶意可执行文件。  
  
- 利用启动脚本和计划任务实现持久化。  
  
- 通过伪装进程降低被发现的概率。  
  
- 修改系统网络工具隐藏攻击者通信。  
  
- 通过 SSH 公钥实现额外的远程访问。  
  
此外，研究人员在恶意脚本中发现了中文开发者注释，涉及持久化机制、信息上报以及与 GL.iNet 云管理软件之间的交互。这表明脚本可能由能够使用中文的开发者编写，但不能据此确定攻击者的真实身份或所属组织。  
### 3.2 Lua 恶意载荷  
  
除 ni.sh 外  
，BleepingComputer 还分析了从一台 GL-MT2500 路由器中提取的恶意 Lua 载荷。  
  
该载荷通过被篡改的系统启动  
脚本 urandom_seed 启动  
。  
  
其主要功能包括：  
- 收集设备相关信息。  
  
- 与外部远程服务器建立通信。  
  
- 下载其他恶意程序。  
  
- 执行额外的恶意命令。  
  
- 借助系统启动流程维持恶意程序运行。  
  
这种通过修改合法系统启动文件实现持久化的方式，使恶意代码能够随着设备启动自动执行，也增加了普通用户通过简单检查发现恶意程序的难度。  
## 四、受感染设备及攻击规模  
  
RemoteToHome Consulting 对其拥有或管理的 183 台 GL.iNet 路由器进行了安全分析，发现其中 152 台存在可疑活动。  
  
具体统计如下：  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">统计项目</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">数量</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">分析设备总数</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">183 台</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">存在可疑活动</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">152 台</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">发现 Root 级反向 Shell</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">24 台</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">发现旧版持久化后门</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">47 台</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">发现攻击者命令执行痕迹</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">81 台</span></section></td></tr></tbody></table>  
研究人员还发现：  
- 24 台设备存在 Root 级反向 Shell。  
  
- 47 台设备存在旧版本持久化后门。  
  
- 81 台设备存在攻击者命令执行痕迹。  
  
需要注意，上述不同类型的异常可能发生在同一台设备上，不能简单相加计算受感染设备数量。  
  
另外，研究人员指出，部分旧版后门可能早于 2026 年 11 月此次攻击事件，并不一定与本次攻击活动存在直接关联。  
  
上述数据来自 RemoteToHome Consulting 自身管理的设备样本，并不代表 GL.iNet 全球客户的实际感染比例。  
## 五、攻击者基础设施与 IOC  
### 5.1 IP 地址  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">IOC</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">类型</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">相关行为</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">170.39.226[.]219</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">IP 地址</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Root 级反向 Shell 通信</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">66.245.219[.]252</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">IP 地址</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">恶意脚本及载荷下载、远程通信</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">65.20.71[.]131</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">IP 地址</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">恶意基础设施相关通信</span></section></td></tr></tbody></table>  
建议安全团队在防火墙、DNS、网络流量监控及 SIEM 平台中对上述地址进行检索。  
### 5.2 文件及进程 IOC  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">IOC</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">类型</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">描述</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">ni.sh</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">恶意脚本</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">针对 GL.iNet 设备的恶意安装脚本</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">gl-helper</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">可执行文件</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">恶意载荷</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">gb</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">可执行文件</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">恶意载荷</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">crond -i /tmp/.log</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">可疑进程</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">攻击者用于伪装的恶意进程名称</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">urandom_seed</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">启动文件</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">曾被发现遭到恶意修改</span></section></td></tr></tbody></table>  
其中，crond -i /tmp/.log 是   
GL.iNet 最初公布的入侵指标，但后续研究发现，部分感染设备可能使用其他后门或反向 Shell，因此不能仅依靠该进程判断设备是否安全。  
### 5.3 其他异常网络行为  
  
在一台被入侵的 GL-RM1 KVM 设备中，用户发现设备于 2026 年 11 月 19 日从以下地址下载恶意脚本和其他可执行文件：  
```
66.245.219[.]252
```  
  
随后，该设备出现了异常网络通信行为：  
- 每天与约 1800 个住宅 IP 地址建立通信。  
  
- 反复访问 TRON 区块链 API。  
  
- 表现出一定的 P2P 网络通信特征。  
  
目前尚不清楚这些通信的具体用途，也没有证据确认受感染设备已经被用于住宅代理服务或大型僵尸网络。  
## 六、受影响设备与固件版本  
  
此次事件涉及 GL.iNet 路由器和 KVM 设备。  
  
GL.iNet 官方建议所有用户检查设备安全状态，并及时安装针对具体型号发布的安全固件。  
  
根据 BleepingComputer 报道，相关 KVM 设备的建议固件版本如下：  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">KVM 设备型号</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">建议固件版本</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">RM10</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.10.3 或更高版本</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">RM1</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.10.3 或更高版本</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">RM1V2</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.10.3 或更高版本</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">RM1PE</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.10.3 或更高版本</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">RM4PE</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.10.3 或更高版本</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">RM10RC</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.10.3 Beta 或更高版本</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">RMQ1</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.8.7 或更高版本</span></section></td></tr></tbody></table>  
路由器用户应根据设备具体型号，安装 GL.iNet 官方提供的最新安全固件。  
  
**注意：**  
 固件升级不能代替恶意软件清除。对于已经感染的设备，仍需进行专门的检测、清理或重新安装固件。  
## 七、安全检测与修复建议  
  
GL.iNet 已修复云端攻击入口，但已经被植入恶意软件的设备仍可能保留后门。  
  
因此，建议用户按照以下步骤进行处理。  
### 7.1 检查设备是否存在异常进程  
  
首先检查设备是否存在 GL.iNet 公布的可疑进程：  
```
ps | grep 'crond -i /tmp/.log'
```  
  
同时检查是否存在异常网络连接：  
```
netstat -antp
```  
  
重点关注是否存在与以下 IP 地址通信的连接：  
```
170.39.226[.]21966.245.219[.]25265.20.71[.]131
```  
  
需要注意，攻击者可能已经替换系统原  
有的 netstat 工具  
，使其隐藏部分恶意连接。因此，不能完全依赖设备本地的网络命令进行检测。  
### 7.2 使用社区开发的检测与清理工具  
  
GL.iNet 已在官方支持说明中提供社区开发的恶意软件检测和清理工具。  
  
该工具能够检查：  
- 已知恶意文件。  
  
- 可疑运行进程。  
  
- 被修改的系统工具。  
  
- 被篡改的配置文件。  
  
- 异常持久化启动机制。  
  
- 可能存在的恶意网络通信。  
  
此外，该工具还支持收集取证信息，并在适当情况下清除已知恶意软件组件。  
  
但开发者明确提醒：该工具无法保证检测所有感染情况。  
  
对于已经被深度入侵的设备，可能需要重新安装固件，并且不能保留原有配置，以避免恶意文件通过配置目录继续存在。  
### 7.3 执行设备恢复  
  
针对不同设备，建议采取以下措施。  
  
**GL.iNet 路由器：**  
1. 在设备完全启动后，按住 Reset 按钮约 10 秒。  
  
1. 松开按钮，执行恢复出厂设置。  
  
1. 重新配置设备，不要直接恢复来源不明或可能已经感染的配置备份。  
  
1. 安装对应型号的最新安全固件。  
  
1. 重新检查设备运行状态和网络连接。  
  
**GL.iNet KVM 设备：**  
1. 设备通电状态下，按住 Reset 按钮约 10 秒。  
  
1. 重新配置 KVM 设备。  
  
1. 安装最新安全固件。  
  
1. 检查与 KVM 连接的计算机是否存在异常程序、浏览器扩展、未知用户账户和异常启动任务。  
  
1. 使用更新后的安全软件对连接计算机执行完整扫描。  
  
### 7.4 修改相关凭据  
  
如果设备存在被入侵的可能，应及时轮换可能暴露的凭据，包括：  
- 路由器管理账户密码。  
  
- GoodCloud 云管理账户密码。  
  
- SSH 登录凭据及公钥。  
  
- 连接计算机使用的账户密码。  
  
- 其他曾经保存在受感染设备上的访问密钥。  
  
如果发现连接 KVM 的计算机也存在异常，应进一步调查该计算机是否遭到未授权访问。  
## 八、安全风险分析  
  
此次事件反映出云端远程管理服务一旦出现安全问题，可能成为攻击者控制大量边缘网络设备的重要入口。  
  
与传统针对单台路由器的攻击相比，此次事件具有几个值得关注的特点。  
  
**1. 云端管理平台成为集中攻击入口**  
  
GoodCloud 为多种 GL.iNet 设备提供统一远程管理能力。一旦云端管理服务存在可被利用的漏洞，攻击者就可能通过统一管理通道影响多个设备。  
  
**2. 恶意软件具备较强的持久化能力**  
  
攻击者不仅部署恶意程序，还通过启动脚本、计划任务、SSH 公钥等方式建立多层持久化机制。  
  
普通重启操作无法保证清除恶意软件，常规固件升级也可能无法删除保存在配置目录中的恶意文件。  
  
**3. 利用系统工具替换隐藏恶意行为**  
  
攻击者  
修改 netstat 工具  
，隐藏特定恶意 IP 地址的连接记录。这种行为增加了管理员排查设备异常网络通信的难度。  
  
**4. KVM 设备可能成为进入内部计算机的跳板**  
  
KVM 设备本身具有远程控制计算机的能力。如果攻击者成功控制 KVM 设备，可能进一步影响与其连接的服务器和工作站。  
  
因此，KVM 设备遭到入侵后，不能只检查设备自身，还必须对连接计算机开展安全排查。  
## 九、总结  
  
此次 GL.iNet 安全事件并非简单的路由器配置被篡改，而是一起涉及云端远程管理服务、恶意软件植入、Root 级反向 Shell 和多层持久化机制的网络设备安全事件。  
  
攻击者利用 GoodCloud 云管理服务中的漏洞，对部分 GL.iNet 路由器及 KVM 设备实施未授权访问，并部署定制恶意脚本和恶意可执行文件。  
  
虽然 GL.iNet 已经修复云端漏洞并阻断相关攻击路径，但此前已经感染的设备仍可能保留后门，甚至在固件升级后继续存在。  
  
对于使用 GL.iNet 网络设备的个人用户、企业和网络运维人员，应尽快检查设备运行状态、核查异常网络连接、安装最新安全固件，并对疑似感染设备执行完整恢复。  
  
**尤其需要注意的是，KVM 设备一旦遭到入侵，其影响范围可能从网络设备本身扩展到连接的服务器和工作站。**  
  
****  
引用来源:   
bleepingcomputer  
  
