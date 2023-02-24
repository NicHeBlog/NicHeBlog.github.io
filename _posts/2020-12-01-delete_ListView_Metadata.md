---
title: Delete ListView Metadata
categories:
- Tech 
date: 2020-12-01
excerpt: "by using destructiveChanges.xml"
feature_image: "https://picsum.photos/id/870/600?image=872"
---
<div id="阅读时长10min" style="color:rgb(168,173,172)">阅读时长：10min</div>
---
### 背景简介
在Salesforce系统中，List View属于系统的metadata数据。<br/>
由于业务上踩了一点关于List View的坑，之前并没有在所有的Profile中把Manage Public List Views这个属性勾掉，导致所有的User都有权限创建给所有人可见的ListView，<br/>
现在公司进行业务梳理，需要删除这些List View。
本文总结了如何管理以及删除List View<br/>
* --- Part 1： List View的管理<br/>
* --- Part 2： List View的删除<br/>
* --- Part 3： 小结<br/>


### Part 1 List View的管理
<hr/>
首先List View是用来方便的用户创建方便查看的视图，可以展示，过滤以及分组数据。List View也可以在创建之后方便的share给对应的Role或者是一些自建的Group。<br/>
但是作为管理员应该尽早和业务部门澄清以及取消Manage Public List Views这个权限。不然会造成List混乱的情况发生。后期处理起来比较棘手，下面会说明一下我是如何批量处理系统中上千条List View数据的。


### Part 2 List View的删除
<hr/>
由于List View在系统中属于Metadata，并不能够使用Salesforce提供的一些批量工具进行处理。所以需要用destructiveChanges.xml配合Workbench来进行处理。<br/>
分三个步骤进行处理： 
1. 从系统中Retrieve需要处理的文件
2. 准备destructiveChanges文件
3. 部署destructiveChanges文件
4. 一些注意事项
<br/>

**1. 从系统中Retrieve需要处理的文件** <br/>
  可以使用[上篇文章](https://nicheblog.github.io/tech/2020/11/16/using_package_xml/)中介绍的方式，准备一下package.xml文件。格式如下：
  ```
    <types>
        <members>Custom_object__c.ListViewAPI</members>
        <name>ListView</name>
    </types>
  ```
  但是这种直接获取List View API比较适用于少量的处理。<br/>
  要处理的文件数量比较多的情况，则最好需要先Retrieve整个Object的数据，这样命令行便可以一次性获取整个Object下的所有List View API
  ```
    <types>
        <members>Custom_object__c</members>
        <name>CustomObject</name>
    </types>
  ```
  另外一点小技巧是在获取到List View API之后，可以方便的使用Sublime的批量编辑功能，把数据处理成需要的格式。
  >ctrl+A全选，ctrl+shift+L进入待操作的状态，通过←和→可以控制光标的位置进行批量编辑。


  **2. 准备destructiveChanges文件**<br/>
  需要准备两个文件,一个文件名称为package.xml,内容如下,不同于一般的package.xml文件，这个文件中什么内容也不用写，只需要指明version版本就可以 <br/>

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <Package xmlns="http://soap.sforce.com/2006/04/metadata">
      <version>47.0</version>
  </Package>
  ```

  另一个文件名称为destructiveChanges.xml，内容如下：
  ```
    <?xml version="1.0" encoding="UTF-8"?>
    <Package xmlns="http://soap.sforce.com/2006/04/metadata">
        <types>
            <members>Account.AllAccounts</members>
            <members>Account.NewThisWeek</members>
            <members>Account.KeyAccounts</members>
            <members>Account.Test_Accounts</members>
            <members>Account.Target_Accounts_by_BDM</members>
            <members>Account.Northeast_Targets</members>
            <members>Account.Northeast_Targets1</members>
            <members>Account.Northeast_Targets2</members>
            <members>Account.Northeast_Targets3</members>
            <members>Account.Northeast_Targets4</members>
            <members>Account.Northeast_Targets5</members>
            <name>ListView</name>
        </types>
    </Package>
  ```
  只需要替换<members>xxx.xxx</members>就可以。
  ![files](/assets/destructiveChanges/files.png "files")

 
 **3. 部署destructiveChanges文件** <br/>
  把准备好的两个文件放在一个文件夹下，随便命名文件夹，然后压缩文件夹为.zip格式。<br/>
  **此处踩了个坑**，使用Mac系统进行文件压缩的时候，发现明明是压缩成为了.zip格式，每次部署都会遇到'No Package.xml Found'错误。<br/>
  后来发现这个是由于使用Mac电脑进行文件压缩的的时候，系统会创建一些隐藏的文件，也被压缩进了文件夹，部署的时候，Salesforce并不认这些文件，所以报错。<br/>

  **解决方案**： <br/>
  使用命令行进行压缩。先cd命令进入将要压缩的文件夹下，执行命令。
  ```
    zip -r -X destructiveChanges.zip *   
      //表示讲当前目前目录下的所有文件和文件夹打包为当前目录下的destructiveChanges.zip
        //-r   递归处理，将指定目录下的所有文件和子目录一并处理。
        //-X   不保存额外的文件属性
        //*    通配，会把当前目录下的所有文件和文件夹都打包到destructiveChanges.zip中
        //也可以这么使用 *.png就只会打包当前目录下的png文件格式。
        //示例：zip -r mcw.zip /root/mcw_test 表示将/root/mcw_test/这个目录下所有文件和文件夹打包为当前目录下的 mcw.zip
        //扩展 -> 使用unzip命令:
        //unzip archive.zip 把archive.zip文件进行解压缩
        //unzip archive.zip -d target-folder 把archive.zip文件进行解压缩到已存在的目标文件夹
  ```
  
  打开Workbench页面，登录后选择**migration > Deploy** <br/>
  选择刚刚才压缩好的.zip文件，一定要记得**勾选Single Package**，<br/>
  可以选择勾选**Rollback On Error**，有错误的话会回滚版本，<br/>
  **Test Level**选择RunLocalTests,跑一下系统中除了安装的package包的所有测试 <br/>
  然后选择Next。

  ![deploy](/assets/destructiveChanges/deploy.png "deploy")
  

  ### Part 4 总结一下需要注意的事项
  <hr/>
     - 系统中对于List View的管理，无特殊情况，不要给Profile勾选Manage Public List Views<br/>
     - 创建package.xml文件以及destructiveChanges.xml文件<br/>
     - 记得用命令行进行压缩操作<br/>
     - 别忘记deploy时候勾选Rollback On Error,Single Package <br/>
