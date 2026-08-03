---
type: 学习笔记
title: Linux 常见命令
created: 2026-08-03
updated: 2026-08-03
tags: [Linux, 命令, 基础, 运维, 目录, 文件, 压缩, 文本编辑]
subject: 部署/Linux
---

> Linux 命令是用户在终端中输入的文本指令，涵盖目录与文件管理、拷贝移动、压缩打包、文本编辑和查找等核心操作，熟练掌握这些命令是进行服务器部署和运维的前提。

## 目录

- [一、目录操作命令](#一目录操作命令)
- [二、文件操作命令](#二文件操作命令)
- [三、拷贝与移动命令](#三拷贝与移动命令)
- [四、打包与压缩命令](#四打包与压缩命令)
- [五、文本编辑命令](#五文本编辑命令)
- [六、查找命令](#六查找命令)
- [七、系统监控命令](#七系统监控命令)
- [八、权限管理命令](#八权限管理命令)
- [小结](#小结)

---

## 一、目录操作命令

### 查看目录内容

| 命令 | 作用 | 说明 |
|------|------|------|
| `ls` | 列出目录内容 | 默认列出当前目录 |
| `ls -l` | 长格式列出 | 显示权限、所有者、大小、修改时间 |
| `ls -a` | 显示隐藏文件 | 包括以 `.` 开头的文件 |
| `ls -lh` | 人性化显示大小 | 与 `-l` 结合，文件大小以 KB/MB 显示 |

```bash
ls -lh          # 详细列出当前目录，文件大小人性化显示
ls -la /etc     # 查看 /etc 目录下所有文件（含隐藏）
```

### 切换目录

| 命令 | 作用 |
|------|------|
| `cd /path` | 切换到指定目录 |
| `cd ..` | 返回上级目录 |
| `cd ~` 或 `cd` | 返回用户家目录 |
| `cd -` | 返回上一次所在目录 |

```bash
cd /var/log       # 切换到 log 目录
cd ..             # 返回上一级
cd ~              # 回到家目录
```

### 创建与删除目录

| 命令 | 作用 | 说明 |
|------|------|------|
| `mkdir dir` | 创建目录 | 一次创建一个 |
| `mkdir -p a/b/c` | 递归创建 | 父目录不存在时一并创建 |
| `rmdir dir` | 删除空目录 | 目录必须为空才能删除 |
| `rm -r dir` | 递归删除目录 | 可删除非空目录，**慎用** |

```bash
mkdir project        # 创建 project 目录
mkdir -p src/main/java   # 递归创建多层目录
rm -rf test_dir      # 强制递归删除目录（不可恢复）
```

> **注意**：`rm -rf` 删除不可恢复，执行前务必确认路径正确。

---

## 二、文件操作命令

### 查看文件内容

| 命令 | 作用 | 适用场景 |
|------|------|---------|
| `cat file` | 查看文件全部内容 | 小文件快速查看 |
| `less file` | 分页查看 | 大文件，可上下翻页（q 退出） |
| `head -n 10 file` | 查看前 10 行 | 查看文件头部 |
| `tail -n 10 file` | 查看后 10 行 | 查看文件尾部 |
| `tail -f file` | 实时追踪文件变化 | 监控日志文件 |

```bash
cat config.yml          # 直接输出全部内容
less /var/log/syslog    # 分页查看日志，空格翻页，q 退出
tail -f app.log         # 实时查看日志增长
tail -n 50 error.log    # 只看最后 50 行
```

### 创建与编辑文件

| 命令 | 作用 |
|------|------|
| `touch file` | 创建空文件（或更新修改时间） |
| `echo "内容" > file` | 写入内容到文件（覆盖） |
| `echo "内容" >> file` | 追加内容到文件 |

```bash
touch readme.txt            # 创建空文件
echo "Hello" > test.txt     # 写入并覆盖
echo "World" >> test.txt    # 追加内容
```

### 删除文件

| 命令 | 作用 |
|------|------|
| `rm file` | 删除文件 |
| `rm -f file` | 强制删除，不提示确认 |
| `rm -r dir` | 递归删除目录及其内容 |
| `rm -rf dir` | 强制递归删除（**危险**） |

```bash
rm old.txt          # 删除文件，会提示确认
rm -f temp.log      # 强制删除，不提示
```

### 查看文件信息

| 命令 | 作用 |
|------|------|
| `file file` | 查看文件类型 |
| `wc file` | 统计行数、单词数、字节数 |
| `stat file` | 查看文件详细元数据 |

```bash
file app.jar        # 查看文件类型
wc -l log.txt       # 统计 log.txt 的行数
```

---

## 三、拷贝与移动命令

### 拷贝文件/目录

| 命令 | 作用 |
|------|------|
| `cp src dest` | 拷贝文件 |
| `cp -r src dest` | 递归拷贝目录 |
| `cp -i src dest` | 拷贝前提示确认 |
| `cp -p src dest` | 保留原文件属性（时间、权限） |

```bash
cp config.yml backup/          # 拷贝文件到 backup 目录
cp -r project/ project_bak/    # 递归拷贝整个目录
cp -p important.txt /mnt/      # 拷贝并保留时间戳和权限
```

### 移动与重命名

| 命令 | 作用 |
|------|------|
| `mv src dest` | 移动文件或目录 |
| `mv old_name new_name` | 重命名文件 |

```bash
mv file.txt /tmp/           # 移动到 /tmp 目录
mv old.log new.log          # 重命名文件
mv data/ archive/           # 移动整个目录
```

> `mv` 在同一文件系统内是原子操作，速度极快；跨文件系统时相当于拷贝+删除。

---

## 四、打包与压缩命令

Linux 中最常用的压缩格式是 `.tar.gz`（tar 打包 + gzip 压缩）。

### tar 命令（打包/解包）

| 参数 | 作用 |
|------|------|
| `-c` | 创建归档 |
| `-x` | 解包 |
| `-f` | 指定文件名（必须放在参数最后） |
| `-z` | 通过 gzip 压缩/解压 |
| `-j` | 通过 bzip2 压缩/解压 |
| `-v` | 显示过程（verbose） |
| `-C` | 指定解包目标目录 |

```bash
# 压缩：将目录打包为 .tar.gz
tar -czvf archive.tar.gz project/
# -c 创建  -z gzip压缩  -v 显示过程  -f 指定文件名

# 解压：从 .tar.gz 解压到当前目录
tar -xzvf archive.tar.gz

# 解压到指定目录
tar -xzvf archive.tar.gz -C /tmp/extract/
```

### 其他压缩工具

| 命令 | 扩展名 | 说明 |
|------|--------|------|
| `gzip file` / `gunzip file.gz` | `.gz` | 单独压缩文件，不保留原文件 |
| `zip -r out.zip dir/` / `unzip out.zip` | `.zip` | 兼容 Windows，广泛使用 |
| `bzip2 file` / `bunzip2 file.bz2` | `.bz2` | 压缩率更高，速度更慢 |

```bash
# zip 压缩（保留源文件）
zip -r backup.zip project/

# unzip 解压
unzip backup.zip

# 查看压缩包内容（不解压）
tar -tzvf archive.tar.gz
zipinfo backup.zip
```

---

## 五、文本编辑命令

### vi / vim（命令行编辑器）

vim 有三种工作模式，是 Linux 最基础的文本编辑器：

| 模式 | 进入方式 | 说明 |
|------|---------|------|
| 普通模式 | 默认 | 移动光标、删除、复制 |
| 插入模式 | 按 `i` | 编辑文本内容 |
| 命令模式 | 按 `:` | 保存、退出、查找替换 |

**常用操作**：

```
i        # 进入插入模式（在光标前插入）
I        # 在行首插入
a        # 在光标后插入
Esc      # 返回普通模式

dd       # 删除当前行
yy       # 复制当前行
p        # 粘贴（插入到光标后）
u        # 撤销上一步操作
gg       # 跳到文件开头
G        # 跳到文件末尾

:w!      # 强制保存并退出（写文件）
:q       # 退出
:q!      # 强制退出（不保存）
:%s/old/new/g   # 全文替换 old 为 new
```

### nano（轻量编辑器）

nano 比 vim 更易上手，适合新手：

```bash
nano file.txt        # 打开或创建文件
Ctrl+O               # 保存
Ctrl+X               # 退出
Ctrl+K               # 删除当前行
```

提示：底部有操作快捷键提示，直观易用。

---

## 六、查找命令

### find（按条件查找文件）

```bash
# 在当前目录查找所有 .log 文件
find . -name "*.log"

# 查找大于 100MB 的文件
find / -size +100M

# 查找最近 7 天内修改过的文件
find . -mtime -7

# 查找并删除匹配文件（谨慎使用）
find . -name "*.tmp" -delete

# 查找可执行文件
find /usr -type f -perm +111
```

常用参数：
- `-name`：按文件名匹配（支持 `*` 通配符）
- `-type`：按类型查找（`f` 文件、`d` 目录、`l` 软链接）
- `-size`：按文件大小（`+100M` 大于 100MB，`-7d` 7 天内）
- `-mtime`：按修改时间（单位：天）
- `-exec`：对查找结果执行命令

### grep（文本内容搜索）

```bash
# 在文件中搜索包含 "error" 的行
grep "error" app.log

# 递归搜索目录中所有文件的匹配内容
grep -r "TODO" ./src/

# 忽略大小写搜索
grep -i "warning" log.txt

# 显示匹配行及其上下文（前后 3 行）
grep -C 3 "exception" app.log

# 统计匹配行数
grep -c "fail" *.log

# 只显示文件名（不显示匹配内容）
grep -l "password" *.conf
```

常用参数：
- `-r`：递归搜索目录
- `-i`：忽略大小写
- `-n`：显示行号
- `-v`：反向匹配（显示不匹配的行）
- `-C n`：显示前后 n 行上下文

### locate（快速查找，基于索引）

```bash
locate filename      # 快速查找文件（需更新数据库）
sudo updatedb        # 更新 locate 数据库
```

> `locate` 比 `find` 快很多，因为它查询的是预建索引，但可能存在短暂延迟（索引不是实时同步）。

### which / whereis（查找命令位置）

```bash
which java          # 查找 java 命令的完整路径
whereis python3     # 查找命令的二进制、源码、手册位置
```

---

## 七、系统监控命令

| 命令 | 作用 |
|------|------|
| `df -h` | 查看磁盘使用情况（人性化显示） |
| `du -sh dir` | 查看目录占用空间 |
| `free -h` | 查看内存使用情况 |
| `top` / `htop` | 查看进程和 CPU 使用情况 |
| `ps aux` | 查看所有进程列表 |
| `kill PID` | 终止指定进程 |
| `uname -a` | 查看系统信息 |
| `whoami` | 查看当前用户 |
| `hostname` | 查看主机名 |

```bash
df -h                 # 磁盘空间使用情况
free -h               # 内存使用情况
top                   # 实时进程监控（按 q 退出）
ps aux | grep java    # 查找 java 相关进程
kill -9 1234          # 强制终止 PID 为 1234 的进程
uptime                # 查看系统运行时间
```

---

## 八、权限管理命令

| 命令 | 作用 |
|------|------|
| `chmod 755 file` | 修改文件权限 |
| `chown user:group file` | 修改文件所有者 |
| `sudo command` | 以 root 权限执行命令 |

```bash
chmod 755 script.sh     # 所有者可读写执行，其他人可读执行
chmod +x app            # 添加可执行权限
chown -R www-data:www-data /var/www    # 递归修改目录所有者
```

权限数字说明：
- `4` = 读（r）
- `2` = 写（w）
- `1` = 执行（x）
- `7` = 读写执行（rwx）
- 格式：`所有者 组用户 其他人`，如 `755` = `rwx r-x r-x`

---

## 小结

1. **目录操作**：`ls`、`cd`、`mkdir`、`rmdir` 是日常最常用的基础命令
2. **文件操作**：`cat`、`less`、`head`、`tail -f` 覆盖查看文件的各种场景
3. **拷贝移动**：`cp -r` 和 `mv` 是文件管理中最频繁使用的操作
4. **压缩打包**：`tar -czvf` 打包、`tar -xzvf` 解包是最核心的用法，`zip/unzip` 次之
5. **文本编辑**：`vim` 功能强大但学习曲线陡，`nano` 更易上手
6. **查找命令**：`find` 按条件查找，`grep` 按内容搜索，两者配合使用效率最高
7. **系统监控**：`df -h`、`free -h`、`top` 用于日常资源诊断
8. **权限**：`chmod` 和 `chown` 是权限管理核心，`sudo` 用于临时提权

> 建议在 WSL 或虚拟机中多动手练习，命令类知识只有实际操作过才能内化。
