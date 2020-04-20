---
layout: post
title: "在MacOS下配置AnyQ"
subtitle: '使用AnyQ搭建智能问答系统'
author: "lingjye"
header-style: 'text'
tags:
  - iOS
---

[AnyQ](https://github.com/baidu/AnyQ)

### 代码编译：

##### 安装Docker

由于AnyQ基于Linux，所以这里需要使用Docker安装。

下载 [Docker](https://www.docker.com/){:target="_blank"} 并安装，启动Docker，选择 Preferences -> Advanced, 配置CPUs：4，Memory：7.0GiB，Swap:3.0GiB, 尽量大一些（至少8G）。

##### 启动Docker

命令行执行：

```
# paddle国内镜像
docker pull hub.baidubce.com/paddlepaddle/paddle:latest-dev

```

之后控制台输出：

```
bogon:anyq chunsheng$ docker run -t -i hub.baidubce.com/paddlepaddle/paddle:latest-dev /bin/bash
λ d4bbeb1b5c3d /home
```

##### 克隆AnyQ

```
git clone https://github.com/keejo125/AnyQ.git
```

注：这里没有使用原版代码了，因为原版代码的cmake中xgboost部分未指定版本，会导致后面编译出错。

如果拉取了原版git，那么需要进行如下修改：

```
# 安装vim
apt-get install vim
# 修改xgboost.cmake
cd AnyQ/cmake/external
vim xgboost.cmake
```

修改部分为：

```
DOWNLOAD_COMMAND git clone --recursive https://github.com/dmlc/xgboost.git && cd xgboost && git checkout v0.81
```

等待下载完成：

```
λ d4bbeb1b5c3d /home git clone https://github.com/keejo125/AnyQ.git
Cloning into 'AnyQ'...
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 748 (delta 0), reused 0 (delta 0), pack-reused 743
Receiving objects: 100% (748/748), 5.53 MiB | 37.00 KiB/s, done.
Resolving deltas: 100% (406/406), done.
Checking connectivity... done.
λ d4bbeb1b5c3d /home 
```

##### 编译并执行

进入AnyQ，创建并进入build

```
cd AnyQ
mkdir build && cd build
```

执行cmake

```
cmake ..
```

编译

```
make
```

* 关于g++版本低于4.8.2的问题，请执行以下命令修改
    
```
cd /home/AnyQ/build/third_party/lac/src/lac/
vim CMakeLists.txt
```

将11行修改为:

```
if (GCC_VERSION VERSION_LESS 4.8)
```

控制台：

```
λ d4bbeb1b5c3d /home cd AnyQ
λ d4bbeb1b5c3d /home/AnyQ {master} mkdir build && cd build
λ d4bbeb1b5c3d /home/AnyQ/build {master} cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/AnyQ/build
λ d4bbeb1b5c3d /home/AnyQ/build {master} make
Scanning dependencies of target extern_leveldb
[  1%] Creating directories for 'extern_leveldb'
[  1%] Performing download step for 'extern_leveldb'
Cloning into 'leveldb'...
```

等待下载完成。

##### 安装jdk1.8

进入tmp

```
cd /tmp
```

下载jdk

```
wget http://anyq.bj.bcebos.com/jdk-8u171-linux-x64.tar.gz
```

解压

```
tar -zxvf jdk-8u171-linux-x64.tar.gz 
```

移动

```
mv /tmp/jdk1.8.0_171 /home/jdk1.8.0_171
```

配置

```
vim /etc/profile
```

如果提示：`bash: vim: command not found`,使用下面命令安装vim

```
apt-get install vim
```

然后增加：

```
#set java env
export JAVA_HOME=/home/jdk1.8.0_171
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

然后回到AnyQ目录下

```
cd /home/AnyQ/build
```

配置

```
# 获取anyq定制solr，anyq示例配置
cp ../tools/anyq_deps.sh .
sh anyq_deps.sh
```

启动solr, 依赖python-json, jdk>=1.8

```
cp ../tools/solr -rp solr_script
sh solr_script/anyq_solr.sh solr_script/sample_docs
```

##### 创建镜像

新建命令行窗口：

查看所有容器

```
docker ps -a
```

例如：

```
localhost:~ chunsheng$ docker ps -a
CONTAINER ID        IMAGE                                             COMMAND                  CREATED             STATUS                     PORTS               NAMES
d4bbeb1b5c3d        hub.baidubce.com/paddlepaddle/paddle:latest-dev   "/bin/bash"              4 hours ago         Up 4 hours                 22/tcp 
```

复制 CONTAINER ID

提交AnyQ镜像：

```
docker commit d4bbeb1b5c3d anyq/anyq
```

其中d4bbeb1b5c3d为CONTAINER ID， anyq/anyq为镜像名字


##### 运行镜像

在AnyQ工程的parent目录下，

```
docker run -p 8999:8999 -p 8900:8900 -i -t -v $(PWD)/AnyQ:/AnyQ anyq/anyq
```


**参考：**

[AnyQ 是什么](https://zhuanlan.zhihu.com/p/55403810){:target="_blank"}

[AnyQ之编译安装](http://keejo.coding.me/AnyQ%E4%B9%8B%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85.html){:target="_blank"}

