---
title: Salesforce Trigger
categories:
- Tech 
date: 2020-02-26
excerpt: "Apex Trigger小结"
feature_image: "https://picsum.photos/2560/600?image=872"
---
<div id="阅读时长20min" style="color:rgb(168,173,172)">阅读时长：20min</div>
---
#### 什么是Apex Trigger？

Apex Trigger在Salesforce中是一个脚本触发器，该脚本会在发生数据操作语言（DML）事件之前或之后执行。 例如往数据库中插入，更新或删除数据时被触发。和传统SQL数据库系统支持的触发器一样，Apex Trigger为管理数据记录提供触发器支持。

#### 系统中Trigger种类
系统总共有如下7种trigger：<br>
    - before insert<br>
    - before update<br>
    - before delete<br>
    - after insert<br>
    - after update<br>
    - after delete<br>
    - after undelete<br>

#### Before和After Trigger的区别

**Before Trigger**： 在数据被插入，更新或删除之前需要使用触发器执行脚本任务时，此时数据还没有被保存到数据库。

**After Trigger**：  在数据被插入，更新或删除之后，需要用到系统的信息，比如Record Id, LastModifiedDate等字段信息的时候，或者是需要联动更新其他相关记录的时候，此时数据已经被保存到数据库并生成了系统的ID信息。

#### List of Trigger Context Variables
事实上所有的Context Variables变量都保存在系统的System.Trigger类中：

**Trigger.isExecuting：** Returns true if the current context for the Apex code is a trigger, not a Visualforce page, a Web service, or an executeanonymous() API call.<br>
**Trigger.isInsert:**  Returns true if this trigger was fired due to an **insert** operation, from the Salesforce user interface, Apex, or the API.<br>
**Trigger.isUpdate:** Returns true if this trigger was fired due to an **update** operation, from the Salesforce user interface, Apex, or the API.<br>
**Trigger.isDelete：** Returns true if this trigger was fired due to a delete operation, from the Salesforce user interface, Apex, or the API.<br>
**Trigger.isBefore：** Returns true if this trigger was fired before any record was saved.<br>
**Trigger.isAfter：** Returns true if this trigger was fired after all records were saved.<br>
**Trigger.isUndelete：** Returns true if this trigger was fired after a record is recovered from the **Recycle Bin** (that is, after an undelete operation from the Salesforce user interface, Apex, or the API.)<br>
**Trigger.new：** Returns a list of the new versions of the sObject records. This sObject list is only available in insert, update, and undelete triggers, and the records can only be modified in before triggers<br>
**Trigger.newMap：** A map of IDs to the new versions of the sObject records. This map is only available in before update, after insert, after update, and after undelete triggers.<br>
**Trigger.old：** Returns a list of the old versions of the sObject records. This sObject list is only available in update and delete triggers.<br>
**Trigger.oldMap：** A map of IDs to the old versions of the sObject records. This map is only available in update and delete triggers.<br>
**Trigger.size：** The total number of records in a trigger invocation, both old and new.<br>

#### 使用时的一些考量

Before Insert Trigger
```
Trigger.new - 即将被插入数据库的记录list，[not commited yet, id = null]
Trigger.old - Null  
Trigger.newMap - Null
Trigger.oldMap - Null
```
Afeter Insert
```
Trigger.new - 新插入到数据库的记录list
Trigger.old - Null
Trigger.newMap - New Map<ID, new Record>
Trigger.oldMap - Null
```
Before Update
```
Trigger.new - 被更新后的记录list，新值
Trigger.old - 更新前的记录list，旧值
Trigger.newMap - New Map<ID, new Record>
Trigger.oldMap - Old Map<ID, old Record>
```
After Update
```
Trigger.new - 被更新后的记录list
Trigger.old - 更新前的记录list.
Trigger.newMap - New Map<ID, new Record>
Trigger.oldMap - Old Map<ID, old Record>
```
#### 什么时候使用"Before" Vs "After" Trigger

![BeforeOrAfterTrigger](/assets/SFTrigger/BeforeOrAfter.png "BeforeOrAfterTrigger")

#### 一些简单的示例：

比如有个需求是当客户的手机号码更新的时候，应该在客户的描述里自动备注上旧的手机号码+新的手机号码：
```
trigger NewVsOld on Account (before update) {
    for(integer i=0; i<trigger.new.size();i++){
        if(trigger.old[i].phone!=trigger.new[i].phone){
               trigger.new[i].description='old phone number is  ' + trigger.old[i].phone + ' and  New Phone number is ' +trigger.new[i].phone ;
   }
 }
} 
```
每当销售创建一个新的客户时，系统都会自动创建一个相关的销售机会待跟进。
```
trigger AutoOpp on Account(after insert) {
  List<Opportunity> newOpps = new List<Opportunity>();
  for (Account acc : Trigger.new) {
    Opportunity opp = new Opportunity();
    opp.Name        = acc.Name + ' Opportunity';
    opp.StageName   = 'Prospecting';
    opp.CloseDate   = Date.today() + 90;
    opp.AccountId   = acc.Id; // Use the trigger record's ID
    newOpps.add(opp);
  }
  insert newOpps;
}
```


                            