# rewin.ubsi.common.Crypto 国密算法

---



### SM3数据散列

```
public static byte[] sm3Digest(byte[] data);
```

参数：

* data - 源数据

返回：

* 散列值，长度32字节



### HMAC数据签名

```
public static byte[] sm3HMAC(byte[] data, byte[] key);
```

参数：

- data - 源数据
- key - 密钥，长度不限


返回：

* 签名，长度32字节



### 计算SM4加密数据的长度

```
public static int sm4EncryptSize(int size);
```

参数：

- size - 源数据的长度

返回：

- 加密数据的长度



### SM4数据加密

```
public static byte[] sm4EncryptEcb(byte[] data, byte[] key);
```

参数：

- data - 源数据，以128位（16字节）为一组，会自动补位
- key - 密钥，长度必须16字节

返回：

- 加密数据




### SM4数据解密

```
public static byte[] sm4DecryptEcb(byte[] data, byte[] key);
```

参数：

- data - 加密数据，长度必须为16字节的整倍数
- key - 密钥，长度必须16字节

返回：

- 解密数据

