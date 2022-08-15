# Docker

# $\color{red}***入门篇***$

## 一、安装

### 1.   安装脚本

```shell
# 删除旧环境
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 删除所有旧的数据
sudo rm -rf /var/lib/docker              
# 安装依赖工具 
sudo yum install -y yum-utils
# 指定docker源（选国内的）
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
# 更新yum软件包索引
sudo yum makecache fast    

# 安装最新版docker核心  docker-ce社区版 -ee企业版
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 配置镜像加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
EOF

# 启动docker引擎并设置开机启动
sudo systemctl start docker
sudo systemctl enable docker

# 配置当前用户对docker的执行权限
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker

echo "安装完成"
```

安装结束后，使用：`docker version`查看是否安装成功

在使用：`docker run hello-world`，从镜像源拉取hello-world镜像，看能否运行，如果运行成功，则使用`docker images`会发现hello-world镜像已经被拉取到本地镜像仓库中。

## 二、基本常用命令

### 1.   帮助命令

```shell 
docker version
docker info
# 查看一个命令如何使用
docker  命令  --help
```

### 2.  镜像命令

#### （1）查看本地镜像（docker images）

```shell
# 查看所有镜像
docker images   或   docker images -a
# 只显示镜像id
docker images -q
```

#### （2）搜索镜像（docker search）

```shell
# 搜索镜像信息
docker search mysql

# 选项，通过搜索来过滤
--filter=STARS=3000  （搜索出来的镜像就是STARS大于3000的）
# 示例
docker search mysql --filter=STARS=3000
```

#### （3）下载镜像（docker pull）

```shell
# 下载镜像（默认是最新版）
docker pull mysql
# 下载信息中 最后一行代表真实地址，即镜像从哪里下载的（docker.io/library/mysql:latest）
# 二者等价
docker pull mysql  =  docker pull docker.io/library/mysql:latest
# 可以通过tag指定版本 5.7版（版本一定要在仓库中存在[废话]）
docker pull mysql:5.7
```

#### （4）删除镜像（docker rmi）

```shell
# 基本命令
docker rmi -f 镜像ID # -f选项 强制，rmi代表remove image
# 也可以删除多个
docker rmi -f 镜像ID1 镜像ID2 镜像ID3
# 删除全部的镜像（实际上跟上面一样）
docker rmi -f $(docker images -aq)
```

### 3.  容器命令

**说明**：有了镜像才能创建容器

#### （1）新建容器并启动（docker run、docker exec、docker attach）

```shell
# 基本使用
docker run [选项] image名字
# 参数说明
--name="Name"    # 容器名字，用来区分容器
-d     # 后台方式运行
-it    # 使用交互方式运行，进入容器查看内容
-p     # 指定容器的端口（小写p）
	-p ip:主机端口:容器端口
	-p 主机端口:容器端口
	-p 容器端口
-P	   # 随机指定端口（大写P）
```

测试：

```shell
# 下载一个centos镜像
docker pull centos
# 启动并进入容器（-it代表交互运行，进入到容器中，后接/bin/bash表示执行一个命令，这里是打开/bin/bash，也可以执行其他命令）
docker run -it centos /bin/bash
# 容器内也可以使用ls等命令查看容器内的环境
# 退出容器交互环境
exit 	# 直接容器停止并退出
ctrl+P+Q  #容器不停止退出
```

创建容器后，如果想要进入容器中进行交互，可以使用：

```shell
# docker attach 在有多个终端在一个容器进行交互时，看到的是同一个终端（同步的）
docker attach 容器id
# docker exec 新开启一个终端进行交互
docker exec -it 容器id /bin/bash
```





#### （2）查看容器状况（docker ps）

```shell
# 查看正在运行的容器
docker ps
# 查看所有运行记录
docker ps -a
# 其他选项
-n=x    # 显示最近创建的x个容器
-q      # 只显示容器的编号
```

#### （3）删除容器（docker rm）

```shell
# 删除指定的容器 ， 如果容器正在运行则无法删除，可通过-f强制删除
docker rm 容器id
# 删除所有容器
docker rm -f $(docker ps -aq)
docker ps -aq |xargs docker rm # 表示将前面指令的容器ID作为参数（xargs关键字）传入后面指令
```

