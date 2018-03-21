# Shell编程实用句式总结

检查是否以root用户执行。
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

类似于Java properties中key=value形式的字符串，取key和value的值。
```shell
username_line="username=test"
#key is username
key=${username_line%=*}
#val is test
val=${username_line#*=}
```

实现trim效果。
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
