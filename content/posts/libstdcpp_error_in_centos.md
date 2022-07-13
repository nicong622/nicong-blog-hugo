---
title: "Centos 中缺少 CXXABI_1.3.8 的问题"
date: 2022-07-13T16:29:30+08:00
draft: false
toc: false
images:
tags: 
  - centos
  - linux
---

最近在 Centos7.6 中利用 [sequelize](https://sequelize.org/) 连接 sqlite 的时候出现了如下错误：

```shell
/lib64/libstdc++.so.6: version `CXXABI_1.3.8' not found
```

出现这个错误是因为CentOS7当前版本默认的GCC的版本太老，里面的动态链接库没有CXXABI_1.3.8。

执行下面的命令可以看到没有找到 `CXXABI_1.3.8`：

```shell
strings /usr/lib64/libstdc++.so.6 | grep CXXABI
```

## 解决方案

```sh
cd /usr/local/lib64/
# 下载最新版本的`下载最新版本的libstdc.so_.6.0.26`
wget http://www.vuln.cn/wp-content/uploads/2019/08/libstdc.so_.6.0.26.zip
# 解压
unzip libstdc.so_.6.0.26.zip
# 将下载的最新版本拷贝到 /usr/lib64
cp libstdc++.so.6.0.26 /usr/lib64
cd  /usr/lib64
# 查看 /usr/lib64下libstdc++.so.6链接的版本
ls -l | grep libstdc++
# 删除原先的软连接(不放心可以备份)
rm libstdc++.so.6
# 使用最新的库建立软连接
ln -s libstdc++.so.6.0.26 libstdc++.so.6
# 查看新版本，成功
strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX

```

