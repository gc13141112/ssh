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

a.在root下vi /etc/sysconfig/network  修改hostname=master(管理节点);

b.在root下vi /etc/hosts 添加各个节点ip 及名称
