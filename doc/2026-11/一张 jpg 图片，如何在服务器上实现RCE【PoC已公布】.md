#  一张 jpg 图片，如何在服务器上实现RCE【PoC已公布】  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-11-23 01:33  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/  
  
**不想错过任何消息？设置星标****↓ ↓ ↓**  
#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlbXyV4tJfwXpicwdZ2gTB6XtwoqRvbaCy3UgU1Upgn094oibelRBGyMs5GgicFKNkW1f62QPCwGwKxA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZgN2tgRCcVG1ibjfKZicP46DqmzqpiciawtI5j0ib72yBL2ukv0eicbFQuQicNic3mfASoVDX9RVr92548SfAoLpf20hKwA5TR3ibqvh63w/640?wx_fmt=png&from=appmsg "")  
  
先说结论：一台服务器只是在为上传的图片生成缩略图，就可能已经执行了攻击者写的程序。  
  
这毫不夸张。安全公司 pwn.ai 的研究员 Paulos Yibelo 在 Ghostscript 的字体加载器里找到一个 use-after-free，编号 CVE-2026-125396。构造一份文档，就能让它在渲染过程中拿到"向任意地址写入任意数据"的能力，然后把 -dSAFER  
 沙盒的开关一个个拨到关闭，最后以渲染进程的身份执行命令 [1][2]。  
  
Ghostscript 10.08.0 已于 2026 年 9 月 8 日修复这个缺陷 [7]。CVE 在 11 月 9 日公开，CVSS 4.0 评分 8.5 [4]。截至公开时，暂时没有在野利用的报告 [1]。  
  
但这条链真正有意思的地方不在评分，而在于它是怎么被接回网络的：**攻击者不需要碰到 Ghostscript，甚至不需要应用调用 Ghostscript。**  
## 一条没人设计过的调用链  
  
Ghostscript 是 PostScript 和 PDF 的解释器。很多人以为它只出现在"打印"或"转 PDF"这类功能里，实际上它更常见的出场方式是：某个图像库在处理一份 EPS/PS 内容时，悄悄把活儿转包给了它。  
  
应用代码里写的是 Imagick::readImage()  
，是 Pillow 的 Image.open().load()  
，是 libvips 的 thumbnail()  
。这些调用背后，格式判断看的是**文件内容**  
，不是扩展名。文件叫 quarterly-results.jpg  
，内容却是 PostScript，图像库照样会把它交给 Ghostscript [2]。  
  
pwn.ai 在测试中记录的路径大致是这样 [2]：  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">常见调用</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">实际走到的路径</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">Pillow</span></code><section><span leaf=""> 的 </span><code><span leaf="">load()</span></code><span leaf=""> / </span><code><span leaf="">crop()</span></code></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">Pillow EPS 解码器 → Ghostscript</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">imageio.imread()</span></code><section><span leaf="">、</span><code><span leaf="">skimage.io.imread()</span></code></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">图像读取 → Pillow → Ghostscript</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">PHP </span><code><span leaf="">Imagick::readImage()</span></code></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">ImageMagick 的 PostScript delegate → Ghostscript</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">libvips</span></code><section><span leaf=""> 的 </span><code><span leaf="">thumbnail()</span></code></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">系统构建的 </span><code><span leaf="">magickload</span></code><span leaf=""> → ImageMagick → Ghostscript</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">Qt 的 </span><code><span leaf="">QImageReader.read()</span></code></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">KImageFormats 的 EPS 插件 → Ghostscript</span></section></td></tr></tbody></table>  
  
还有一个容易忽略的细节：Image.open()  
 本身只做格式识别，真正启动渲染器的是后面的"加载像素""裁剪""转换"。也就是说，**一个缩略图功能就足够**  
 [2]。  
  
这意味着安全评审里最常见的那道防线——"我们只允许上传 jpg/png/pdf"——在原理上就是漏的。文件类型白名单约束的是文件名和 MIME，而决定谁来解析内容的是下层的库。  
## -dSAFER 到底是一道什么边界  
  
Ghostscript 有一个 -dSAFER  
 模式，用来限制文档能读哪些文件、能不能执行命令。听起来像沙盒，但它和 seccomp 完全不是一回事。  
  
seccomp 由内核过滤系统调用，进程想绕过就得先攻破内核。-dSAFER  
 的检查跑在解释器自己内部：PostScript 层的 PermitFileReading  
