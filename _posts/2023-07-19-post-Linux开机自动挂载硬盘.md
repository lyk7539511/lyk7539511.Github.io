---
title: "Ubuntu开机自动挂载硬盘到非root权限目录"
categories:
  - Blog
  - Tech
tags:
  - Linux
  - Mount Disk
  - Ubuntu
---

# 1. 查看硬盘信息

```bash
sudo fdisk -l
```

找到你要挂载的硬盘分区，比如我的是`/dev/sda1`，然后查看硬盘的UUID。
这里一定要注意，不是硬盘，而是硬盘分区的UUID。

```bash
sudo blkid /dev/sda1

# 结果应该是这样的
/dev/sda1: UUID="0145ed18-dfd5-644c-8895-xxxxxxxxx" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux data partition" PARTUUID="cbab8849-86bc-406b-a58b-0xxxxxxxxxx"
```

# 2. 创建挂载目录

在任何你想要挂载的地方创建一个目录，比如我创建在`/home/username`下。

```bash
mkdir DiskUp
```

此时，该文件夹的所属用户与组应该是你当前用户，而不是root用户。

# 3. 修改`/etc/fstab`

```bash
sudo vim /etc/fstab
# 在文件末尾添加一行
UUID=0145ed18-dfd5-644c-8895-xxxxxxxxx /home/username/DiskUp ext4 defaults 0 0
```

# 4. 重启

```bash
sudo reboot
```

# 5. 检查

此时你应该能看到你的硬盘已经挂载到了`/home/username/DiskUp`下。

但是，该文件夹的所属用户与组被自动修改为root用户，而不是你当前用户，因此当前没有权限访问该文件夹。

```bash
ls -la
# 结果应该是这样的
drwxr-xr-x  1 root root 4096  7月 19 16:00 DiskUp
```

# 6. 修改所属用户与组以及权限

```bash
sudo chown -R username:username DiskUp
chmod -R 775 DiskUp
```

# 7. 检查

此时你的硬盘就挂载到了`/home/username/DiskUp`下，并且你有权限访问该文件夹。

```bash
ls -la
# 结果应该是这样的
drwxrwxr-x  1 username username 4096  7月 19 16:00 DiskUp
```

重启后，硬盘依然会自动挂载到`/home/username/DiskUp`下，并且权限不变。