#### （4）启动与停止容器（docker start、docker stop、docker kill、docker restart）

```shell
docker start 容器ID       # 启动容器
docker restart 容器ID		# 重启容器
docker stop 容器ID		# 停止当前正在运行的容器
docker kill 容器ID		# 强制停止当前容器
```

#### （5）其他常用命令

**后台启动**

```shell
### 后台启动

docker run -d centos
# 问题：后台启动后 用docker ps查看运行的容器，发现centos停止了

# 解释：docker容器使用后台运行，就必须要有一个前台进程，如果docker发现没有应用，就会自动停止
# 坑：容器启动后，发现自己没有提供服务（即自己什么都不用干），就会立即停止
```

**查看日志**

```shell
### 查看日志
# 可以先写个脚本让容器内部有日志（否则查看日志发现什么都没有）
docker run -d centos /bin/bash -c "while true;do echo "hello";sleep 1;done"
# 查看日志
docker logs -tf --tail 10 容器ID
	-tf					#显示日志
	--tail number		#要显示日志的条数
```

**查看容器中的进程信息**

```shell
docker top 容器ID

```

![image-20220717140006493](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220717140006493.png)

**查看镜像的元数据**

```shell
docker inspect 容器ID [多个容器ID]
```

```json
// 查看结果
[
    {
        "Id": "d5be2f8e9fbd62190820f17540457c4f78e6ec842c68b8613d0934e0a025162d",
        "Created": "2022-07-17T05:56:24.005724836Z",
        "Path": "/bin/bash",
        "Args": [
            "-c",
            "while true;do echo hello;sleep 1;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 6245,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2022-07-17T05:56:24.319535972Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6",
        "ResolvConfPath": "/var/lib/docker/containers/d5be2f8e9fbd62190820f17540457c4f78e6ec842c68b8613d0934e0a025162d/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/d5be2f8e9fbd62190820f17540457c4f78e6ec842c68b8613d0934e0a025162d/hostname",
        "HostsPath": "/var/lib/docker/containers/d5be2f8e9fbd62190820f17540457c4f78e6ec842c68b8613d0934e0a025162d/hosts",
        "LogPath": "/var/lib/docker/containers/d5be2f8e9fbd62190820f17540457c4f78e6ec842c68b8613d0934e0a025162d/d5be2f8e9fbd62190820f17540457c4f78e6ec842c68b8613d0934e0a025162d-json.log",
        "Name": "/relaxed_shirley",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/94a827542a9ae13628a352c1372defc2416b8bce3d3f2de55fb22f83f5eaf091-init/diff:/var/lib/docker/overlay2/32b6e455f50a37da56ed86bda9e7eea49613358ea5b70ecf19dfa839cd497868/diff",
                "MergedDir": "/var/lib/docker/overlay2/94a827542a9ae13628a352c1372defc2416b8bce3d3f2de55fb22f83f5eaf091/merged",
                "UpperDir": "/var/lib/docker/overlay2/94a827542a9ae13628a352c1372defc2416b8bce3d3f2de55fb22f83f5eaf091/diff",
                "WorkDir": "/var/lib/docker/overlay2/94a827542a9ae13628a352c1372defc2416b8bce3d3f2de55fb22f83f5eaf091/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "d5be2f8e9fbd",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash",
                "-c",
                "while true;do echo hello;sleep 1;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20210915",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "c04499323e12c34391cc9dd64eb709694c3764db5df0918a5cacf3fcb426d875",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/c04499323e12",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "c6f1e48b644679de6dba45a9b72814cacbcbcf97e555e68d1b9049e08b477a6c",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "90d5112d020230665eff0be7e23d06cb8196090528bb8bc6fafe33409ae89883",
                    "EndpointID": "c6f1e48b644679de6dba45a9b72814cacbcbcf97e555e68d1b9049e08b477a6c",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]

```

**进入当前正在运行的容器**

