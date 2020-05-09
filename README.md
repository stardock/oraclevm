# oraclevm
oracle甲骨文免费机换内核centos7

```  
yum install git -y
git clone -b master https://www.github.com/stardock/CentOS-Kernel-415
cd CentOS-Kernel-415
rpm -ivh kernel-ml-4.15.4-1.el7.elrepo.x86_64.rpm
rpm -ivh kernel-ml-devel-4.15.4-1.el7.elrepo.x86_64.rpm
rpm -ivh kernel-ml-headers-4.15.4-1.el7.elrepo.x86_64.rpm
```  

显示默认启动内核~这个在安装完后，最新安装的内核会自动设置为启动内核~  

grubby --default-kernel  

查看所以安装的内核信息~  

grubby --info=ALL  

设置启动内核~  

grubby --set-default /boot/[查到的内核信息]  
比如  
grubby --set-default /boot/vmlinuz-5.5.1-1.el8.elrepo.x86_64  

然后重启就完事了~  

Reference:  
https://blog.csdn.net/KnYoboy/article/details/104147009  
