# ssh
1. 在node0和node1上进行如下操作：

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
### 1. 在NFS server的EC2上，编辑/etc/exports文件，将/home/ec2-user/data 共享

    $ cat /etc/exports
    /home   54.222.142.217(rw,sync,no_root_squash,no_subtree_check)
    /home   10.0.1.59(rw,sync,no_root_squash,no_subtree_check)
