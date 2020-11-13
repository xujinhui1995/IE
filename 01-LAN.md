**交换及端口安全实验**

未开启端口安全

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201111091430.png)

开启端口安全

	[LSW1]int g0/0/1
	[LSW1-GigabitEthernet0/0/1]port-security enable
	
修改主机1的mac地址后，网络不通，该配置会老化

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113083245.png)

端口安全修改为sticky，重启依旧有效

	[LSW1]int g0/0/1
	[LSW1-GigabitEthernet0/0/1]port-security mac-address sticky
	查看
	[LSW1]display mac-address sticky


![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113083725.png)

端口安全相关参数配置

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113084252.png)


**MAC地址漂移实验**

