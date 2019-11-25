---
layout: post
title:  "在阿里云上搭建hyperledger-fabric版本的环境"
date:   2019-11-25 11:07:54
categories: 区块链
tags: hyperledger,fabric
---

## 基础资源

* CentOS 7
* Hyperledger-fabric版本：v1.4.4

## 一、更新源

```shell
yum update
```

## 二、安装Git

```shell
yum install git
```

## 三、安装docker

查看这个文档https://help.aliyun.com/document_detail/60742.html

```shell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start
```

查看安装状态

```shell
docker version
```

出现以下内容说明安装成功

```shell
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



## 四、设置docker的镜像加速器

登录你的容器镜像服务控制台https://cr.console.aliyun.com/cn-shenzhen/instances/mirrors

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://rz50yojt.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 五、安装Docker Compose

查看最新版本https://github.com/docker/compose/releases

```shell
curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

太慢的话把`https`改成`http,或者本地下载再上传给服务器`

## 六、安装1.4.4版本的fabric-samples，这一步会做以下三件事

1. 客隆`fabric-samples`项目并切换到指定版本，如1.4.4
2. 下载二进制文件
3. 下载镜像文件

```shell
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh | bash -s -- 1.4.4 1.4.4 0.4.18
```

以上命令如果在网速很慢的机器上时，要等待很久，建议fabric-samples和二进制文件单独客隆和下载

* 单独客隆

```shell
git clone -b master https://github.com/hyperledger/fabric-samples.git && cd fabric-samples && git checkout v1.4.4
```

* 下载二进制文件，选择自己的系统版本

```
https://github.com/hyperledger/fabric/releases/tag/v1.4.4
```

* 下载镜像文件

