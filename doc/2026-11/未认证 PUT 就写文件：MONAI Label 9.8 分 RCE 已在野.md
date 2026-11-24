#  未认证 PUT 就写文件：MONAI Label 9.8 分 RCE 已在野  
红队安全圈
                    红队安全圈  红队安全圈   2026-11-24 00:00  
  
一个 PUT 请求，不用登录，就能把文件写到服务器上任意可写路径。MONAI Label 这个 9.8 分的洞已经被用起来了，11 月 23 日进了已知在野利用清单。  
  
## 01  事情是什么  
  
出问题的接口是   
PUT /datastore/image  
。它把调用方传来的 image 查询参数直接当影像 ID 用，不做任何校验。LocalDatastore.add_image() 拿到这个 ID 后拼上文件扩展名，join 到 datastore 的影像目录，再 shutil.copy() 把上传的文件写过去。  
  
问题就在这个路径从头到尾没检查过是否还在 datastore 里面。传一个 ../../../tmp/x，文件就落到目录外面。PUT /datastore/label 走的 save_label() 是同一个写法，一样能穿出去，只是拼接的是标签目录。归类 CWE-24 路径穿越，CVSS 3.1 打 9.8。  
  
## 02  为什么危险  
  
任意文件写在这个场景里基本等于拿 shell。落点很直接：覆盖应用自身的 Python 代码等下次加载、往 .ssh/authorized_keys 追加公钥、或者在 cron 目录里塞一个任务。整条链上没有一步需要猜、需要爆破或者需要等回调。  
  
请求形状大致是这样：  
  
PUT /datastore/image?image=  
  
  ../../tmp/x  
  
Content-Type: multipart/form-data  
  
（下面是上传的文件体）  
  
真正让它变严重的是认证状态。MONAI Label 默认 MONAI_LABEL_AUTH_ENABLE=False，也就是默认没人拦。默认配置下，任何能连到服务端口的客户端都能打。就算把认证打开了，ANNOTATOR 这一个角色就够。  
  
漏洞记录里写得很清楚：2026 年 9 月起已在野利用。这不是一个只存在于理论里的写法，前面已经有人在用了。  
  
## 03  谁受影响  
  
影响组件  
pip 包 monailabel 0.8.5 及以下  
  
修复版本  
0.8.6（PR 1915）  
  
利用状态  
已在野利用  
  
前置条件  
能连到服务端口（默认无认证）  
  
最典型的是影像科、科研单位和 AI 标注平台这类内部系统：几个人要用、上线赶时间，于是端口直接放开、认证没开。这个洞打的就是这种部署。  
  
先看装的版本：  
  
pip show monailabel \  
  
  | grep -i version  
  
再看服务打没打补丁。补丁会把空 ID 和点开头的 ID 直接拒掉回 400，没打补丁的照旧接收：  
  
curl -s -o /dev/null \  
  
  -w "%{http_code}\n" \  
  
  -X PUT \  
  
  "http://host:8000/datastore/\  
  
image?image=.probe" \  
  
  -F file=@1.png  
  
返回 400 说明补丁在；返回 200 说明这台机器还能被写。  
  
## 04  怎么修  
  
1. 升到 0.8.6；版本还没落地就先合 PR 1915 的两个校验，add_image() 和 save_label() 都会判断最终路径必须落在 datastore 内。  
  
2. 打开认证：MONAI_LABEL_AUTH_ENABLE=True，ANNOTATOR 角色不要发给不可信的人。  
  
3. 把服务从公网撤回来，前面挂反向代理再加一层身份验证。  
  
4. 进程降权运行，写权限只留 datastore 目录。  
  
5. 排查历史利用：看 datastore 之外有没有多出来的文件，检查 authorized_keys、crontab 和近期被改动的代码。  
  
## 05  一句判断  
  
只在内部跑、就两三个人用、端口没出网的，优先级可以往后排。但只要这个服务在公网上，或者让外部标注人员接进来过，这就是一台现成的 RCE 机器，两个晚上之内修完。  
  
红队视角多说一句：AI 标注平台这类科研工具直接上生产的系统，默认关认证、端口裸奔、日志基本没有，进了内网之后往往比业务系统好打得多，值得单独列一张资产表。  
  
## 06  修复参考  
  
https://github.com/Project-MONAI/MONAILabel/security/advisories/GHSA-qc22-rmgc-vg94  
  
https://github.com/Project-MONAI/MONAILabel/pull/1915  
  
https://nvd.nist.gov/vuln/detail/CVE-2026-133968  
  
公开信息只有公告和修复 PR，暂无公开 PoC 仓库。  
  
  
下期继续蹲厂商补丁和 PoC 动静，**关注**  
后漏洞先知道，修复不踩坑。  
  
— END —  
  
  
