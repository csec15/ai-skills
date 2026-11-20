---
name: crypto-ctf
description: Crypto 类 CTF 题的标准解题流与常用脚本模板。当题目给出密文、加密脚本、密码学参数时使用。
---

# Crypto 打法

## 0. 先分类
- **纯编码**（base64/hex/url/摩斯/凯撒/栅栏/ROT）：无密钥结构，直接 CyberChef 或爆破。
- **古典密码**（凯撒/维吉尼亚/仿射/培根）：有工具就爆破，密钥空间小。
- **现代密码**（RSA/AES/ECC）：给了脚本 → 找实现漏洞；给了参数 → 套攻击。

## 1. RSA 高频攻击（按参数判断）
```python
# 模板：pip install pycryptodome gmpy2 sympy
from Crypto.Util.number import long_to_bytes
# ① n 可分解（factordb.com 查，或 yafu）
p, q = factor(n); d = inverse(e, (p-1)*(q-1)); m = pow(c, d, n)
# ② 共模攻击（同 n 不同 e）
# ③ 低加密指数：e=3 且 m^e < n → m = c 开三次方
# ④ 低解密指数：Wiener 攻击（e 很大）
# ⑤ p,q 相近：费马分解
# ⑥ dp/dq 泄露、p 高位泄露
print(long_to_bytes(m))
```

## 2. AES 高频攻击
```
- 明文/密文/IV 可控 → CBC 字节翻转
- ECB 模式 → 块重放、选择明文攻击
- 密钥弱（题面提示）→ 爆破
```

## 3. 古典密码识别
```
凯撒/ROT：看偏移爆破（CyberChef ROT13 可调）
维吉尼亚：先猜密钥长度（重合指数），再逐位爆破
栅栏：栏数爆破
培根：只有 AB 两种字符
```

## 4. 数学类
```
格密码（LLL）、离散对数（sage）、椭圆曲线（异常曲线）
sage 没有就 pip install sageall 或在线思路套 python sympy
```

## 5. 通用习惯
- 拿到题目脚本先读一遍：出题人常留后门（固定 IV、可预测随机数、自己实现的加密）。
- 答案一般是 `long_to_bytes` 或直接字符串，注意大小写和格式。
