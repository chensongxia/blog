---
title: Maven学习笔记
date: 2019-03-09 22:06:01
author: 陈松夏
tags:
  - 工具
  - Maven
categories: 
  - 工具
---

--------------------------

Maven是我们在做Java开发过程中用经常用到的一个辅助工具。本篇博客是我学习Maven的一个记录博客，学习过程主要参考《Maven实战》这本书。同时也参考了[Maven的官方文档](http://maven.apache.org/)。

--------------------------

## 1. Maven简介
### 1.1 什么是Maven
**Maven**是一个开源的Java项目**构建、依赖管理和项目信息管理**工具。

### 1.2 何为构建
所谓**构建**是指项目编译、运行单元测试、生成项目文档、打包和项目部署等一系列操作。

### 1.3 Maven的优点

Maven是一个优秀的构建工具：帮我们**规范了项目的构建过程**，我们不在需要像以前那样自己编写不通用的项目构建脚本，**大大降低自写脚本出错的效率**，同时也规范了团队项目的标准化。只需执行一些简单的Maven命令就可以实现项目构建工作。另外，Maven还提供了很多现成插件来完成各种构建任务，不需要我们自己去写脚本。
<!-- more -->
Maven抽象了一个完整的项目构建的生命周期模型，这个模型吸取了其他构建工具的优点，总结了大量项目的实际需求。如果遵循这个模型，可以避免很多不必要的错误。Maven已经提供了大量成熟的Maven插件来完成它抽象的这套生命周期模型，如果我们的项目有特殊的构建需求，可以通过实现自己的构建插件来完成项目特殊的构建需求。

统一的依赖管理（中央仓库）。

Maven还可以管理如下信息：项目描述、开发者列表、版本控制系统地址、许可证和缺陷管理系统地址等。  



### 1.3 同类技术对比
在Maven之前，Make和Ant也是两个比较流行的项目构建工具。但是使用这两个工具，**针对每个项目我们必须自己编写项目构建脚本，而且这个两个工具都没有依赖管理的功能（Ant依赖于Lvy进行依赖管理）**，所以这两个工具逐渐成为过去式，Maven成为了Java世界中项目构建的标准。（Gradle也是一款不错的项目构建工具）

## 2. Maven安装配置
### 2.1 windows环境安装

```java
    step1：安装JDK，配置JAVA_HOME,添加到path；
    step2：下载Maven安装包，解压到指定目录；
    step3：将Maven的安装目录添加到path；
    step3：使用mvn -v校验是否安装成功。
```

> 配置M2_HOME 和 MAVEN_HOME两个环境变量（指代Maven的安装目录，到bin那层目录）
> 通常需要设置**MAVEN_OPTS**的值为-Xms128m -Xmx512m，因为Java默认的最大可用内存往往不能够满足Maven运行的需要，比如在项目较大时，使用Maven生成项目站点需要占用大量的内存，如果没有该配置，则很容易得到java.lang.OutOfMemeoryError异常。因此，一开始就配置该环境变量是推荐的做法。


### 2.2 Linux环境安装
安装方式和上述类似

### 2.3 .m2目录
安装完Maven后，一般会在用户的主目录下有一个.m2目录。这个下面会有一个仓库目录，默认情况下，缓存下来的Jar包会存放在这个目录下。我们可以在这个目录下添加settings.xml文件配置仓库目录。注意这个settings文件只对当前用户生效。M2_HOME/config下的settings文件对所有用户生效。推荐使用用户范围的settings。

### 2.4 配置Http代理
有时候公司为了安全起见，内部网络不能直接访问外部互联网。但是提供了代理服务器进行外部网络访问，这是想要让Maven访问到外部的Maven中心仓库需要配置代理。

```xml
    <settings>
	    ...
	    <proxies>
	        <proxy>
	           <id>optional</id>
	           <active>true</active>
	           <protocol>http</protocol>
	           <host>10.47.22.247</host>
	           <port>80</port>
               <username>xx</username>
               <password>xx</pawwword>
	           <nonProxyHosts>some.host.com|*.google.com</nonProxyHosts>
	        </proxy>
	    </proxies>
	    ...
    </settings>
```

proxies下面可以配置多个代理，默认情况下第一个被激活的代理生效。如果代理服务器需要认证，那我们还需要提供用户名密码。<nonProxyHosts>标签用来指定访问哪个地址时不需要经过这个代理。

### 2.5 Maven安装最佳实践

#### 2.5.1 配置MAVEN_OPTS
通常需要设置**MAVEN_OPTS**的值为-Xms128m -Xmx512m，因为Java默认的最大可用内存往往不能够满足Maven运行的需要，比如在项目较大时，使用Maven生成项目站点需要占用大量的内存，如果没有该配置，则很容易得到java.lang.OutOfMemeoryError异常。因此，一开始就配置该环境变量是推荐的做法。

#### 2.5.2 配置用户范围的settings.xml
我们可以配置M2_HOME/conf/settings.xml，也可以配置~/.m2/settings.xml。前者会对所有用户生效，后者只会对当前用户生效。推荐配置用户范围内的setting，这样不会影响其他用户的配置，在升级Maven时也不需要重新配置。当然，如果你需要做全局配置，就需要配置做全局配置。

## 3. Maven坐标和依赖
### 3.1 Maven坐标
在Maven中，坐标可以理解为一个依赖的唯一标识。Maven的坐标元素包括groupId、artifactId、version、packaging、classfier。只要我们提供正确的坐标元素，Maven就能找到对应的构件，首先去你的本地仓库查找，没有的话再去远程仓库下载。**如果没有配置远程仓库，会默认从中央仓库地址(http://repo1.maven.org/maven2)下载构件**，该中央仓库包含了世界上大部分流行的开源项目构件，但不一定所有构件都有。下面介绍下坐标中每个标签的含义：

- groupId:定义当前Maven项目的隶属的实际项目。比如xx公司xx团队的xx项目，xx项目下又包含几个模块项目。这时groupId我们就可以命名为com.xxCompany.xxTeam.xxProject，子项目的名字可以通过artifactId来指定；
- artifactId：定义实际项目中的一个Maven模块（子项目）的名字。一般推荐用项目的名称作为前缀。比如我们整个大项目的名称是xxProject，这个子模块的名称是core。那artifactId可以设置为xxProject-core。
- version：项目的版本
- packaging：项目打包方式（jar、war、pom等）默认的是jar。
- classifier：用来定义javadoc和源代码source等构件，这个元素不能直接定义。

> 邮件测试时我们可以考虑使用GreenMail这个类库

### 3.2 dependeny标签的内容

dependeny标签下面可以有下面的标签：

- groupId、artifactId和version这几个标签来定位具体的依赖；
- type这个标签对用maven坐标中的packaging属性，一般不需要指定，默认是jar；
- scope：见3.3节详解；
- optional：值是true或false，可选依赖不会被传递；
- exclusions：排除传递依赖。

### 3.3 scope属性详解
Maven环境下有三种classpath，分别是编译classpath、测试classpa和运行classpath。scope属性和这三种classpath有关。scope的值可以是下面几种：

- compile：如果将一个依赖的scope属相设置为compile（默认就是compile），那么这个依赖对于三种classpath都有效；
- test：编译测试classpath有效，这种Jar包不会被打包进最终的项目；
- provide：对于编译和测试有效，运行时无效。说明在项目运行时这个依赖会被提供，就不需要我们打进项目了。典型的就是Servlet-api（这个依赖在运行时会由web容器提供）
- runtime：典型的应用就是JDBC的驱动实现（就是这个依赖的Jar只在项目运行是才会用到）
- system：依赖的Jar包在本地（不建议使用这个属性）

```xml
<dependency>
    <groupId>org.xx.xx</groupId>
    <artifactId>xx-web</artifactId>
    <scope>system</scope>
    <systemPath>{java.home}/lib/xx.jar</systemPath>
</dependency>
```

- import：该属性只会在dependenceManagement属性中生效。用来导入类型为pom的依赖，典型的应用看Springboot。

|Scope|编译classpath|测试classpath|运行时classpath|列子|
|-----|:-----------:|------------|-------------:|----|
|compile| Y | Y | Y | Spring-core |
|test | - | Y | - | junit |
|provide| Y | Y | - | servlet-api |
|runtime| - | Y | Y | jdbc驱动实现 |
|system| Y | Y | - |  |

### 3.4 传递依赖
Maven的依赖是具有传递性的，比如A->B,B->C,那么A间接的依赖于C，这就是依赖的传递性，其中A对于B是第一直接依赖，B对于C是第二直接依赖，C为A的传递性依赖。这里还有一个依赖scope的问题。如果A->B的scope是compile，B->C的scope也是compile，那么A->C的scope也是compile。

表格：传递依赖
|     |compile|test|provided|runtime|
|-----|:-----:|----|------:|-------|
|compile|compile| - | - | runtime |
|test|test| - | - | test |
|provided|provided| - | provided | provided |
|runtime|runtime| - | - | runtime |

上表中最左边一列表示A->B的第一直接依赖，最上面一行表示B->C的第二直接依赖，表格中的内容表示A->C的依赖内容。举个列子，当A->B的直接依赖范围是test，B->C的第二直接依赖是compile，那么A->C的依赖范围是test。

### 3.5 依赖调解

下面我们来思考这样一个问题，如果A->B->C->X(1.0),A->D-X(2.0),即A间接依赖X，我们可以看到有两条路径都依赖X，那么maven将会选择哪个版本的X？maven当中有一套自己的规则，我们来说明一下，maven传递性依赖的一些规则以及如何排除依赖冲突。

Maven里面对于传递性依赖有以下几个规则：

1. 最短路径原则：如果A对于依赖路径中有两个相同的jar包，那么选择路径短的那个包，路径最近者优先，上述会选X(2.0)。

2. 第一声明优先原则：如果A对于依赖路径中有两个相同的jar包，路径长度也相同，那么依赖写在前面的优先。例如：A->B->F(1.0),A->C->F(2.0)，会选F(1.0)。

3. **可选依赖不会被传递**，如A->B,B->C,B->D,A对B直接依赖，B对C和D是可选依赖，那么在A中不会引入C和D。可选依赖通过optional元素配置，true表示可选。如果要在A项目中使用C或者D则需要显式地声明C或者D依赖。

### 3.6 依赖排除
```xml
<dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-core</artifactId>
     <version>3.2.8</version>
     <exclusions>
           <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
           </exclusion>
      </exclusions>
</dependency>
```

## 4. Maven仓库
**Maven的仓库分为本地仓库和远程仓库**。Maven在寻找依赖的时候会先从本地仓库寻找，本地没有的话会从远程仓库去下载到本地然后再使用。

### 4.1 仓库的分类
![仓库分类](http://images2015.cnblogs.com/blog/915951/201612/915951-20161205142602226-1306419231.png)

#### 4.1.1 本地仓库
本地仓库需要我们在settings.xml配置文件中配置。注意，默认情况下.m2目录下是没有这文件的，需要我们自己配置。所有的依赖都会被缓存到本地仓库，本地没有的话回去远程仓库拿。

    <localRepository>D:\software\maven\Repository</localRepository>

#### 4.1.2 远程仓库
远程仓库有好几种分类：**Maven的中央仓库、Nexus搭建的私有服务器（如果Nexus上没有相应的依赖，Nexus会自动从外部的仓库下载）和JBoss、谷歌的中央仓库等**。一般我们只配置一个远程仓库。如果中央仓库中没有相应的依赖，就会报错。下面是Maven配置的默认的远程仓库，这个仓库中包含了绝大多数的依赖构建，如果我们没做其他配置，Maven就会从这个仓库中下载依赖。
```xml
    <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
    </repositories>
```

Maven还允许用户配置私有服务器。如果我们将远程仓库配置成我们自己搭建的私有服务器，那么Maven每次都会从私有服务器上去请求依赖，如果私服上不存在我们需要的依赖，私服会先从外部仓库下载依赖然后缓存到服务器上再提供给我们。另外私服也允许我们自己上传依赖。架设Maven私服有如下优点：

- 节省公司外网贷款；
- 因为依赖会被缓存到私服上，所以依赖的下载速度会很快；
- 方便上传团队内部的依赖，统一管理，共享。


### 4.2 远程仓库的配置
Maven已经默认为我们配置了一个远程仓库，在maven安装目录下的：/lib/maven-model-builder-${version}.jar中我们可以找到这个配置。
```xml
    <repositories>  
     <repository>  
	    <id>central</id>  
	    <name>Central Repository</name>  
	    <url>https://repo.maven.apache.org/maven2</url>  
	    <layout>default</layout>  
	    <snapshots>  
	      <enabled>false</enabled>  
	    </snapshots>  
     </repository>  
    </repositories>  
```
这里我们只要知道，中央仓库的id为central，远程url地址为http://repo.maven.apache.org/maven2，它关闭了snapshot版本构件下载的支持。

当然我们也可以自己在pom中配置repository，有可能你依赖的一个jar在central中找不到，它只存在于某个特定的公共仓库，这样你也不得不添加那个远程仓库的配置。
```xml
    <project>  
	...  
	  <repositories>  
	    <repository>  
	      <id>maven-net-cn</id>  
	      <name>Maven China Mirror</name>  
	      <url>http://maven.net.cn/content/groups/public/</url>  
	      <releases>  
	        <enabled>true</enabled>  
	      </releases>  
	      <snapshots>  
	        <enabled>false</enabled>  
	      </snapshots>  
	    </repository>  
	  </repositories>  
	  <pluginRepositories>  
	    <pluginRepository>  
	      <id>maven-net-cn</id>  
	      <name>Maven China Mirror</name>  
	      <url>http://maven.net.cn/content/groups/public/</url>  
	      <releases>  
	        <enabled>true</enabled>  
	      </releases>  
	      <snapshots>  
	        <enabled>false</enabled>  
	      </snapshots>      
	    </pluginRepository>  
	  </pluginRepositories>  
	...  
	</project>  
```

我们先看一下<repositories>的配置，你可以在它下面添加多个<repository> ，每个<repository>都有它唯一的ID，一个描述性的name，以及最重要的，远程仓库的url。此外，<releases><enabled>true</enabled></releases>告诉Maven可以从这个仓库下载releases版本的构件，而<snapshots><enabled>false</enabled></snapshots>告诉Maven不要从这个仓库下载snapshot版本的构件。禁止从公共仓库下载snapshot构件是推荐的做法，因为这些构件不稳定，且不受你控制，你应该避免使用。当然，如果你想使用局域网内组织内部的仓库，你可以激活snapshot的支持。

**这边整理下Maven查询仓库的顺序**：

1. 先查询本地仓库，如果本地仓库中没有，进入第二步；
2. 查询Maven给central仓库配置镜像，如果配置了镜像就去镜像查，如果没有，就去中央仓库查；
3. 查询Maven有没给你自己配置的仓库指定镜像，如果配置了镜像就去镜像查，否则就从你自己配置的仓库查。


#### 4.2.1 远程仓库的认证
大部分公共的远程仓库无须认证就可以直接访问，但我们在平时的开发中往往会架设自己的Maven远程仓库，出于安全方面的考虑，我们需要提供认证信息才能访问这样的远程仓库。配置认证信息和配置远程仓库不同，**远程仓库可以直接在pom.xml中配置，但是认证信息必须配置在settings.xml文件中**。这是因为pom往往是被提交到代码仓库中供所有成员访问的，而settings.xml一般只存在于本机。因此，在settings.xml中配置认证信息更为安全。
```xml
    <servers>
	    <server>
	       <id>releases</id>
	       <username>pub_mvn_deploy</username>
	       <password>mvn.Deploy</password>
	     </server>
    </servers>
```
上面代码我们配置了一个id为releases的远程仓库认证信息。Maven使用settings.xml文件中的servers元素及其子元素server配置仓库认证信息。认证用户名为admin，认证密码为admin123。这里的关键是id元素，settings.xml中server元素的id必须与pom.xml中需要认证的repository元素的id完全一致。正是这个id将认证信息与仓库配置联系在了一起。

#### 4.2.2 部署构建到远程仓库
```xml
     <distributionManagement>
	         <repository>
	             <id>releases</id>
	             <name>public</name>
	             <url>http://59.50.95.66:8081/nexus/content/repositories/releases</url>
	         </repository>
	         <snapshotRepository>
	             <id>snapshots</id>
	             <name>Snapshots</name>
	             <url>http://59.50.95.66:8081/nexus/content/repositories/snapshots</url>
	         </snapshotRepository>
	 </distributionManagement>
```
distributionManagement包含repository和snapshotRepository子元素，前者表示发布版本（稳定版本）构件的仓库，后者表示快照版本（开发测试版本）的仓库。这两个元素都需要配置id、name和url，id为远程仓库的唯一标识，name是为了方便人阅读，关键的url表示该仓库的地址。往远程仓库部署构件的时候，往往需要认证，配置认证的方式同上。

配置正确后，运行命令mvn clean deploy，Maven就会将项目构建输出的构件部署到配置对应的远程仓库，如果项目当前的版本是快照版本，则部署到快照版本的仓库地址，否则就部署到发布版本的仓库地址。

### 4.3 快照版本
在Maven世界中版本分为发布版本和快照版本。1.0.0、1.3-alpha-4和2.0这种版本是稳定的发布版本。2.1-SNAPSHOT或者2.1--20091214-13这种版本是快照版本。
如果我们依赖一个版本为快照的依赖，这种情况下我们我们每次build的时候都会从远程仓库拉取这个依赖的最新版本。（远程仓库会给这个依赖维护一个时间戳，所以Maven能知道最新的版本）

release（最新发布版本）、latest（最新版本）、snapshot这三种版本都是上面的策略。在我们平时使用时不建议使用release和latest版本，因为这些版本更新比较频繁，你的项目依赖了这些版本的Jar包，今天还能编译通过，明天可能就编译不过了。推荐使用稳定发布版本。
```xml
    <repository>
        <releases>
            <enabled></enabled>
        </releases>
        <snapshots>
            <enabled></enabled>
        </snapshots>
    </repository>
```
### 4.4 从仓库解析依赖的机制
。。。待完成

### 4.5 镜像
如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。关于镜像的一个更为常见的用法是结合私服。由于私服可以代理任何外部的公共仓库(包括中央仓库)，因此，对于组织内部的Maven用户来说，使用一个私服地址就等于使用了所有需要的外部仓库，这可以将配置集中到私服，从而简化Maven本身的配置。在这种情况下，任何需要的构件都可以从私服获得，私服就是所有仓库的镜像。这时，可以配置这样的一个镜像：
```xml
    <mirrors>
	    <mirror>
			<!--This sends everything else to /public -->
			<id>nexus</id>
			<mirrorOf>*</mirrorOf>
			<url>http://maven.dev.xxxx.com/nexus/content/groups/public</url>
	    </mirror>
    </mirrors>
```        
mirrorOf标签可以有多种配置方式：

    <！-- 匹配所有仓库 -->
    <mirrorOf>*</mirrorOf>
    <！-- 匹配仓库rep1和rep2 -->
    <mirrorOf>rep1,rep2</mirrorOf>
    <！-- 匹配rep1之外的所有 -->
    <mirrorOf>*,！rep1</mirrorOf>
	<！-- 匹配不在本机上的远程仓库 -->
    <mirrorOf>external:*</mirrorOf>

## 5. Maven生命周期和插件
在Maven中，我们需要掌握的重要概念有**坐标、依赖、仓库、生命周期、插件**。这个章节我们讲解生命周期和插件的主题。

Maven通过对大量项目构建过程的总结，抽象出了一套高度完善的项目构建的生命周期。整个生命周期包含**项目的清理、初始化、编译、测试、打包、集成测试、验证、部署、站点生成等几乎所有的过程**。Maven的整个生命周期是抽象的，这意味着整个生命周期本身不会干任何事情。在Maven项目的整个构建过程中，实际的构建任务都是由Maven插件完成的。Maven本身内置了很多插件，所以我们在项目进行编译、测试、打包的过程是没有感觉到。比如编译就是通过maven-compile-plugin实现的、测试是通过maven-surefire-plugin实现的。

### 5.1 生命周期详解
Maven拥有3套**独立的生命周期**：clean周期、default周期和site生命周期。clean周期的主要作用是清理项目，default周期的作用是构建项目，site周期的作用是生成项目站点。每个生命周期都若干个阶段，比如使用mvn clean命令，就是调用了clean生命周期的clean阶段。

#### 5.1.1 clean生命周期
Clean生命周期一共包含了三个阶段：

- pre-clean：执行一些需要在clean之前完成的工作。
- clean：移除所有上一次构建生成的文件。
- post-clean：执行一些需要在clean之后立刻完成的工作。

mvn clean中的clean就是上面的clean，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，mvn clean等同于 mvn pre-clean clean，如果我们运行 mvn post-clean，那么 pre-clean、clean 都会被运行。这是Maven很重要的一个规则，可以大大简化命令行的输入。但是执行clean生命周期的某个阶段，不会触发其他生命周期的阶段，因为Maven的每个生命周期都是相互独立的。

#### 5.1.2 site生命周期
下面看一下Site生命周期的各个阶段：

- pre-site：执行一些需要在生成站点文档之前完成的工作。
- site：生成项目的站点文档。
- post-site：执行一些需要在生成站点文档之后完成的工作，并且为部署做准备。
- site-deploy：将生成的站点文档部署到特定的服务器上。

这里经常用到的是site阶段和site-deploy阶段，用以生成和发布Maven站点，这可是Maven相当强大的功能，Manager比较喜欢，文档及统计数据自动生成，很好看。

#### 5.1.3 default生命周期
最后，来看一下Maven的最重要的default生命周期，绝大部分工作都发生在这个生命周期中，这里我只解释一些比较重要和常用的阶段：

- validate
- initialize
- generate-sources
- process-sources：处理项目主资源文件，一般来说，是对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中；  
- generate-resources
- process-resources
- compile 编译项目的源代码，一般来说，是编译src/main/java目录下的Java文件至项目输出的主classpath目录中。
- process-classes
- generate-test-sources 
- process-test-sources：处理项目测试资源文件，一般来说，是对src/test/resources目录的内容进行变量替换等工作后，复制到项目输出的测试classpath目录中。
- generate-test-resources
- process-test-resources    
- test-compile：编译项目的测试源代码，一般来说，是编译src/test/java目录下的Java文件至项目输出的测试classpath目录中。
- process-test-classes
- test：使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
- prepare-package
- package：接受编译好的代码，打包成可发布的格式，如 JAR 。
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install：将包安装至本地仓库，以让其它项目依赖。
- deploy：将最终的包复制到远程的仓库，以让其它开发人员与Maven项目使用。
 
基本上，根据名称我们就能猜出每个阶段的用途，关于阶段的详细解释以及其她阶段的解释，请参考 http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html 。

#### 5.1.4 命令行和生命周期

    mvn clean install site-deploy


### 5.2 插件详解
这边先理解一个重要的概念：插件目标（goal）。每个插件的单个功能就可以理解为是一个插件的目标。举个例子，maven-dependency-plugin这个插件可以分析项目的依赖树、可以分析潜在无用的依赖，这一个个功能我们就可以理解为插件的goal。

#### 5.2.1 默认插件绑定
Maven生命周期的某个阶段会和插件的某个goal绑定来完成某个特定的任务。Maven已经在生命周期的某个阶段默认绑定了一些goal，让用户可以直接使用。

表格:clean生命周期绑定的插件
| 生命周期阶段 | 插件目标 |
|------------|----------|
| pre-clean  |    -     |
| clean | maven-clean-plugin:clean|
| post-clean |   -       |

表格:site生命周期绑定的插件目标
| 生命周期阶段 | 插件目标 |
|------------|----------|
| pre-site  |    -     |
| site | maven-site-plugin:site|
| post-site |   -       |
| site-deploy  | maven-site-plugin:deploy |

表格：default生命周期绑定的插件目标
| 生命周期阶段 | 插件目标 | 执行的任务 |
|------------|----------|-----------|
| process-resources  |  maven-resources-plugin:resources  |复制主资源文件到主输出目录 |
| compile | maven-compiler-plugin:compile| 编译主代码到主输出目录 |
| process-test-resources |  maven-resources-plugin:testResources  | -|
| test-compile  | maven-compiler-plugin:testCompile | - |
| test    | maven-surefire-plugin:test |  执行测试用例 |
| package | maven-jar-plugin：jar | 项目打包（如果打成war包的话会绑定其他插件） |
| install | maven-install-plugin:install | 安装到本地仓库 |
| deploy | maven-deploy-plugin | 项目部署 |

上表只是列出了拥有插件绑定关系的阶段，default生命周期还有很多其他阶段，默认他们没有绑定任何插件，因此这些阶段没有任何实际行为。

**除了默认的打包类型jar之外，常见的打包类型还有war、pom、maven-plugin和ear等。**

#### 5.2.2 自定义插件绑定
除了内置的绑定之外，用户还能自己选择将某个插件的目标绑定到生命周期的某个阶段上。这种自定义的绑定方式能让Maven执行更多功能。下面是一个插件配置的列子：
```xml
    <build>
		<plugins>
			<plugin>
			    <groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-source-plugin</artifactId>
				<version>2.1.2</version>
				 <executions>
				  <execution>
				   <goals>
						<goal>jar-no-fork</goal>
				   </goals>
				   <phase>package</phase>
				  </execution>
				 </executions>
			</plugin>
		</plugins>
	</build>
```
在上面的列子中我们将>maven-source-plugin插件的jar-no-fork目标绑定到了default周期的package阶段。所以在执行到package阶段的时候jar-no-fork会被调用。有时候我们会发现我们没配置任何phase，插件的目标也能被调用。这是因为插件已经为目标默认绑定到生命周期的某个阶段上了。使用下面的命令可以查看某个目标绑定到了哪个阶段上
```dos
	mvn help:describe -Dplugin=org.apache.maven.plugins:maven-dependency-plugin -Ddetail
    mvn help:describe -Dplugin=dependency -Ddetail
```

插件仓库和依赖的仓库不是公用的，需要我们另外配置。
```xml
    <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
    </pluginRepositories>
```


## 6. Maven聚合和继承

### 6.1 聚合
聚合最主要的表现形式是使用modle标签来管理多个子模块的构建：
```xml
    <modules>
        <module>spring-boot-quickstart</module>
        <module>spring-boot-web</module>
    </modules>
```
### 6.2 继承
如果多个模块出现相同的依赖包，这样在pom.xml文件的内容出现了冗余、重复的内容，解决这个问题其实使用Maven的继承机制即可，就像Java的继承一样，父类就像一个模板，子类继承自父类，那么有些通用的方法、变量都不必在子类中再重复声明了。Maven的继承机制类似，在一个父级别的Maven的pom文件中定义了相关的常量、依赖、插件等等配置后，实际项目模块可以继承此父项目 的pom文件，重复的项不必显示的再声明一遍了，相当于父Maven项目就是个模板，等着其他子模块去继承。不过父Maven项目要高度抽象，高度提取公共的部分（交集），做到一处声明，多处使用。

可继承的POM元素：
groupId和version是可以被继承的，那么还有哪些POM元素可以被继承呢？以下是一个完整的列表，并附带了简单的说明：

- groupId：项目组 ID，项目坐标的核心元素；
- version：项目版本，项目坐标的核心元素；
- description：项目的描述信息；
- organization：项目的组织信息；
- inceptionYear：项目的创始年份；
- url：项目的 url 地址；
- develoers：项目的开发者信息；
- contributors：项目的贡献者信息；
- distributionManagerment：项目的部署信息；
- issueManagement：缺陷跟踪系统信息；
- ciManagement：项目的持续继承信息；
- scm：项目的版本控制信息；
- mailingListserv：项目的邮件列表信息；
- **properties**：自定义的 Maven 属性；
- **dependencies**：项目的依赖配置；
- **dependencyManagement**：醒目的依赖管理配置；
- **repositories**：项目的仓库配置；
- **build**：包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等；
- reporting：包括项目的报告输出目录配置、报告插件配置等。

### 6.3 插件管理
和依赖管理（dependencyManagement）的理念相同，我们可以在父pom中配置插件管理。这样子模块就可以继承父模块的插件。
```xml
    <build>
        <pluginManagement>
            <plugins>
                
            </plugins>
        </pluginManagement>
    </build>
```
## 7. Nexus创建私服
私服是一种特殊的Maven远程仓库。Nexus分为免费版和专业版。安装包分为附带web容器的Bundle版本，和不带web容器的war包版本。

### 7.1 Nexus仓库类型

仓库有四种类型： 

- group（仓库组） ：没有实际的内容，就是将若干个其他仓库组合成一个组，在这种类型的仓库中下载依赖的时候，Maven会从这个组中包含的仓库依次轮询，知道下载到依赖为止；
- hosted（宿主） ：建在Nexus一台机器上的本地Maven仓库；
- proxy（代理） ：是一个代理，代理了其他远程仓库，比如Maven的central仓库；
- virtual（虚拟）：兼容Maven1 版本的jar或者插件。

每个仓库的格式为maven2或者maven1。此外，仓库还有一个属性为Policy（策略），表示该仓库为发布（Release）版本仓库还是快照（Snapshot）版本仓库。最后两列的值为仓库的状态和路径。

- Maven Central：该仓库代理Maven中央仓库，其策略为Release，因此只会下载和缓存中央仓库中的发布版本构件。
- Releases：这是一个策略为Release的宿主类型仓库，用来部署组织内部的发布版本构件。
- Snapshots：这是一个策略为Snapshot的宿主类型仓库，用来部署组织内部的快照版本构件。
- 3rd party：这是一个策略为Release的宿主类型仓库，用来部署无法从公共仓库获得的第三方发布版本构件。
- Apache Snapshots：这是一个策略为Snapshot的代理仓库，用来代理Apache Maven仓库的快照版本构件。
- Codehaus Snapshots：这是一个策略为Snapshot的代理仓库，用来代理Codehaus Maven仓库的快照版本构件。
- Google Code：这是一个策略为Release的代理仓库，用来代理Google Code Maven仓库的发布版本构件。
- java.net-Maven 2：这是一个策略为Release的代理仓库，用来代理java.net Maven仓库的发布版本构件。
- **Public Repositories：该仓库组将上述所有策略为Release的仓库聚合并通过一致的地址提供服务。**
- Public Snapshot Repositories：该仓库组将上述所有策略为Snapshot的仓库聚合并通过一致的地址提供服务。

![图片地址](http://img.blog.csdn.net/20170521190802328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTF9TYWls/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 7.2 配置Nexus仓库

1. 在POM中配置仓库
```xml
	    <project>  
		  <repositories>  
		    <repository>  
		      <id>maven-net-cn</id>  
		      <name>Maven China Mirror</name>  
		      <url>http://maven.net.cn/content/groups/public/</url>  
		      <releases>  
		        <enabled>true</enabled>  
		      </releases>  
		      <snapshots>  
		        <enabled>false</enabled>  
		      </snapshots>  
		    </repository>  
		  </repositories>  
		  <pluginRepositories>  
		    <pluginRepository>  
		      <id>maven-net-cn</id>  
		      <name>Maven China Mirror</name>  
		      <url>http://maven.net.cn/content/groups/public/</url>  
		      <releases>  
		        <enabled>true</enabled>  
		      </releases>  
		      <snapshots>  
		        <enabled>false</enabled>  
		      </snapshots>      
		    </pluginRepository>  
		  </pluginRepositories>  
		</project>  
```
这种配置方式只会对当前项目生效，如果想对所有项目生效，可以在settings文件中配置。

2. 在Settings.xml中配置
```xml
		<settings>  
		  ...  
		  <profiles>  
		    <profile>  
		      <id>Nexus</id>  
		      <!-- repositories and pluginRepositories here-->  
		    </profile>  
		  </profiles>  
		  <activeProfiles>  
		    <activeProfile>Nexus</activeProfile>  
		  </activeProfiles>  
		  ...  
		</settings>  
```
3. 使用镜像
如果你的地理位置附近有一个速度更快的central镜像，或者你想覆盖central仓库配置，或者你想为所有POM使用唯一的一个远程仓库（这个远程仓库代理的所有必要的其它仓库）
```xml
	    <settings>  
		...  
		  <mirrors>  
		    <mirror>  
		      <id>maven-net-cn</id>  
		      <name>Maven China Mirror</name>  
		      <url>http://maven.net.cn/content/groups/public/</url>  
		      <mirrorOf>*</mirrorOf>  
		    </mirror>  
		  </mirrors>  
		...  
		</settings>  
```
### 7.3 部署构建到Nexus
...

## 8. Maven测试
Maven的测试插件会自动执行src/test/java/下面的

- **/Test*.java；
- **/*Test.java;
- **/*TestCase.java

以上形式的类。

### 8.1 跳过测试

    mvn clean install -DskipTests

    <properties>
        <skipTests>true</skipTests>
    </properties>

### 8.2 指定运行某个测试用例

    mvn test -Dtest=Random1Test，Random2Test
    mvn test -Dtest=Random*Test

### 8.3 排除测试
```xml
    <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0-M3</version>
        <configuration>
          <includes>
           <include>**/*Tests.java</include>
          </includes> 
          <excludes>
            <exclude>**/TestCircle.java</exclude>
            <exclude>**/TestSquare.java</exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
    </build>
```
## 9. 持续集成
Hudson+Maven+git版本控制-->持续集成

## 10. 构建Web项目
使用Cargo可以进行远程Web应用部署...

## 11. Maven版本管理
Maven的版本管理一般会遵循：

    <主版本>.<次版本>.<增量版本>-<里程碑版本>

- 主版本号：表示项目架构有重大的变更;
- 次版本号：较大范围的功能增加和变化，以及Bug修复，但总体项目架构没什么变化；
- 增量版本：一般表示重大Bug修复，例如1.4.0发布后发现一个重大Bug，发现一个重大Bug修复后发布1.4.1；
- 里程碑版本：SNAPSHOT-->alpha-->beta-->release-->GA

   SNAPSHOT：正在开发中的版本
   Alpha: 内部测试的版本（项目组内部测试）
   Beta：用户下载下来测试使用
   Release: 用户使用下来没什么问题就可以发布Release版本
   GA：稳定版本


### 11.1 GPG签名
使用GPG签名来校验依赖包有没被第三方恶意篡改

## 12. 灵活的构建

### 12.1 Maven属性
Maven中有6中基本属性：

1. 内置属性
主要有两个常用内置属性——${basedir}表示项目根目录，即包含pom.xml文件的目录;${version}表示项目版本。

2. POM属性
pom中对应元素的值，例如${project.artifactId}对应了<project><artifactId>元素的值，常用的POM属性包括：
                                
- ${project.build.sourceDirectory}:项目的主源码目录，默认为src/main/java/；
- ${project.build.testSourceDirectory}:项目的测试源码目录，默认为/src/test/java/；
- ${project.build.directory}:项目构建输出目录，默认为target/；
- ${project.build.outputDirectory}:项目主代码编译输出目录，默认为target/classes/；
- ${project.build.testOutputDirectory}:项目测试代码编译输出目录，默认为target/testclasses/；
- ${project.groupId}:项目的groupId；
- ${project.artifactId}:项目的artifactId；
- ${project.version}:项目的version,于${version}等价；
- ${project.build.finalName}:项目打包输出文件的名称，默认为${project.artifactId}${project.version}.
    
3. 自定义属性                                                                                                                                                                                                                                                                                                                                                                                                                                   
在pom中<properties>元素下自定义的Maven属性。

4. Settings属性
与POM属性同理。如${settings.localRepository}指向用户本地仓库的地址。

5. Java系统属性
所有Java系统属性都可以使用Maven属性引用，例如${user.home}指向了用户目录。可以通过命令行mvn help:system查看所有的Java系统属性

6. 环境变量属性
所有环境变量都可以使用以env.开头的Maven属性引用。例如${env.JAVA_HOME}指代了JAVA_HOME环境变量的值。也可以通过命令行mvn help:system查看所有环境变量。

### 12.2 资源过滤
```xml
    <build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
				<excludes>
					<exclude>**/config/*.*</exclude>
				</excludes>
			</resource>
			<resource>
				<directory>src/main/resources/config</directory>
				<filtering>true</filtering>
				<includes>
					<include>application-${profiles.active}.yml</include>
				</includes>
			</resource>
		</resources>
    </build>
```
### 12.3 Maven profile

#### 12.3.1 针对不同环境profile
```xml
    <project>
        ...
	    <profiles>
	        <profile>
	            <id>dev</id>
	            <properties>
	                <profiles.active>dev</profiles.active>
	            </properties>
	            <activation>
                    <!-- 这边可以配置各种激活条件 -- >
	                <property>
	                    <name></name>
	                    <value></value>
	                </property>
	                <file>
	                    <exists></exists>
	                </file>
	                <os>
	                    ...
	                </os>
	                <jdk>
	                    ...
	                </jdk>
	                <activeByDefault>true</activeByDefault>
                </activation>
	        </profile>
	        <profile>
	            <id>stg</id>
	            <properties>
	                <profiles.active>stg</profiles.active>
	            </properties>
	        </profile>
	        <profile>
	            <id>prod</id>
	            <properties>
	                <profiles.active>prod</profiles.active>
	            </properties>
	        </profile>
        </profiles>
        ...
    </project>
```
#### 12.3.2 激活profile
1. 命令行激活

    mvn clean install -Pdev

2. settings文件显示激活
如果你希望某个profile一直处于激活状态，可以直接在settings文件中配置:
```xml
      <profiles>
	    <profile>
	      <id>nexus</id>
	      <repositories>
	          <repository>
	          <id>central</id>
	          <url>http://central</url>
	          <releases><enabled>true</enabled></releases>
	          <snapshots><enabled>true</enabled></snapshots>
	         </repository>
	      </repositories>
	      <pluginRepositories>
	        <pluginRepository>
	          <id>central</id>
	          <url>http://central</url>
	          <releases><enabled>true</enabled></releases>
	          <snapshots><enabled>true</enabled></snapshots>
	        </pluginRepository>
	      </pluginRepositories>
	    </profile>
	  </profiles>
	  <activeProfiles>
	    <!--make the profile active all the time -->
	    <activeProfile>nexus</activeProfile>
	  </activeProfiles>
```
#### 12.3.3 Web项目打包
...

## 13. 项目站点生成
...后续完成...

## 14. 编写Mavem插件
...后续完成...

## 附录
### A. 常用命令
下面我们总结下Maven中常用的命令。

- mvn help:system 调用help这个插件，打印出jvm的系统属性和操作系统的环境变量
- mvn help:describe -Dplugin=org.apache.maven.plugins:maven-dependency-plugin -Ddetail
- mvn archetype：generate 这个命令可以帮助我们生成一个Maven项目的骨架   
```
     |--Pom.xml  
       |--src  
           |--main
                 |--java
                 |--resource
           |--test
                 |--java
                 |--resource
```
- mvn dependency:list
- mvn denpedency:tree
- mvn dependency:analyze 可以分析当前项目依赖的问题
- mvn clean install -DskipTests
- mvn clean deploy 编译打包并发布到远程仓库
- mvn clean install site


### B. POM文件总结

            元素名称                             简   介
          <project>                         POM的xml根元素
          <parent>                          声明继承
          <modules>                         声明聚合
          <groupId>                         坐标元素之一
          <artifactId>                      坐标元素之一
          <version>                         坐标元素之一
          <packaging>                       坐标元素之一，默认值jar
          <name>                            名称
          <description>                     描述
          <organization>                    所属组织
          <licenses><license>               许可证
          <mailingLists><mailingList>       邮件列表
          <developers><developer>           开发者
          <contributors><contributor>       贡献者
          <issueManagement>                 问题追踪系统
          <ciManagement>                    持续集成系统
          <scm>                             版本控制系统
          <prerequisites><maven>            要求Maven最低版本，默认值为2.0
          <build><sourceDirectory>          主源码目录
          <build><scriptSourceDirectory>    脚本源码目录
          <build><testSourceDirectory>      测试源码目录
          <build><outputDirectory>          主源码输出目录
          <build><testOutputDirectory>      测试源码输出目录
          <build><resources><resource>      主资源目录
          <build><testResources><testResource> 测试资源目录
          <build><finalName>                输出主构件的名称
          <build><directory>                输出目录
          <build><filters><filter>          通过properties文件定义资源过滤属性
          <build><extensions><extension>    扩展Maven的核心
          <build><pluginManagement>         插件管理
          <build><plugins><plugin>          插件
          <profiles><profile>               POM Profile
          <distributionManagement><repository>  发布版本部署仓库       
          <distributionManagement> <snapshotRepository> 快照版本部署仓库       
          <distributionManagement> <site>   站点部署
          <repositories><repository>        仓库
          <pluginRepositories><pluginRepository>   插件仓库
          <dependencies><dependency>        依赖                      
          <dependencyManagement>            依赖管理
          <properties>                      Maven属性
          <reporting><plugins>              报告插件

### C. settings文件例子

    <settings>
    <localRepository>D:\software\maven\Repository</localRepository>
    <proxies>
    <proxy>
        <id>optional</id>
        <active>true</active>
        <protocol>http</protocol>
        <host>10.47.22.247</host>
        <port>80</port>
        <nonProxyHosts>some.host.com</nonProxyHosts>
    </proxy>
    </proxies>
    <servers>
    <server>
        <id>releases</id>
        <username>pub_mvn_deploy</username>
        <password>mvn.Deploy</password>
    </server>
    <server>
        <id>snapshots</id>
        <username>pub_mvn_deploy</username>
        <password>mvn.Deploy</password>
    </server>
    <server>
        <id>nexus</id>
        <username>pub_mvn_deploy</username>
        <password>mvn.Deploy</password>
    </server>
    </servers>

    <mirrors>
    <mirror>
        <id>nexus</id>
        <mirrorOf>*</mirrorOf>
        <url>http://maven.dev.xxx.com/nexus/content/groups/public</url>
    </mirror>
    </mirrors>

    <profiles>
    <profile>
        <id>nexus</id>
        <repositories>
            <repository>
                <id>central</id>
                <url>http://central</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>central</id>
                <url>http://central</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
            </pluginRepository>
        </pluginRepositories>
    </profile>
    <!-- if you want to be able to switch to the defaultprofile profile put this in the active profile -->
    <profile>
        <id>defaultprofile</id>
        <repositories>
            <repository>
                <id>maven.default</id>
                <name>default maven repository</name>
                <url>http://repo1.maven.org/maven2</url>
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
            </repository>
            <repository>
                <id>maven.snapshot</id>
                <name>Maven snapshot repository</name>
                <url>http://people.apache.org/repo/m2-snapshot-repository</url>
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
            </repository>
        </repositories>
    </profile>
    </profiles>
    <activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
    </activeProfiles>
    </settings>

### D. settings文件元素参考表

    1，<settings>，settings.xml文档的根元素

	2，<localRepository>，本地仓库
	
	3，<interactiveMode>，Maven是否与用户交互，默认值true
	
	4，<offline>，离线模式，默认值false
	
	5，<pluginGroups>  <pluginGroup>，插件组
	
	6，<servers>  <server>，下载与部署仓库的认证信息
	
	7，<mirrors>  <mirror>，仓库镜像
	
	8，<proxies>  <proxy>，代理
	
	9，<profiles>  <profile>，Settings Profile
	
	10，<activeProfiles>  <activeProfile>，激活Profile

### E. 常用插件总结

    插件名称	                用途	                来源
	maven-clean-plugin	   清理项目	          Apache
	maven-compiler-plugin	编译项目	          Apache
	maven-deploy-plugin	    部署项目	          Apache
	maven-install-plugin	    安装项目	          Apache
	maven-resources-plugin	处理资源文件	      Apache
	maven-site-plugin	    生成站点	          Apache
	maven-surefire-plugin	执行测试	          Apache
	maven-jar-plugin	        构建jar项目	      Apache
	maven-war-plugin     	构建war项目	      Apache
	maven-shade-plugin	构建包含依赖的jar包	  Apache
	maven-changelog-plugin	生成版本控制变更报告	Apache
	maven-checkstyle-plugin	生成CheckStyle报告   Apache
	maven-javadoc-plugin  	生成javadoc文档	   Apache
	maven-jxr-plugin	      生成源码交叉引用文档	   Apache
	maven-pmd-plugin	      生成pmd报告	       Apache
	maven-project-info-reports-plugin	生成项目信息报告	Apache
	maven-surefire-report-plugin	 生成单元测试报告	Apache
	maven-antrun-plugin	    调用Ant任务	        Apache
	maven-archetype-plugin	基于Archetype生成项目骨架	Apache
	maven-assembly-plugin	构建自定义格式的分发包	    Apache
	maven-dependency-plugin	依赖分析及控制	        Apache
	maven-enforcer-plugin	定义规则并强制要求项目遵守	Apache
	maven-pgp-plugin	      为项目构件生成pgp签名	    Apache
	maven-help-plugin	  获取项目及Maven环境的信息	Apache
	maven-invoker-plugin	  自动运行Maven项目构建并验证	Apache
	maven-release-plugin  自动化项目版本发布	        Apache
	maven-scm-plugin	      集成版本控制系统	       Apache
	maven-source-plugin	  生成源码包	               Apache
	maven-eclipse-plugin  生成eclipse项目环境配置	  Apache
	build-helper-maven-plugin 包含各种支持构建生命周期的目标 Codehaus
	exec-maven-plugin	  运行系统程序或者java程序  	Codehaus
	jboss-maven-plugin	 启动、停止jboss、部署项目  	Codehaus
	properties-maven-plugin	从properties文件读写Maven属性	Codehaus
	sql-maven-plugin	     运行sql脚本	                Codehaus
	tomcat-maven-plugin	 启动、停止tomcat、部署项目	Codehaus
	versions-maven-plugin	自动化批量更新pom版本	   Codehaus
	cargo-maven-plugin	 启动/停止/配置各类web容器自动化部署web项目	Cargo
	jetty-maven-plugin	集成jetty容器、实现快速开发测试     	Eclipse
	mave-gae-plugin	    集成google app engine	    Googlecode
	maven-license-plugin	 自动化添加许可证证明至源码文件	Googlecode
	maven-andmid-plugin	 构建Android项目	            Googlecode

### F. 超级POM
这个POM会被每个POM继承。它的定义在model-builder.jar这个包下面。我们解压就可以看到。 


### G. 图片工具
http://techsmith.com/jing这个网站提供了一个不错的制图工具，有时间可以研究下。

### H. 参考书籍

1. 《Maven实战》
2. 《Maven权威指南》

### I. Maven和gradle的对比
- https://www.cnblogs.com/huang0925/p/5209563.html
- http://blog.csdn.net/xad707348125/article/details/46891941
- https://maven.apache.org/settings.html (Maven官网)
