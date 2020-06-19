---
title: SFDX CLI系列（一）：模拟一个完整的项目开发
categories:
- Tech 
date: 2020-06-14
excerpt: "SFDX Command Line模拟开发场景"
feature_image: "https://picsum.photos/id/418/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
### 总述
* --- Part 1： 创建项目，创建开发scratch org，创建开发模拟数据等<br/>
* --- Part 2： 本地创建apex，event，components等<br/>
* --- Part 3： 同步开发内容到测试org并创建permission set等<br/>


### Part 1 创建项目，创建开发scratch org，创建开发模拟数据等
创建项目
```
sfdx force:project:create -n geolocation  
    //创建一个名称为geolocation的project
```
创建一个scratch org
```
sfdx force:org:create -s -f config/project-scratch-def.json -a GeoAppScratch
    //根据配置文件project-scratch-def.json创建一个scratch org 
    // -s 设置默认org
    // -a 设置scratch org alias
```
在新建的scratch org中创建两条记录
```
sfdx force:data:record:create -s Account -v "Name='Marriott Marquis' BillingStreet='780 Mission St' BillingCity='San Francisco' BillingState='CA' BillingPostalCode='94103' Phone='(415) 896-1600' Website='www.marriott.com'"
   // -s 指定sObject
   // -v 指定field以及value

sfdx force:data:record:create -s Account -v "Name='Hilton Union Square' BillingStreet='333 O Farrell St' BillingCity='San Francisco' BillingState='CA' BillingPostalCode='94102' Phone='(415) 771-1400' Website='www.hilton.com'"
   //再创建一条记录到scratch org
```

本地环境创建一个文件夹用于存放数据，以方便之后把数据导入其他org中
```
mkdir data
```
把scratch org中的数据导出来到新建的data文件夹下面<br/>
导出之后，会在data文件夹下面生成一个Account.json文件
```
sfdx force:data:tree:export -q "SELECT Name, BillingStreet, BillingCity, BillingState, BillingPostalCode, Phone, Website FROM Account WHERE BillingStreet != NULL AND BillingCity != NULL and BillingState != NULL" -d ./data
   // -q SOQL语句
   // -d/--outputdir 指定数据导出后存储的文件夹

//如果未来测试中，需要导入数据到其他的org，执行下面的语句
sfdx force:data:tree:import --sobjecttreefiles data/Account.json -u targetOrg
   //--sobjecttreefiles/-f 指定取数据的路径
   // -u 指定要导入到的org名称
```

### Part 2 本地创建apex，event，components等

>具体的代码部分在此会省略

创建一个apex class到scratch org中，并存在指定的文件夹下面
```
sfdx force:apex:class:create -n AccountSearchController -d force-app/main/default/classes
   // -n/--classname 取一个apex class的名称
   // -d/--outputdir 指定class的存储路径 
```
>...省略AccountSearchController代码

创建一个event到scratch org中，并存在指定的文件夹下面
```
sfdx force:lightning:event:create -n AccountsLoaded -d force-app/main/default/aura
   // -n/--eventname 取一个event的名称
   // -d/--outputdir 指定event的存储路径
```
>...省略AccountsLoaded代码

创建一个component到scratch org中，并存在指定的文件夹下面
```
sfdx force:lightning:component:create -n AccountSearch -d force-app/main/default/aura
```
>...省略AccountSearch代码

把本地的代码同步到云端的scratch org中
```
sfdx force:source:push
```

在云端scratch org中创建permission set，并把创建的结果拉取到本地
```
sfdx force:source:pull
```

### Part 3 同步开发内容到测试org并创建permission set
至此，开发阶段已经完成，当然是可以在开发环境的scratch org进行测试的，<br/>
不过按照最佳实践，为了测试所有的内容都是外部化的，<br/>
最好需要把本地的开发内容全部部署到一个新的scratch org中进行测试

新建一个scratch org用于测试；
```
sfdx force:org:create -f config/project-scratch-def.json -a GeoTestOrg
  // -a scratch org的alias
```

把本地的所有资源同步到用于测试的scratch org中<br/>
会同步所有的代码aura component,apex class,tabs,permission set等
```
sfdx force:source:push -u GeoTestOrg
  // -u 目标org
```

向测试环境导入之前的测试数据信息，当然也可以手动创建更多的数据到.json文件中
```
sfdx force:data:tree:import -f data/Account.json -u GeoTestOrg
```
分配permission set
```
sfdx force:user:permset:assign -n Geolocation -u GeoTestOrg
   // -n 当时手动在scratch org中创建的permission set的名称
   // -u 目标org
```

打开测试环境,开始测试吧<br/>
测试如果没有问题，可以遵循同样的逻辑导入开发数据到Pre-Production,Production环境等。
```
sfdx force:org:open -u GeoTestOrg
```




