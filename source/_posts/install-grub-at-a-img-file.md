---
title: install grub at a img file
toc: true
mathjax: false
date: 2021-08-15 14:32:08
categories:
   - [Linux]
tags:
   - Grub
---

# 在磁盘映像文件上安装 Grub 2

> 本文操作环境为 Arch Linux

### 前言

1. 文中的 Grub 均指 Grub 2，而非 Grub Legacy
2. 文中涉及的安装操作仅限于 Arch Linux，其他发行版请参考 Wiki 
3. 分区大小以及文件大小并不一定按照下面所示的设置，但推荐至少 15 M

### 使用 dd 创建一个映像文件

```shell
$ dd if=/dev/zero of=os.img bs=1M count=20
```

此命令将在当前目录创建一个大小为 20M，文件名为 os.img 的文件

### 在 os.img 上创建分区

此处提供 `gdisk` 和 `fdisk` 命令参考

#### 使用 gdisk

```shell
$ gdisk os.img
```

进入 gdisk 交互界面
如下所示

```
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help):
```

> 根据 [Arch Linux Wiki](https://wiki.archlinux.org/title/GRUB#BIOS_systems)
> 需要腾出 1M 空间给 Grub 使用

**创建分区简要步骤：**
- 输入 `n` 创建新的分区
- 回车选择默认的分区号
- 再次回车选择默认的起始块
- 输入 `+1M` 将分区大小设置为 1M
- 输入 `ef02` 将分区类型设置为 Boot 启动分区
- 输入 `n` 创建新的分区
- 之后全部回车，将剩余空间创建为一个分区
- 输入 `p` 确认分区无误后，最后输入 `w` 确认写入文件

可能会出现一个警告，确认即可

**以下为参考：**
```
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): d
Using 1

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-40926, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-40926, default = 40926) or {+-}size{KMGTP}: +1M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef02
Changed type of partition to 'BIOS boot partition'

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-40926, default = 4096) or {+-}size{KMGTP}: 
Last sector (4096-40926, default = 40926) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk ./file.img: 40960 sectors, 20.0 MiB
Sector size (logical): 512 bytes
Disk identifier (GUID): 8AECC85A-5E21-48E1-AB62-1684A9A8BF0D
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 40926
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            4095   1024.0 KiB  EF02  BIOS boot partition
   2            4096           40926   18.0 MiB    8300  Linux filesystem

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to ./file.img.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.
```

#### 使用 fdisk
```shell
$ fdisk ./os.img
```

进入 fdisk 交互界面

输入 `g` 新建一个 GPT 分区表

创建分区过程与 **使用gdisk** 一节类似
但创建分区过程无法选择分区类型，需要在创建分区后进行更改

在创建分区后
- 输入 `t` 更改分区类型
- 随后会要求选择分区，输入 `1` 选择第一个分区
- 然后输入 `4` 将分区类型更改为 BIOS Boot

**参考**
```
命令(输入 m 获取帮助)：t
分区号 (1,2, 默认  2): 1
分区类型或别名（输入 L 列出所有类型）：4

已将分区“Linux filesystem”的类型更改为“BIOS boot”。

命令(输入 m 获取帮助)：
```

最后输入 `w` 确认写入文件


### 将 os.img 挂载为 loop 设备

```shell
$ sudo losetup --show -P -f ./os.img
/dev/loop0
```

命令解释：
   - `--show` 在挂载成功后，显示挂载的设备名
   - `-P` 创建带分区的 loop 设备
   - `-f` 查找第一个未使用的设备

显示的设备名不一定是 `/dev/loop0`
在后问出现的 `/dev/loop0` 更改为你实际操作时显示的名称

### 格式化并挂载分区

> 在文中实例中，需要格式化的是第二个分区，对应的 loop 设备为 `/dev/loop0p2`
> 如上一个步骤中的显示的 loop 设备不同，请自行更改

```shell
$ sudo mkfs.vfat -L "BOOT" /dev/loop0p2
```

不出意外的话，格式化很快就能完成
随后挂载分区

请选择一个没有正在使用的文件夹
在本文中选择 `/mnt` 作为挂载点
这个文件夹在之后会使用到

```shell
$ sudo mount /dev/loop0p2 /mnt
```

没有消息就是最好的消息

### 安装 Grub

> 下面的操作具有一定的风险，请执行命令前一定要确认

此处示例将安装以 **BIOS 方式启动**的**i386 平台**的 Grub
使用上一个步骤的挂载点 `/mnt` 作为安装目录

```shell
$ sudo grub-install --target=i386-pc --boot-directory=/mnt
```

命令解释：
   - `target=i386-pc` 指出安装架构为 i386
   - `boot-directory=/mnt` 指出安装到 `/mnt` 而不是默认的 `/boot`

安装需要 13M 左右的空间
完成后会提示成功

### 配置

照着手册随便糊的一个，~~能用就行~~

```
# Grub 的配置文件

# 设置根分区
set root='(hd0,gpt2)'

# 选择启动项超时时间，0 即立即启动
set timeout=0

# 菜单选项
menuentry EntryName {
    # 载入 file 并启动
    multiboot ($root)/boot/file
    boot
}
```

### 卸载设备

```shell
$ sudo umount /mnt
$ sudo losetup -d /dev/loop0
```

上面的命令将从挂载点卸载 loop 设备
并释放 loop 设备与文件的关联

### 参考资料

- [Grub 手册](https://www.gnu.org/software/grub/manual/grub/grub.html)