```shell
# 我们通常容器都是后台运行的 而时常需要进入容器修改一些配置
# 命令
# 方式一
docker exec -it 容器id /bin/bash
# 方式二
docker attach 容器ID

# docker exec ：进入容器后开启一个新的终端，可以在里面操作
# docker attach ：进入容器正在运行的终端，如果此时终端是一个死循环就寄了
```

**从容器内copy文件到主机上**

```shell
docker cp 容器id:容器文件全路径 主机全路径
```

**查看容器的动态状况**

```shell
docker stats 容器ID
```

## 三、UnionFS

UnionFS，联合文件系统，目前docker支持的联合文件系统有很多种，包括：AUFS、overlay、overlay2、DeviceMapper、VSF等。

### 1.   docker层次

以overlay2为例，docker层级分为"lowerdir"、"upperdir"、"merged"三个层次。

* lowerdir层：只读镜像层，其中包含bootfs/rootfs层，bootfs包含bootLoader与kernel，rootfs就是经典linux系统中的一些标准目录。
* upperdir层：可读可写，实际上就是container层，对容器数据的更改都在这一层。
* merged层：联合挂载层，给用户暴露的统一视觉，将image层和container层结合。

![img](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/1715041-20210226171114031-709396621.png)

## 四、提交镜像

### 1. commit（docker commit）

```shell
# 提交容器成为一个新的副本
docker commit 
# 命令与git原理相似
docker commit -m="提交的信息" -a="作者" 容器id 目标镜像名:[TAG]
# 比如有一个容器 id为111  我们希望这个容器发布为一个镜像 名字叫myImage 版本为1.0
docker commit -m="first commit" -a="hoshino" 111 myImage:1.0
```



# $\color{red}***高级篇***$

## 五、容器数据卷

### 1.   挂载（-v）

​	如果我们删除一个容器，那么容器内的数据也会随之删除，所以我们希望可以把容器内的数据同步到外部主机上，容器数据卷技术即为解决这个需求的。

#### （1）   基本使用

​	基本命令：

```shell
# 将主机目录与容器目录双向同步（绑定）   -v   这样 两个目录的增删查改都会影响对方
docker run -it -v 主机目录:容器目录 centos /bin/bash
```

#### （2）   具名挂载与匿名挂载

```shell
# 匿名挂载 不指定主机目录进行挂载
docker run -d -P --name nginx01 -v /etc/nginx nginx
# 可以查看本地卷（volume）的情况
docker volume ls
# 如下：
[root@VM-12-6-centos ~]# docker volume ls
DRIVER    VOLUME NAME
local     a4c667a576225db152a332910c9b6695d645108b59be21e3532164c202a50230
# ------------------------------------------------------------------------------

# 具名挂载 通过-v 卷名:容器路径 指定具体的卷名 这里指定卷名为juming
docker run -d -P --name nginx02 -v juming:/etc/nginx nginx
# 再查看一下本地卷:
[root@VM-12-6-centos ~]# docker volume ls
DRIVER    VOLUME NAME
local     a4c667a576225db152a332910c9b6695d645108b59be21e3532164c202a50230
local     juming
# ------------------------------------------------------------------------------
# 可以通过inspect查看一下这些卷的位置
[root@VM-12-6-centos ~]# docker inspect juming
[
    {
        "CreatedAt": "2022-07-18T15:25:16+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming/_data",  # juming卷的具体位置 
        "Name": "juming",
        "Options": null,
        "Scope": "local"
    }
]
```

==不建议使用匿名挂载，建议使用具名挂载==

#### （3）  挂载小结

可以看出，挂载有三种：

1. 指定映射路径挂载
2. 具名挂载
3. 匿名挂载

```shell
-v 容器内路径             #匿名挂载
-v 卷名:容器内路径	        #具名挂载
-v 宿主机路径:容器内路径    #映射路径挂载
```

拓展:

```shell
# 通过ro rw改变读写权限，一旦设置了容器权限 容器对挂载的内容就有限制了
# ro readonly
docker run -d -P --name nginx01 -v juming:/etc/nginx:ro nginx
# rw readwrite
docker run -d -P --name nginx01 -v juming:/etc/nginx:rw nginx
# ro说明这个路径只能通过宿主机来操作，容器内部无法操作
```



