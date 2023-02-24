---
title: Salesforce的持续集成-CI Using SFDX 
categories:
- Tech 
date: 2020-07-20
excerpt: "使用SFDX Command Line实现持续集成"
feature_image: "https://picsum.photos/id/403/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
### 总述
* --- Part 1： 什么是持续集成<br/>
* --- Part 2： 准备工作<br/>
* --- Part 3： Salesforce端创建Connected App<br/>
* --- Part 4： CI端解密私钥，创建流程<br/>



### Part 1 什么是持续集成
<hr>
持续集成 Continuous Integration (CI):<br>频繁的（一天多次的）将所有开发者的工作合并到主干上<br>

持续集成是一种软件开发的实践，适合团队协作的项目。鼓励团队的每个开发人员经常集成自己的开发到项目。<br>
而每一次的集成都会通过自动化的构建来进行验证。（当项目代码有变更时，立即进行构建和测试并反馈运行结果）<br>
我们可以根据测试结果，确定新代码是否可以和原有代码正确的集成在一起。


### Part 2 准备工作
<hr>
* [安装Salesforce CLI](https://developer.salesforce.com/tools/sfdxcli)
* 在开发环境中开启DevHub选项，具体的步骤可以参阅[官方文档](https://developer.salesforce.com/docs/atlas.en-us.224.0.sfdx_setup.meta/sfdx_setup/sfdx_setup_enable_devhub.htm)
* 本文使用Travis CI和GitHub为例，Travis CI中可以直接授权使用GitHub进行登录。
* 本文使用这个repo做示例[sfdx-travisci GitHub repo](https://github.com/forcedotcom/sfdx-travisci)，可以fork并git clone项目到本地。项目中已经包含一个SFDX project.
* 使用GitHub账号登录Travis CI，点击右上角的setting，勾选sfdx-travisci项目。如下图：<br/>

   ![sfdx_travisci](/assets/ci_using_sfdx/sfdx_travisci.png "sfdx_travisci")

* 安装[Travis CI CLI](https://github.com/travis-ci/travis.rb#installation)

  ```
  sudo gem install travis --no-document
    //使用sudo命令
  travis  
    //输入travis 检查是否安装成功
  ```

### Part 3 Salesforce端创建Connected App
<hr>
* **创建SSL certificate**
```
cd ..
mkdir certificates
    //回退到项目外（以确保不被git追踪到），创建文件夹以用来存放SSL证书
which openssl
    //检查本地是否已经安装了OpenSSL，如果返回/usr/bin/openssl说明已经安装
    //否则，执行 Homebrew: brew install openssl （MacOS）
cd certificates
openssl genrsa -des3 -passout pass:SomePassword -out server.pass.key 2048
    //生成一个RSA private key
openssl rsa -passin pass:SomePassword -in server.pass.key -out server.key
    //Create a key file from the server.pass.key file:
rm server.pass.key
    //删除server.pass.key
openssl req -new -key server.key -out server.csr
openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
    //填写必要信息，生成certificate
```

此时certificates文件夹下会生成以下三个文件：server.crt，server.csr，server.key

* **创建Connected App**
```
cd ..
   //回退到sfdx-travisci目录下，
sfdx force:auth:web:login -d -a DevHub
   //登录DevHub， 打开DevHub：sfdx force:org:open -u DevHub
```
在打开的DevHub中，填写必要信息如下图：
Setup > APP Manager > New Connected App
* connected app name 可以随便取
* 勾选Enable OAuth Settings
* Callback URL填写： http://localhost:1717/OauthRedirect
* 勾选Use digital signatures，并上传刚才生成的server.crt文件
* OAuth scopes选择如图

  ![connectedAPP](/assets/ci_using_sfdx/connectedAPP.jpeg "connectedAPP")

* Manage > Edit Policies
Permitted  User选择Admin approved users are pre-authorized
* 创建一个Permission Set并assign给自己以及需要的user
* 回到新建的Connected App中，选择Manage Permission Sets并赋予刚才创建的Permission Set
  ![permissionSet](/assets/ci_using_sfdx/permissionSet.png "permissionSet")

创建成功之后，会生成一个Consumer Key，这个key是之后Travis CI端要验证使用的。

### Part 4 CI端解密私钥，创建流程
<hr>
server.key不可以随便暴露出去，所以我们要进行一道加密程序，之后在Travis CI使用验证的时候，再进行一道解密程序。

```
   //打开.travis.yml文件，把类似如下代码删去：
- openssl aes-256-cbc -K $encrypted_0db5e9c4fee8_key -iv $encrypted_0db5e9c4fee8_iv
   -in assets/server.key.enc -out assets/server.key -d
  //把刚才生成的server.key文件放在assets文件夹中， 
travis login --org
  //登录Travis CI 
travis encrypt-file assets/server.key assets/server.key.enc --add
  //使用Travis CI CLI 对文件server.key进行加密，生成文件erver.key.enc在同一个文件夹中。
  //手动删去server.key文件，不过经常使用的方式是在.gitignore文件中，添加一下文件server.key
```

>备注：执行加密程序后，.yml文件会自动生成一个解密程序在文件的头部。

**在Travis CI配置一下环境变量**
CONSUMERKEY是在Salesforce端配置Connected APP时候生成的
USERNAME 是指DevHub中你的用户名，即org登录名

```
travis env set CONSUMERKEY <替换connected app consumer key>
travis env set USERNAME <替换your Dev Hub username>
```

至此使用SFDX配置了CI的流程，具体的执行内容会写在.yml文件中。


##### **小结**
<hr>
1. 本地生成SSL Certificate
2. Salesforce端利用创建的SSL Certificate创建Connected App,并生成consumer key
3. CI工具利用私钥和consumer key进行双重验证，自动自行yml文件中预设的脚本进行自动化的Unit test.
如下图所示： 

![structure](/assets/ci_using_sfdx/structure.png "structure")
