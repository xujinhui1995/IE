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

	取消前期实验配置
	[LSW1-GigabitEthernet0/0/1]undo port-security protect-action
	[LSW1-GigabitEthernet0/0/1]undo port-security mac-address sticky
	[LSW1-GigabitEthernet0/0/1]undo port-security enable

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113091232.png)

地址漂移演示

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113093638.png)

	修改优先级
	[LSW1-GigabitEthernet0/0/1]mac-learning priority 3 #0-3数值越大优先级越高

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113094229.png)

探测防环

	取消优先级
	[LSW1-GigabitEthernet0/0/1]undo mac-learning priority

	全局开启mac漂移探测（默认开启）
	[LSW1]mac-address flapping detection 
	查看是否开启
	[LSW1]display current-configuration | in mac
	drop illegal-mac alarm

	设置环路行为阻断时间10s，尝试次数2次（第二次阻断）
	[LSW1]vlan 1
	[LSW1-vlan1]loop-detect eth-loop block-time 10 retry-times 2
	
	设置接口触发执行动作
	[LSW1-GigabitEthernet0/0/2]mac-address flapping trigger error-down
	
	设置error-down自动恢复
	[LSW1]error-down auto-recovery cause mac-address-flapping interval 30

g0/0/2第一次漂移block了10s
g0/0/1第一次漂移block了10s

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113100225.png)

g0/0/2第二次漂移error down

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113100225.png)

30s后恢复

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113100225.png)