### 3.  容器间数据共享

可以在docker run 时，使用==--volume-from 容器名称==，同步容器间的数据卷。

--volume-from即在创建容器时，将数据卷的指针，指向要引用的容器的数据卷指针指向的目录处：

![image-20220718183333030](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220718183333030.png)

## 六、Dockerfile

### 1.  基本关键字

```shell
FROM				#表示构建这个镜像是基于哪个镜像的
MAINTAINER			#镜像的维护者 一般写姓名+邮箱
WORKDIR				#表示镜像的工作目录
ADD					#添加东西到镜像中
COPY				#构建镜像时，将本机的某个目录copy到镜像的某个目录下
RUN					#构建镜像时，执行的指令
CMD					#构建好镜像后，执行的指令
EXPOSE				#构建的镜像暴露的端口
VOLUME				#挂载容器数据卷
ENV					#指定环境变量
ENTRYPOINT			#与CMD作用相似，但如果docker run的时候指定了选项，CMD选项就失效了，而ENTRYPOINT会追加
```

### 2.   构建过程

​	scratch：DockerHub中99%的镜像都是FROM scratch，它是一个最小的运行环境。

​	写完Dockerfile后，就可以构建镜像了：

```shell
docker build -f Dockerfile -t 镜像名:版本号 .			# 注意后面有个.（也可以加其他命令，“.”是最基础的命令）
```

### 3.  发布镜像（docker push）

```shell
# 使用 docker login登录dockerhub或其他的镜像服务（比如阿里云）
docker login --username=用户名 [server]		#不写server默认登录dockerhub
# 接着输入密码
# 登录成功后，使用docker push发布镜像
docker push 镜像名:版本号
# 可以通过docker tag进行镜像改名，比如:
docker tag ubuntu:15.10 runoob/ubuntu:v3 #表示将ubuntu:15.10改名为ubuntu:v3
```

## 七、docker-compose

### 1.   引言

如果我们要部署许多个镜像容器，按照之前讲的，得一个一个docker build去启动？显然太麻烦，于是docker-compose显得十分必要，docker-compose是处理项目的依赖启动关系的。

### 2.  docker-compose层级

docker-compose基本上可以分为3层：

1. 项目
2. 服务
3. 其他配置

其中最主要的为服务，一个项目可以有多个服务，比如一个项目依赖redis，可以如此构建

docker-compose.yml：

```yaml
version: '3.8'
services:
  hoshinoapp:
  # build可以指定Dockerfile,如果只写一个“.”就会在当前路径下找Dockerfile
    build: .
  # 指定容器运行的镜像
    image: hoshinoapp
  # 依赖
    depends_on:
      - redis
  # 端口映射
    ports:
      - "8080:8080"

  redis:
    image: "redis:alpine"
```

### 3.   基本使用

```yaml
# version可以看官方给出的表，不同的docker版本对应不同的version
version: 'xxx'
# services是最重要的，一个项目可以有多个service，在这下面编写你项目的服务
services:
	web: 
		#自定义容器名称，不用默认的
		container_name: name
		# build可以写“.”（从当前docker-compose所在目录下找dockerfile，），也可以写所需dockerfile的路径
		build: .
		# 指定容器运行的镜像
		image: xxx
		ports:
		  - "宿主机端口:容器端口"
		volumes:
		  - 宿主机卷1:容器卷1
		  - 宿主机卷2:容器卷2
		# 覆盖容器启动的默认命令
		command: ["xxx","xxx"]
		# 覆盖容器默认entrypoint 可以是sh，也可以是多个“-”拼接的命令
		entrypoint: /xxx/xxx.sh
		  (
		  - xxx
          - xxx
		  )
		# 服务依赖于
		depends_on: 
		  - redis
		  - mysql
		# 自定义dns服务器
		dns: 
		  - dns1
		  - dns2
		#暴露端口，但不映射宿主机
		expose:
		  - "3000"
		  - "8080"
		# links连接容器 共享环境变量
		links:
		  - redis
		  - mysql
		# 设置网络模式,有：bridge,host,none,service:[service name],container:[container name|id]
		network_mode: "bridge"
	redis: 
		xxxx
	mysql:
		xxxx
```

