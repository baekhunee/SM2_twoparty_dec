# 实验名称
SM2_twoparty_dec

# 实验简介
implement sm2 2P decrypt with real network communication

# 实验完成人
权周雨 

学号：202000460021 

git账户名称：baekhunee

# 运行指导
需要先启动twoparty_dec_client2，再启动twoparty_dec_client1，两个文件均可以直接启动。

# 实现过程
根据课件指导，sm2 two-party decrypt过程如下所示：
![image](https://user-images.githubusercontent.com/105578152/181195108-5752a481-45fb-440e-9365-af1df6e9f69b.png)

# 部分代码说明
## sm3.py
用于实现SM3算法。后期解密过程中需要用到Hash函数，本次实验选用的即为自己实现的SM3函数。调用方法为：
```
import sm3
m = "..."
sm3.SM3(m)
```

## def epoint_add(x1,y1,x2,y2)
椭圆曲线上的加法,返回值(x,y)=(x1+x2,y1+y2)

## def epoint_mult(x,y,k)
椭圆曲线上的点乘，返回值为k*(x,y)

## def KDF(z,klen)
密钥派生函数

## 核心代码展示
双方具体的解密与收发过程均已在代码中添加注释，不再赘述。
### twoparty_dec_client1
```
# 生成子私钥 d1
d1 = 0x6FCBA2EF9AE0AB902BC3BDE3FF915D44BA4CC78F88E2F8E7F8996D3B8CCEEDEE

# 获取密文 C = C1||C2||C3
C1 = (0x26518fd38aa48284d30ce6e5c42d34b57840d1a03b64947b6a300ffe81797cc8,
      0x208be67614cc4562c219dc0cc060aeca05c52bfc1a990f9f02a4ed972ee91df6)
C2 = 0x4e1d4176afeec9e0ddc7702c1bd9a0393b54bb
C3 = 0xDF31DE4A7A859CF0E06297030D4F8DE7ACA5D182D89FE278423F7D12F9C3E03C

# 计算T1 = d1^(-1) * C1
T1 = epoint_mult(C1[0], C1[1], invert(d1, p))
x, y = hex(T1[0]), hex(T1[1])
klen = len(hex(C2)[2:])*4

# 将T1发送给客户2
addr = (HOST, PORT)
s.sendto(x.encode('utf-8'), addr)
s.sendto(y.encode('utf-8'), addr)

# 从客户2接收到T2
x1, addr = s.recvfrom(1024)
x1 = int(x1.decode(), 16)
y1, addr = s.recvfrom(1024)
y1 = int(y1.decode(), 16)
T2 = (x1, y1)

# 计算T2 - C1
x2, y2 = epoint_add(T2[0], T2[1], C1[0], -C1[1])
x2, y2 = '{:0256b}'.format(x2), '{:0256b}'.format(y2)
# t= KDF(x2||y2,klen)
t = KDF(x2 + y2, klen)
# M2 = C2 ^ t
M2 = C2 ^ int(t,2)
m = hex(int(x2,2)).upper()[2:] + hex(M2).upper()[2:] + hex(int(y2,2)).upper()[2:]

# u = Hash(x2||M2||y2)这里Hash函数选择SM3
u = sm3.SM3(m)
if (u == C3):
    print(hex(M2).upper()[2:])
print("result:",hex(M2)[2:])
```

### twoparty_dec_client2
```
# 生成子私钥 d2
d2 = 0x5E35D7D3F3C54DBAC72E61819E730B019A84208CA3A35E4C2E353DFCCB2A3B53

# 从客户1接收到T1
x, addr = s.recvfrom(1024)
x = int(x.decode(), 16)
y, addr = s.recvfrom(1024)
y = int(y.decode(), 16)
T1 = (x, y)

# 计算T2 = d2^(-1) * T1
T2 = epoint_mult(x, y, invert(d2, p))
x, y = hex(T2[0]), hex(T2[1])
s.sendto(x.encode('utf-8'), addr)
s.sendto(y.encode('utf-8'), addr)
```

# 测试截图
![image](https://user-images.githubusercontent.com/105578152/181196838-a8ae2c70-0b0b-4b77-b3ea-faa722758dbb.png)

![image](https://user-images.githubusercontent.com/105578152/181196926-01c004a1-d54d-49ca-b03e-bf2bcfde2012.png)


