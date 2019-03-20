

基于jmeter-3.0

# 引入和使用Jmeter的原因

## 特点：

1. 开源、轻量级、插件化
2. 适合自动化和持续集成

## 使用场景

1. 性能测试

2. 接口测试 

   http tcp xmpp

3. 造数据

4. 线上巡检

5. 构建部署

   原理：编译打包war包、tar包；把包拷贝到对应的目录；解压，启动服务

   jmeter有一个sh插件，装上之后，可以执行shell命令。

6. 重启服务

   shell命令

7. 清理环境

   脏数据的处理

8. 封装成性能测试平台

   平台，底层调用Jmeter，中间用Jenkins结合起来

   也可以直接变化成自动化测试平台。开发2个插件分别管理脚本，管理调度任务，就是一个轻量级的平台。Jmeter里面的代码可以和MySQL数据库直接打通，保存报告。

   



## 原理和目录解析

Jmeter不管是性能测试还是接口测试，都是封装一个请求，发送，接受返回结果。

### JMeter的工作目录

#### bin：放置各项配置文件（如日志设置、JVM设置）、启动文件、启动Jar包、示例脚本等；

**.properties **

- 解决中文乱码的途径之一，设置utf-8

- 把一个设置设为true，就可以把请求头、请求体，保存在结果里

- 脚本格式，可以设置各种格式 jtl csv xml

- 第三方/自己开发的jar包，放的路径有两种方式，

  1）放在lib文件夹，jmeter启动时自动加载

  2）放到自己建的文件夹，jar包的加载情况在properties中配置

- 多个负载机时，远程服务器的配置

- 日志log配置

**jmeter.sh**  

jmeter并发数一半的时候，jmeter卡死。可以修改如下配置

```shell
##   ==============================================
##   Environment variables:
##   JVM_ARGS - optional java args, e.g. -Dprop=val
##
##   e.g.
##   JVM_ARGS="-Xms512m -Xmx512m" jmeter.sh etc.
##
##   ==============================================
```

#### docs：放置JMeter API的离线帮助文档；

jmeter本身的api

和其他工具做集成，

把jmeter封装成一个平台，调用jmeter中的api

#### extras：JMeter辅助功能，提供与Ant、Jenkins提成的可能性，用来构建性能测试自动化框架；

巡检邮件报告。基于jmeter-results-detail-report_21.xsl 和 jmeter-results-report_21.xsl进行改造  

#### lib：JMeter组件以Jar包的形式放置在lib/ext目录下，如果要扩展JMeter组件，Jar包就放在此目录下，JMeter启动时会加载此目录下的Jar包；

#### printable_docs：放置JMeter的离线帮助文件，可用来学习JMeter；

#### backups：操作过程中自动保存的一些脚本



## 学习掌握jmeter的建议

### 现状

1. jmeter在升级，中文好的资料不多

2. 初级的多，深入的少
3. 没有知识体系，花时间，事倍功半
4. 好高骛远，行动的矮子

### 学习什么

1. http协议，json，html，cookie，session，token
2. jmeter中元件的含义、结构、作用顺序（常用的几个）
3. jmeter脚本的4种方式（手动、badboy、jmeter自带、fiddler转换）
4. 接口测试的原理
5. 性能基础概念和原理
6. 监控相关知识，原理和常用工具
7. java、shell编程

### 怎么学

在项目中使用，解决问题

不懂就搜索，就看英文资料

拜师学艺

jmeter白皮书，基础入门、常用插件、实践