这些是不够的，要学习docker-compose写法，==多去看看别人开源项目或者官方文档怎么写==。

### 4.   部署

这里已部署java的jar为例：

​		当完成了Dockerfile、docker-compose.yml后，把这两个文件连同jar包丢到服务器==同一目录下==，然后启动

docker-compose构建镜像容器即可：

```shell
docker-compose up		#开始构建服务
# 如果中途构建失败了，排除错误后，再次构建时，对于已经整理好的环境不会再从头开始构建，这就有可能出现各种问题，建议重新构建时加上--build
docker-compose up --build	# --build会让镜像容器重新构建
```

### 5.  小结

​		可以看出，docker-compose十分方便，因为写好docker-compose后，项目各种依赖及构建机制以及启动就以及确定了，如果一个项目加入了docker-compose管理，我们便可以直接`docker-compose up`就能启动项目。

## 八、docker网络

​		在docker容器中访问宿主机端口有时候会显示连接失败。宿主机明明有8080的服务，容器内确访问不到：

> localhost:8080

​	这里就要说说docker的网络模式了，docker有4种网络模式：

### 1.   docker网络模式

#### （1） host模式

​		容器和宿主机共享network，这时候localhost就可以访问宿主机端口了。

```shell
docker run -d --network host --name nginx01 nginx
```

#### （2） container模式

​		容器A和容器B共享network，就是说容器之间可以通过localhost直接访问。

```shell
docker run -d --network container --name nginx01 nginx
```

#### （3）none模式

​		容器与宿主机隔绝，不能联网，安全性最高，一般很少用到。

```shell
docker run -d --network none --name nginx01 nginx
```

#### （4）bridge模式（默认）

​		每个容器有自己的network，通过localhost访问不到宿主机。

```shell
docker run -d --name nginx
```

==每个容器的ip可以通过docker inspect 容器id | grep IPAddress查看==

### 2. 容器互联

#### （1）   基本介绍

​	当容器间需要相互通信时，如一个springboot应用的容器与一个mysql容器需要通信，也许你会想可以通过ip通信，但如果ip有变（每次启动docker容器时，docker都会为这个容器随机分配一个ip）岂不是要重新修改配置？这里容器连接--link就有作用了。

#### （2）   --link X:Y

我们在创建容器时可以指定参数--link，此处X指要连接的容器名，Y指要连接的容器别名。

比如，我们有以上的需求时：

```shell
# 创建mysql容器 
docker run --name mysql-test -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=test  -p 3306:3306 -d mysql 
# 此时 如果已经做好了springboot的镜像 就可以：
docker run --link mysql-test:mysql-test --name springboot-app -p 8080:8080 springboot-img
# 原来，两个容器通信只能够以各自容器的ip通信（docker inspect 容器id | grep IPAddress可以查看容器ip），现在link之后，我们可以通过容器的名字进行通信。
```

==这里--link会让springboot-img创建的容器自动记住mysql容器的ip，但是在本机开发时，我们的通信ip都会是127.0.0.1，因此，部署为容器时需要修改后端配置，所以我们link之后就能根据容器名来进行通信（这也是为什么教程里springboot与redis通信时ip为redis），不为容器时ip为127.0.0.1==

![image-20220723230400110](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220723230400110.png)

==注意：--link的连接是单向的，以上例子中，springboot容器ping mysql容器能ping通，但mysql容器ping不通springboot容器==

#### （3）原理

​		实际上，--link的原理十分简单，只是用了域名劫持，当启动容器写上了"--link  连接容器id"，那么就会在启动容器的/etc/hosts（相当于windows的hosts文件）里写上 "连接容器ip   连接容器名字"（简单的域名劫持），这样，当启动容器访问连接容器的名字，实际走的就是这个容器的ip。

### 3.  查看docker网络