、PermitFileWriting  
、LockFilePermissions  
，加上 C 层的一个路径控制状态。**这些开关就住在进程的内存里**  
 [2]。  
  
而 PostScript 是一门图灵完备的语言。渲染一份 PostScript 文档，本质上就是执行一个程序。这一点和 JavaScript 引擎很像：运行 JS 是设计预期，但引擎本身的内存安全漏洞，天生就是沙盒逃逸 [2]。  
  
于是问题变成了：渲染进程以什么身份运行？在 pwn.ai 的测试里，是 www-data  
，是 lp  
。这个账号能读应用文件、能写磁盘、能启动别的程序。沙盒的全部意义，就是不让文档借用这份权限 [2]。  
## 缺陷的第一层：一个必须拥有特权的辅助函数  
  
漏洞的位置在 Resource/Init/gs_fonts.ps  
 里的 .putgstringcopy  
。  
  
Ghostscript 把一部分字体加载逻辑用 PostScript 自己实现了。这个辅助函数需要往受保护的字体字典里写数据，普通文档代码没有这个权限，所以它用了一个内部操作符 .forceput  
。函数被 bind  
 过，也标了 executeonly  
，实现细节对外不可见 [2][5]。  
  
它要执行的序列很短：  
```
dup length string copy.forceput setglobal
```  
  
第一行复制一个字符串，第二行做受保护的写入，然后恢复分配模式。问题在于，**它没有先检查操作数是不是预期的类型**  
 [2][5]。  
  
这就像一间上锁的房间里放着一把备用钥匙。攻击者拿不到钥匙，但可以让持有钥匙的人走到一半时绊一跤，然后替他把手按到按钮上。  
## 第二层：让错误处理器替它按下按钮  
  
攻击者注册一个 Type 1 字体，用 findfont  
 加载。这个字体的加载代码能看到字体加载器的操作数栈，于是它把栈上的文件名字符串换成一个本地数组。  
  
在 global 转 local 的分支里，length  
 和字符串分配都成功了，轮到 copy  
 时，因为源是数组而不是字符串，抛出 typecheck  
 [2]。  
  
正常情况下，报错就结束了。但 PostScript 允许文档自定义 errordict  
 里的错误处理器。攻击者的处理器做了五件事——准确地说是五次 pop  
：  
```
pop pop pop pop popuserdict /ForceTarget getuserdict /ForceKey getuserdict /ForceValue get
```  
  
这五次 pop  
 依次丢掉失败的 copy  
、新分配的目标字符串、被替换的文件名、加载器的 /PathLoad  
 键、以及字体字典，只留下那个保存分配模式的布尔值。紧接着压入攻击者选好的字典、键、值。  
  
于是执行流回到那个已经 bind  
 死的辅助函数里，**从 .forceput 那一步继续跑**  
。攻击者从头到尾没有直接调用过这个隐藏操作符 [2]。  
  
这就是这类漏洞反复出现的根源：权限没有被"提升"，而是被"借用"了。函数本身有权限，攻击者只是劫持了它的执行上下文。  
## 第三层：一个活过了对象本身的引用  
  
拿到一次受保护的写入，还不足以执行原生代码。真正致命的是下一步，它破坏的是对象生命周期的规则。  
  
PostScript 区分 local 和 global 两种"虚拟内存"——这里指的是解释器的对象空间。save  
 记录当前 local 内存状态，restore  
 丢弃这之后创建的对象；global 对象可以活过 restore  
 [2]。  
  
这就带来一条约束：一个能活下来的 global 字典，不能持有指向即将被释放的 local 对象的可用引用。否则 restore  
 之后，字典就指进了已回收的内存。  
  
而 .forceput  
 的实现里有一个细节：在 psi/zgeneric.c  
 中，它会临时把 systemdict  
 当作 local 来处理存储，事后再恢复空间标志。结果是一个 local 对象也能通过存储检查，被塞进一个能跨 restore  
 存活的字典 [2]。  
  
于是：  
```
restore 之前：  存活的 global 字典 → local 对象restore 之后：  存活的 global 字典 → 已释放的内存分配被复用后：  存活的 global 字典 → 同地址上的活跃解释器数据
```  
  
