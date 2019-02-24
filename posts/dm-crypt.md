# dm-crypt 简介

`dm-crypt` 是 linux 内核中提供的用于加密的模块。

`cryptsetup` 是操作 `dm-crypt` 的命令行工具，`cryptsetup` 和 `dm-crypt` 的关系相当于 `iptables` 和 `netfilter` 或 `tc` 和 `netem` 的关系。

## 几种加密模式

1. LUKS (Linux Unified Key Setup): dm-crypt 最常用的模式
2. Plain: 这个是给老手用的
3. TCRYPT: 支持 `TrueCrypt` 的模式，但无法修改密码和使用 keyfiles 功能

## 基本操作

### 查看性能指标

使用如下命令查看 dm-crypt/cryptsetup 针对不同“加密算法”和“散列算法”的性能指标。

```bash
cryptsetup benchmark
```

### 创建加密盘

```bash
cryptsetup [options...] luksFormat device-or-file
```

相关参数

参数|含义|推荐值|说明
:--|:--|:--|:--------
--cipher|加密方式|aes-xts-plain64|AES
--key-size|密钥长度|512|因为 XTS 模式需要两对密钥，每个的长度是256
--hash|散列算法|sha512|
--iter-time|迭代时间|最好大于 10000|单位毫秒。该值越大，暴力破解越难；但是你在打开加密盘时也要等待更久

如：

```bash
cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 luksFormat /dev/sda2

# 使用文件作为逻辑卷
fallcate -l 1G /test.vol
cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 luksFormat /test.vol
```

### 打开加密盘

```bash
cryptsetup open --type luks device-or-file mapping-name
# 如：
cryptsetup open --type luks /dev/sdb myname

# 等价于其简写方式

cryptsetup luksOpen /dev/sdb myname
```

随后，该盘的解密形式就会出现在 `/dev/mapper/myname`

### 查看加密盘状态

```bash
cryptsetup status mapping-name
```

### 关闭加密盘

```bash
cryptsetup close mapping-name
```

## 使用 keyfile 作为认证因素

### 创建一个 keyfile

```bash
dd if=/dev/urandom of=mykey bs=1k count=64
```

### 查看加密盘的认证因子

```bash
cryptsetup luksDump device-or-file
```

LUKS 的默认 key slot 有 8 个，用任意一个都可以解密数据

### 增加 keyfile 认证

```bash
cryptsetup luksAddKey device-or-file keyfile
```

注：如果不加 keyfile 则提示你可以添加一个新的密码

### 使用 keyfile 打开加密盘

```bash
cryptsetup --key-file keyfile luksOpen device-or-file mapping-name
```

### 删除 key slot

```bash
cryptsetup luksKillSlot device-or-file slot编号
```

## resize 一个逻辑卷

### 确保该卷没有被挂载，没有被映射

```bash
umount /mnt
cryptsetup close /dev/mapper/mytest
losetup -a  # 查看 loop 设备使用情况中没有 test 文件
```

### 添加新的空间

1. 使用 `dd`
```bash
dd if=/dev/zero of=test oflag=append conv=notrunc bs=1M count=1024
```
2. 内核 4.9 及以上使用 `fallocate`（未验证）
```bash
fallocate -i -l 1G test
```

### resize

resize 加密部分

```bash
cryptsetup resize test
```

resize 文件系统

```bash
e2fsck -f /dev/mapper/mytest
resize2fs /dev/mapper/mytest
```

## 加密一个存在未加密数据的设备

### 将现有文件系统 shrink 到最小

```bash
e2fsck -f /dev/vdb
resize2fs -M /dev/vdb
```

### 开始加密

```bash
cryptsetup-reencrypt /dev/vdb --new --reduce-device-size 4096S --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000

# --reduce-device-size 4096S 表示腾出 4096 个 512-byte sector(共 2M 大小) 给 LUKS header
```

### 映射、恢复大小

映射

```bash
cryptsetup luksOpen /dev/vdb vdb
```

resize 文件系统

```bash
e2fsck -f /dev/mapper/vdb
resize2fs /dev/mapper/vdb
```
