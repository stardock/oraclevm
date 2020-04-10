# oraclevm
oracle甲骨文免费机换内核


去年那一波搞的日本节点两台免费机，选的是【映像: CentOS-7-PV-2019.08.20-0】，docker了v2由于回国丢包严重一直吃灰。
昨天想把bbr加上看看能不能凑合用，按这个方式手工处理的
https://www.hostloc.com/forum.php?mod=viewthread&tid=591266&extra=&highlight=oracle%2Bcentos%2B%2Bbbr&page=1

    sudo -i
    centos7 开启原生BBR
    rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
    yum --enablerepo=elrepo-kernel -y install kernel-ml kernel-ml-devel kernel-ml-headers

    ###查看最新内核
    cat /etc/grub2-efi.cfg |grep menuentry

    #######设置启动项
    vi  /boot/grub2/grubenv
    # GRUB Environment Block
    saved_entry=CentOS Linux (5.3.0-1.el7.elrepo.x86_64) 7 (Core)

    开启BBR
    vi /etc/sysctl.conf
    net.core.default_qdisc = fq
    net.ipv4.tcp_congestion_control = bbr

    关闭selinux
    vi  /etc/sysconfig/selinux
    SELINUX=disabled

    重启 init 6

复制代码


为了保险按前辈方法重启前把VNC开开了，结果启动时VNC卡在中间一步不动了，控制台强制关机也要等5分钟才能关闭，试了几次都没启动成功。
新内核好像是5.5.9，VNC里还能看到旧的3.x内核（此时在VNC切回旧内核应该会正常），不过昨晚太晚了关机后没切换回旧内核就睡了。

今天起来发现vnc连接已经断了，且控制台里也没有VNC连接了，控制台里“新建控制台连接”是灰色的，不能再开VNC了，不知道是不是没启动的缘故。

抢救过程：  由于还有第二个正常的centos7免费机，所以用DD**抢救
-----------------------------------------
停止 分离 挂载到新机器
dd if=/dev/sda of=/dev/sdb
从新机器分离 附加到旧机器 开机
-----------------------------------------
结果DD后还是不能启动，点了启动过一两分钟就自动终止了，控制台也不能打开VNC，看不到启动过程。

不想删了免费机再抢，还有别的方法挽救吗？

另：关机时间太长以后免费名额会不会被别人抢走，再启动时由于没有资源所以启动不成功？（好像在论坛里看到有人这么说过）

最后晚上10:30删机，运气不错用脚本花了30分钟又抓了一只小机。


===================== 无敌分割线（2020.3.4更新） =====================

拿新抓的小机还是选的centos7，还是用一键安装（wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh）
报错：
Error: /boot/grub2/grub.cfg not found, please check it.

然后安装 grub2 更新以下 grub 配置：
yum install -y grub2
grub2-mkconfig -o /boot/grub2/grub.cfg

重启还是卡壳（脚本安装的最新内核是5.5.7）

成功方法，供大家参考：
去年美西的oracle 免费机也是用这个一键脚本成功开的BBR，上去看了用的是5.3.9，我怀疑是不是5.5不兼容，所以下面是手工指定安装5.3.9尝试：

wget http://mirror.rc.usf.edu/compute_lock/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-5.3.9-1.el7.elrepo.x86_64.rpm
yum install ./kernel-ml-5.3.9-1.el7.elrepo.x86_64.rpm -y
//重新创建内核配置
grub2-mkconfig -o /boot/grub2/grub.cfg

//查看系统中已有内核：
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
//调整默认内核
grub2-set-default 0

//开启BBR
vi /etc/sysctl.conf
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

//关闭selinux
vi  /etc/sysconfig/selinux
SELINUX=disabled

reboot （成功）
然后查看lsmod |grep bbr

------------------------------------------------------------
显示所有内核
rpm -qa | grep kernel
最后，删除不能启动的内核
yum remove kernel-ml-5.5.7-1.el7.elrepo.x86_64


REF: https://www.hostloc.com/thread-652292-1-1.html
