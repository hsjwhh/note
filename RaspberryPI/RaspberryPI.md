## 项目用 RaspberryPI 4B 系统配置大概流程

1. 刻录镜像，如启动报错，需安装固件补丁；
	```shell
	# 报错信息
	start4.elf: is not compatible This board requires newer sofrware
	# GitHub 下载最新固件启动文件更新 SD 卡上的文件
	地址：https://github.com/raspberrypi/rpi-firware
	fixup4.dat
	srar4.elf
	```
2. 登录路由器查看 RaspberryPI 的 ip 地址，ssh 登录；
3. 扩容 SD 卡，直接运行脚本`/usr/bin/rootfs-expand`；
4. 开启 Wi-Fi（可选）`nmcli dev wifi con "SSID 名称" password "Wi-Fi 密码"；
5. 安装所需插件；
6. 关闭 selinux 及 firewalld
	```shell
	# 修改以下文件
	vi /etc/selinux/config
	# 将 SELINUX=enforcing 改为 SELINUX=disable
	```