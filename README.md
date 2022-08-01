# deploy-kafka-on-docker
![image](https://github.com/qiaonanasen/deploy-kafka-on-docker/blob/local/image/1.png)
### 1.什么是docker?
开发人员开发完成了某个功能，接下来需要QA人员来测试该功能，QA人员的电脑上并没有安装开发人员所配置的环境，这时QA就得安装配置一遍环境。如果另一个人再运行又要重新装环境，这无疑增加了很多工作量，环境如果搭建的不好还可能出现“在我的电脑上能跑怎么到你那就不行”，怨气buff+++。有什么办法可以把代码和它所运行的环境一起发送给对方呢？
Docker 就是这样一个平台，它以容器的形式将应用程序及其所有依赖项打包在一起。这种容器化确保应用程序可以在任何环境中工作。每个应用程序都在单独的容器上运行，并且有自己的一组依赖项和库。这确保每个应用程序独立于其他应用程序，让开发人员确信他们可以构建不会相互干扰的应用程序。如下图所示：
![image](https://github.com/qiaonanasen/deploy-kafka-on-docker/blob/local/image/2.png)
图1.docker运行框架

### 2.docker基本概念
![image](https://github.com/qiaonanasen/deploy-kafka-on-docker/blob/local/image/3.png)
图2.docker基本构成
Dockerfile： Dockerfile 是一个文本文档，它的作用类似于一个纲要，用于指导image怎么构建。Docker  daemon顺序读取 docker file 中的指令来构建镜像。
Docker daemon:接收Docker Client发送的操作指令并执行；另一个重要职能是管理Docker的对象，如镜像、容器、数据卷、网络。
Docker Image：通俗地说，Docker Image 可以比作一个用于创建 Docker 容器的模板。因此，这些只读模板是容器的构建块。您可以使用docker run来运行映像并创建容器。就类似于我们做蛋糕用的模具一样，通过dockerfile中的说明来添加不同的原材料，进而做出一模一样的蛋糕。这就是前文提到的，bulid ,ship ，and run anywhere。
Docker Container： 它是 Docker Image 的运行实例，可以简单的把image理解为可执行程序，container就是运行起来的进程。
Docker Registry :是用来管理image的，Docker 镜像存储在 Docker Registry 中。Docker Hub 是一个公共存储库，当我们需要一些image又不想自己去建造时，可以直接从Docker Hub 搜索拉取。
Docker compose:用于管理多个image
### 3.docker和虚拟机的区别？
虚拟机设计的初衷是分割硬件，减少在服务器设备上的支出，将一个物理服务器切分成多个独立的虚拟机来完成许多工作。
由于只有一台主机，因此可以利用虚拟机管理程序的集中功能高效地管理所有虚拟环境。这些guestOS完全相互独立，这意味着你可以在不同的虚拟机里安装不同的系统环境。
但是当我们开发某个需求时，可能只需要使用linux中很小一部分的功能，但是由于虚拟机是一个完整的系统，它会配置整个的linux。虚拟机可能占用主机的大量系统资源，虚拟机的大小为数GB。在虚拟服务器上运行单个应用程序意味着还要运行Guest OS以及Guest OS运行所需的所有硬件的虚拟副本。这样就增加了很多RAM和CPU资源消耗。迁移虚拟机上运行的应用程序的过程也可能很复杂，因为它始终附加在操作系统上。因此，必须同时迁移应用程序和操作系统。同样，在创建虚拟机时，系统管理程序会分配专用于VM的硬件资源。 不过相比再购买一个实体服务器这虚拟机的成本还是很实惠的。
而docker相比虚拟机需要虚拟出一整个系统，它更像微信，docker里的container类似于微信的小程序，不管你是安卓还是IOS用户，都可以在微信里使用同一个小程序，而不用像APP一样需要从专门的applestore和android store下载。手机内存的限制，导致我们如果安装100个APP可能很困难，但是在微信中使用超过100个小程序很简单。
一句话概括，虚拟机是系统虚拟化，docker是进程虚拟化。

3.png
![image](https://github.com/qiaonanasen/deploy-kafka-on-docker/blob/local/image/4.png)
图3 docker和虚拟机的区别
### 4.docker底层原理？
（1）docker使用go语言编写
* （2）docker依赖于 Linux 内核以下几个特性来实现隔离：
    * 利用linux的命名空间(Namespaces)
    * 利用linux控制组(Control Groups)
    * 利用linux的联合文件系统(Union File Systems)
    * 这也就意味着Docker只能在linux上运行。
    * 在windows、MacOS上运行Docker，其实本质上是借助了虚拟化技术，然后在linux虚拟机上运行的Docker程序。
（3）cgroup :用来限制、控制与分离一个进程组的资源（如CPU、内存、磁盘输入输出等）。
（4）namespace ：封装抽象，限制，隔离，使命名空间内的进程看起来拥有他们自己的全局资源；namespace
* UTS 主机和域名 提供了主机名与域名的隔离，这样每个docke容器就可以拥有独立的主机名和域名了，在网络上可以被视为一个独立的节点，而非宿主机上的一个进程。
* IPC 信号量、消息队列和共享内容 实现了容器与宿主机、容器与容器之间的IPC隔离
* PID 进程编号 用来隔离进程的ID空间，使得不同pid namespace里的进程ID可以重复且相互之间不影响。
* network 网络设备、网络栈、端口等 提供了关于网络资源的隔离
* mount 挂载点(文件系统) 通过隔离文件系统挂载点对隔离文件系统提供支持，具有独立的挂载点信息，使每个mnt namespace可具有独立的目录层次
* user 用户和用户组 对用户和用户组进行隔离，user namespace一个特定的目标就是允许一个进程在容器中的操作拥有root特权。docker中默认root登录，挂载宿主机任意目录后，可通过容器里的root账号对宿主机数据进行破坏，风险很大，user namespace可以让容器有一个“假”的root用户，它在自己的容器内是root，被映射到容器外宿主机是一个非root用户。

### 5.docker安装
（1）MAC环境下可参考以下两个教程：本文不再赘述
    简约安装教程：https://blog.csdn.net/qq_42405688/article/details/124468315
    详细安装教程：https://dockerdocs.cn/docker-for-mac/index.html
（2）安装完成后如果出现：bash : docker :command not found 的解决办法（未配置环境变量导致） 
     https://blog.csdn.net/shifengwang123/article/details/99416921
（3）安装完成后如果出现下载image过慢的情况：（使用国内 Docker Hub 镜像服务器）
     https://zhuanlan.zhihu.com/p/291280980
### 6.docker常用命令
* 查看命令帮助
docker 命令名称 —help
* 查看有哪些命令
docker help
* 搜索image
docker docker search 关键字
* 下载image
docker pull <image>
* 查看已安装image
docker images
* 删除镜像
Docker rmi imagename
* 启动容器
Sudo docker run -d -p 80:80 imagename 
（-d(detach)表示在后台执行, -p 80:80表示要使用的端口）
* 查看正在运行的container
docker ps
* 查看运行的container使用的资源
docker  stats
* 启动，停止，删除，重启
docker start <id-conteneur>
docker stop  <id-conteneur>
docker rm <id-conteneur>. # rmi是删除镜像rm删除容器
docker restart <id-conteneur>


### 7.docker实战：在docker上部署并使用kafkahttps://www.cnblogs.com/shanfeng1000/p/14638455.html
* 拉取镜像
docker pull wurstmeister/kafka
docker pull wurstmeister/zookeeper

* 检查拉取镜像是否成功
docker image ls