```shell
#!/bin/bash
#
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

# if version not passed in, default to latest released version
VERSION=1.4.4
# if ca version not passed in, default to latest released version
CA_VERSION=1.4.4
# current version of thirdparty images (couchdb, kafka and zookeeper) released
THIRDPARTY_IMAGE_VERSION=0.4.18
ARCH=$(echo "$(uname -s|tr '[:upper:]' '[:lower:]'|sed 's/mingw64_nt.*/windows/')-$(uname -m | sed 's/x86_64/amd64/g')")
MARCH=$(uname -m)

printHelp() {
    echo "Usage: bootstrap.sh [version [ca_version [thirdparty_version]]] [options]"
    echo
    echo "options:"
    echo "-h : this help"
    echo "-d : bypass docker image download"
    echo "-s : bypass fabric-samples repo clone"
    echo "-b : bypass download of platform-specific binaries"
    echo
    echo "e.g. bootstrap.sh 1.4.4 -s"
    echo "would download docker images and binaries for version 1.4.4"
}

# dockerPull() pulls docker images from fabric and chaincode repositories
# note, if a docker image doesn't exist for a requested release, it will simply
# be skipped, since this script doesn't terminate upon errors.

dockerPull() {
    image_tag=$1
    shift
    while [[ $# -gt 0 ]]
    do
        image_name="$1"
        echo "====> hyperledger/fabric-$image_name:$image_tag"
        docker pull "hyperledger/fabric-$image_name:$image_tag"
        docker tag "hyperledger/fabric-$image_name:$image_tag" "hyperledger/fabric-$image_name"
        shift
    done
}

cloneSamplesRepo() {
    # clone (if needed) hyperledger/fabric-samples and checkout corresponding
    # version to the binaries and docker images to be downloaded
    if [ -d first-network ]; then
        # if we are in the fabric-samples repo, checkout corresponding version
        echo "===> Checking out v${VERSION} of hyperledger/fabric-samples"
        git checkout v${VERSION}
    elif [ -d fabric-samples ]; then
        # if fabric-samples repo already cloned and in current directory,
        # cd fabric-samples and checkout corresponding version
        echo "===> Checking out v${VERSION} of hyperledger/fabric-samples"
        cd fabric-samples && git checkout v${VERSION}
    else
        echo "===> Cloning hyperledger/fabric-samples repo and checkout v${VERSION}"
        git clone -b master https://github.com/hyperledger/fabric-samples.git && cd fabric-samples && git checkout v${VERSION}
    fi
}

# This will download the .tar.gz
download() {
    local BINARY_FILE=$1
    local URL=$2
    echo "===> Downloading: " "${URL}"
    curl -s -L "${URL}" | tar xz || rc=$?
    if [ -n "$rc" ]; then
        echo "==> There was an error downloading the binary file."
        return 22
    else
        echo "==> Done."
    fi
}

pullBinaries() {
    echo "===> Downloading version ${FABRIC_TAG} platform specific fabric binaries"
    download "${BINARY_FILE}" "https://github.com/hyperledger/fabric/releases/download/v${VERSION}/${BINARY_FILE}"
    if [ $? -eq 22 ]; then
        echo
        echo "------> ${FABRIC_TAG} platform specific fabric binary is not available to download <----"
        echo
        exit
    fi

    echo "===> Downloading version ${CA_TAG} platform specific fabric-ca-client binary"
    download "${CA_BINARY_FILE}" "https://github.com/hyperledger/fabric-ca/releases/download/v${VERSION}/${CA_BINARY_FILE}"
    if [ $? -eq 22 ]; then
        echo
        echo "------> ${CA_TAG} fabric-ca-client binary is not available to download  (Available from 1.1.0-rc1) <----"
        echo
        exit
    fi
}

pullDockerImages() {
    command -v docker >& /dev/null
    NODOCKER=$?
    if [ "${NODOCKER}" == 0 ]; then
        FABRIC_IMAGES=(peer orderer ccenv tools)
        case "$VERSION" in
        1.*)
            FABRIC_IMAGES+=(javaenv)
            shift
            ;;
        2.*)
            FABRIC_IMAGES+=(nodeenv baseos javaenv)
            shift
            ;;
        esac
        echo "FABRIC_IMAGES:" "${FABRIC_IMAGES[@]}"
        echo "===> Pulling fabric Images"
        dockerPull "${FABRIC_TAG}" "${FABRIC_IMAGES[@]}"
        echo "===> Pulling fabric ca Image"
        CA_IMAGE=(ca)
        dockerPull "${CA_TAG}" "${CA_IMAGE[@]}"
        echo "===> Pulling thirdparty docker images"
        THIRDPARTY_IMAGES=(zookeeper kafka couchdb)
        dockerPull "${THIRDPARTY_TAG}" "${THIRDPARTY_IMAGES[@]}"
        echo
        echo "===> List out hyperledger docker images"
        docker images | grep hyperledger
    else
        echo "========================================================="
        echo "Docker not installed, bypassing download of Fabric images"
        echo "========================================================="
    fi
}

DOCKER=true
SAMPLES=true
BINARIES=true

# Parse commandline args pull out
# version and/or ca-version strings first
if [ -n "$1" ] && [ "${1:0:1}" != "-" ]; then
    VERSION=$1;shift
    if [ -n "$1" ]  && [ "${1:0:1}" != "-" ]; then
        CA_VERSION=$1;shift
        if [ -n  "$1" ] && [ "${1:0:1}" != "-" ]; then
            THIRDPARTY_IMAGE_VERSION=$1;shift
        fi
    fi
fi

# prior to 1.2.0 architecture was determined by uname -m
if [[ $VERSION =~ ^1\.[0-1]\.* ]]; then
    export FABRIC_TAG=${MARCH}-${VERSION}
    export CA_TAG=${MARCH}-${CA_VERSION}
    export THIRDPARTY_TAG=${MARCH}-${THIRDPARTY_IMAGE_VERSION}
else
    # starting with 1.2.0, multi-arch images will be default
    : "${CA_TAG:="$CA_VERSION"}"
    : "${FABRIC_TAG:="$VERSION"}"
    : "${THIRDPARTY_TAG:="$THIRDPARTY_IMAGE_VERSION"}"
fi

BINARY_FILE=hyperledger-fabric-${ARCH}-${VERSION}.tar.gz
CA_BINARY_FILE=hyperledger-fabric-ca-${ARCH}-${CA_VERSION}.tar.gz

# then parse opts
while getopts "h?dsb" opt; do
    case "$opt" in
        h|\?)
            printHelp
            exit 0
            ;;
        d)  DOCKER=false
            ;;
        s)  SAMPLES=false
            ;;
        b)  BINARIES=false
            ;;
    esac
done

if [ "$SAMPLES" == "true" ]; then
    echo
    echo "Clone hyperledger/fabric-samples repo"
    echo
   #  cloneSamplesRepo
fi
if [ "$BINARIES" == "true" ]; then
    echo
    echo "Pull Hyperledger Fabric binaries"
    echo
    # pullBinaries
fi
if [ "$DOCKER" == "true" ]; then
    echo
    echo "Pull Hyperledger Fabric docker images"
    echo
    pullDockerImages
fi
```