字典里的引用一个字节都没变。变的是那块地址上住着谁。  
  
用一个比喻：租客退租了，钥匙还在房东手里，而新租客正好住进了同一个房间。你手里的钥匙依然能开门，只是门后的人已经换了。  
## 从悬空指针到"我能改内存了"  
  
验证这一步不需要碰原生内存。研究者先用一个 8 元素数组占住被释放的位置，通过悬空引用写一个值，再从新数组里读回来——两个引用读到了同一块字节，UAF 就被证实了 [2]。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZhfYKLbIarnGeAzH2fbeac2xNzcHiaAUf0gPagzBbX5og6PKJOeseh2rAqIciaG4iceyYWRETBRPxsKtTMuyE8c5a8ao2pM97Pu1o/640?wx_fmt=png&from=appmsg "")  
  
接下来是把"能读写一块被复用的内存"升级成"能读写任意原生内存"。关键约束是：**不能猜地址**  
。  
  
他们的做法很巧。用一条 2048 字节的悬空字符串，加上一批填满 0x12345678  
 的新数组。通过悬空字符串扫到标记字节，把最低字节从 0x78  
 改成 0x79  
，再去找哪个数组元素变成了 0x12345679  
。变了的那个，就是和悬空引用重叠的槽位 [2]。  
  
在 64 位构建上，Ghostscript 的一个 ref  
 占 16 字节：前 8 字节是类型、属性和长度，后 8 字节是值或指针。研究者把 currentdevice  
 放进重叠槽位，通过悬空字符串读出它真实的 device 引用指针，再把这个 ref 临时改造成指向该指针的字符串视图——从此可以顺着解释器自己的指针走 [2]：  
```
ref + 8            → psi_device_refpsi_device_ref + 0 → gx_devicegx_device + 24     → gs_memory_tgs_memory_t + 208  → gs_lib_ctxgs_lib_ctx + 8     → gs_lib_ctx_core_tgs_lib_ctx_core_t + 156 → path_control_active
```  
  
这些偏移量只对测试的 x86-64 构建成立，但每个对象的地址都来自运行中的真实引用，所以整个利用不依赖固定的进程地址 [2]。  
  
为了证明"这确实是原生控制"，他们做了一件很干净的事：复制 mul  
 的 16 字节引用，拿到一个真实的 sub  
 引用，只替换偏移 8 处的 8 个指针字节，调用同一个 ref 一次，再把原始 16 字节写回。于是 29 × 11 的输出依次是 [2]：  
```
before:   319changed:   18restored: 319
```  
  
中间那个 18 是 29 减 11。这不是在 PostScript 层重定义 mul  
，而是真的改了 C 函数指针。  
## 把沙盒的开关拨到 0  
  
最后一步是命令执行。目标依然是那个 gs_lib_ctx_core_t  
，偏移 156 处的 path_control_active  
。四次 put  
，四个 0：  
```
Target 156 0 putTarget 157 0 putTarget 158 0 putTarget 159 0 put
```  
  
写入前读回 1,0,0,0  
，写入后是 0,0,0,0  
，.currentpathcontrolstate  
 返回 false。**-dSAFER 还挂在命令行上，但它在本地的路径检查已经在内存里被关掉了**  
 [2]。  
  
还没完。%pipe%  
 还受两道门约束：PostScript 层的 PermitFileWriting  
 列表，以及 parse_file_name()  
 会查的一个缓存标志 LockFilePermissions  
。研究者用恢复出来的 .forceput  
 把前者改成 [(*)]  
，再通过一次字体加载把后者强写为 false，并用 save  
/restore  
 让缓存值同步 [2]。  
  
三道门都开之后，Debian 13 上那次打印任务以这样的方式结束：  
```
(PIPE_RCE_STOPPED=) print{  (%pipe%printf POST_SAFER_UAF_PIPE_RCE > /var/tmp/GS_CUPS_PIPE_RCE)  (w) file closefile} stopped
```  
  
调度器记录 PIPE_RCE_STOPPED=false  
，标记文件出现在打印服务器上。false  
 的意思是 stopped  
 块正常跑完了。命令以 uid=7(lp)  
 执行——那是渲染进程的账号，不是提交打印任务的人 [2]。  
## 回到网络：三个入口  
  
解释器层面的原语成立之后，剩下的问题是：远程用户能把这份文档送到哪里？  
  