```shell
# 查看docker的所有网络
docker network ls
# 有network id、name、driver、scope四个列，driver代表网络模式
[root@VM-12-6-centos ~]# docker network ls
NETWORK ID     NAME                 DRIVER    SCOPE
90d5112d0202   bridge               bridge    local
1b6c70228b91   hoshinoapp_default   bridge    local
9f1b35e5695f   host                 host      local
99c611461267   none                 null      local
# 这里第一个名为bridge的是docker0网桥
# 我们创建容器时，默认会加上一个参数：--net bridge，表示桥接模式使用默认的网桥（即docker0）
```

### 4.  自定义网络

​		虽然说--link能够使用容器的名称来ping通。但是，我们一般不会这么做，毕竟容器间进行需要通信时，创建容器每次都得--link，当容器多起来后十分麻烦混乱，自定义网络是一个好的解决方案。

```shell
# 我们也可以自定义一个网络，使用docker network create命令创建
# 常用参数：
	--driver：网络模式
	--subnet：子网
	--gateway：网关ip
# 例如，我们创建一个名为mynet，子网为192.168.0.0/16，网关为192.168.0.1，driver为桥接模式的网络：
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
# 简单介绍一下网络知识吧，怕有人在这里还不会：
# 局域网如何划分？首先指定一个网段，在相同网段下的ip可以相互通信，像上面，我们指定的子网192.168实际上就是一个虚拟局域网段，后面的16是指定网段的长度，每个“.”的两边实际上是8位2进制数字的十进制转换，16就是两个小段，后面剩下的16位用于分配子网ip，总共可以分配255*255=65535个ip地址。指定的网关为192.168.0.1，ip通信均经过网关，而docker0默认用172.17.0.1作为网关。

# 到这一步，我们的docker网络就创建完成了




# 前面我们说到，容器创建默认使用--net bridge，也就是默认的docker0网络。那么，我们也可以自己指定自己的网络，让容器加入到我们创建的网络中：
docker run -d -P --name mynet-redis01 --net mynet redis
docker run -d -P --name mynet-redis02 --net mynet redis

# 这样，两个容器就能加入到我们创建的网络中，使用docker network inspect 网络id查看网络状态，可以看到网络中有我们刚刚创建的两个容器：
"Containers": {
            "33b4c24a8dd3e41fc0ae927fd7a487daa686cc1eed1da54ab60cf210a8c11aa9": {
                "Name": "mynet-redis01",
                "EndpointID": "492492c9b8ae7a2ddee2a01f92b6935069182d366417474975ed3d25bf883999",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "962bcf5052f0fcb1fb55a2838d9e2fed4e0989d5ba85c2972a2ad2acf239b56d": {
                "Name": "mynet-redis02",
                "EndpointID": "d2f4ba5847cdaf9d4522260a60acdf2590b93f44a3332d514e0265704f86393f",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        }
```

可以看到，我们的mynet-redis01与mynet-redis02分别被分配了192.168.0.2与192.168.0.3。

到这里，我们就可以测试一下自定义网络相较于--link的优点了：

```shell
# 毫无疑问，如果这两个容器互相去ping对方的ip肯定是能ping通的（毕竟它们在同一个网段下嘛），那么使用名字ping如何：
docker exec -it mynet-redis01 ping mynet-redis02
# 发现也是能ping通的。
# 总结优点：
自定义网络内的所有容器相互可以使用名称进行通信，不需要双方进行繁琐的--link
```

注意：==默认的docker0网络中的容器是无法相互通信的==

### 5.   网络连通

​		在同一网络中的容器可以相互通信，如果我们想要在不同的网络中的容器相互通信怎么办呢？docker network为我们提供了一个connect操作：

```shell
docker network connect [连通网络名] [要连通的容器名]
# 比如一个在docker0下的redis0，需要与一个在mynet下的mynet-mysql0通信，方法是连通redis0与mynet，而不是两个容器互相连通：
docker network connect mynet redis0

# 使用docker network inspect mynet可以看到，connect方法只是将redis0加入到了mynet网络下，也就是说，现在redis0有两个ip，存在于两个虚拟网络下 
```

