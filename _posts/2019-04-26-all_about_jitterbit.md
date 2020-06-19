---
title: JitterBit使用全解
categories:
- Tech 
date: 2019-04-26
excerpt: "JitterBit使用手顺"
feature_image: "https://picsum.photos/2560/600?image=872"
---
<div id="阅读时长30min" style="color:rgb(168,173,172)">阅读时长：30min</div>
---
### 什么是JitterBit?
Jitterbit 是一款开源的数据集成工具, 用户可以使用Jitterbit 来集成不同的应用, 不同的databases, 以及不同的数据来源.
> 官网地址： https://www.jitterbit.com/

### 如何使用JitterBit 同步数据
**概述**： 可以通过jitterbit工具进行数据的 query, upsert, insert, update, delete, bulk Process操作。 

**举例**： 通过jitter设置对系统中的数据做update操作： 
+ ###### a. 点击下图所示的 New Update 

![jitter01](/assets/BlogJitterbit/jitterbit01.png "pic01")

+ ###### b. 可以连接Salesforce 的测试或者正式环境
> 测试环境的 Server host: https://test.salesforce.com/ <br/>
> 正式环境的 Server host: https://login.salesforce.com/

+ ###### c.  通过点击 Test Salesforce Login 可以测试有没有成功的连接上Salesforce 

+ ###### d. 点击Next ， 选择要操作的salesforce 的对象，有三个选项列表，分别是Standard Common Objects（系统标准对象）,  All Custom Objects（自定义对象）, All Objects（所有的对象）

![jitter02](/assets/BlogJitterbit/jitterbit02.png "pic02")

+ ###### e. 点击Next，选择数据源，支持csv文件的导入以及各种数据库的连接。 
    输入数据库的地址，表名称以及账户
    点击Test Connection可以测试数据库连接成功与否

![jitter03](/assets/BlogJitterbit/jitterbit03.png "pic03")

+ ###### f. 点击Next，选择匹配关系。
可以进行拖拽匹配，或者点击左侧的数据库的字段名，然后双击右侧的API名称即可完成单个字段的匹配

![jitter04](/assets/BlogJitterbit/jitterbit04.png "pic04")

+ ###### g. 可以设定任务定时执行。  
     - g-1. 可以设置每日，每周，每月执行，
     - g-2. 对于每日执行的操作，可以设置每日执行的频率。
     - g-3. 可以设定执行的有效周期


![jitter05](/assets/BlogJitterbit/jitterbit05.png "pic05")

### 如何管理JitterBit下的Operations
+ ###### 部署云端
点击右上角的上传按钮，可以部署你配置的操作到JitterBit 的云端，这样即便自己的电脑关机状态，也能保证实时的数据同步

![jitter06](/assets/BlogJitterbit/jitterbit06.png "pic06")
                                      
+ ###### 云端下载配置         
点击左上角的Actions→Restore Project from Cloud 
可以获取你已经部署到云端的所有配置（注意：此时会覆盖你本地的配置）

![jitter07](/assets/BlogJitterbit/jitterbit07.png "pic07")

### 关于JitterBit的云端管理平台
双击下图中标红的 Cloud Console 可以进入JitterBit 的云端管理平台

![jitter08](/assets/BlogJitterbit/jitterbit08.png "pic08")

下图便是JitterBit 的云端平台，Doshboard 页面可以看到连接的数量以及正在工作的配置信息

![jitter09](/assets/BlogJitterbit/jitterbit09.png "pic09")

                            