---
title: SFDX刷Sandbox Metadata
categories:
- Tech 
date: 2020-08-22
excerpt: "使用SFDX Command Line"
feature_image: "https://picsum.photos/id/403/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
#### 目标
<hr>
把本地的sfdx project repo deploy到一个sandbox中

#### 步骤
<hr>
Step1: <br/>
首先转换本地source的格式，从Source Format改为Metadata Format，使用以下命令:<br/>

```
sfdx force:source:convert -r force-app/main/default -d mdapipackage/
```
-r 是指目标目录<br/>
-d 是转换后导出的目录，如果系统中不存在这个目录，则会新建一个文件夹叫做mdapipackage，并且会把转换后的Metadata文件放在该目录下面<br/>

Step2: <br/>
把mdapipackage文件夹中的文件部署到目标sandbox中

```
sfdx force:mdapi:deploy -d mdapipackage/  -u nicdevsdb
```
-d 指定Metadata Format的文件的目录<br/>  
-u 指定部署的org alias名称(如果连接了目标org,可以不用指定)<br/>

Step3: <br/>
查看当前部署的状态
方式一：
sfdx force:mdapi:deploy:report
terminal中会返回部署的状态

方式二： 
打开你的sandbox，setup中选择Deploy Status，可以查看部署状态
