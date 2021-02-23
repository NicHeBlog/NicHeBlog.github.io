---
title: Lead Convert Summary
categories:
- Tech 
date: 2021-02-23
excerpt: "Lead Convert From A to Z"
feature_image: "https://picsum.photos/id/870/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
### 简介
本文会介绍Lead Convert的使用以及<br/>
转换过程需要注意（踩过）的地方（坑）<br/>
* --- Part 1： Lead Convert 功能介绍<br/>
* --- Part 2： Lead Convert 需要注意的地方<br/>
* --- Part 3： 小结<br/>


### Part 1: Lead Convert 功能介绍
<hr/>
在客户关系管理中，当对一个Lead的跟进到达一定程度之后(比如，lead对于公司售卖的产品有明显的兴趣和意愿)，这时候就要考虑把lead转换成为系统中Contact,Account或者Opportunity了，以便更好的在公司内部由不同的团队进行跟进流转(不同公司可能是有不同的销售策略)。<br/>

在Salesforce中转换Lead，其实就是把Lead中的信息流转到不同的Object中进行存储。<br/>

Lead可以被同时转换成为全新的Contact,Account和Opportunity，<br/>
Lead也可以被转换成已经存在的系统中Contact,Account和Opportunity<br/>
比如:两个Lead同属于一家公司(Account)；两个来自不同渠道的Lead其实都是对应的同一个人(Contact)<br/>

![leadConvert](/assets/leadConvertSummary/leadConvert.png "leadConvert")

### Part 2: Lead Convert 需要注意的地方
<hr/>
Lead Convert的过程是Salesforce Sales Cloud产品提供的一个标准功能，看似简单，其实里面暗藏着不少的坑，下面本文会一一道来，希望会对看到本文的人有所帮助：<br/>

1. Lead Convert之后Lead记录并不会被删除
2. LeadStatus字段（API：Status）和Lead Path 
3. Lead Field Mapping
4. 转换到Con/Acc/Opp的值覆盖问题
5. **Lead Convert执行顺序问题**
6. Default Value的问题
<br/>

**1. Lead Convert之后Lead记录并不会被删除** <br/>
  在Lead Convert之后，从效果和页面上来看，Lead记录确实是找不到了。但是该条记录其实并没有被系统删除，还会继续存留在数据库中，只是Lead页面中找不到了而已。<br/>

  isConverted, ConvertedContactId .....<br/>
  #TODO

**2. LeadStatus字段（API：Status）和Lead Path** <br/>
  LeadStatus是一个系统的标准Picklist字段，它里面存储了Lead的所有可能状态，不同公司可以根据自己独特的业务场景进行定制，可以设定多个Record Type来使用不同的状态值。<br/>

  根据不同的Record Type可以创建不同的 SetUp > Path Setting（在Salesforce中很多Object都可以创建一个可视化的Path来管理流程流转的过程）。<br/>
  
  Path Setting中可以自定义section的显示以及不同的‘成功指南’和‘帮助文本’等，是Salesforce提供的一个很好用的开箱即用的工具包。<br/>
  具体Path长什么样,见下图：<br/>

  ![leadPath](/assets/leadConvertSummary/leadPath.jpeg "leadPath")

  Path Setting里面可以指定Object的Record Type，比如Lead Path指定了某个特定的Record Type之后，对应section显示的值就可以通过Setup > Lead Processes来进行设定（Lead Processes也绑定了对于的Record Type）<br/>
  
  LeadStatus字段的API是Status，他的Label也可以更改，但是不同于自定义字段，这些系统的字段变更Label的方式是，<br/>
  Setup > rename Tabs and Labels 选定具体的Object然后进行变更。<br/>

  当然LeadStatus的Retrieve也比较特殊，具体可以参见[之前文章](https://nicheblog.github.io/tech/2020/11/16/using_package_xml/)中的方式。<br/>

**3. Lead Field Mapping** <br/>
  前面说到Lead转换成Con/Acc/Opp的时候，可以选定一些带过去的字段值，这就需要一个字段值之间的Mapping关系。<br/>Salesforce也提供了一个页面方便进行字段的匹配 Object > Fields & Relationships > Map Lead Fields <br/>

  **此处有个需要注意的地方**： <br/>
  LeadStatus这个字段并不支持被直接Map到Con/Acc/Opp中的字段。<br/>
  当然为了解决这个问题，也有Workround，要想Map这个字段，需要在lead下创建一个返回值为text的formula字段（TEXT（Status））来实时获取字段值，然后把这个自定义字段再Map到相应的Con/Acc/Opp中的字段。<br/>
  
**4. 转换到Con/Acc/Opp的值覆盖问题** <br/>
  Field Map过程其实有一个规则是如果Target Object中的字段已经有值存在了，从Lead中map过去的字段值并不会覆盖已有的值。<br/>
  只有Target Object中的字段没有值的时候，才会把lead中对应字段的值带过去。<br/>


**5. Lead Convert执行顺序问题** <br/>
  这个问题可能翻阅了很多资料没能找到准确的解释，下面是自己试探摸索出来的执行顺序供参考： <br/>
  - （如果有像第3部分中说到的创建了一个自定义的字段用来map的话），
  formula获取LeadStatus的值最先执行；<br/>
  - 然后是：按照Lead对象下设定的Map Lead Fields进行字段的匹配（有值不覆盖）；<br/>
  - 然后是：Lead自己把Convert页面中选定的‘Converted Status’ 值再赋值给Lead自己本身，（就是说转换之后的Lead其LeadStatus的值）；<br/>
  - 然后是： 执行相应的 Lead before trigger；<br/>
  - 然后是：validation rule等；<br/>
  - 最后是：after trigger；<br/>
  当然在Lead Setting中还可以进行设定，来指定是不是需要执行Con/Acc/Opp这三个Objects对应的before trigger和after trigger。如果执行的话，顺序会如下: <br/>

  ```
   Account Custom Field Mappings
   Account Before Trigger          (如果lead settings指定执行便执行，不指定执行，会直接插入Account记录)
   Account After  Trigger
   Contact Custom Field Mappings
   Contact Before Trigger          (如果lead settings指定执行便执行，不指定执行，会直接插入Contact记录)
   Contact After  Trigger
   Opportunity Field Mappings
   Opportunity Before Trigger      (如果lead settings指定执行便执行，不指定执行，会直接插入Opportunity记录)
   Opportunity After  Trigger     
   Lead中把LeadStatus设定为指定好的值（LeadStatus中勾选了Converted的picklist value）    
   Lead Before Trigger
   Lead After  Trigger             
  ```
   
   #TODO 如果有lead convert相关trigger，测试写法：

**6. Default Value的问题** <br/>
  发现有时候Picklist字段中明明在某个值后面勾选了default value但是实际却不生效。<br/>
  原因是这个object存在多个Record Type，需要到对应的Record Type中（Object > Record Types > 对应Picklist字段）设定字段的default value才行。<br/>
  因为这个设定会overwrite掉直接在字段上设定的default value。


### Part 3: 小结
  <hr/>
  小结几个重点吧，后续有时间会画一个顺序图出来更直观的知晓执行顺序，<br/>
     - Lead Field Mapping有值不覆盖<br/>
     - Convert过程顺序很tricky<br/>
     - Picklist Field Default Value有优先<br/>