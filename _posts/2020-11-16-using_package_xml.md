---
title: Some commonly used package.xml metadata
categories:
- Tech 
date: 2020-11-16
excerpt: "Commonly used package.xml commands for easy searching"
feature_image: "https://picsum.photos/id/870/600?image=872"
---
<div id="阅读时长10min" style="color:rgb(168,173,172)">阅读时长：10min</div>
---
### 总述
使用Visual Studio Code做SF开发时候，连接了dev org之后，有时候需要使用package.xml文件进行配置类开发的as code行为。<br/>
或者进行Lightning开发的时候，需要本地查看LWC或者Aura Component（暂时SF不支持在browser端直接查看编辑LWC）<br/>
本文总结了一些常用的命令，方便之后的查阅使用，并保持持续更新。<br/>
* --- Part 1： 一个sample package.xml长什么样子?<br/>
* --- Part 2： 如何创建一个package.xml<br/>
* --- Part 3： 一些常用的命令查阅<br/>


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

### Part 2 如何创建一个package.xml
一般在使用VsCode做Salesforce开发的时候，创建了一个SFDX Project之后，系统不会自动的帮助创建一个package.xml文件。
需要我们手动的在Project下创建一个manifest文件夹，然后在manifest文件夹下创建一个package.xml文件。<br/>
manifest文件夹和force-app/main/default文件夹平行。


### Part 3 一些常用的命令查阅
以下是一些常用的命令<br/>
也可以通过Workbench中的info > Metadata Types and Components,
方便的查看都有哪些metadata type.
![metadata type](/assets/using_package/metadataType.png "metadata type")
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

##### Custom Tabs 
```
    <types>
        <members>API name of the tab</members>
        <name>CustomTab</name>
    </types>
```

##### Apex Class
```
    <types>
        <members>SampleController</members>
        <members>SampleControllerTest</members>
        <name>ApexClass</name>
    </types>
```

##### Apex Trigger
```
    <types>
        <members>TriggerNameAPI</members>
        <name>ApexTrigger</name>
    </types>
```

##### Aura Component
```
    <types>
        <members>SampleAuraComp</members>
        <name>AuraDefinitionBundle</name>
    </types>
```

##### List View of Object
```
    <types>
        <members>Custom_object__c.ListViewAPI</members>
        <name>ListView</name>
    </types>
```

##### Flexipage
```
    <types>
        <members>SampleFlexipageAPI</members>
        <name>FlexiPage</name>
    </types>
```

##### Page Layout
```
    <types>
        <members>Lead-CRM Sales Lead Page</members>
        <name>Layout</name>
    </types>
```

##### Compact Page Layout
```
    <types>
        <members>Lead.CRM_Sales_Compact_Layout</members> 
        <name>CompactLayout</name>
    </types>
```

##### Record Type
```
    <types>
        <members>Task.RecordTypeName</members>
        <name>RecordType</name>
    </types>
```

##### Profile
```
    <types>
        <members>ProfileName</members>
        <name>Profile</name>
    </types>
```

##### Picklist Value
```
    <types>
        <members>FieldAPI</members>
        <name>StandardValueSet</name>
    </types>
```


##### Lead Status 比较特殊，虽然该字段的API为Status
```
    <types>
        <members>LeadStatus</members> 
        <name>StandardValueSet</name> 
    </types>
```


##### Business Process   //lead process
```
    <types>
        <members>Lead.Test Process</members>
        <name>BusinessProcess</name>
    </types>
```

##### Validation Rule
```
    <types>
        <members>Lead.CouldNotChooseConvertedFromPicklist</members>
        <name>ValidationRule</name>
    </types>
```

##### Themes and Branding
```
    <types>
        <members>APIofTheme</members>
        <name>LightningExperienceTheme</name>
    </types>
//上面是创建的主题，下面是具体的主题配置
    <types>
        <members>LEXTHEMINGThoughtWorks_Blue</members>    //具体的members名称用上面的命令retrieve之后可得到
        <name>BrandingSet</name>
    </types>
```

##### Global Action
  >global action 也是用QuickAction标签来retrieve
```
    <types>
		<members>LogACall</members>
		<name>QuickAction</name>
	</types>
```

##### Static Resource
```
    <types>
        <members>Name_Of_The_Static_Resource</members>
        <name>StaticResource</name>
    </types>
```

##### Account Team Member Role / Opportunity Team Member Role
```
    <types>
        <members>SalesTeamRole</members>
        <name>StandardValueSet</name>
    </types>

    //OpportunityTeamMember Role field
    <types>
        <members>OpportunityTeamMember.TeamMemberRole</members>
        <name>CustomField</name>
    </types>

    //AccountTeamMember Role field
    <types>
        <members>AccountTeamMember.TeamMemberRole</members>
        <name>CustomField</name>
    </types>
```
##### User Pagelayout / User Profile Pagelayout  
```
    <types>
        <members>User-User Layout</members>
        <members>UserAlt-User Profile Layout</members>
        <members>CommunityMemberLayout-Community Member Layout</members>
        <name>Layout</name>
    </types>
```

##### Report
```
    <types>
        <members>CRMUsageReports</members>
        <members>CRMUsageReports/Report_API</members>
        <name>Report</name>
    </types>
    // 需要注意1 ： retrieve report的时候也需要一并把report folder信息一并retrieve
    // 需要注意2 ： report的Edit页面，Save下拉可以看到Report的Properties,最好folder name没有空格
```

##### Dashboard
Dashboard比较特别，retrieve时候需要通过workbench找到唯一的名称<br/>
Workbench > info > Metadate types&Component > 选择Dashboard > 找到Dashboard所在的folder > 找到对应的dashboard fullname, 如下图： <br/>

![dashboard](/assets/using_package/dashboard.png "dashboard")

```
    <types>
        <members>CRMStandardUserDashboards/WMzeoFyohVbWXvaFwATUkWaczasdsadawJgF</members>
        <name>Dashboard</name>
    </types>
```

##### Public Group
```
    <types>
        <members>Public_Group_API</members>
        <name>Group</name>
    </types>
```

##### Lead Convert Field Mapping Relationship
```
    <types>
        <members>*</members>
        <name>LeadConvertSettings</name>
    </types>
```

##### StandardValueSet 
```
    // https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/standardvalueset_names.htm
```

##### LWC Lightning Web Component
```
    <types>
        <members>*</members>
        <name>LightningComponentBundle</name>
    </types>
```

##### Aura Component
```
    <types>
        <members>*</members>
        <name>AuraDefinitionBundle</name>
    </types>
```

##### App logo
```
    <types>
        <members>logoNameRetrieveFromApplicationXMLFile</members>
        <name>ContentAsset</name>
    </types>
```

##### Visualforce Page
```
    <types>
        <members>visualforcePageAPI</members>
        <name>ApexPage</name>
    </types>
```

##### Custom Label
```
    <types>
        <members>*</members>
        <name>CustomLabels</name>
    </types>
```

......... To Be Continue
