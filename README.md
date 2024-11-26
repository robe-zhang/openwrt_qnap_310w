## openwrt_qnap_310w 刷 openwrt 系统方法：  
# 1，下载 openwrt 系统：  
下载链接：https://downloads.openwrt.org/releases/23.05.5/targets/ipq807x/generic/openwrt-23.05.5-ipq807x-generic-qnap_301w-squashfs-sysupgrade.bin  
下载 -squashfs-sysupgrade.bin 结尾的文件  
复制到 ubuntu 系统下，运行命令：binwalk -e openwrt-23.05.5-ipq807x-generic-qnap_301w-squashfs-sysupgrade.bin  
得到 kernel 和 root 两个文件。稍后把这两文件写入 qnap-310w 分区，刷机完成  
# 2，打开 ssh：  
qnap-310w 原生的系统，开机状态下，长按 wps 按键，直到听到 di-di 两声，开启成功  
# 3，ssh 登录：端口号不是常用的 22，是 22200  
ssh admin@192.168.100.1 -p 22200  
# 4，检查当前系统：  
sudo fw_printenv -n current_entry  
如果显示 1 ，下面步骤跳过。开始传输文件刷机  
显示不是 1 ，运行以下命令  
sudo fw_setenv current_entry 1  
sudo reboot  
等重启后，再次登录 ssh：ssh admin@192.168.100.1 -p 22200  
检查当前系统：  
sudo fw_printenv -n current_entry  
现在显示 1。开始传输文件刷机  
# 5，传输文件：  
scp -P 22200 kernel admin@192.168.100.1:/tmp/
scp -P 22200 root admin@192.168.100.1:/tmp/  
把 kernel root 文件传输到 qnap-310w 的 /tmp 目录下  
# 6，刷机：  
sudo dd if=/tmp/kernel of=/dev/mmcblk0p1  
sudo dd if=/tmp/root of=/dev/mmcblk0p4  
sudo fw_setenv current_entry 0  
sudo fw_setenv boot_0 good  
sudo reboot  
写入 kernel root，修改环境变量，从 openwrt 系统启动，openwrt 系统就绪。  
重启，进入了 openwrt 系统   
刷机已经完成。openwrt 系统已经可以用了。  
  
## 下面是备份和更新 10G 网口 phy 芯片固件  
# 1，查看10G 网口 phy 芯片固件，所在的分区  
ssh 登录 openwrt 系统：用户名 root，没有密码，端口号 22 是默认端口，不用 -p 参数  
ssh root@192.168.1.1  
查看10G 网口 phy 芯片固件，所在的分区  
cat /proc/mtd|grep "0:ethphyfw"  
显示哪个分区，就备份哪个分区  
比如显示 mtd10，备份 mtd10 分区：   
dd if=/dev/mtd10 of=/tmp/ethphyfw.backup  
把 ethphyfw.backup 文件复制到电脑中，保存，备份  
# 2，下载最新 10G 网口 phy 芯片固件    
稍后补充  
# 3，更新10G 网口 phy 芯片固件  
mtd erase /dev/mtd_NUM  
mtd -n write /tmp/<10G_phy_firmware> mtd_NUM  
比如上面显示 10G 网口 phy 芯片固件，所在的分区是 mtd10 和 mtd11  
mtd erase /dev/mtd10  
mtd -n write /tmp/<10G_phy_firmware> mtd10  
mtd erase /dev/mtd11  
mtd -n write /tmp/<10G_phy_firmware> mtd11  
  
## 后续更新 openwrt 系统，直接在 openwrt 的 luci 界面中 system >> backup/flash firmware 中的 flash new firmware image 中更新  
  
## openwrt_qnap_310w 双系统切换命令：  
上面的刷机方法，把 openwrt 刷在 0 中，原生系统在 1 中  
想切换到 原生系统：sudo fw_setenv current_entry 1  
想切换到 openwrt系统：sudo fw_setenv current_entry 0  
  