将上面的文本内存放进`bootstrap.sh`的文件中，文本内容已经注释掉客隆fabric-samples和下载二进制文件的代码

```shell
vim bootstrap.sh
# 将内容复制到这个文件中
# 保存退出，修改文件权限
chmod +x bootstrap.sh
# 执行该文件，下载镜像文件
./bootstrap.sh
# 查询下载的镜像文件
docker images | grep hyperledger
```

```shell
$ docker images | grep hyperledger
hyperledger/fabric-javaenv                       1.4.4               4648059d209e        8 days ago          1.7GB
hyperledger/fabric-javaenv                       latest              4648059d209e        8 days ago          1.7GB
hyperledger/fabric-ca                            1.4.4               62a60c5459ae        9 days ago          150MB
hyperledger/fabric-ca                            latest              62a60c5459ae        9 days ago          150MB
hyperledger/fabric-tools                         1.4.4               7552e1968c0b        10 days ago         1.49GB
hyperledger/fabric-tools                         latest              7552e1968c0b        10 days ago         1.49GB
hyperledger/fabric-ccenv                         1.4.4               ca4780293e4c        10 days ago         1.37GB
hyperledger/fabric-ccenv                         latest              ca4780293e4c        10 days ago         1.37GB
hyperledger/fabric-orderer                       1.4.4               dbc9f65443aa        10 days ago         120MB
hyperledger/fabric-orderer                       latest              dbc9f65443aa        10 days ago         120MB
hyperledger/fabric-peer                          1.4.4               9756aed98c6b        10 days ago         128MB
hyperledger/fabric-peer                          latest              9756aed98c6b        10 days ago         128MB
hyperledger/fabric-zookeeper                     0.4.18              ede9389347db        2 weeks ago         276MB
hyperledger/fabric-zookeeper                     latest              ede9389347db        2 weeks ago         276MB
hyperledger/fabric-kafka                         0.4.18              caaae0474ef2        2 weeks ago         270MB
hyperledger/fabric-kafka                         latest              caaae0474ef2        2 weeks ago         270MB
hyperledger/fabric-couchdb                       0.4.18              d369d4eaa0fd        2 weeks ago         261MB
hyperledger/fabric-couchdb                       latest              d369d4eaa0fd        2 weeks ago         261MB
hyperledger/fabric-baseos                        amd64-0.4.18        c256a6aad46f        2 weeks ago         80.8MB
```

## 七、进入`fabric-samples/first-network`快速测试所有环境是否已经准备就绪，执行

```shell
# 生成MSP证书和创世区块
./byfn.sh -m generate
# 启动网络
./byfn.sh -m up
```

看到这个说明环境都部署成功

```shell
========= All GOOD, BYFN execution completed =========== 


 _____   _   _   ____   
| ____| | \ | | |  _ \  
|  _|   |  \| | | | | | 
| |___  | |\  | | |_| | 
|_____| |_| \_| |____/ 
```



如果执行失败了，修改某些配置后就重新启动时需要先清理环境，执行

```shell
# 关闭网络并清理环境
./byfn.sh -m down
# 再执行
./byfn.sh -m up
```

