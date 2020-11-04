---
title: go build and  deploy
date: 2020-10-28 14:16:33
categories:
	- 编程实战
tags:
	- 编程
	- GOLANG
	- DOCKER
---

# 1. 准备环境

CentOS本身无法用yum直接安装Golang，需要设置源地址或手动安装。

## 1.1 安装Golang

### 1.1.1 手动安装

Golang默认安装在`/usr/local/go` 下，所以下载好安装包直接解压到该目录下，然后导入到环境变量中。

```
wget https://golang.org/doc/install?download=go1.15.3.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.15.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
```

卸载的话可以直接删除该文件夹。

### 1.1.2 安装源

```
rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO
curl -s https://mirror.go-repo.io/centos/go-repo.repo | tee /etc/yum.repos.d/go-repo.repo
yum install golang
```

## 1.2 安装Docker

```
cat /etc/redhat-release
yum install -y gcc gcc-g++
yum install -y yum-utils device-mapper-persistent-data lvm2
添加docker仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo    
yum makecache fast
yum install -y docker-ce docker-ce-cli containerd.io
配置镜像仓库
vim /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com"
  ]
}
```

## 1.3 编译代码

与Java不同的，Java在不同平台会生成能够跨平台执行的字节码，而Golang在不同平台生成的二进制执行文件，是不能跨平台执行的。

所以，需要在Linux下编译代码。

```
You can try one of the following:
Method 1:
* Assume that your project name is MyProject
* Go to your path, run go build
* It will create an executable file as your project name ("MyProject")
* Then run the executable using ./MyProject
You can do both steps at once by typing go build && ./MyProject. Go files of the package main are compiled to an executable.
Method 2:
* Just run go run *.go. It won't create any executable but it runs.
运行：go run
测试：go test
编译：go build
```

Go分为单个文件执行，多个文件在一个package下执行，或整个项目编译执行（file，pacakge，directory）

File只能运行单独一个文件的代码，而package能把多个文件链接起来运行，Directory是从可以控制从目录名开始运行，层级逐渐升高！

### 1.3.1 异地编译

在windows平台进行编译，目标系统为linux，目标平台是x64：

```
SET CGO_ENABLED=0
SET GOOS=linux			#目标操作系统
SET GOARCH=amd64		#目标平台
go build main.go
```

在linxu平台进行编译，目标系统为windows，目标平台是x86：

```
export CGO_ENABLED=0
export GOOS=windows	
export GOARCH=386
go build main.go
```

其他平台、系统，自行替换其中环境变量的值即可。

## 1.4 制作Dockerfile

```
FROM alpine:latest        					 #使用基础镜像
MAINTAINER hujunchina@outlook.com    #维护者
WORKDIR /home/src/myproject             #工作目录
ADD ./myprojectbin /home/src/myproject
EXPOSE 1883
ENTRYPOINT ["./myprojectbin"]					#执行程序
```

创建并运行镜像：

```
docker build -t "hl:v5" .
docker run -it --name hl5 -p 1883:1883 imageid
```

### 1.4.1 注意

如果项目里没有用类似Mqtt的依赖库，可以先禁止CGO_ENABLED=0，再go build，可以在alpine基础镜像上运行成功。

但是如果有类似mqtt的基于cgo编译的依赖库，就需要使用可以运行c环境的镜像，在alpine是无法运行的。