pwn.ai 追了大约十款应用的调用路径，其中三条最有代表性。  
  
**Debian 13 的 CUPS。**  
 向一个共享网络打印队列发送匿名 IPP Print-Job  
，CUPS 把 PostScript 文档交给 gstopdf  
，后者用发行版的 Ghostscript 以 -dSAFER  
 启动。命令在打印服务器上以 lp  
 身份执行。给队列加上认证，这条路就断了 [2]。  
  
**Roundcube。**  
 攻击者只需要发一封邮件。附件声明为 image/jpeg  
，名字叫 quarterly-results.jpg  
，内容是 PostScript。收件人**只要打开邮件**  
，不需要点击、不需要下载，Roundcube 就会为它请求缩略图，缩略图路径把附件交给 ImageMagick，ImageMagick 调起 Ghostscript。命令以 www-data  
 执行，同时还生成了一张缩略图 [2]。  
  
**WordPress。**  
 在 Core 里，只需要 upload_files  
 权限，而 Author 及以上角色默认就有。更宽的口子在插件：Forminator 的公开 PDF 表单如果配置成存入媒体库，未登录访客提交就能触发；Essential Addons for Elementor 的公开注册组件配了上传字段，同样不需要账号 [1][2]。  
  
完整清单还包括 MediaWiki + PdfHandler（允许匿名上传的站点）、DokuWiki（启用 ImageMagick 后端）、Mayan EDMS（配置了 POP3 邮件收件源）、ResourceSpace、以及 Plesk Obsidian 自带的 Roundcube 组件 [2]。  
  
顺带一提，WordPress 在 8 月 12 日就修了自己的问题，编号 CVE-2026-65640，7.0.4 起修复并回溯到 4.7 分支 [8]。但它修的是"上传后不该进入那条转换路径"，Ghostscript 修的是"进入之后不能逃出沙盒"，**两个补丁覆盖的是链条上的不同环节，装一个不能代替另一个**  
 [1][2]。  
## 8.5 分，向量却写着"本地"  
  
CVE-2026-125396 的 CVSS 4.0 评分是 8.5，向量是 AV:L/AC:L/AT:N/PR:N/UI:P/VC:H/VI:H/VA:H  
 [4]。AV:L  
 是本地。  
  
这看起来和"远程执行命令"矛盾，其实不矛盾。评分针对的是**解释器本身**  
：要触发这个缺陷，得先让 Ghostscript 处理一份文件，从解释器的视角看，这是本地输入。至于这份文件是打印队列送来的、邮件附件送来的，还是网页表单传上来的，属于投递方式，不在这个向量里 [2]。  
  
研究者的工作恰恰是把"投递"这一环补齐了。CUPS 提供了网络可打印路径，Roundcube 提供了邮件路径，WordPress 和插件提供了上传路径，而且多次试验可重复 [2]。  
  
这个区分很重要：**评分低不代表不可远程利用，只代表评分对象不是整条链。**  
## 补丁，以及补丁到不了的地方  
  
Artifex 的修复分两步。  
  
第一个提交（1484854，9 月 2 日）在校验操作数类型，让 copy  
 不再抛错，从而切断"错误处理器接管执行流"这条路 [5]。提交信息里写得很坦率：  
> 我们正在消除，或至少把 .forceput  
 的使用降到绝对最低。理想情况下我们想完全移除它，正是为了避免这类问题。不幸的是这仍是进行中的工作；撤销 30 年的假设并不容易。  
  
  
维护者 Ken Sharp 在 bug 报告里也承认，那段代码"是众所周知存在漏洞、但难以根除的古老代码" [5]。  
  
第二个提交（e0421767，9 月 10 日）把 .putgstringcopy  
 改写成 C 实现，从根本上不再依赖 .forceput  
 [6]。这才是结构性修复：不是把错误路径堵上，而是不再需要那把备用钥匙。  
  
Ghostscript 10.08.0 于 9 月 8 日发布 [7]。Ubuntu 在 10 月 21 日的 USN-8949-1 里为 7 个发行版提供了修复包 [2]。Debian 13 通过 DSA-6516-1 修复，Debian 12 后来通过 DLA-4914-1 补上，跟踪页现在显示 bookworm 的安全更新已标记为 fixed [9]。  
  
