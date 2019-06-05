<!-- TOC -->

- [常用命令](#常用命令)
- [命令参数解析](#命令参数解析)
    - [拉取镜像](#拉取镜像)
    - [运行镜像](#运行镜像)
    - [进入容器的 bash](#进入容器的-bash)
- [Swarm 模式(集群)](#swarm-模式集群)
    - [管理节点](#管理节点)
    - [工人节点](#工人节点)
    - [建立 Swarm(3 个管理节点，3 个工人节点，需要 6 台机器或虚拟机)](#建立-swarm3-个管理节点3-个工人节点需要-6-台机器或虚拟机)
    - [建立服务(Service)](#建立服务service)
    - [Stack](#stack)
- [Volume(卷)](#volume卷)
    - [解析](#解析)
    - [挂载卷](#挂载卷)
- [Dockfile](#dockfile)
    - [常用指令](#常用指令)
    - [ASP.NET Core使用的Dockerfile](#aspnet-core使用的dockerfile)

<!-- /TOC -->

## 常用命令

- docker run <镜像> _(运行镜像)_
  - docker run -it <镜像> _(前台运行镜像)_
  - docker run -d <镜像> _(后台运行镜像)_
- docker pull <镜像> _(拉取镜像)_
- docker images _(查询本机所有镜像)_
- docker rmi <镜像> _(删除指定镜像)_
- docker ps _(查询当前运行的容器)_
- docker ps -a _(查询所有容器)_
- docker stop <容器> _(停止容器)_
- docker rm <容器> _(删除容器)_
- docker start <容器> _(重新运行容器)_

## 命令参数解析

### 拉取镜像

```shell
docker pull microsoft/dotnet-samples
```

**解析**

> `microsoft/dotnet-samples` 为镜像名字

### 运行镜像

```shell
docker run -it --rm -p 8000:80 --name aspnetcore_sample microsoft/dotnet-samples:aspnetapp
```

**解析**

> `microsoft/dotnet-samples:aspnetapp` 为镜像名

> `--name aspnetcore_sample` 为运行的容器取名为`aspnetcore_sample`

> `-p 8000:80` 将容器里的`80`端口映射到容器外的`8000`端口，即容器外的`8000`端口相当于容器内的`80`端口

> `-rm` 表示容器运行完之后自动将容器删除

> `-it` 表示运行容器之后停留在容器里的`bash`，即前台运行容器，关闭命令行则容器停止

> `-d` 与`-it`相对应，指后台运行容器，执行之后会得到容器 id

### 进入容器的 bash

```shell
docker exce -it 33982a1e1234 /bin/bash
```

**解析**

> `33982a1e1234`指容器 Id

> `/bin/bash` 指容器里的 bash 程序的位置

## Swarm 模式(集群)

### 管理节点

- 管理节点维护着 swarm
- 管理节点是高可用的
- 管理节点的数目建议 3/5 个(基数)
- 只有一个首领

### 工人节点

- 执行首领给的任务

### 建立 Swarm(3 个管理节点，3 个工人节点，需要 6 台机器或虚拟机)

1. 初始化 Swarm(在机器 A 创建一个管理节点)

```shell
docker swarm init --advertise-addr 10.0.0.1:2377 --listen-addr 10.0.0.1:2377
```

> 此时机器 A 作为 Swarm 的 leader

> 10.0.0.1 为本机 IP，2377 为监听端口

2. 在机器 A 上查询加入 Swarm 管理节点的命令

```shell
docker swarm join-token manager
```

3. 在机器 A 上查询加入 Swarm 工人节点的命令

```shell
docker swarm join-token worker
```

4. 将机器 B 作为管理节点加入到 Swarm 集群中

```shell
docker swarm join --token xxx 10.0.0.1:2377 --advertise-addr 10.0.0.2:2377 --listen-addr 10.0.0.2:2377
```

> 10.0.0.2 为本机 IP，2377 为监听端口

> 其中 xxx 为步骤 2 得到的 token

5. 在管理节点上查看当前 swarm 节点

```shell
docker node ls
```

6. 将机器 D 作为工人节点加入到 Swarm 集群中

```shell
docker swarm join --token xxx 10.0.0.1:2377 --advertise-addr 10.0.0.4:2377 --listen-addr 10.0.0.4:2377
```

> 10.0.0.4 为本机 IP，2377 为监听端口

> 其中 xxx 为步骤 3 得到的 token

7. 将指定的工人节点提升为管理节点(在 leader 上执行)

```shell
docker promoted xxx
```

> xxx 是通过执行`docker node ls`得到的节点 Id

### 建立服务(Service)

1. 建立服务(会在 swarm 中自动分配机器来运行服务，N 个机器运行 M 个容器)

**这里表示任意一个发往 swarm 中任意一个节点中 80 端口的请求，都会到达 service**

```shell
docker service create --name aspcore -p 80:80 --replicas 5 microsoft/dotnet-samples:aspnetapp
```

> `--replicas 5`表示这个镜像要运行复制 5 个容器

> `microsoft/dotnet-samples:aspnetapp` 为 service 要运行的镜像名称

2. 查看当前运行的 service

```shell
docker service ls
```

3. 查看指定 service 里的复制品

```shell
docker service ps aspcore
```

> `aspcore`为服务名字

4. 更新 service 中的复制品数量

```shell
docker service update --replicas 7 aspcore
```

> `--replicas 7`为要更新的数量

> `aspcore`为服务名字

### Stack

一组相互有关的 Service，这些 Service 共享了一些依赖项，这些 Service 可以被一起编排，一起扩展。Stack 比 Service 更上一个层次结构

1. 创建一个 Stack
   1. 创建目录 myStack
   2. 创建文件 `docker-compose.yml`
   3. 输入文件内容
   ```yml
   version: "2"
   services:
   web:
       image: microsoft/dotnet-samples:aspnetapp
       deploy:
       replicas: 5
       restart_policy:
           condition: on-failure
       resource:
           limits:
           cpus: "0.1"
           memory: 50M
       ports:
       - "80:80"
       networks:
       - webnet
   visualizer:
       image: dockersamples/visualizer:stable
       ports:
       - "8080:8080"
       volumes:
       - "/var/run/docker.sock:/var/run/docker.sock"
       deploy:
       placement:
           constraints: [node.role == manager]
       networks:
       - webnet
   networks:
   webnet:
   ```
   4. 执行命令
   ```shell
   docker stack deploy -c docker-compose.yml mystacksample
   ```
   5. 查询 stack 列表
   ```shell
   docker stack ls
   ```
   6. 查询指定 stack 里的 service 列表
   ```shell
   docker stack services mystacksample
   ```
   > `mystacksample`为 stack 名字

## Volume(卷)

### 解析

1. Volume(卷)是容器中一个特别种类的目录，通常叫数据 volume，里面可以存放各种类型的数据，例如代码，日志文件，数据文件等
2. Volume 可以在容器间被共享和复用。可以让多个容器对同一个 volume 进行读写，也可以让一个容器读写多个 volume
3. 对镜像更新并不会影响 volume
4. Volume 是被持久化的，即使容器删除，volume 仍然存在

### 挂载卷

```shell
docker run -it -p 8080:5000 -v /Users/jiamiao.x/repo/docker/volSample:/app --workdir '/app' -v /Users/jiamiao.x/repo/docker/volOutput:/var/www/ microsoft/dotnet bash
```

> `-v /Users/jiamiao.x/repo/docker/volSample:/app` 其中 `-v` 表示挂在 volume，`/Users/jiamiao.x/repo/docker/volSample`为本机文件夹路径，`/app`为容器里的目录，即绑定本机与容器里的文件夹

> `--workdir '/app'` 指定工作目录为`/app`，即进入`bash`默认目录为`/app`

> `-v /Users/jiamiao.x/repo/docker/volOutput:/var/www/` 其中`/Users/jiamiao.x/repo/docker/volOutput`为本机文件夹路径，`/var/www/`为容器里的目录，在代码里输出文件到`/var/www/`文件夹中，在本机`/Users/jiamiao.x/repo/docker/volOutput`目录中可以直接看到代码输出的文件


## Dockfile
### 常用指令
- FROM 通常情况下，要创建的镜像是基于另一个镜像的，这里就需要使用FROM
- MAINTAINER 该镜像的维护人
- RUN 这里可以定义一些需要运行的命令，例如 `npm install`,`dotnet restore`等
- COPY 开发的时候，可以将源码放在volumes里，而在生产环境下，经常需要将源码复制到容器里面，使用COPY就可以做到
- ENTRYPOINT 定义容器的入口，把容器配置成像exe一样的运行文件，通常是一些例如`dotent`命令，`node` 命令等等
- CMD 设置容器运行的默认命令和参数，当容器运行的时候，这个可以在命令行被执行
- WORKDIR 设定容器运行的工作目录
- EXPOSE 暴露端口
- ENV 设定环境变量
- VOLUME 定义volume，并控制如何在宿主中进行存储 

### ASP.NET Core使用的Dockerfile
```docker
FROM microsoft/dotnet:2.2-sdk AS build-env
WORKDIR /app

# Copy csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# Copy everything else and build
COPY . ./
RUN dotnet publish -c Release -o out

# Build runtime image
FROM microsoft/dotnet:2.2-aspnetcore-runtime
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]
```
- 生成镜像命令
```shell
docker build -t xiejiamiao/myaspnetcore .
```
> `xiejiamiao/myaspnetcore`为要生成的镜像名字
- 运行容器
```shell
docker run -d -p 8080:5000 --name mycore -v /Users/jiamiao.x/repo/docker/volSample:/app/out xiejiamiao/myaspnetcore
```
> Dockerfile中指定里容器运行时会执行 `/app/out`目录里的`WebApplication1.dll`文件，所以在运行容器的时候需要挂载volume

> **如果修改代码需要重新生成镜像**