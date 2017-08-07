---
title: Eclipse EE Neon 使用说明
date: 2016-09-20 22:5
tags: [eclispe, 工具]
---

## 下载 `Eclipse`
* eclipse 是开源免费软件，可到[eclipse 官网](https://eclipse.org/) 处获取到下载链接地址。
* eclipse 有两种方式下载，直接下载独立安装的ZIP程序包，或者使用 `Eclipse Installer` 进行在线安装.
* eclipse 下载页面地址：  
[https://eclipse.org/downloads/eclipse-packages/](https://eclipse.org/downloads/eclipse-packages/)  
![Eclipse 下载页面](./image/下载Eclipse.png)  

## Eclipse 安装、启动，注意事项

eclipse 是免安装程序,直接将zip包解压即可，然后双击 `eclipse.exe` 即可使用

>需要注意的是 `eclipse 4.6` 版本需要 JDK 1.8。若系统环境变量 `JAVA_HOME`有配置指定到jdk1.8，那么eclipse直接可用。 <br/>

另外，也可以在 `eclipse.ini` 文件中配置eclispe 所使用的jdk,可在 `--launcher.appendVmargs` 之后 `-vmargs`之前增加如下参数，参数值为安装的jdk bin目录路径

```ini
-vm
C:/Program Files/Java/jdk1.8.0_101/bin
```
* Eclispe 工作区  
![Eclispe 工作区](./image/workspace Java EE - Eclipse.png)

## Eclispe 插件安装

### 通过 `Eclipse Marketplace` 在线安装插件

1. 在eclipse菜单栏点击 `Help > Eclipse Marketplace...`  
![安装插件-01](./image/安装插件-01.PNG)
2. 在`Find`输入框输入插件名称搜索插件  
![安装插件-02](./image/Eclipse-Eclipse Marketplace.PNG)
3 点击 `Install` 安装。

### 使用 `update sit url` 在线安装插件

1. 在 `eclipse` 菜单栏，点击 `Help > Install New Software`。
2. 点击 `Add` ，弹出 `Add Repository` 对话框。
3. 在 `name` 输入框里输入名称, 在 `location` 输入框里输入插件安装`Update site URL` 。
4. 点击`OK`。

![](./image/Install New Software.png)

### 离线安装

1. 将插件下载解压，删除`features`与`plugins` 之外的所有文件，然后将此两文件夹放入一个新建的名为`eclispe` 的文件夹里，然后将这个eclipse文件夹放入以插件名为名的文件夹。然后进行下一步。
2. 将插件文件放置到 `eclipse\dropins\`目录下，*`最终第一步解压的两个插件文件将会位于形如{eclipse根目录}\dropins\Subclipse-1.10.13-1.9.x\eclipse`的路径下* 重启eclipse即可

![](./image/离线安装插件-1.PNG)
![](./image/离线安装插件-2.PNG)

## Eclipse常用插件列表

* [Spring Tool Suite (STS) 4.6](http://dist.springsource.com/release/TOOLS/update/e4.6/)
  *开发Spring 相关功能*  
* MyBatispse  
  *MyBatis 开发插件*  
* Mybatis Generator  
  *为eclipse提供自动生成MyBatis相关代码的功能*
* Buildship Gradle Integration  
  *集成 `Gradle`*
* Subclipse  
  *`Subversion`插件*  
  **
* [JBoss Tools 4.4.2.Final](http://download.jboss.org/jbosstools/neon/stable/updates/)
  *提供集成JBoss Application Server，Hibernate, 查看与编辑 `properties`、`xml` 、`web.xml`文件，等功能*  
  *[离线安装包下载地址](http://download.jboss.org/jbosstools/static/neon/development/updates/core/)*
* [Activiti BPMN 2.0 designer](http://activiti.org/designer/update/)  
  *`Activiti`工作流引擎插件*  
* [Thymeleaf Eclipse Plugin repository](http://www.thymeleaf.org/eclipse-plugin-update-site/)
  *`Thymeleaf`模板引擎*  

> 其中 `Spring Suite Tool`,`MyBatispse`,`Mybatis Generator`,`Buildship Gradle Integration`,`Subclipse`,`JBoss Tools`,亦可直接从`Eclipse Marketplace`安装。

## 插件使用
### Subclipse 使用

* svn ignore 设置排除版本控制的文件

![svn-ignore](./image/svn-ignore.png)

## 导入 Maven 项目

1. 如下所示点击"File-Import"<br/>
![导入](./image/01-导入Maven项目.PNG)

2. 选择导入Existing Maven Projects<br/>
![选择导入Existing Maven Projects](./image/02-选择导入Existing Maven Projects.PNG)

3. 选择源码路径,然后点击`Finish`<br/>
![选择源码路径](./image/03-选择源码路径.PNG)

## Eclipse 设置Build Path
![Alt text](./image/Eclipse EE - Java Build Path.PNG)

## 问题解决

* **JPA project Change Event Handler (Wating)**

这是Eclipse中的一个GUG：

[Bug 386171 - JPA Java Change Event Handler (Waiting)](https://bugs.eclipse.org/bugs/show_bug.cgi?id=386171)

解决方法：

1.) 退出Myeclipse（或eclipse）；

2.) 进入Myeclipse（或eclipse）的安装目录；

* linux中：

```shell
mkdir disabled
mkdir disabled/features disabled/plugins

mv plugins/org.eclipse.jpt.* disabled/plugins
mv features/org.eclipse.jpt.* disabled/features
```

* windows 中:

创建名为disabled的文件夹；

在disabled文件夹下创建两个文件夹，名字分别为features 、plugins；

将plugins目录下，以org.eclipse.jpt开头的jar文件剪切到disabled\plugins目录下；

将features目录下，以org.eclipse.jpt开头的j文件夹剪切到disabled\features目录下；

重新Myeclipse（或eclipse）；

重启后第一次会提醒你重新配置content-assist；

可以执行以下命令替代以上操作：

```shell
mkdir disabled
mkdir disabled\features
mkdir disabled\plugins

move plugins\org.eclipse.jpt.* disabled\plugins
move features\org.eclipse.jpt.* disabled\features
```


卸载完DALI/JPT的eclipse插件后，就再也不会出现UI卡顿与保存文件的时候需要等待几秒的问题了。


## 普通 Java 项目转换为 Maven 项目

> 本文提供一个完整可行的将遗留项目转换为Maven项目的步骤。至于转换Maven构建项目的好处不在本文涉及范围之内，希望了解的读者可以使用`持续集成、自动化测试 与 Maven`等进行搜索

### 使用工具

* Eclipse Neon (4.6)
* JBoss Tools 4.4.2.Final

### 转换步骤

1. 在Eclipse Marketplace 安装 `JBoss Tools 4.4.2.Final` 插件  
![安装 `JBoss Tools 4.4.2.Final` 插件](./image/01-安装JBoss Tools.png)
2. 选中待转换的项目，“右键 > Configure > Convert to Plug-in Projects”  
![](./image/02-选中项目，右键，Configure,Convert to Plug-in Projects.PNG)  
3. 在弹出框填写Maven项目的GAV信息和打包信息（Java Web项目使用war,Java项目使用jar)  
![](./image/03-填写Maven项目的GAV信息和项目打包类型.png)
4. 等待自动识别jar依赖，注意不要勾选“Delete original references from project”
![](./image/04-自动匹配jar包依赖.jpg)
5. 记录上一步里的匹配结果，已匹配的（绿色）和未匹配成功的（红叉）,建议使用截图将识别记录保存下来。
6. 回到项目文件夹，将自动识别成功的jar删除。接下来开始处理无法匹配的依赖包
7. 将剩余未成功匹配的jar包移动到另一个位置，新建名为test的项目，将这些jar加入该项目的`build path`。
8. 在test项目的“Referenced Libraries > ”之下依次点击jar左侧的箭头按钮查看jar的类信息  
![](./image/05-处理无法匹配的依赖包-1.png)
9. 打开jar的`META-INF`目录，这个时候有两种情况：由Maven打包的jar和不是的。Maven打包的jar处理起来很简单，`META-INF`下将可以看到名为`maven`文件夹打开它知道最里层会看到一个`pom.properites`的文件,这个文件里含有该jar的Maven依赖信息将该信息填入pom即可。示例如下：  
![](./image/07-Maven构建的jar-1.png)
10. 处理非Maven打包的同时也不在Maven中央仓库存在的依赖包，这样的依赖包一般是非开源的私有包。这是自己编织该jar的Maven GAV信息，将该信息填入pom，并将将jar发布到自己建立的代理仓库(*一般使用Nexus OSS搭建*)即可。
11. 回到被转换的项目，在Eclipse里按 “Alt+F5” 更新Maven即可看到构建成功。
12. 至于其他既不能在jar里找到`pom.properites`,看上去由不像是私有包，那么可以在`[search.maven.org](http://search.maven.org/#advancedsearch)`进行搜索.  
![](./image/8.其他.png)
