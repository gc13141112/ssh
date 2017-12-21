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
