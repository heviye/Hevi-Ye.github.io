---
title:  "在阿里云的ECS上手动部署Hyperledger-fabric网络"
tags: hyperledger
---

## 基础环境

* CentOS - v7.7
* Git - v1.8.3.1
* Golang - v1.13.3
* Docker - v19.03.5
* Docker-Compose - v1.25.0
* Hyperledger-fabric - v1.4.4



## 一、更新源

```sh
$ yum update
```



## 二、安装Git

```sh
$ yum install git
# 查看版本
$ git --version
```



## 三、安装Golang

```sh
$ yum install golang
```

查看环境

```sh
$ go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/root/.cache/go-build"
GOENV="/root/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/root/go"
GOPRIVATE=""
GOPROXY="direct"
GOROOT="/usr/lib/golang"
GOSUMDB="off"
GOTMPDIR=""
GOTOOLDIR="/usr/lib/golang/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build981408195=/tmp/go-build -gno-record-gcc-switches"
```

配置goang

```sh
# 进入`.bashrc`文件
$ cd ~
$ vim .bashrc
# 增加下列内容
export GOROOT=/usr/lib/golang
export GOPATH=/root/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

# 保存退出
# 使用配置生效
$ source .bashrc
```



## 四、安装Docker

查看这个文档https://help.aliyun.com/document_detail/60742.html

```sh
# step 1: 安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2

# Step 2: 添加软件源信息
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Step 3: 更新并安装Docker-CE
yum makecache fast
yum -y install docker-ce

# Step 4: 开启Docker服务
service docker start

```

查看安装情况

```sh
$ docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:25:41 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea
  Built:            Wed Nov 13 07:24:18 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

设置镜像加速器

登录你的容器镜像服务控制台https://cr.console.aliyun.com/cn-shenzhen/instances/mirrors

```shell
vim /etc/docker/daemon.json

# 加入以下内容
{
  "registry-mirrors": ["https://rz50yojt.mirror.aliyuncs.com"]
}
# 加载
systemctl daemon-reload
# 重启docker
systemctl restart docker

```



## 五、安装Docker-Compose

```sh
curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# 查看版本
docker-compose version
```

下载太慢的话，建议本地下载再上传给服务器



## 六、客隆Fabric仓库

```sh
mkdir -p $GOPATH/src/github.com/hyperledger/
cd $GOPATH/src/github.com/hyperledger/
git clone https://github.com/hyperledger/fabric.git
# 切换版本
cd fabric
git checkout -b v1.4.4 v1.4.4
```



## 七、客隆相关依赖

```sh
# 执行成功后会在`$GOPATH/bin`下生成「protoc-gen-go」文件
go get github.com/golang/protobuf/protoc-gen-go
# 将上面生成的文件复制到这个目录来
mkdir -p $GOPATH/src/github.com/hyperledger/fabric/.build/docker/gotools/bin
```



## 八、编译Fabric模块

```sh
cd fabric
make release
```



## 九、下载镜像

```sh
make docker
```

最终会在`$GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin`这个目录生成以下二进制文件

```sh
configtxgen  configtxlator  cryptogen  discover  idemixgen  orderer  peer
```

将这些文件复制到`/usr/local/bin`下载，方便直接使用