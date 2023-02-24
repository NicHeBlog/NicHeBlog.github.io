---
title: SFDX CLI系列（二）：创建一个Unlocked Package
categories:
- Tech 
date: 2020-06-18
excerpt: "SFDX Command Line创建一个包含Permission set的Package"
feature_image: "https://picsum.photos/id/403/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
### 总述
* --- Step 1： 为了模拟真实的开发场景，我们从GitHub上下载一个project到本地环境<br/>
* --- Step 2： 我们在本地做些修改之后，部署project到一个临时的scratch org中，并在scratch org中创建一个permission set集<br/>
* --- Step 3： 把在scratch org中做的permission set同步到本地环境<br/>
* --- Step 4： 在DevHub中创建一个package，把本地的project打包成一个版本，部署安装到DevHub中<br/>
* --- Step 5： 安装package

### Step 1 安装project到本地
VS Code中新建一个Terminal, 

![terminal](/assets/unlockedPackage/newterminal.png "terminal")

克隆GitHub上的项目源码到本地环境,并进入project目录
```
git clone https://github.com/developerforce/PermSetUnlockedPackage.git

cd PermSetUnlockedPackage
```

### Step 2 创建一个临时的scratch org，并配置permission set
新建一个Scratch org ，输出结果如下：
```
sfdx force:org:create -f config/project-scratch-def.json -s
```
![createScratchOrg](/assets/unlockedPackage/createScratchOrg.png "createScratchOrg")


把本地的metadata推到scratch org中，可以看到一些object，listView和customtab等信息被同步到云端org
```
sfdx force:source:push
```
![scratchPush](/assets/unlockedPackage/scratchPush.jpeg "scratchPush")

打开scratch org，并在org中配置一下permission sets
```
sfdx force:org:open
```

>在此省略在scratch org中配置permission set的过程。。。


### Step 3 把scratch org中的permission set同步到本地

然后把云端scratch org中配置好的permission set下载到本地环境中
```
sfdx force:source:pull
```

![permissionSuccess](/assets/unlockedPackage/permissionSuccess.jpeg "permissionSuccess")

此时这个scratch org的使命就算完成了（临时的组织，负责线上配置，配置完把配置同步到本地后，使命就结束了），可以选择直接删除这个scratch org
```
sfdx force:org:delete -u aliasTargetOrg
``` 

### Step 4 创建package
在我们的DevHub下创建一个package

```
sfdx force:package:create --name salesApps --description "My Package" --packagetype Unlocked --path force-app --nonamespace --targetdevhubusername DevHub

备注： 
     --name/-n package的名称为salesApps
     --description/-d package的描述
     --packagetype/-t 种类有两种Unlocked和Managed
     --path/-r  package存放路径
     --targetdevhubusername/-v DevHub的alias
```

打开本地文件中的sfdx-project.json文件，可以看到，此时文件中多了package的相关信息，如packageAliases,versionName,versionNumber等

![package1](/assets/unlockedPackage/package1.jpeg "package1")

尝试修改versionName和versionNumber参数如下： <br/>

```
"versionName": "Version 1.0",
"versionNumber": "1.0.0.NEXT"
```
这样未来package的版本信息初版就是salesApps@Version1.0.0，<br/>
未来更新的版本为salesApps@Version1.0.0-1，salesApps@Version1.0.0-2等等<br/>

创建一个package的Version： 
```
sfdx force:package:version:create -p salesApps -d force-app -k test1234 --wait 10 -v DevHub

备注： 
     -p package的alias
     -d package的路径
     -k 安装package时候需要提供的安装key
     -v DevHub的alias

```
此时会看到sfdx-project.json文件中会多出"salesApps@1.0.0-1": "04t2x0000048jCYAAY"信息

![package2](/assets/unlockedPackage/package2.png "package2")

Promote一下package的version

>这里要补充的是，package是有状态的，package被初始创建的时候状态都是beta,
>beta状态的package是不可以直接安装到production环境的，为了保证要安装的package是production ready的,
>我们需要promote the package version to released.

```
sfdx force:package:version:promote -p salesApps@1.0.0-1 -v DevHub

备注： 
     -p 要promote的package版本
     -v DevHub的alias
```

### Step 5 安装package
至此，package已经打包完成了，你可以随时把这个package安装到任何环境了。
我们尝试安装package到DevHub中，
```
sfdx force:package:install --wait 10 --publishwait 10 --package salesApps@1.0.0-1 -k test1234 -r -u DevHub

备注： 
     --package package的alias
     -k 提供一下之前设置的安装key
     -u 提供一下要安装的org的alias
```

Voila！安装成功。

![package3](/assets/unlockedPackage/package3.png "package3")


**如果未来想更新已经安装的package的信息，可以使用如下命令进行**

```
sfdx force:package:update -p 0Ho2x000000XZQCCA4 -v MyDevHub -d "new description for package" -n "new package name"
sfdx force:package:update -p “package alias” -v MyDevHub -d "new description for package" -n "new package name"

备注： 
     -p/--package package的id或者alias,可以利用sfdx force:package:list -v MyDevHub查出(显示MyDevHub下的所有已安装的package信息)，
     -v/--targetdevhubusername package所在的DevHub org的alias, 
     -d/--description 新的description  
     -n/--name 新的name for package
```



