# docker容器添加对外映射端口

参考文章：[docker容器添加对外映射端口](https://www.cnblogs.com/zhumengke/articles/13525837.html)

## 一、背景

一般在运行容器时，我们都会通过参数 -p（使用大写的-P参数则会随机选择宿主机的一个端口进行映射）来指定宿主机和容器端口的映射，例如

```
docker run -it -d --name [container-name] -p 8088:80 [image-name]
```

这里是将容器内的80端口映射到宿主机的8088端口

参数说明

-   -d 表示后台运行容器
-   -t 为docker分配一个伪终端并绑定到容器的标准输入上
-   -i 是让容器的标准输入保持打开状态
-   -p 指定映射端口

在运行容器时指定映射端口运行后，如果想要添加新的端口映射，可以使用以下两种方式：

## 二、方法一：重新制作镜像

将现有的容器打包成镜像，然后在使用新的镜像运行容器时重新指定要映射的端口

大概过程如下：

1.   停止现有容器

     ```bash
     docker stop container-name
     ```

2.   将容器commit成为一个镜像

     ```bash
     docker commit container-name new-image-name
     ```

3.   用新镜像运行容器

     ```bash
     docker run -it -d --name container-name -p p1:p1 -p p2:p2 new-image-name
     ```

## 三、方法二：修改配置文件

修改要端口映射的容器的配置文件

3.   停止现有容器

    ```bash
    docker stop 容器ID
    ```
    
2.   停止docker服务，**必须先停止，不然修改不生效** 

     ```bash
     systemctl stop docker
     ```

3.   进到/var/lib/docker/containers 目录下找到与 Id 相同的目录

     ```
     cd /var/lib/docker/containers/容器ID
     ```

4.   修改 hostconfig.json 和 config.v2.json文件：

     ![img](https://img2020.cnblogs.com/blog/827744/202008/827744-20200818203654715-1021369230.png)

5.   修改hostconfig.json，在PortBindings后加上要绑定的端口，添加端口绑定：

     ```bash
     # 表示绑定端口 3306
     "3306/tcp": [{"HostIp": "","HostPort": "3306"}]
     ```

     ![img](https://img2020.cnblogs.com/blog/827744/202008/827744-20200818203701169-1817258600.png)

6.   修改config.v2.json在ExposedPorts后加上要暴露的端口

     ```bash
     # 表示暴露端口 3306
     "3306/tcp":{}
     ```

     ![img](https://img2020.cnblogs.com/blog/827744/202008/827744-20200818203705896-316450832.png)

7.   改完之后保存启动docker，查看添加的端口是否已映射绑定上

    ```
    systemctl start docker
    ```

## 四、查看端口映射成功

在容器外执行

```
netstat -an |grep 3306
```

如果有进程存在则表示有映射