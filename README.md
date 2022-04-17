# video-demo
## 1、demo介绍
### 1.1 实现功能
* 通过 baetyl 远程向边缘侧下发进行视频 AI 推断的应用
* 在边缘侧查看部署的视频推断服务，包括查看 AI模型结构、截图文件、mqtt推断消息、最终合成视频
* 在云端控制台远程调试 AI 推断服务相关应用

### 1.2 部署应用介绍
部署**3**个video应用

* video-demo-0  
AI 视频推断服务，具体包含视频抽帧、图片推断、MQTT消息发送、视频流合成等功能模块

* dcell-service  
边缘端服务编排后台服务

* dcell-console  
边缘服务编排控制台服务

## 2、操作实践
### 2.1 登录
登录百度云边缘计算控制台[BIE](https://console.bce.baidu.com/iot2/bie/node/list)

### 2.2 导入配置
1. 点击左侧【配置管理】Tab
2. 点击【导入配置】按钮，选择**vide-demo-conf.json**文件导入，此操作将导入**10**个配置文件，这些文件将匹配到后续导入的应用
3. 各配置文件的作用可以看配置对应的描述信息
![导入配置](images/import-conf.png)

### 2.3 导入应用
1. 点击左侧【应用部署】Tab
2. 点击【导入应用】按钮，选择**video-demo-app.json**文件导入，此操作将导入**3**个AI应用
3. 点击可以查看各个应用详情  
![导入应用](images/import-app.png)

### 2.4 创建节点
1. 点击左侧【边缘节点】Tab
2. 点击【创建节点】按钮，按如下内容创建节点
  * 名称: **video-demo**
  * 类型：根据情况选择
  * AI加速卡：无
  * 选择官方系统模块：不需要勾选  
![创建节点](images/create-node.png)

### 2.5 给节点打标签
1. 点击刚创建的**video-demo**节点
2. 点击节点标签右侧的编辑按钮，增加**dcell:dcell**标签  
![增加标签](images/add-label.png)

### 2.6 安装节点
1. 点击左侧【边缘节点】Tab
2. 点击之前创建的**video-demo**节点
3. 点击【安装】按钮，如果系统为linux且没有docker及k3s，则分别复制docker及k3s安装命令在控制台执行安装，如果系统为macos，则k8s安装流程如下：[mac安装k8s](https://github.com/AliyunContainerService/k8s-for-docker-desktop)  
![安装命令](images/install-command.png)

4. 在环境安装完成后，复制节点安装命令并在控制台执行  
![安装](images/install.png)

5. 通过 kubectl -n baetyl-edge-system get pod 可以查看 baetyl 系统模块的运行情况
6. 通过 kubectl -n baetyl-edge get pod 可以查看用户模块的运行情况（AI推断程序即在此命名空间下），等待镜像下载启动后，状态如下  
![运行状态](images/install-status.png)

### 2.7 服务访问
#### 2.7.1 查看抽帧文件
1. 抽帧文件存储路径参见应用**video-demo-0**中的**dcell-system**卷的配置  
![卷](images/vol.png)
2. 然后在边缘端设备对应目录查看抽帧及推断文件  
![抽帧](images/pic.png)

#### 2.7.2 查看MQTT消息
1. 点击左侧【边缘节点】Tab，选择**video-demo**
2. 编辑 baetyl-broker 配置，暴露一个外部端口供测试使用  
![broker](images/broker.png)  
![broker-vol](images/broker-vol.png)  
3. 修改变量值为  
![broker-vol](images/broker-conf-val.png)  
	
	```yaml
	listeners:
	  - address: 'tcp://0.0.0.0:38003'
	principals:
	  - username: test
	    password: test
	    permissions:
	      - action: pub
	        permit:
	          - '#'
	      - action: sub
	        permit:
	          - '#'
	session:
	  sysTopics:
	    - $link
	    - $baetyl
	logger:
	  level: debug
	  encoding: console
	
	```  
4. 新增 38003 端口供测试使用。 并且需要设置映射宿主机端口以供连接(可以此处直接暴露，或者在端侧使用kubectl port-forward 命令暴露)  
![修改端口](images/30083.png)

5. 然后在本地使用MqttBox连接baetyl-broker  
```
kubectl -nbaetyl-edge-system port-forward baetyl-broker-xxx 38003:38003
```
![本地连接](images/mqttbox.png)

#### 2.7.3 查看本地编排信息
1. 映射**dcell-service**服务的8022端口到主机，映射**dcell-console**服务的8888端口到主机
```
kubectl -nbaetyl-edge port-forward sis-device-xxx 8022:8022
kubectl -nbaetyl-edge port-forward dcell-node-console-xxx 8888:8888
```

2. 浏览器访问 127.0.0.1:8888 可以打开边缘编排控制台  
![console](images/console.png)

#### 2.7.3 查看本地视频合流
1. 映射**video-demo**服务的80端口到主机
```
kubectl -nbaetyl-edge port-forward video-demo-xxx 38080:80
```
2. 使用 VLC 打开 http://127.0.0.1:38080/realtime-video-demo-0/0.flv  
![vlc](images/vlc.png)

### 2.8 远程调试
1. 点击【边缘节点】Tab，选择**video-demo**节点进入节点详情页，点击查看【应用部署】下的【服务状态】
2. 选择待调试服务，点击右侧[+]按钮，选择【远程调试】  
![远程调试1](images/debug1.png)
![远程调试2](images/debug2.png)