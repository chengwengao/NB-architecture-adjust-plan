# NB-IoT火灾初期预警系统架构调整方案

---

备注：NB-IoT火灾初期预警系统项目，后文简称烟感项目。

# 一、背景

##  1、烟感项目运行终端

 

 - web端软件系统平台 

 - Andoriod app

 - IOS app

 - 微信公众号

## 2、烟感项目原始技术栈

 烟感项目外包人员选用的主要技术栈如下：

\`mirth connect + vue + h5 + hbuilder + Tomcat\` 等\(下文 mirth connect 简称mirth\)



 ## 3、原始技术栈解读



 - 接口实现

 web后端采用mirth作为后台接口提供者，web前端采用vue开发；

 Android和IOS app采用vue开发的h5页面，然后通过hbuilder打包成apk或ipa的安装包实现，移动端app的后台接口均由mirth提供；

 微信公众号主要采用html+css方式实现前端页面，后端还是采用mirth接口提供方式；



 - 运行容器

 前端页面采用Tomcat作为静态资源运行容器，后台接口采用mirth作为运行容器和请求接口转发工具；

## 4、原始技术栈优点

 备注：后文所有技术对比将着重针对mirth，其他技术点不做重点探讨！

 - 现有技术选型满足目前业务应用场景，包括与第三方平台对接与新的业务需求实现；

 - mirth后端支持热部署，可实现在不重启后端服务器的情况下几乎实时更新后台代码，用户零感知；

 - mirth + postman可快速开发与第三方数据对接的需求，充分发挥mirth强大的数据接口引擎特色；

 - 简单的配置可以让mirth支持跨域访问；

## 5、原始技术栈的缺点



 - 免费版的mirth未提供诸如java一样发达、完善的源代码管理与追踪体系，采用mirth开发的后端代码在版本管理和过程追溯方面体验相当差；

 - mirth比较小众，新人入职有学习成本，相对目前生态良好的java开发人员而言，招一个懂mirth甚至是愿意学mirth的人比招java的开发人员要难得多；

 - mirth作为外包的技术壁垒，每次开发新的需求都伴随着高昂的成本，如果让内部研发使用mirth开发，在项目紧急急需扩充研发人员的情况下，其他项目组的研发无法提供开发支持；

 - mirth中没有提供的api只能通过调用外部jar的形式引入，jar包修改后需求重启mirth服务，如果jar包众多，这就为以后jar的管理带来很多的麻烦；

 - mirth中的javascript脚本虽然支持大多数js数据类型和语法，但是在调用外部jar包的传参时，对非json或者字符串格式的数据（比如对象）无法支持；

# 二、风险

 目前外包开发的这套系统已经商用，预案和演练准备不充分替换失败将导致严重后果。

# 三、技术选型

 ## 1、选型原则

 

 - 保证现有系统稳定；

 - 对现有系统架构设计代码侵入尽可能少；

 - 选用新架构和框架必须是经过生产环境检验，保证稳定运行的；

 - 升级过程相对简单可靠；

 - 新架构和框架可维护性高，要能有效解决mirth现有暴露的缺点；

##  2、选型思路：

 \(1\)微信公众号：

 - 前端：微信公众号目前只需维护若干html和js文件，暂无替换必要；

 - 后端：由mirth提供后台接口，需要替换；

 

 （2）Android和IOS APP

 - 前端：移动端前端均采用vue，编译后再通过Hbuilder打包生成安装包，暂无替换必要；

 - 后端：由mirth提供后台接口，需要替换；

 （3）web端软件平台

 

 - 前端：采用vue,暂无替换必要；

 - 后端：由mirth提供后台接口，需要替换；

 总结：目前负责前端交互的vue暂无需替换，由于负责后端交互的mirth与业务逻辑耦合较大，重构需要的时间成本较高，且短时间无法保证稳定性，因此\*\*现阶段暂只考虑新功能模块采用新的架构，老的代码还是保持原有的架构运行\*\*。

##  3、选型方案

 前端保持原有的vue，后端选用已经在农机、电动车等项目组生产环境稳定运行的nginx + Spring Boot + zookeeper + Dubbo分布式前后端解决方案。

##  4、选型依据

 

 ### （1）nginx

 - 前端使用vue打包后的是静态资源，nginx对静态文件的处理能力比tomcat\(tomcat主要用于处理servlet\)快很多；

 - Nginx的使用对项目并发能力有很大的提升；

 - Nginx通过简单的配置可以解决跨域问题，对代码侵入小；

 - 经验证使用Nginx替换Tomcat对所有终端代码的侵入性较小，基本无需改动源代码，只需改变一些部署方式；

### （2）Spring Boot + zookeeper + Dubbo

Spring Boot自带Tomcat容器，项目打成jar包后直接执行java -jar xxx.jar命令后即可运行，无需依赖其他外部容器；

该方案的有点可参考附件中的文档：《架构演变》

## 5、可行性验证

备注：该验证性方案均是基于Nginx（不再使用Tomcat）作为处理前端页面的容器可能存在的风险性测试得出的结论；

 ### （1）微信公众号

将wx项目文件放在nginx指定目录下，在微信公众平台配置好域名访问即可正常访问微信公众号。基于现有架构只需调整wx项目文件的发布路径即可，无需调整微信公众平台相关配置；

 ### （2）Android、IOS APP

 前端页面无需做任何改变，后端接口也能通过nginx代理访问；

 ### （3）web端软件平台

 前端页面无需做任何改变，后端接口也能通过nginx代理访问，包括图片、导入导出等静态资源文件的下载、websocket推送；为了不影响客户存放在浏览器的书签地址失效，调整web端项目的存放路径即可保证原有书签正常使用；

 

# 四、具体实施方案

## 1、nginx核心配置

![](/assets/5.png)

## 2、配置说明
![](/assets/1.png)
dist文件夹是前端vue页面打包和静态资源的根目录，与nginx的可执行程序在同一目录下，所有的前端页面和静态资源文件都放在dist目录下，excel文件夹下存放的事是web端导入导出文件模板，logo文件夹下存放的是web端图片文件夹,static文件夹存放的是web端打包的静态资源文件，wx文件夹下存放的是微信前端页面资源文件；为了确保客户之前存放的浏览器书签能正常访问，web端的编译打包好的首页index.html文件存放在zc文件夹下。
![](/assets/2.png)
![](/assets/3.png)![](/assets/4.png)
我们可以将其与之前Tomcat目录下的部署存放方式做一下对比：
![](/assets/6.png)
对比发现差别不大，对现有方式的改动较小。

# 五、故障回退预案
使用原有的Tomcat容器重新发布项目，待解决问题后开启nginx替换Tomcat运行。


