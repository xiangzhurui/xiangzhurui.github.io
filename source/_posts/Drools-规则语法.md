---
title: Drools Rule 规则语言结构
tags: [Drools,语法]
categories:
- 规则引擎
- Drools
date: 2018-03-07 16:15:57

---

## 规则语言结构

### 规则文件

在 Drools 当中，一个标准的规则文件就是一个以“.drl”结尾的文本文件，由于它是一个标准的文本文件，所以可以通过一些记事本工具对其进行打开、查看和编辑。

规则是放在规则文件当中的，一个规则文件可以存放多个规则，除此之外，在规则文件当中还可以存放用户自定义的函数、数据对象及自定义查询等相关在规则当中可能会用到的一些对象。常用的有
规则文件结构：
```
package package-name
imports
globals
functions
queries
rules
```
对于一个规则文件而言，首先声明package 是必须的，除package 之外，其它对象在规则文件中的顺序是任意的，也就是说在规则文件当中必须要有一个package 声明，同时package 声明必须要放在规则文件的第一行。在Drools 的规则文件当中package 对于规则文件中规则的管理只限于逻辑上的管理，，而不管其在物理上的位置如何，这点是规则与Java 文件的package 的区别。对于同一package 下的用户自定义函数、自定义的查询等，不管这些函数与查询是否在同一个规则文件里面，在规则里面是可以直接使用的，这点和Java 的同一package 里的Java类调用是一样的。
### Package：

与java的package类似，一个package代表了一组规则以及其他相关资源（imports、globals）的集合。同时package表示一个名称空间。对于业务或者作用相似的规则这个package应该是唯一的，并且这个package不与任何的物理文件夹或文件相关。
有时我们从多个package中加载规则，当存在相同的package路径时，则所有的规则都保存到这个顶级package路径下。建议的做法是将相关的规则都放到一个package路径下。

同时，任何的规则属性（比如salience,dialect）也可以写在包级别,这种写法为全局默认配置，相同package路径下的规则文件如果没有自定义，则默认使用全局配置。

### Import：
同Java的import，默认导入java.lang

### Global：
global定义全局变量，类似于java的public static 的成员变量。通常用于提供规则使用的数据或服务，以及将结果返回应用程序中，或者还可以添加一下服务对象（如Datasource，Logger等）与应用程序进行交互。

### rules
* 规则结构包括四部分
    - 规则名称 (rule name)
    - 属性部分（rule attribute）
    - 条件部分（LHS）
    - 结果部分（RHS

```JavaScript
rule "name"
attributes
    when
        LHS
    Then
        RHS
end
```
一个规则中规则名称是必须要有的，而另外三个部分都是可选的，也即是说如下所示的规则也是是合法的：
```JavaScript
rule "name"
    when
    then
end
```
#### 规则名称

规则必须有规则名称，并且在一个`package`中唯一这个`name`是唯一的。需要注意的是：
* 如果在一个DRL文件中定义相同的规则名称，则drools加载规则时报错；
* 如果规则引擎已存在相同的规则名称，则后加载的规则会覆盖原有规则；
* 如果规则名称中包含空格，则必须使用双引号。
