# JMeter02-JMeter内部原理

在使用JMeter这款工具之前，首先要了解JMeter是如何工作的，也就是JMeter的内部原理，理解了原理，我们才能更好地使用工具。

### 1、JMeter的体系结构

 JMeter的结构图如下所示。这是一个三维空间，三维坐标轴分别是X、Y、Z。这张图来源于网络，在很多讲JMeter原理的文章中都可以看到，但是理解起来比较抽象，很难简单明了地看懂，我在此尝试描述一下。

![img](JMeter02.assets/auto-orient.jpeg)

X维度实际上是描述的是不同的组件，这些组件是独立的个体，我们依靠这些组件完成性能测试中负载的模拟，这些组件分别是：

- X1 采样器（Sampler）：采样器用来向服务器发送HTTP请求（Request），并接受服务器的响应数据（Response）；JMeter的采样器有很多种元件（采样器称为组件，其中的具体一种采样器如HTTP采样器称为元件），基本涵盖了常见的协议，如HTTP、FTP、JAVA、JMS、LDAP、MAIL、MongoDB、SMTP、SOAP、TCP、JUnit等；
- X1 断言（Assertions）：断言用来验证结果是否正确，即判断请求是否成功、返回数据是否符合要求等，通过预设的结果和实际返回的结果进行比较，匹配到了就说明断言成功，JMeter的断言组件也有多种元件，如响应断言、XML断言、BeanShell断言；
   -X1 监听器（Listener）：监听器用来收集测试结果，监听器有两个任务，即添加结果监听和展示结果，监听器组件也有多种元件；

> 采样器+断言+监听器，组合在一起就可以完成发送请求、验证结果、记录结果这样的工作；

- X2 前置处理器（Pre Processors）：前置处理器用来完成请求发送前的一些环境或者参数的准备工作，比如操作数据库前需要建立一个数据库连接；
- X2 配置原件（Config Element）：配置原件用来为采样器提供预备数据，比如做参数化，生成动态数据，或从文件中读取测试数据等；
- X2 后置处理器（Post Processors）：后置处理器用来处理服务器响应中的数据，比如获取登录后的session_id，后置处理器一般放在采样器之后；

> 前置处理器+配置元件+后置处理器，都是为采样器提供数据支持的，而采样器关注的是业务逻辑。

- X3 控制器（Logic Controller）：控制器用来控制采样器的执行逻辑，如执行顺序、执行次数等，各种控制器组合在一起，也能完成各种奇葩等需求；
- X4 定时器（Timer）：定时器用来完成集合的功能， 比如为了足够真实地模拟用户负载，会需要模拟这些请求在同一时刻发送，定时器也有很多种类的元件；
- X5 线程组（Threads）：线程组用来模拟大量用户负载的情况，可以设置运行的线程数（一个线程一个用户）、运行时长、定时运行等；

Y维度实际上是对性能测试进行了一个划分，Y1是负责模拟用户请求的部分，Y2是负责验证结果正确的部分，因为这两部分同时需要线程组，所以是相交的。

Z维度只有一个监听器（Listener），监听器用来负责结果的收集，监听器不仅可以放在线程组之内，也可以放在线程组之外，所以监听器与它们是相交的。

> 上述的这些组件是JMeter的主要组成部分，除此之外，还有其他的一些组件，比如Test Fragment，这是一个辅助的组件，在此节点下可以放置任何JMeter测试元件，但它一般不会被运行，它的作用主要有两个：在脚本开发过程中备份元件、利用Test Fragment来模块化请求，供模块控制器调用。

### 2、JMeter运行原理

JMeter是以线程的方式运行的，通过线程组来驱动多线程运行测试脚本，对被测服务器发起负载，每一个负载机上都可以运行多个线程组。

JMeter的场景运行，基于操作方式，可以分为两种：

- GUI：图形用户界面运行模式，可视化，更加直观，使用鼠标操作，方便实时查看运行状态，如测试结果、运行线程数等；
- 非GUI：命令行模式，对负载机的资源消耗更小，GUI模式会影响负载量的生成，比如非GUI模式100个线程可以产生100TPS的负载，而GUI模式只能产生80TPS的负载；

> 所以每次启动JMeter的时候，你都会看到这个提示“Don't use GUI mode for load testing !”。

基于运行架构，也可以分为两种，即（本地化运行或称单机运行）、远程运行，不论是GUI模式还是非GUI模式，都支持本地运行与远程运行。

- 本地运行：只运行一台JMeter机器，所有的请求从一台机器发出；
- 远程运行：一台JMeter控制机（Master）控制远程的多台机器（Slave）来产生负载；

> - 负载机：向被测服务器发起负载请求的机器，与其他支持远程运行的测试工具一样，负载机受控制机管理时首先启动一个客户端Agent程序，控制机才可以接管负载机；
> - 控制机：控制机也是一台负载机，只不过是多台负载机中被选中作为管理机的那台机器，所以控制机也可以参与脚本的运行，同时担负着管理和指挥远程的负载机运行的任务，并且收集远程的负载机的测试结果；

远程运行的逻辑是：

- 远程机首先启动Agent程序（运行jmeter-server.bat）；
- 控制机连接上远程负载机（修改配置文件，会自动探测并连接）；
- 控制机发送指令（脚本及启动命令）启动线程（参数化文件或依赖包需要手工拷贝到远程负载机）；
- 负载机运行脚本，回传状态（包括测试结果）；
- 控制机收集结果并显示；

### 3、JMeter测试计划的要素

JMeter中一个脚本，就是一个测试计划，也是一个管理单元。JMeter的请求模拟、并发数（即设置的线程数，一个线程代表一个虚拟用户）的设置都在脚本文件中一起设置。

> LoadRunner中，脚本与虚拟用户的设置是分开的。

- 要素一：脚本中的测试计划只能有一个
- 要素二：测试计划中至少要有一个线程组
- 要素三：至少要有一个采样器
- 要素四：至少要有一个监听器

至于JMeter中的其他元件，都是为这些要素服务的。

### 4、JMeter的工作目录

学习完JMeter的基础原理后，我们再来看下JMeter的工作目录，更好地了解JMeter。JMeter的工作目录下主要有以下的文件夹/文件：

- bin：放置各项配置文件（如日志设置、JVM设置）、启动文件、启动Jar包、示例脚本等；
- docs：放置JMeter API的离线帮助文档；
- extras：JMeter辅助功能，提供与Ant、Jenkins提成的可能性，用来构建性能测试自动化框架；
- lib：JMeter组件以Jar包的形式放置在lib/ext目录下，如果要扩展JMeter组件，Jar包就放在此目录下，JMeter启动时会加载此目录下的Jar包；
- printable_docs：放置JMeter的离线帮助文件，可用来学习JMeter；

### 5、小结

本篇文章框架性地介绍了JMeter的组成及运行原理，实际上与大多数的性能测试工具原理上是相似的，比较理论化，虽然这些理论比较枯燥，但这是后续学习JMeter的一个基础，有了理论，才能更好地实践。

JMeter的运行逻辑主要是：

- 采样器模拟用户请求；可使用前置管理器做环境及数据的准备；可使用后置处理器做响应数据的处理；
- 控制运行：使用线程组来设置运行场景，利用逻辑控制器来控制业务（控制取样器）；
- 收集结果：利用断言来验证测试结果，利用监听器来收集显示测试结果；

同时JMeter也支持远程运行，弥补单台机器负载不够的情况。
