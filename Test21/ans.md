# Crypto-分组密码工作模式

## 题意
`cbc.py`

## 题解

```python
import socket
HOST = '172.17.0.15'  # 服务器地址
PORT = 16488          # 端口号
# 创建 socket 连接
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    try:
        s.connect((HOST, PORT))  # 连接到服务器
        s.recv(1024)  # 接收初始消息
        s.send(b"1\n")  # 发送创建账户请求
        s.recv(1024)  # 接收中间响应
        # 接收token
        response = s.recv(1024).decode()
        start_index = response.find("token:") + len("token: ")
        end_index = response.find("\n", start_index)
        token = response[start_index:end_index].strip()
        # 计算用户密钥
        user_key = bytes.fromhex(token[:32])  # 前32个字节为user_key
        cur = b'HUSTCTFer!______'
        tar = b'AdminAdmin!_____'
        user_key = bytes([user_key[i] ^ cur[i] ^ tar[i]
                         for i in range(len(cur))])
        s.send(b"3\n")  # 发送登录请求
        s.recv(1024)  # 接收中间响应
        # 发送计算后的用户密钥和密文
        s.send(f'{user_key.hex() + token[32:]} \n'.encode())
        s.recv(1024)  # 接收中间响应
        # 输出最终响应
        print("\n\n", s.recv(1024).decode(), "\n")
        s.send(b"4\n")  # 发送退出请求
    except Exception as e:
        print("发生错误:", e)
```
