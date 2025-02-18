---
title: 对linux磁盘进行无损扩容
date: 2024-08-05 20:37:35
categories: linux
tag: 扩容磁盘
---

执行 lsblk 命令得到的数据如下
   ```bash
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0  150G  0 disk 
├─sda1   8:1    0   49G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1  4.6G  0 rom  
   ```
此时，想将sda1从49G扩容到150G。进行无损扩展 `/dev/sda1` 的步骤较为复杂，但可以通过以下步骤来安全地扩展分区而不丢失数据。这包括使用 `fdisk` 或 `parted` 工具来调整分区大小，然后扩展文件系统。

### 无损扩展分区的步骤

#### 1. **备份数据**

在进行任何磁盘操作之前，务必备份重要数据以防万一。

#### 2. **使用 `parted` 调整分区大小**

假设现有的 `/dev/sda1` 是 `ext4` 文件系统。

1. **查看当前磁盘布局**

   ```bash
   lsblk
   sudo fdisk -l /dev/sda
   ```

2. **调整分区大小**

   使用 `parted` 调整分区大小。

   ```bash
   sudo parted /dev/sda
   ```

   在 `parted` 交互界面中执行以下步骤：

   ```bash
   (parted) print
   ```

   确认分区表，并记下 `/dev/sda1` 的起始位置（例如 `2048s`）。

   ```bash
   (parted) resizepart 1 150GB
   ```

   这会扩展分区1到150GB。

   ```bash
   (parted) quit
   ```

#### 3. **更新分区表**

有时，系统需要重新加载分区表。

```bash
sudo partprobe /dev/sda
```

#### 4. **扩展文件系统**

根据你的文件系统类型，执行以下步骤来扩展文件系统。

##### 对于 `ext4` 文件系统：

```bash
sudo resize2fs /dev/sda1
```

##### 对于 `xfs` 文件系统：

```bash
sudo xfs_growfs /dev/sda1
```

#### 5. **验证扩展是否成功**

```bash
df -h
```

### 示例步骤

完整的命令和步骤如下：

```bash
# 查看当前磁盘布局
lsblk
sudo fdisk -l /dev/sda

# 调整分区大小
sudo parted /dev/sda

# 在 parted 交互界面中
(parted) print
(parted) resizepart 1 150GB
(parted) quit

# 更新分区表
sudo partprobe /dev/sda

# 扩展文件系统（假设是 ext4 文件系统）
sudo resize2fs /dev/sda1

# 验证扩展是否成功
df -h
```

### 注意事项

- 确保你的 `/dev/sda1` 分区后面有足够的未分配空间来扩展到 150GB。
- 在操作之前，确保系统分区未被使用或者在单用户模式下进行操作，以防止数据损坏。
- 如果你无法进入系统，可以通过 live CD/USB 启动系统并执行这些操作。

这些步骤旨在提供一种相对安全的方法来无损扩展分区。如果对操作不熟悉，建议请教专业人士或使用专业的分区管理软件进行操作。