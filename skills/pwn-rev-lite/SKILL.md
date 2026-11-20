---
name: pwn-rev-lite
description: PWN 与逆向题的最低限度打法。仅当题面明确是二进制/逆向/溢出时使用；15 分钟无进展立即放弃转下一题。
---

# PWN / RE 最低限度打法（低优先级，止损优先）

## RE
```bash
file 程序                # 位数/加壳（upx -d 脱壳）
strings 程序 | grep -iE 'flag|correct|wrong|pass'
# 有 IDA/ghidra：Shift+F12 找字符串 → 交叉引用 → 找到比较函数
# 常见：硬编码 flag、简单 xor 加密、md5 比较
# 动态调试：输入后下断点看寄存器/内存
```

## PWN
```bash
checksec 程序            # PIE/NX/canary 情况
file 程序
# 流程：找溢出点 → 构造 payload → 拿 shell → cat /flag
# 常见：ret2text（有后门函数直接跳）、ret2libc（泄漏地址）、
#       fmt 格式化字符串（%n 写）、栈溢出基础
```

## 止损规则（重要）
- 10 分钟没找到溢出点/关键逻辑 → 写 notes.md 跳过。
- 明确难的（堆利用、内核、虚拟机逃逸）直接跳过，不碰。
- RE 简单题（硬编码 flag）值得先 strings 扫一遍，其余不强求。
