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

**VLAN实验**

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113103857.png)

- 基于port端口划分vlan

------------

	[LSW1]vlan 10
	[LSW1]vlan 20

查看VLAN信息

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113103624.png)

	配置链路类型及缺省VLAN
	[LSW1]interface e0/0/1
	[LSW1-Ethernet0/0/1]port link-type access
	[LSW1-Ethernet0/0/1]port default vlan 10
	
	[LSW1]interface e0/0/2
	[LSW1-Ethernet0/0/2]port link-type access
	[LSW1-Ethernet0/0/2]port default vlan 20

查看所有接口vlan信息

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113103833.png)

- 基于MAC地址划分VLAN

-------
	将PC3划入VLAN10
	[LSW2]vlan 10
	[LSW2-vlan10]mac-vlan mac-address 5489-9835-52BD

	接口开启mac vlan
	[LSW2-Ethernet0/0/1]mac-vlan enable
	[LSW2-Ethernet0/0/1]port hybrid untagged vlan 10

	设置接口类型及允许通过的vlan
	[LSW1-Ethernet0/0/10]port link-type trunk
	[LSW1-Ethernet0/0/10]port trunk allow-pass vlan 10 20

	[LSW2-Ethernet0/0/10]port link-type trunk
	[LSW2-Ethernet0/0/10]port trunk allow-pass vlan 10 20


- 基于IP子网划分VLAN

-----------------------
	
	设置vlan子网，启用ip划分vlan，设置接口类型
	[LSW2-vlan20]ip-subnet-vlan ip 1.1.2.0 24
	[LSW2-Ethernet0/0/2]ip-subnet-vlan enable
	[LSW2-Ethernet0/0/2]port hybrid untagged vlan 20 

- 配置hybrid接口实现不同vlan间的访问

------

	PC2、3、4由1.1.1.0/24修改为1.1.0.0/16网段
	
	修改链路类型
	[LSW1-Ethernet0/0/2]undo port default vlan
	[LSW1-Ethernet0/0/2]port link-type hybrid
	创建vlan
	[LSW1]vlan 100
	[LSW2]vlan 100
	
	修改所属vlan及允许通过vlan 
	[LSW1-Ethernet0/0/2]port hybrid pvid vlan 100
	[LSW1-Ethernet0/0/2]port hybrid untagged vlan 10 20 100
	接口 允许pvid 100的vlan通过
	[LSW2-Ethernet0/0/1]port hybrid untagged vlan 100
	[LSW2-Ethernet0/0/2]port hybrid untagged vlan 100
	接口允许通过vlan 100
	[LSW1-Ethernet0/0/10]port trunk allow-pass vlan 100
	[LSW2-Ethernet0/0/10]port trunk allow-pass vlan 100



**VLAN间路由**

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113144208.png)

- 单臂路由

---------

	配置VLAN
	[LSW1-Ethernet0/0/1]port link-type access
	[LSW1-Ethernet0/0/1]port default vlan 10
	[LSW1-Ethernet0/0/2]port link-type access
	[LSW1-Ethernet0/0/2]port default vlan 20
	[LSW1-Ethernet0/0/3]port link-type access
	[LSW1-Ethernet0/0/3]port default vlan 30
	[LSW1-Ethernet0/0/4]port link-type access
	[LSW1-Ethernet0/0/4]port default vlan 30
	
	路由至交换机线路
	[LSW1-Ethernet0/0/10]port link-type trunk
	[LSW1-Ethernet0/0/10]port trunk allow-pass vlan 10 20

	配置路由
	[AR1]int g0/0/1.10
	[AR1-GigabitEthernet0/0/1.10]dot1q termination vid 10
	[AR1-GigabitEthernet0/0/1.10]arp broadcast enable
	[AR1-GigabitEthernet0/0/1.10]ip add 1.1.1.254 24 
	[AR1-GigabitEthernet0/0/1.10]int g0/0/1.20
	[AR1-GigabitEthernet0/0/1.20]dot1q termination vid 20
	[AR1-GigabitEthernet0/0/1.20]arp broadcast enable
	[AR1-GigabitEthernet0/0/1.20]ip address 1.1.2.254 24


- SVI

---------

	[LSW1]interface Vlanif 30
	[LSW1-Vlanif30]ip address 1.1.3.254 24
	[LSW1]interface Vlanif 40
	[LSW1-Vlanif40]ip address 1.1.4.254 24

接口UP条件：存在该VLAN、且该VLAN有活动接口

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113143648.png)


**VLAN聚合**

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113155333.png)

	创建VLAN，启用聚合，并设置access VLAN
	[LSW1]vlan batch 10 20 100
	[LSW1]vlan 100
	[LSW1-vlan100]aggregate-vlan
	[LSW1-vlan100]access-vlan 10 20

	配置接口
	[LSW1-GigabitEthernet0/0/1]port link-type access
	[LSW1-GigabitEthernet0/0/1]port default vlan 10
	[LSW1-GigabitEthernet0/0/2]port link-type access
	[LSW1-GigabitEthernet0/0/2]port default vlan 20

	配置网关
	[LSW1]interface Vlanif 100
	[LSW1-Vlanif100]ip address 100.1.1.254 24

	设置代理
	[LSW1-Vlanif100]arp-proxy inter-sub-vlan-proxy enable
	配置接口
	[LSW2-GigabitEthernet0/0/10]port link-type trunk
	[LSW2-GigabitEthernet0/0/10]port trunk allow-pass vlan 10
	配置接口
	[LSW2-GigabitEthernet0/0/3]port link-type access
	[LSW2-GigabitEthernet0/0/3]port default vlan 10

不允许vlan100通过

![](https://raw.githubusercontent.com/xujinhui1995/IE/main/image/20201113155256.png)


	VLAN与super VLAN间路由
	[LSW2]vlan 200
	[LSW2-GigabitEthernet0/0/4]port link-type access
	[LSW2-GigabitEthernet0/0/4]port default vlan 200

	允许通信
	[LSW2-GigabitEthernet0/0/10]port trunk allow-pass vlan 200
	[LSW1-GigabitEthernet0/0/10]port trunk allow-pass vlan 200

	增加路由
	[LSW1]int vlan 200
	[LSW1-Vlanif200]ip add 200.1.1.254 24
	