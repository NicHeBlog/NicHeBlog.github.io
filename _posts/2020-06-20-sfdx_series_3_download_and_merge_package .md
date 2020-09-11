---
title: SFDX CLI系列（三）：下载package并合并到本地开发环境
categories:
- Tech 
date: 2020-06-20
excerpt: "SFDX Command Line下载一个package并合并到本地进行二次开发"
feature_image: "https://picsum.photos/id/403/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
### 总述
* --- Step 1： 本地创建一个project<br/>
* --- Step 2： 从目标org下载一个package到本地环境<br/>
* --- Step 3： 解压package，转换数据format并合并到本地环境<br/>



### Step 1 本地创建一个project

在本地的Projects文件夹下面创建一个project，名称取为testRetrievePackage
```
cd Projects  
    //进入文件夹Projects

sfdx force:project:create -n testRetrievePackage
    //创建project
    // -n 取一个project名字
cd testRetrievePackage
    //进入项目的文件夹
```

### Step 2 从目标org下载一个package到本地环境
这里我们用 [SFDX CLI系列（二）：创建一个Unlocked Package](https://nicheblog.github.io/tech/2020/06/18/sfdx_series_2_create_an_unlocked_package/)中创建的package salesApps作为目标package<br/>
把package中的内容从云端取到本地,并存储在本地新建文件夹mdapipackage中

```
sfdx force:mdapi:retrieve -s -r ./mdapipackage -p salesApps -u MyTP -w 10
   // -s/--singlepackage 指定是singlepackage
   // -p/--packagenames 要取的package的名称
   // -u/--targetusername 目标org的alias
```

![retrieveResult](/assets/installPackage/retrieveResult.png "retrieveResult")

### Step 3 解压package，转换数据format并合并到本地环境

可以看到在项目文件夹testRetrievePackage下面生成了一个新的文件夹mdapipackage <br/>
并且在该文件夹下面生成了一个unpackaged.zip文件<br/>
手动解压该.zip文件,可以看到文件如下<br/>
package.xml中存储了该package的所有的metadata信息。

![unpackage](/assets/installPackage/unpackage.png "unpackage")

#### 插播：metadata format和source format的区别
>在salesforce的开发中，metadata format存储方式是一个整体文件，如上文中的package.xml一个文件中存储了所有的object信息以及object下的所有field，listview信息等等<br/>
>在实际开发中，如果多个developer同时在开发，就都在修改这个package.xml文件，会导致merge的问题。
>而source format存储方式就不一样，它会在多个文件中分别存放不同的信息，比如：<br/>
>object本身的信息单独存放在一个.xml文件中，每个field信息单独存放等等。这样developer之间就少了开发冲突。<br/>

插播完毕<br>


**下面开始转换下载下来的package metadata format的文件为source format**
```
sfdx force:mdapi:convert -r mdapipackage/
  // -r/--metadatapath 要转换的package源路径
```

这个是metadata format的信息,可以看到只有一个package.xml文件

![metadataformat](/assets/installPackage/metadataformat.png "metadataformat")

这个是转换之后source format的信息，可以看到拆分成了多个.xml文件

![sourceformat](/assets/installPackage/sourceformat.png "sourceformat")

此时本地存有两份package信息，一份metadata format，一份source format，<br/>
我们可以将metadata format格式的信息删除了，
```
rm -rf mdapipackage
  //删除package
```
至此，合并完毕。

**小结：**<br/> 
我们创建了一个项目project<br/>
我们从云端DevHub下载了目标package<br/>
由于解压之后，package格式是metadata format，我们转换成了source format<br/>
我们把转换后的package资源都整合到了我们的project中了，可以愉快的在本地二次开发了。<br/>
等开发完之后，我们还可以把项目打包成package供安装，怎么打包：参见[SFDX CLI系列（二）：创建一个Unlocked Package](https://nicheblog.github.io/tech/2020/06/18/sfdx_series_2_create_an_unlocked_package/)
