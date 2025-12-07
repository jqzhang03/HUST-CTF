# Crypto-Hill密码

## 题意
给你一段密文和一些已知信息，让你求出明文

## 题解
使用[SageMath](https://cocalc.com/features/sage)在线求出
```python
from Crypto.Util.number import *
raw = b'>u\x10l9\npI,0\x04^J\x00ib\x03\x0c\x158d\x1f\x08Ixk\nF\x19fz\x14PT\x04\x03>R~'
rawi = [int(x) for x in raw]
p = matrix(Zmod(127), 3, 3, [118, 109, 99, 123, 0, 0, 125, 32, 32])
c = matrix(Zmod(127), 3, 3, [62, 117, 16, 108, 57, 10, 62, 82, 126])
for i in range(0, 127):
    for j in range(0, 127):
        p[1, 1] = j
        p[1, 2] = i
        flag = True
        if p.is_invertible():
            K = p.solve_right(c)
            decode = ''
            for k in range(13):
                tmp = matrix(Zmod(127), 1, 3, rawi[k*3:(k+1)*3])
                A = K.solve_left(tmp)
                for l in range(3):
                    char = chr(A[0, l])
                    if not char == ' ' and not char.isprintable():
                        flag = False
                        break
                    decode += char
                if flag is False:
                    break
            if flag is True:
                print(f"{i},{j}|{decode}")
```