但覆盖并不均匀。截至 11 月 12 日，Amazon Linux 2/2023/2027 Preview 仍列为"待修复"，SUSE 多个分支处于进行中，openSUSE Leap 16.0/16.1 列为受影响；研究团队也未能确认当时 RHEL、Rocky、AlmaLinux、Oracle Linux 和稳定版 Fedora 的包是否已包含修复 [2]。  
  
更麻烦的是"补丁到不了"的问题：**更新宿主操作系统，不会更新容器镜像或应用目录里自带的那份 Ghostscript。**  
 文档处理 worker 完全可能继续跑着旧代码，而宿主的包早就换过了 [1][2]。  
## 那么，该做什么  
  
如果只记一句话：**去确认你实际调用的那个 Ghostscript 是哪个版本。**  
  
具体来说：  
- 查渲染路径上的真实二进制，而不是宿主 dpkg -l  
 的结果。容器镜像、应用 bundle、打印驱动的 PPD 目录都可能有独立副本。  
  
- 打印队列要求认证。pwn.ai 的 CUPS 链路就是靠匿名 Print-Job  
 打通的 [2]。  
  
- 限制图像处理库接受的格式。ImageMagick 的 policy 可以禁用 PS/EPS/PDF delegate；Pillow、libvips 也有对应的收紧方式。  
  
- 加一道进程外的边界。测试中一个 AppArmor 配置成功阻止了被攻陷的解释器启动 shell [1]；seccomp、landlock、低权限容器是同一个思路。  
  
- 调整处理顺序：先确认文件的真实类型，再生成缩略图。Roundcube 的问题恰恰是"先缩略图、后判断"，等于把第一次解释的机会直接交给了渲染器 [2]。  
  
## 更值得记住的  
  
这个漏洞的技术细节很漂亮：借来的权限、被劫持的错误处理、跨 restore  
 的悬空引用、顺着解释器自己的指针走而不猜地址。但它给出的教训比技巧更朴素。  
  
第一，**"文件类型白名单"不是安全边界。**  
 它约束的是名字，而解析器看的是内容。真正决定风险的是"这份文件最终会被哪个解释器执行"，这个问题在依赖树里往往要往下挖三四层才能回答 [2]。  
  
第二，**解释器内的沙盒挡不住解释器内的内存破坏。**  
 只要 -dSAFER  
 的开关是进程里的几个字节，一个写原语就等于沙盒失效。想要真正的边界，得让边界跑在进程外面。  
  
第三，**补丁的有效性取决于它有没有到达执行点。**  
 一个修好的上游版本、一个修好的发行版包、一个还在跑旧镜像的 worker，是三件不同的事。  
  
还有一点值得一提。pwn.ai 在技术分析里写道，这个问题是 GPT 5.6 Sol 与 Paulos Yibelo 一起发现的 [2]。这是研究者的自述，无法独立验证；但结合过去一年漏洞披露节奏的变化，它至少说明一件事：**依赖树深处的老代码，正在被更快地翻出来。**  
 而 Ghostscript 这样的组件，恰好是那种"没人记得它在那儿、但它一直在处理陌生人的文件"的角色。  
  
一张改了名字的图片，之所以能让服务器执行命令，不是因为某个配置写错了。是因为我们默认了一条看不见的流水线：上传 → 识别 → 转换 → 渲染。而流水线的最后一站，是一门图灵完备的语言。  
## 参考资料  
1. Researchers Demonstrate Ghostscript Sandbox-Escape RCE via Crafted File Uploads in Popular Apps  
  
1. How We Hacked the Pipes — Upload2Shell  
  
1. Upload2Shell · CVE-2026-125396 影响索引  
  
1. Ghostscript before 10.08.0 -dSAFER Bypass via .putgstringcopy .forceput Recovery  
  
1. PS interpreter - validate operands to .putgstringcopy（commit 1484854）  
  
1. Implement .putgstringcopy in C（commit e0421767）  
  
1. Ghostscript/GhostPDL 10.08.0  
  
1. Remote code execution vulnerability via malicious file upload by an Author level user or higher（GHSA-8vr3-7mxf-gx8w）  
  
1. CVE-2026-125396  
  
1. Ghostscript Bug 709684  
  
1. Upload2Shell (CVE-2026-125396)  
  
- END -  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
