---
title: FLS & Record Sharing
categories:
- Tech 
date: 2021-09-20
excerpt: "FLS and records sharing"
feature_image: "https://picsum.photos/id/870/600?image=872"
---
<div id="阅读时长25min" style="color:rgb(168,173,172)">阅读时长：25min</div>
---
### 简介
本文会通过一个具体的例子来切身感受一下字段级别和记录级别的安全控制。<br/>

* --- Part 1： Object Level and Field Level Security<br/>
* --- Part 2： 案例背景介绍<br/>
* --- Part 3： 针对字段以及针对记录不同的处理方法<br/>
* --- Part 4： 其他以及小结<br/>


### Part 1: Object Level, Field Level, Record Level的管控
<hr/>
相信大家对于Salesforce中的对象(Object)，字段(Field)，记录(Record)已经有充分了解，多啰嗦几句权限控制，算个小结；<br/>
系统中对于Object和Field的管控是Metadata层面的，<br/>
而对于Record级别的管控是Data层面的；<br/>

* 系统中涉及Object Level访问的管控有Profile，Permission Set，Permission Set Group可以进行管控；<br/>
* 系统中涉及Field Level的访问管控也是会有Profile，Permission Set，Permission Set Group来控制，当然除此之外还能通过Page Layout, Record Type来从UI层面进行权限控制。<br/>
总体来说系统是遵循一个最小权限原则进行设计，三者配合使用来更好更易于维护和扩展的管理系统中表和字段级别的权限，更详细的我会在这篇文章（TODO）中做详细说明。<br/>

* 系统中涉及Record Level的访问管控稍微复杂一点，下面这张图可以很好的进行解释： <br/><br/>
 ![RecordLevelAccess](/assets/access_visibility/record_level_access.png "RecordLevelAccess")

<br/>
分别进行一下简单的解释：<br/>
如果一个Object的OWD设置为Public(read/write,read only)，那就直接出门右转跳转到此文结束就好了，所有人都能看到了，没有秘密可言。<br/>

如果OWD设置为Private,那么只有两种情况，<br/>
第一种是只有自己才能看到自己创建的record;<br/>
第二种是只有role hirarchy在自己之上的人才能看到自己创建的record。<br/>

那么如果其他人也想看到这些记录怎么办呢？铺垫完成，答案就是sharing rules, manual sharing以及apex sharing。
就如这张图一样，对于record级别的权限，是这么一级一级打开的。

如果我们把role hirarchy理解为**纵向**的权限打开，因为它是上下级之间的权限开放，
那么sharing rules就是**横向**的权限打开，因为它可以根据某些特定的字段规则进行同事间，群组间的分享；
manual sharing更加自由，可以随意指定个人进行记录分享；
apex sharing顾名思义是通过Apex代码后端进行分享。

