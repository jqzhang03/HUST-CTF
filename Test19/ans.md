# Crypto-签名算法

## 题意
`server.py`

## 题解

```python
import socket  # 导入 socket 模块用于网络通信
import time  # 导入 time 模块用于时间控制
import math  # 导入 math 模块用于数学运算
from gmpy2 import invert  # 从 gmpy2 导入 invert 函数，用于求逆元
import hashlib  # 导入 hashlib 模块用于哈希计算

HOST = '172.17.0.15'
PORT = 16486
loop_times = 7  # 定义请求次数

def get_signatures():
    # 初始化签名和哈希值列表
    sign_s, hash_m = [], []
    for i in range(loop_times):
        print(f"\r获取到第{i + 1}组签名与消息", end='', flush=True)
        time.sleep(1.1 if i > 0 else 0)  # 等待一下，让服务端哈希值更新
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:  # 使用 with 语句管理 socket 连接
            try:
                s.connect((HOST, PORT))  # 连接到指定的主机和端口
                s.recv(1024)  # 接收初始响应
                s.send(b"1\n")  # 发送请求以获取签名和哈希值
                response = s.recv(1024).decode().split(",")  # 接收响应并解析
                sign_r = int(response[0])  # 提取签名 r 值
                sign_s.append(int(response[1]))  # 保存签名 s 值
                hash_m.append(int(response[2]))  # 保存消息哈希值
            except Exception as e:
                print("发生错误:", e)  # 捕获并打印异常
    return sign_r, sign_s, hash_m  # 返回最后一个 r 值和所有签名、哈希值

def compute_keys(sign_r, sign_s, hash_m):
    # 计算签名差值和哈希差值
    delta_s = [sign_s[i + 1] - sign_s[i] for i in range(len(sign_s) - 1)]
    delta_h = [hash_m[i + 1] - hash_m[i] for i in range(len(hash_m) - 1)]
    q = 0  # 初始化 q 值
    for i in range(2, len(delta_s)):
        # 计算 kq1 和 kq2，用于求解 q
        kq1 = abs(delta_s[i - 2] * delta_h[i - 1] -
                  delta_s[i - 1] * delta_h[i - 2])
        kq2 = abs(delta_s[i - 1] * delta_h[i] - delta_s[i] * delta_h[i - 1])
        kq = math.gcd(kq1, kq2)  # 计算 kq 的最大公约数
        q = math.gcd(q, kq) if q > 0 else kq  # 更新 q 值
    # 计算临时密钥 k 和密钥 x
    k = delta_h[0] * invert(delta_s[0], q) % q
    x = (k * sign_s[0] - hash_m[0]) * invert(sign_r, q) % q
    return q, k, x  # 返回计算出的 q, k, x

def send_final_signature(sign_r, sign_s):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:  # 使用 with 语句管理 socket 连接
        try:
            s.connect((HOST, PORT))  # 连接到指定的主机和端口
            s.recv(1024)  # 接收初始响应
            s.send(b"2\n")  # 发送请求以验证签名
            response = s.recv(1024)  # 接收响应
            signature_part = response.decode().split('"')[1]  # 提取签名部分
            to_check = int(hashlib.md5(signature_part.encode()
                                       ).hexdigest(), 16)  # 计算待检查的哈希值
            sign_s = (to_check + x * sign_r) * invert(k, q) % q  # 计算最终签名
            s.send(f"{sign_r}, {sign_s}\n".encode())  # 发送签名和 r 值
            final_response = s.recv(1024).decode()  # 接收最终响应
            final_response = s.recv(1024).decode().strip()  # 再次接收并去除空格
            print('\n\n', final_response, '\n\n')  # 打印最终响应
        except Exception as e:
            print("发生错误:", e)  # 捕获并打印异常
            
if __name__ == '__main__': 
    sign_r, sign_s, hash_m = get_signatures()  # 获取签名和哈希值
    q, k, x = compute_keys(sign_r, sign_s, hash_m)  # 计算公钥和临时密钥
    send_final_signature(sign_r, x)  # 发送最终签名
```