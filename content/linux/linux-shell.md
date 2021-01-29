---
title: 'Shell 编程实用句式'
date: 2021-01-29T23:54:37+08:00
draft: false
---

检查是否以 root 用户执行。

```shell
# check if run as root user
if [[ `id -u` -ne 0 ]]; then
  echo "You need root privileges to run this script."
fi
```

获取正在执行脚本的绝对路径，注意直接用 $0 或 pwd 获取的可能都不要你想要的。

```shell
current_dir=$(cd `dirname $0`;pwd)
```

为当前目录包含子目录下所有 .sh 文件增加可执行权限。

```shell
chmod +x `find . -name '*.sh'`
```

将提示信息显示到终端（控制台），同时也写入到文件里。

```shell
log_file=/var/log/test.log
echo "This line will echo to console and also write to log file." | tee -a ${log_file}
```

类似于 Java properties 中 key=value 形式的字符串，取 key 和 value 的值。

```shell
username_line="username=test"
#key is username
key=${username_line%=*}
#val is test
val=${username_line#*=}
```

实现 trim 效果。

```shell
#trim string by echo
val_trim=$(echo -n ${val})
```

解压缩，以下示例请根据文件名后缀自行选择解压缩命令。

```shell
tar -xf test.tar
gzip -d test.gz
gunzip test.gz
tar -xzf test.tar.gz
bzip2 -d test.bz2
bunzip2 test.bz2
tar -xjf test.tar.bz2
```

压缩，以下示例请根据需要选择压缩算法。

```shell
# 将当前目录下所有jpg格式的文件打包为pictures.tar
tar -cf pictures.tar *.jpg
# 将Picture目录下所有文件打包并用gzip压缩为pictures.tar.gz
tar -czf pictures.tar.gz Picture/
# 将Picture目录下所有文件打包并用bzip2压缩为pictures.tar.bz2
tar -cjf pictures.tar.bz2 Picture/
```

字体输出颜色及终端格式控制

```shell
#字体色范围：30-37
echo -e "\033[30m 黑色字 \033[0m"
echo -e "\033[31m 红色字 \033[0m"
echo -e "\033[32m 绿色字 \033[0m"
echo -e "\033[33m 黄色字 \033[0m"
echo -e "\033[34m 蓝色字 \033[0m"
echo -e "\033[35m 紫色字 \033[0m"
echo -e "\033[36m 天蓝字 \033[0m"
echo -e "\033[37m 白色字 \033[0m"
#字背景颜色范围：40-47
echo -e "\033[40;37m 黑底白字 \033[0m"
echo -e "\033[41;30m 红底黑字 \033[0m"
echo -e "\033[42;34m 绿底蓝字 \033[0m"
echo -e "\033[43;34m 黄底蓝字 \033[0m"
echo -e "\033[44;30m 蓝底黑字 \033[0m"
echo -e "\033[45;30m 紫底黑字 \033[0m"
echo -e "\033[46;30m 天蓝底黑字 \033[0m"
echo -e "\033[47;34m 白底蓝字 \033[0m"
```