当然这里面还有一些隐式分享(implicit sharing)，此文不做细致讨论，可以参见这两篇文章，<br/>
[Parent Implicit Sharing](https://www.simplysfdc.com/2016/11/salesforce-parent-implicit-sharing.html) <br/>
[Child Implicit Sharing](https://www.simplysfdc.com/2016/11/salesforce-child-implicit-sharing.html) <br/>
简而言之：隐式分享是系统原生自带的，你不可以把它关掉也不可以刻意打开这个功能。<br/>
**父级隐式分享**是指，当一个用户对于account的子对象opportunities, cases 或者contacts有访问权限，那么他就会对这条子记录关联的account有访问权限。（其实也只有对于account有这个功能，自定义的对象统统没有隐式分享这一说）；<br/>
**子级隐式分享**是指，当编辑用户或者记录的Owner的Role的时候，系统会出现几个对于Account的子记录（contact,opportunity,case）附加访问权限的选项。（View/ Edit/ No Access）
1. 一个用户是某条Account的Owner,这个Account关联的Opp, contact, case等统统都可以直接访问（**View**），即便这些Opp, contact, case的Owner并不是这个用户；
2. 一个用户是某条Account的Owner,这个Account关联的Opp, contact, case等统统都可以直接编辑（**Edit**），即便这些Opp, contact, case的Owner并不是这个用户；
3. 也可选择设定，不让这个用户看到相关的这些Opp, contact, case子记录。除非它本身就是这些子记录的Owner；



 
> 这里留下两个小问题，可以自己亲自实验一下：<br/>
> Question 1:  <br/>
>             有销售A和销售B， 销售B的manager是销售经理C，但是C不是A的上级经理
>             A把一条记录share给了B， 问C能看到这条记录吗？  <br/> 
>Answer: <br/>
>             YES，C可以看到的!! <br/>
>             所以分享记录的时候要注意，你不仅分享给了张三，还分享给了张三的上级们；张三能看到的，张三的上级也都能看到。<br/>

>Question2 ： <br/>
>            现在我们知道，C能看到记录了，那请问这时候C可以再把这条记录分享给销售D吗？<br/>
>Answer： <br/>
>            NO!! <br/>
>            因为只有记录的所有者以及在所有者的上级才可以再次进行分享,当然admin是任何时候都有权限进行分享的<br/>
        

关于记录级别的访问，在此附上另一张我觉得挺好的图，一级一级的排查权限设定。

 ![ViewRecord](/assets/access_visibility/view_record.png "ViewRecord")

其中需要解释的一点：<br/>
‘View All Data’权限，像一个通天密钥，一旦有这个权限，就可以看到该对象下的所有的记录。


### Part 2: 案例介绍
<hr/>
假设有HealthInfo这么一个对象，它有以下6个字段：

![fields](/assets/access_visibility/fields.png "fields")

系统中有3个不同的Profile，不同的Profile对于字段的访问权限如下图：
![access](/assets/access_visibility/access.png "access")

我们知道BMI是身体质量指数，计算公式是：<br/>
BMI=体重÷身高² <br/>

需要注意的是Nurse这个profile对于体重和身高都是有访问权限的，但是她对于BMI这个字段是没有访问权限的。<br/>
系统后台会有Trigger根据病人的身高和体重，自动计算更新病人的BMI值。
那么现在的问题就是如果护士通过页面UI对身高或者体重进行了更新，那么这个时候，病人的BMI值会不会被更新呢？
答案当然是会被更新。
可能你就疑惑了，明明护士对于这个字段并没有访问权限（CRUD权限都没有），为啥他能引起BMI字段的更新。

这就引出了下面要讲的内容： **System Mode**以及**User Mode**。

![mode](/assets/access_visibility/system_mode.png "mode")

**System Mode:**
1. 登陆用户对于object没有访问权限，也可以access to这个object
2. 登陆用户对于field没有访问权限，也可以access to这个field
3. 系统代码此时对所有的object& fields 都有访问权限，不考虑对于字段级别做的权限设定以及sharing rule的设定

为什么系统要这么设定呢？ 其实应该是为了确保代码不会因为用户对某些字段/对象没权限 或者UI页面上的权限改动而导致程序跑崩。瞎猜的。

**User Mode：** 
1. 遵循执行者的权限设定以及sharing设定。profile level， field-security level, sharing rule level
2. 只有vf或者Aura中的standard controller以及匿名函数, Test method with System.runAs() 执行会走User Mode
3. 但是custom controller, 或者standard controller with extension controller 都会是system mode.

讲到这里其实还可以多讲一点，就是在Apex里面还有sharing的关键词可以对记录级别进行控制。

![sharing_keywords](/assets/access_visibility/sharing_keywords.png "sharing_keywords")

**With sharing：**<br/>
        执行的时候，会考虑到当前用户的sharing权限，只能看到其有权限看到的记录。<br/>
        匿名函数执行时候默认是with sharing模式跑。<br/>
        With sharing里面调用without sharing, 那结果还是with sharing执行<br/>
        后果： SOQL，SOSL获取到的row可能会变少，DML操作可能会失败。<br/>
        
**Without sharing：** <br/>
        执行的时候，会忽略对当前用户的权限设定，此时当前用户可以看到系统中本来无权限看到记录。<br/>
        Without sharing 里面调用with sharing: 那结果会是with sharing <br/>
        
**Inherited sharing:** <br/>
         直接调用的时候，默认是with sharing模式跑。<br/>
         如果显式的从另外一个without sharing class中调用inherited sharing apex那么inherited sharing此时是without sharing模式跑。<br/>

**Omitted sharing：** <br/>
        是指没有任何关键词修饰的时候，默认是用with sharing进行（前提是不是从其他类中call的）,<br/>
        没有官方答案，但是查阅所有的资料都是说system mode跑，without sharing ,但是最终的实验结果是with sharing.<br/>
        比如这个高票答案： https://salesforce.stackexchange.com/questions/84006/default-class-is-with-sharing-or-without-sharing    <br/>
        <br/>

**强调:** <br/>
        With sharing的关键词只会影响系统中记录层面sharing的设定，并不会影响CRUD and FLS 层面的设定。


