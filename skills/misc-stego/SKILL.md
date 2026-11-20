---
name: misc-stego
description: Misc、隐写、取证类 CTF 题的标准排查流。当题目给图片/音频/压缩包/流量包/磁盘镜像等附件时使用。
---

# Misc / 隐写 / 取证打法

## 0. 附件到手第一件事
```bash
file 附件            # 真实类型（改后缀的题靠这个）
binwalk 附件         # 看是否藏着别的文件
binwalk -e 附件      # 自动提取
strings 附件 | grep -iE 'flag|ctf|{'    # 直接找字符串
```

## 1. 图片隐写（png/jpg 出现）
```bash
exiftool 图             # 元数据（作者/坐标/注释藏 flag）
zsteg 图                # LSB 隐写一步到位
steghide extract -sf 图 -p ''    # 无密码提取
# PNG：改高度（CRC 校验不过→宽高被改）
# JPG：FFD9 后追加数据 → dd/foremost 提取
```

## 2. 压缩包
```bash
# 伪加密：010 编辑器改加密标志位（zip 中 0x50 0x4B 附近）
zip2john x.zip > h && john h --wordlist=rockyou.txt
# 爆破可用 hashcat -m 13600/17200
```

## 3. 流量包 pcap
```bash
tshark -r x.pcap -Y 'http.request.method==POST' -T fields -e http.file_data
tshark -r x.pcap --export-objects http,outdir   # 导出 http 对象
binwalk -e x.pcap    # 藏文件
# FTP/明文协议抓账号：tshark -r x.pcap -Y 'ftp.request.command' -T fields -e ftp.request.arg
```

## 4. 磁盘/内存镜像
```
取证题（volatility/autopsy）：先 strings 镜像找 flag，再挂载看文件系统
```

## 5. 音频
```
Audacity 频谱图看 flag；DTMF 拨号音识别；慢速播放
```

## 6. 编码杂项
```
CyberChef 万能：base64/hex/url/凯撒/ROT13/摩斯/栅栏/XX编码
多层编码循环解，直到出现 flag{
```

**习惯**：binwalk 和 zsteg 永远先跑，一半的 misc 题在这两步就结束了。
