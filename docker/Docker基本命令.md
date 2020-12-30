[toc]

### 镜像

1. 版本

   ```sh
   docker version
   ```

   

2. 登录

   ```sh
   docker login -u User -p Pasword <docker-server>
   ```

   Docker会将token存储在~/.docker/config.json文件中，从而作为拉取私有镜像的凭证。

3. 注销

   ```sh
   docker logout <docker-server>
   ```

   

4. 查询

   ```sh
   docker images
   ```

   

5. 搜索镜像

   ```sh
   docker search image
   ```

   

6. 拉取镜像

   ```sh
   docker pull image:tag
   ```

   ```sh
   docker pull ubuntu 
   
   Using default tag: latest
   latest: Pulling from library/ubuntu
   da7391352a9b: Pull complete 
   14428a6d4bcd: Pull complete 
   2c2d948710f2: Pull complete 
   Digest: sha256:c95a8e48bf88e9849f3e0f723d9f49fa12c5a00cfc6e60d2bc99d87555295e4c
   Status: Downloaded newer image for ubuntu:latest
   docker.io/library/ubuntu:latest
   ```

   

   - 默认抓取的地址是官方的 [docker hub](https://hub.docker.com/)，官方存放镜像的位置默认在 library 目录下
   - 在抓取的时候，先有一个下载的过程，镜像是多层存储所构成，分层下载，并非单一文件，下载过程中给出了每一层的前 12 位 ID
   - 下载结束后，显示 pull complete，并给出该镜像的完整 `sha256`摘要

7. 创建镜像

   ```sh
   docker image build -t <image name>:<tag> .
   ```

   - -t 指定 image 文件，最后的` .` 表示上下文环境，Dockerfile 在当前路径
   - -f 指定dockfile 文件

8. 查看镜像

   ```sh
   # 列出所有镜像，不包含中间层镜像
   docker images
   # 或者
   docker image ls
   
   # 列出所有虚悬镜像
   docker image ls -f dangling=true
   
   # 列出所有镜像，包含中间层镜像
   docker images -a
   # 或者
   docker image ls -a
   
   # 列出指定镜像
   docker images <image name>
   ```

   ```tex
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   mysql                                  8.0                 ab2f358b8612        10 days ago         545MB
   ```

   说明:

   - REPOSITORY 表示镜像名
   - TAG 表示镜像标记，通常是版本号，或者其他可以区别镜像不同版本的标记
   - IMAGE ID 表示镜像 ID，镜像可以通过`<REPOSITORY>:<TAG>`来识别，也可以通过`<IMAGE ID>`来识别
   - CREATED 表示这个镜像制作的时间
   - SIZE 表示这个镜像的大小

   注意:

   ​	无标签的镜像`<none>`，与之前的虚空镜像不同，这些无标签的镜像很多都是中间层镜像，是其他镜像的依赖对象，只要删除了这些中间层镜像，这些依赖他们的镜像也将会被删除。

9. 查看镜像的详细信息

   ```sh
   docker inspect <image name>:<tag>
   ```

   

10. 删除镜像

   ```sh
   docker rmi <image name>:<tag>
   # 或者
   docker image rm <image name>:<tag>
   
   # 删除所有镜像 
   docker rmi $(docker images -q)
   # 或者
   docker image prune -f -a
   ```

   标签名缺省是latest，如果标签名是latest，则不用添加标签名

   - -f 参数： 强制删除此镜像，并会删除通过此镜像创建的容器

11. 发布镜像

    ```sh
    docker push <image name>:<tag>
    ```

    

### 容器

容器的实质是进程，容器进程运行于属于自己的独立的命名空间。容器也是分层存储，每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层。

1.  列出正在运行的容器文件

   ```sh
   docker ps
   # 或者
   docker container ls
   
   # 列出所有的容器文件(包括停止运行的容器)
   docker ps -a 
   # 或者
   docker container ls -a
   ```

   ```tex
   CONTAINER ID        IMAGE               COMMAND                   CREATED             STATUS                      PORTS                               NAMES
   6f4e1e80e33c        echo1:1.0           "/bin/sh -c 'echo us…"    55 minutes ago      Exited (0) 55 minutes ago                                       strange_mirzakhani
   0094e5d008c9        mysql:8.0           "docker-entrypoint.s…"    4 days ago          Exited (255) 4 days ago     33060/tcp, 0.0.0.0:3308->3306/tcp   slave_mysql
   174711ccb475        mysql:8.0           "docker-entrypoint.s…"    4 days ago          Exited (255) 4 days ago     33060/tcp, 0.0.0.0:3307->3306/tcp   master_mysql
   ```

   - CONTAINER ID 表示容器 ID
   - IMAGE 表示运行的镜像，镜像和容器的关系，就像是面向对象程序设计中`类`和`实例`一样，镜像是静态的定义，容器是运行时的实体
   - CREATED 表示运行的时间
   - STATUS UP表示运行
   - PORTS 表示可以通过指定的端口号来访问
   - NAMES 表示对镜像容器的描述
   - COMMAND 表示表示容器启动后运行的命令
   - STATUS UP表示运行，Exited表示已经停止运行了

2. 运行容器

   ```sh
   docker run -p 3307:3306 --name <container name> -d <image name>:<tag>
   ```

   - -p参数：容器的 3306 端口映射到本机的 3307 端口
   - --name参数：指定容器名
   - -d参数: 后台运行

3. 进入容器

   ```sh
   docker exec -it <container ID>|<container name> /bin/bash
   ```

   - -it参数：容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器
   - /bin/bash：容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell

4. 停止容器

   ```sh
   docker stop <container ID>|<container name>
   
   # 停止所有容器
   docker stop $(docker ps -aq) 
   ```

   

5. 删除容器

   ```sh
   docker rm <container ID>|<container name>
   
   # 删除所有停止的容器
   docker container prune
   # 或者
   docker rm $(docker ps -aq) 
   ```

   

6. 查询容器ip

   ```sh
   docker inspect --format='{{.NetworkSettings.IPAddress}}' container_id|container_name
   ```

   

7. 查看容器日志

   ```sh
   docker logs -f <container ID>|<container name>
   ```

   - -f 参数： 代表会实时刷新日志信息

8. 查看容器的详细信息

   ```sh
   docker inspect <container ID>
   ```

9. 复制文件

   ```sh
   # docker -> local
   docker cp <container ID>:/opt/file.txt /opt/local/
   # local -> docker
   docker cp /opt/local/file.txt <container ID>:/opt/
   ```

   

### Dockfile

Dockerfile 中的每一条指令都会建立一个层,所以多条`sh`指令最好使用`&&`,`;`等连接，这样生成的层会很少，镜像也会小很多。

- Dockerfile 必须以`FROM` 开头
- 一个 Dockerfile，只能有一个`ENTRYPOINT`，如果有多个以最后一个为准
- 一个 Dockerfile，只能有一个`CMD`，如果有多个以最后一个为准
- 在Dockerfile中，ENTRYPOINT指令或CMD指令，至少必有其一

1. Dockerfile

   ```dockerfile
   FROM alpine:latest as production
   LABEL Author="feiquanxx@gmail.com"
   RUN mkdir /lib64 &&\
   	ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2 &&\
   	echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories &&\
   	echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories &&\
   	apk add tzdata &&\
   	ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
   	echo "Asia/Shanghai" > /etc/timezone &&\
   	mkdir -p /app/ /config/
   COPY ./bin/* /app/*
   COPY ./config/* /config/
   EXPOSE 8080
   ENTRYPOINT ["/app/server","-c","/config/app.yaml"]
   ```

   - FROM 指定基础镜像为 alpine，版本为 latest，并重命名为`production`

   - LABEL 指定署名邮箱

   - RUN 指定在容器内执行的命令

     ```sh
     # musl和glibc是兼容的，通过创建该符号链接修复缺少的依赖项，可以保证编译后 go 程序可以正常运行
     mkdir /lib64
     ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2
     
     # 更新源
     echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories
     echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
     
     # 同步时间
     apk add tzdata
     ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
     echo "Asia/Shanghai" > /etc/timezone
     
     # 创建项目目录
     mkdir -p /app/ /config/
     ```

   - COPY 将当前本机上的文件复制到容器内指定位置

   - EXPOSE 容器将使用 8080 端口，这里只是声明并不会建立和本机的端口映射

   - ENTRYPOINT 容器启动时执行的命令

     ```sh
     /app/server -c /config/app.yaml
     ```

     

2. FORM 

   基础镜像必须指定，FROM指令指定基础镜像，因此一个 Dockerfile 文件中 FROM 是必备指令，而且是第一个指令

   `FROM scratch`指定一个空白镜像，scratch 不以任何镜像为基础，接下来写的指令将作为镜像第一层开始存在

3. LABEL 

   添加元数据到镜像。`LABEL`是以键值对形式出现的

4. RUN 

   在容器内执行指定命令

5. COPY

   将本机==构建上下文==中的指定文件复制到镜像中

6. ADD

   仅在需要自动解压缩的情况下才使用ADD指令，如果只是复制文件就使用COPY指令

7. ENV

   设置容器的环境变量

   ```dockerfile
   ENV name=linux
   ```

   

8. EXPOSE

   声明容器将要使用的端口，并不建立和主机的端口映射。当运行容器时使用`-p`才是真正的建立端口映射

9. ENTRYPOINT

   - exec格式用法 [推荐]

     程序入口点,执行`docker run`命令时，设置的==任何命令参数或CMD指令==的命令，都将作为ENTRYPOINT指令的命令参数，==追加到==ENTRYPOINT指令的命令之后。

     ```dockerfile
     ENTRYPOINT ["top","-b","-H"]
     ```

     如果此时创建docker 实例

     ```sh
     docker run <container-name> -v
     ```

     则实际执行的命令是 `top -b -H -v`

   - shell 格式用法

     ==屏蔽追加任何参数==，即CMD指令或docker run … 的参数都将被忽略。

     ```sh
     ENTRYPOINT top -b -H
     ```

     在创建容器后会首先调用Shell，即自动在命令前面追加`/bin/sh -c`。既容器启动时，将执行`/bin/sh -c top -b -H`。这样会导致`top`命令就不是容器中的第一个进程PID 1，这样在容器停止的时候就无法收到系统的SIGTERM信号。要想收到SIGTERM信号，务必使用Bash的内置exec命令使得top的PID 1，定义ENTRYPOINT指令如下:

     ```dockerfile
     ENTRYPOINT exec top -b -H
     ```

     

10. CMD

    用于指定==默认==的容器主进程的启动命令。

    不同格式：

    - exec格式用法 [推荐]

      ==参数追加模式==，第一个参数必须是命令的全路径

      

      如果`docker run`没有指定任何的执行命令或参数，但是，dockerfile 中有CMD和ENTRYPOINT, 那么 CMD 中的全部内容将作为 ENTRYPOINT 的参数

      ```dockerfile
      FROM ubuntu:latest
      CMD ["echo","username"]
      ENTRYPOINT ["echo","ENTRYPOINT"]
      ```

      ```sh
      # 构建
      docker image build -t echo1:2.0 -f Dockerfile .
      
      # 无参运行
      docker run echo1:2.0  # 输出：ENTRYPOINT
      
      # 带参数运行
      docker run echo1:2.0 echo "test"  # 输出：ENTRYPOINT echo test
      ```

      输出变量：

      ```sh
      FROM ubuntu:latest
      ENV user=feiquan
      
      # 这种写法无法输出变量,会直接输出 username:$user
      # CMD ["username:$user"]
      # ENTRYPOINT ["/bin/echo,"echo ENTRYPOINT:$user"]
      
      CMD ["/bin/sh","-c","echo username:$user"]
      ENTRYPOINT ["/bin/sh","-c","echo ENTRYPOINT:$user"]
      ```

      ```sh
      # 构建
      docker image build -t echo1:4.0 -f Dockerfile .
      
      # 无参运行
      docker run echo1:4.0  # 输出：ENTRYPOINT:feiquan
      
      # 带参数运行
      docker run echo1:4.0 docker run echo1:4.0  && echo "test:$user" 
      # 输出：
      ```

      ```tex
      ENTRYPOINT:feiquan
      test:
      ```

      由于在传入`&& echo "test:$user"`  时，`$user`使用的本机中环境变量，它为空，所以这里命令在追加后执行的其实是`/bin/sh -c echo ENTRYPOINT:$user && echo "test:"`。

      设置`user=xx`后，再次执行输出：

      ```tex
      ENTRYPOINT:feiquan
      test:xx
      ```

      

    - shell 格式用法

      ==屏蔽追加任何参数==，即CMD指令或docker run … 的参数都将被忽略。在创建容器后会首先调用Shell，即自动在命令前面追加`/bin/sh -c`。

      

      如果`docker run`没有指定任何的执行命令或者dockerfile里面也没有ENTRYPOINT，那么，就会使用cmd指定的默认的执行命令执行。

      ```dockerfile
      FROM ubuntu:latest
      CMD echo username
      ```

      ```sh
      # 构建
      docker image build -t echo1:1.0 -f Dockerfile .
      # 无参运行
      docker run echo1:1.0  # 输出：username
      
      # 带参数运行
      docker run echo1:1.0 echo "test"  # 输出：test
      ```

      输出变量：

      ```dockerfile
      FROM ubuntu:latest
      ENV user=feiquan
      CMD echo username:$user
      ```

      ```sh
      # 构建
      docker image build -t echo1:3.0 -f Dockerfile .
      # 无参运行
      docker run echo1:3.0  # 输出：username:feiquan
      
      # 带参数运行
      docker run echo1:3.0 echo "test"  # 输出：test
      ```

      

11. WORKDIR

    指定工作目录，也就是启动容器后终端所在路径,默认`/`。如果WORKDIR指 定的目录不存在，即使随后的指令没有用到这个目录，都会创建 

12. VOLUME

    实现数据的持久化，为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

    - 管理卷

      ```sh
      # 创建一个自定义容器卷
      docker volume create mdata
      # 查看所有容器卷
      docker volume ls
      # 查看指定容器卷详情信息
      docker volume inspect mdata
      ```

      ```json
      [
          {
              "CreatedAt": "2020-12-22T10:41:39Z",
              "Driver": "local",
              "Labels": {},
              "Mountpoint": "/var/lib/docker/volumes/mdata/_data",
              "Name": "mdata",
              "Options": {},
              "Scope": "local"
          }
      ]
      ```

    - 使用指定卷

      ```dockerfile
      FROM ubuntu:latest
      
      VOLUME /data
      RUN ls -ah / > /data/dir.txt
      ENTRYPOINT sleep 10000
      ```

      ```sh
      # 构建
      docker image build -t echo1:5.0 -f Dockerfile .
      # 挂载
      docker run -d -v mdata:/data echo1:5.0
      ```

      - -v参数：-v代表挂载数据卷，这里使用自定义卷`mdata`,并将其挂载到`/data`

    - 进入容器查看：

      ```sh
      docker exec -it 90177637380d /bin/bash
      
      ls data
      ```

      ```tex
      root@90177637380d:/# ll /data/
      total 12
      drwxr-xr-x 2 root root 4096 Dec 22 11:26 ./
      drwxr-xr-x 1 root root 4096 Dec 22 11:12 ../
      -rw-r--r-- 1 root root  119 Dec 22 11:10 dir.txt
      ```

    - 本机查看

      ```sh
      # mac 主机需要先执行
      # screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
      ll /var/lib/docker/volumes/mdata/_data
      ```

    - 删除卷

      ```sh
      docker volume rm <volume-name>
      ```

      