# ssh
## 1. 在node0和node1上进行如下操作：

    在/etc/hosts文件中添加如下内容：

    10.141.4.36      master

    10.141.4.39      computer

    在/home/cluster(也就是当前目录)目录下执行如下命令，生成.ssh目录：

    ssh-keygen -t rsa

    A：cat /root/.ssh/id_rsa.pub

    echo "miyao" >>/root/.ssh/authorized_keys

    “”号中间的是你刚才复制出来的内容
  
    echo 在B机器上执行一下
   
    然后再试
    
# 亚马逊集群配置

## 1.更改主机名

### 管理节点

#### a.在root下vi /etc/sysconfig/network  修改hostname=master(管理节点);

#### b.在root下vi /etc/hosts 添加各个节点ip 及名称

    1.2.3.4 master

    1.2.3.4 node1

### 计算节点

#### a.在root下vi /etc/sysconfig/network  修改hostname=node1(计算节点);

#### b.在root下vi /etc/hosts 添加各个节点ip 及名称

    1.2.3.4 master

    1.2.3.4 node1

## 2.开始安装torque (注意在此目录下）

    yum -y install libxml2-devel openssl-devel gcc gcc-c++ boost-devel libtool

    ./configure --prefix=/home/ec2-user/data/torque-4.2.9 --with-scp --with-default-server=master
    
    make && make install && make packages
    
    cp contrib/init.d/{pbs_{server,sched,mom},trqauthd} /etc/init.d/

    for i in pbs_server pbs_sched pbs_mom trqauthd; do chkconfig --add $i; chkconfig $ion; done
    
    vi /etc/profile
    export PATH=/home/ec2-user/data/torque-4.2.9/bin:$PATH
    export PATH=/home/ec2-user/data/torque-4.2.9/sbin:$PATH
    source /etc/profile
    ./torque.setup root
    ./gterm -t quick
    
    for i in pbs_server pbs_sched pbs_mom trqauthd; do service $i start; done
    vi /var/spool/torque/server_priv/nodes
    master np=40
    node1 np=60
    vi /var/spool/torque/mom_priv/config
    pbsserver master
    logevent 255
    
    ps -e | grep pbs
    for i in pbs_server pbs_sched pbs_mom trqauthd; do service $i restart; done
    for i in pbs_mom trqauthd; do service $i start; done
    source /home/ec2-user/data/LIBRARY/intel/composer_xe_2015.2.164/bin/ifortvars.sh intel64
    source /home/ec2-user/data/LIBRARY/intel/composer_xe_2015.2.164/bin/iccvars.sh intel64
    export LD_LIBRARY_PATH=/home/ec2-user/data/LIBRARY/intel/lib/intel64:$LD_LIBRARY_PATH
    
# NFS server配置：
## 1. 在NFS server的EC2上，编辑/etc/exports文件，将/home/ec2-user/data 共享



    $ cat /etc/exports
    
    /home/ec2-user/data   54.222.142.217(rw,sync,no_root_squash,no_subtree_check)
    /home/ec2-user/data   10.0.1.59(rw,sync,no_root_squash,no_subtree_check)
## 2. 重启NFS服务

sudo /etc/init.d/nfs restart(注意要在你的管理节点，也就是你要给别人的)

    Shutting down NFS daemon:                                  [  OK  ]
    Shutting down NFS mountd:                                  [  OK  ]
    Shutting down NFS quotas:                                  [  OK  ]
    Shutting down RPC idmapd:                                  [  OK  ]
    Starting NFS services:                                     [  OK  ]
    Starting NFS quotas:                                       [  OK  ]
    Starting NFS mountd:                                       [  OK  ]
    Starting NFS daemon:                                       [  OK  ]
    Starting RPC idmapd:                                       [  OK  ]
## 3. 编辑安全组，允许2049端口能被client访问

Ingress rules:

    Protocol 	Port range 	Source 	Description
    TCP 	        2049 	        10.0.0.0/16 	
## 4.客户端配置

### 1. 安装NFS client

     $ sudo yum -y install nfs-utils

### 2. 挂载共享路径

     $ sudo mount 10.0.1.23:/home/ec2-user/data /home/ec2-user/data

     这里10.0.1.23为NFS server的地址
     
## 5.疑难问题
    
    1.ulimit -s unlimited 自启动生效
    
    http://www.cnblogs.com/52php/p/5666755.html
    
    vi /etc/rc.d/rc.local
    
    ulimit -s ulimited
    ulimit -c ulimited
    
    最后重启 reboot
    
    2.彻底卸载 Intel Parallel Studio
    
    Intel 的开发工具默认安装在 /opt/intel 目录下，由于一时手贱，用 sudo rm -rf /opt/intel 把整个 intel 文件夹给删除了。
    在重新安装 Intel Parallel Studio 的时候一直提示 “已安装该产品”，进而导致无法再次安装。实际上 是因为在上一次安装的时候，安装了一大堆 rpm 文件，     并将文件的信息写入了 rpm 数据库中，而在删除的 时候数据库没有被更新。再次安装的时候，会在数据库中检测到，继而出现 “已安装该产品” 的提示。
    解决办法很简单，找到所有已安装的 intel 相关的包，然后删除。
    
    rpm -qa | grep intel | awk '{print"yum remove -y",$1}' > uninstall.sh
    
    查看 uninstall.sh 文件，删除其中不以 intel 开头的包。然后用 root 权限执行该脚本即可。
