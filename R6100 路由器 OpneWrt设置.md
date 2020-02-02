# R6100 路由器 OpneWrt设置

1. ## Openwrt固件下载

   1. [Openwrt  R6100 19.0.7.1版本 ](https://downloads.openwrt.org/releases/19.07.1/targets/ar71xx/nand/)
   2. [Openwrt所有固件下载](https://downloads.openwrt.org/releases/)

2. ## 刷固件

   1.  ### TFTP方法刷固件(img文件)

      具体方法：
       首先，路由插好网线连上电脑，接上电源。
       打开CMD，ping 192.168.1.1 –t，监测路由联通状态
       将原厂固件R6100-V1.2.0.4.img文件复制到c盘根目录，并改个简单的名字，我改名为r6100.img。
       再打开cmd，输入TFTP -i 192.168.1.1 PUT r6100.img，不要着急点回车确认，接下来将路由进入tftp模式，安装reset不松手，拔掉电源再接上，路由灯闪烁后或者ping通了后立马回车，将TFTP -i 192.168.1.1 PUT r6100.img这条命令执行，过一会就出现传送成功的，松开reset键。

      ![img](https:////upload-images.jianshu.io/upload_images/2042197-607035dd7c60e74e.png?imageMogr2/auto-orient/strip|imageView2/2/w/388/format/webp)

      

      接着路由器会自动重启，等ping能通后过几分钟就可以访问192.168.1.1了，进去后发现已经是原厂固件了，刷原厂固件完毕。

      [官方参考 U-Boot TFTP](https://openwrt.org/docs/guide-user/installation/generic.flashing.tftp)

      [官方参考 安装OpenWrt](https://openwrt.org/docs/guide-user/installation/generic.flashing#method_1via_oem_firmware)

   2. ### sysupgrade升级固件

      sysupgrade命令参数:

      -i 交互模式

      -c 保留 /etc 中所有修改过的文件

      -n 重刷固件时不保留配置文件

      -v 详细的输出信息

      -h 显示帮助信息

      

      具体使用:

      ``` shell
      sysupgrade -i -n /tmp/r6100-squashfs-sysupgrade.tar
      ```

      

3. ## 恢复出厂设置

   1. ### firstboot命令恢复出厂设置

      ```` shell
      mount_root	#切换到root
      firstboot #一次性擦除指令
      reboot -f #重启
      ````

      

   2. ### rm命令删除方法来恢复出厂设置

      ````Shell
      df -h #硬盘查看
      
      rm -rf /overlay/ #清理数据
      reboot -f #重启
      ````

      

      原理:

      OpenWRT使用的是Overlay透明挂载技术，首先将/rom挂载为/根文件，然后再用/overlay覆盖在/之上。 这样的话进行文件系统的变更，修改，所做的操作将在overlay中记录。rom是不改变的。 而最简单的恢复出厂设置方法，即是删除掉/overlay下所有文件。

      ```` shell
      root@OpenWrt:~# df -h
      Filesystem                Size      Used Available Use% Mounted on
      /dev/root                 3.0M      3.0M         0 100% /rom
      tmpfs                    60.7M    620.0K     60.1M   1% /tmp
      /dev/ubi0_1              99.6M      1.9M     93.0M   2% /overlay
      overlayfs:/overlay       99.6M      1.9M     93.0M   2% /
      tmpfs                   512.0K         0    512.0K   0% /dev
      ````

      

      这里看到/rom的空间占有永远都是100%，这一部分是是只读的，不可以更改的。
      你所有可以更改的配置是在/overlay下。
      通过overlay技术，将/overlay（可读写）和/rom（只读）连在一起，这个就是你当前的OpenWRT的完整系统文件rootfs。

      

4. ## 软件安装

   1. ### luci界面汉化

      ````shell
      opkg update #更新软件库
      opkg install luci-i18n-base-zh-cn #下载汉化界面	
      ````

      

   2. ### usb移动硬盘支持

      ```` shell
      opkg update #更新软件库
      opkg install kmod-usb-storage kmod-usb-storage-extras block-mount kmod-fs-btrfs #安装USB支持,btrfs文件格式支持
      ll /dev/sda* #查看移动硬盘
      free -m #查看内存开销
      mkswap /dev/sda1 #设置sda1为虚拟内存
      swapon /dev/sda1 #开启虚拟内存  swapoff /dev/sda1 为关闭
      mkdir /mnt/udisk	#创建挂载点
      mount -o subvol=homefs /dev/sda2 /mnt/udisk/ #挂载 sda2到 挂载点
      ````

      

   3. ### 常用软件安装

      ````shell
      opkg install luci-i18n-aria2-zh-cn luci-app-cjdns luci-i18n-base-zh-cn luci-i18n-commands-zh-cn luci-i18n-ddns-zh-cn luci-i18n-diag-core-zh-cn luci-i18n-hd-idle-zh-cn luci-i18n-minidlna-zh-cn luci-i18n-simple-adblock-zh-cn luci-app-sqm luci-i18n-samba-zh-cn luci-i18n-shadowsocks-libev-zh-cn luci-i18n-transmission-zh-cn luci-i18n-statistics-zh-cn luci-ssl-openssl ca-bundle
      ````

      

   4. ### 防火墙安装

      ```shell
      opkg install luci-i18n-firewall-zh-cn
      ```

      

   

5. ## 软件设置

   1. ### 时间设置

      1. 查看时间

         ````shell
         date
         ````

         

      2. 方法1.luci界面设置

         设置:系统-系统

         

         ![image-20200202145620044](C:\Users\Administrator\Desktop\image-20200202145620044.png)

         

      3. 方法2.设置时区,时间

         ````shell
         vi /etc/config/system
         
         option zonename 'Asia/Shanghai'
         option timezone 'CST-8'
         ````

         

         

   2. ### DDNS设置

      1. #### 免费域名申请

         免费域名申请地址:

         ````html
         https://my.freenom.com/domains.php
         ````

         

         参考:

         [1. 教你申请免费域名](https://blog.csdn.net/mwz1tn/article/details/90241676)

         [2. \.ML、.CF、.GA、.TK四大顶级域名免费注册](https://www.douban.com/group/topic/43212963/)

         

      2. #### DDNS设置

      3. #### 验证

         

         

   3. ### Samba网络共享

      ```` shell
      opkg install shadow-useradd shadow-groupadd #添加 useradd  groupadd 命令
      useradd admin	#添加admin用户
      chown -R amdin:admin /mnt/udisk #设置/mnt/udisk 的owner 是admin
      smbpasswd -a admin	#在smbpasswd文件添加访问人员
      service samba restart	#重启samba服务
      ````

      ![image-20200201221432380](C:\Users\Administrator\Desktop\image-20200201221432380.png)

      

      

      

   4. ###  Shadowsocks-libev

   5.  ### DNSCrypt-Proxy设置

   6. ### 硬盘休眠

      设置:

      ![image-20200202124039549](C:\Users\Administrator\Desktop\image-20200202124039549.png)

      

   7. ### aria2c设置

      设置:

      1. #### 需要的包

         ````shell
      opkg install  luci-ssl-openssl ca-bundle luci-i18n-aria2-zh-cn
         ````
   
         

      2. #### 下载的目录要对运行的用户要有读写权限

      3. #### 在luci界面上设置

         1. ##### 基础选项

            ![image-20200202153822417](C:\Users\Administrator\Desktop\image-20200202153822417.png)

         2. ##### RPC选项

            ![image-20200202153902048](C:\Users\Administrator\Desktop\image-20200202153902048.png)

         3. ##### Http/Ftp/SFtp选项

            图一:![image-20200202154107562](C:\Users\Administrator\Desktop\image-20200202154107562.png)

            图二:![image-20200202154203128](C:\Users\Administrator\Desktop\image-20200202154203128.png)

         4. ##### BT选项

            图一:![image-20200202154239624](C:\Users\Administrator\Desktop\image-20200202154239624.png)

            图二:![image-20200202154304242](C:\Users\Administrator\Desktop\image-20200202154304242.png)

         5. ##### 高级选项

            ![image-20200202154336002](C:\Users\Administrator\Desktop\image-20200202154336002.png)

         

         

         

         

         

      4. #### 下载的界面(WebUi)

         [地址1](http://webui-aria2.ghostry.cn/)

      5. #### WebUi链接到aria2

         设置:

         设置-链接设置

         ![image-20200202154857272](C:\Users\Administrator\Desktop\image-20200202154857272.png)

         

      

      

   8. ### cjdns设置

   9. ### miniDLNA

      设置:

      ![image-20200202161235297](C:\Users\Administrator\Desktop\image-20200202161235297.png)

      

      

   10. ### 简易 AdBlock

       

   11. ### SQM Qos

       1. 基础设置

          ![image-20200202162651038](C:\Users\Administrator\Desktop\image-20200202162651038.png)

       2. 队列选择

          ![image-20200202162727140](C:\Users\Administrator\Desktop\image-20200202162727140.png)

          

          

          

       