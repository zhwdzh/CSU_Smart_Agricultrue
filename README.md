# CSU_Smart_Agricultrue
中南大学物联网生产实习的项目，题目是智慧农业大棚
```
这里我负责的是后端部分，只上传后端部分的代码哦
```

后端设计结构图：<br />
![image](https://user-images.githubusercontent.com/50700026/126894322-7c7949b2-9568-4156-8a14-9c8060417b67.png)

后端详细设计：
本项目的后端采用Django框架，主要完成接口的编写以及硬件设备自动控制的功能，在该框架下编写了以下几个应用：

	users用户模块
	environment_parameters环境参数模块
	controlled_devices 可控设备模块
	alarmsystem报警系统模块
	automode自动控制模块
	logs日志模块


1.	environment_parameters环境参数模块
本模块的作用是为前端提供查看各个环境参数历史数据的接口，主要有五个环境参数：

		temperature温度
		humidity湿度
		light_intensity光照强度
		carbon_dioxide二氧化碳
		ultraviolet紫外线强度
五个环境参数的模型基本相同，有id，数值，时间，设备号四个字段，其中温度temperature温度和humidity湿度在一个传感器上，中间层也是一起插入数据库，所以把这两个参数安排在一个模型中。这五个环境参数数据都只需要调用，而不需要增删改，所以只用编写查数据的接口。接口继承自类ListAPIView，用其自带的get方法。
		
2.	controlled_devices 可控设备模块
本模块的作用是为前端提供查看各个可控设备历史状态数据的接口，以及改变可控设备状态的接口。主要有3个可控设备：

		light高亮LED灯
		fan风机
		engine洒水喷头
		
三个可控设备的模型基本相同，有id，状态值，时间，设备号四个字段。
这三个可控设备除了需要提供查历史状态数据的接口外，还要提供修改可控设备状态的接口。查找可控设备历史状态数据的接口继承自ListAPIView，用其自带的get方法。修改可控设备历史状态数据的接口继承自CreateAPIView，重写其自带的post方法，使得该接口可以在数据库中插入一条心得状态值数据，并加入向中间层发送数据的代码，采用socket方式发送
3.	alarmsystem报警系统模块
本模块的作用是为前端提供查看各个传感器及报警装置的历史数据的接口，以及改变报警灯和报警器的接口。主要有3个传感器及报警灯和报警器：

		fumes烟雾传感器
		methane甲烷传感器
		firelight火光传感器
		alarmlight报警灯
		alerter报警器
三个传感器设备及报警灯和报警器的模型基本相同，有id，状态值，时间，设备号四个字段。
这三个传感器设备及报警灯和报警器除了需要提供查历史状态数据的接口外，还要提供修改报警灯和报警器状态的接口。查找设备历史状态数据的接口继承自ListAPIView，用其自带的get方法。修改报警灯和报警器状态的接口继承自CreateAPIView，重写其自带的post方法，使得该接口可以在数据库中插入一条心得状态值数据，并加入向中间层发送数据的代码，采用socket方式发送

4.	automode自动控制模块
该模块和其他四个模块不同，该模块调用apscheduler库中的函数，实时对数据库中的各个设备的最新数据进行查看，并判断其是否处于适宜的参数范围，判断是否需要修改可控设备状态以调控环境参数，使其恢复正常，并监控烟雾，甲烷及火光，检测到有其中一个以上时，打开洒水喷头并报警。调控内容主要如下：

		温度过高时开启风机
		温度恢复正常时关闭风机
		湿度过低时打开洒水喷头
		湿度恢复正常时关闭洒水喷头
		光照强度过低时打开高亮LED灯
		光照强度恢复正常时关闭高亮LED灯
		检测到烟雾，甲烷，火光时打开洒水喷头和报警灯，报警器
		
5.	logs日志模块
本模块的作用是为前端提供查看用户操作日志及自动控制系统自动控制设备的日志的接口。

	建立日志模型类Logs
	add_time	日志添加时间
	log_type	所属模块、操作类型
	log_content	日志内容
	log_ip	访问IP
	user	用户ID，外键关联到用户表

	添加内置函数add_logs(LogType, LogContent, LogIp, UserId)，利用原生sql语句和游标，执行insert语句，将传过来的参数插入到日志表中。
	接口设计：POST /logs/logs    GET /logs/logs

	自动控制系统自动控制设备的日志模型主要由id，时间，设备号，操作内容四个字段构成，提供查看日志历史数据的接口。该接口继承自ListAPIView，采用其中的get方法。

6.	users用户模块<br />
	（1.登录、注册、查看、修改<br />
	```登陆时，根据token获取用户信息，根据HTTP_AUTHORIZATION拿到token，然后利用queryset.filter 过滤器拿到相应id的用户信息。```<br />
	（2.用户的详情、编辑、删除<br />
	```利用UserRetrieveUpdateDeleteView(RetrieveAPIView, DestroyAPIView, UpdateAPIView):，重写相关的POST、GET方法。```<br />
	（3.用户注册<br />
	```RegisterView(mixins.CreateModelMixin, GenericAPIView)类新增用户```<br />
