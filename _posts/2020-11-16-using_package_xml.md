---
title: 一些常用的package.xml命令 
categories:
- Tech 
date: 2020-07-20
excerpt: "常用的package.xml方便查阅"
feature_image: "https://picsum.photos/id/870/600?image=872"
---
<div id="阅读时长10min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
### 总述
使用Visual Studio Code做SF开发时候，连接了dev org之后，有时候需要使用package.xml文件进行配置类发开的as code行为。
本文总结了一些常用的命令，方便之后的查阅使用，并保持持续更新。<br/>
* --- Part 1： 一个sample package.xml长什么样子<br/>
* --- Part 2： 一些常用的命令查阅<br/>


### Part 1 一个sample package.xml长什么样子？
<hr/>
```
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>MyCustomObject__c</members>
        <name>CustomObject</name>
    </types>
    <types>
        <members>*</members>
        <name>CustomTab</name>
    </types>
    <types>
        <members>Standard</members>
        <name>Profile</name>
    </types>
    <version>47.0</version>
</Package>

```


### Part 2 一些常用的命令查阅
<hr>
##### Custom Object or Custom Setting 
```
    <types>
        <members>Custom_object__c</members>
        <members>CustomSetting__c</members>
        <name>CustomObject</name>
    </types>
```

##### Custom Field 
```
    <types>
        <members>Custom_object__c.Field1__c</members>
        <members>Custom_object__c.Field2__c</members>
        <name>CustomField</name>
    </types>
```

##### Validation Rule 
```
    <types>
        <members>Custom_object__c.validationRule01API</members>
        <members>Custom_object__c.validationRule02API</members>
        <name>ValidationRule</name>
    </types>
```

##### Custom Button 
```
    <types>
        <members>Custom_object__c.buttonAPI</members>
        <name>WebLink</name>
    </types>
```
