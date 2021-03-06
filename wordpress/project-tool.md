# 项目小工具

平时自己有写一些小项目，但是写小项目的过程中如果能有一些好用的工具来提升效率的话就最好了。比如自动化测试、自动发布、代码自动检查等功能。今天就来谈谈我自己在用或者接触过的一些工具。

## Git服务器

虽然全球最大同性交友网站github现在对私人的仓库也免费开放了，但是有些代码我们还是自己保存比较妥当，特别是近几年鹰酱老是搞些小动作，什么科学无国界、开源无国界都变扯淡了。所以搭建一个私人的git服务器是非常有必要的，下面就来谈谈常见的几种可以取代github的git服务器吧。

### GitLab

![gitlab](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-15-50-40.png)

官网：[https://about.gitlab.com/](https://about.gitlab.com/)

GitLab 是由 GitLab Inc.开发，一款基于 Git 的完全集成的软件开发平台。另外，GitLab 且具有wiki以及在线编辑、issue跟踪功能、CI/CD 等功能。可以说在这个领域是很强大了，类似全家桶的存在，连CICD、看板这些东西一套都给你上了。

优点：集成代码开发全家桶，包括持续集成、持续交付、任务看板、工单系统、pages等几乎所有功能，对中大型企业来说是很方便的

缺点：功能多的收费，只能使用社区版免费。吃配置，2G起跑、4G偶尔还500错误，要是不缺钱能堆配置就可以考虑。

### gogs

![gogs](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-15-59-11.png)

官网：[https://gogs.io/](https://gogs.io/)

开源地址：[https://github.com/gogs/gogs](https://github.com/gogs/gogs)

Gogs 的目标是打造一个最简单、最快速和最轻松的方式搭建自助 Git 服务。使用 Go 语言开发使得 Gogs 能够通过独立的二进制分发，并且支持 Go 语言支持的 所有平台，包括 Linux、Mac OS X、Windows 以及 ARM 平台。

优点：免费，轻量级，资源占用少，支持多平台，可以二进制部署也可以docker部署等。

缺点：功能太少，更新慢，听说是某个不太愿意接受社区贡献的大神写的，因此衍生出了后面的gitea。

### gitea

官网：[https://gitea.io/](https://gitea.io/)

开源地址：[https://github.com/go-gitea/gitea](https://github.com/go-gitea/gitea)

Gitea 是一个开源社区驱动的轻量级代码托管解决方案，后端采用 Go 编写，采用 MIT 许可证。他和GitHub, Bitbucket or Gitlab等比较类似。他是从 Gogs 发展而来，不过已经Fork并且命名为Gitea。Gitea是流行的自托管Git服务Gogs的社区分支，是由不断增长的前Gogs用户和贡献者组成的小组，他们发现Gogs的单一维护者管理模型令人沮丧，因此决定努力建立一个更加开放和更快的开发模型。

同Gogs一样，Gitea的首要目标是创建一个极易安装，运行非常快速，安装和使用体验良好的自建 Git 服务。采用Go作为后端语言，这使我们只要生成一个可执行程序即可。并且他还支持跨平台，支持 Linux, macOS 和 Windows 以及各种架构，除了x86，amd64，还包括 ARM 和 PowerPC。

优点：免费，轻量级，资源占用少，支持多平台，可以二进制部署也可以docker部署等。功能比Gogs多，相当于Gogs plus版本。推荐使用此版本，本人目前也在使用Gitea。

缺点：功能没有gitlab那么多，CI/CD的支持还是要自己搭建然后使用钩子连接，不过还好有很多CI/CD工具可用。

### gitbucket

![gitbucket](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-16-15-59.png)

官网：[https://gitbucket.github.io/](https://gitbucket.github.io/)

开源地址：[https://github.com/gitbucket/gitbucket](https://github.com/gitbucket/gitbucket)

GitBucket 是JVM上的开源Git平台由Scala提供支持的Git平台，具有易于安装，高度可扩展性和GitHub API兼容性的特点。只需下载并运行java -jar gitbucket.war。GitBucket是开源的，可在遵循Apache License Version 2.0协议下使用。

由于我没使用过这个，所以暂时不清楚优缺点，网上知道的好像也比较少，有使用过的小伙伴可以讨论一下。

## 持续集成/持续交付

首先我们要了解持续集成\(CI:Continuous Integration\)/持续交付\(CD:Continuous Delivery\)是什么东西。持续集成与持续交付是软件开发和交付中的实践。其实就是一个自动化的过程，通过配置一些信息，让系统自动帮我们完成代码质量检查、单元测试、基本BUG查找、编译新版本、自动发布等等。简化工作量的同时又规范了开发流程，可以帮助我们做出更好的产品。

持续集成：软件开发中，集成是一个很可能发生未知错误的过程。持续集成是一种软件开发实践，希望团队中的成员频繁提交代码到代码仓库，且每次提交都能通过自动化测试进行验证，从而使问题尽早暴露和解决。持续集成无法消除bug，但却能大大降低修复的难度和时间。

持续交付：持续交付指的是，频繁地将软件的新版本，交付给质量团队或者用户，以供评审。如果评审通过，代码就进入生产阶段。持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。这里需要注意的是，CD代表持续交付（Continuous Delivery）而不是持续部署（Continuous Deploy），因为部署也包括部署到测试环境，而持续交付代表的是功能的上线，交付给用户使用。

持续部署：持续交付的下一步，指的是代码通过评审以后，自动部署到生产环境。持续部署的目标是，代码在任何时刻都是可部署的，可以进入生产阶段。持续部署的前提是能自动化完成测试、构建、部署等步骤。

### Gitlab

![gitlab](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-16-55-09.png)

官网：[https://docs.gitlab.com/ee/ci/](https://docs.gitlab.com/ee/ci/)

GitLab CI / CD是内置在GitLab中的功能强大的工具，它使您可以将所有连续方法（连续集成，交付和部署）应用于软件，而无需第三方应用程序或集成。将.gitlab-ci.yml配置文件添加到存储库后，GitLab将检测到该文件并使用名为GitLab Runner的工具运行脚本，该工具的工作原理与终端类似。

优点：如果使用gitlab做代码管理的话，那这就是最佳工具了，毕竟内部集成，整体上品牌机用起来总是比组装机舒服的。 缺点：和上面gitlab一样，耗资源，以前在公司的时候就经历过一次中午的时候公司运维停服加内存的情况。所以，如果不缺钱的话那gitlab全家桶绝对是最佳选择。

### jenkins

![jenkins](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-17-03-55.png)

官网：[https://www.jenkins.io/](https://www.jenkins.io/)

开源地址：[https://github.com/jenkinsci/jenkins](https://github.com/jenkinsci/jenkins)

Jenkins是一款由Java编写的开源的持续集成工具。在与Oracle发生争执后，项目从Hudson项目复刻。Jenkins提供了软件开发的持续集成服务。它运行在Servlet容器中（例如Apache Tomcat）。它支持软件配置管理（SCM）工具（包括AccuRev SCM、CVS、Subversion、Git、Perforce、Clearcase和RTC），可以执行基于Apache Ant和Apache Maven的项目，以及任意的Shell脚本和Windows批处理命令。Jenkins的主要开发者是川口耕介。Jenkins是在MIT许可证下发布的自由软件。可以通过各种手段触发构建。例如提交给版本控制系统时被触发，也可以通过类似Cron的机制调度，也可以在其他的构建已经完成时，还可以通过一个特定的URL进行请求。

优点：开源免费、跨平台、插件多，一般和Java项目有较好的配合。

缺点：上古时代的页面UI，极其丑陋，配置项较多，好像不支持文件配置。

### Drone

![Drone](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-17-11-03.png)

官网：[https://drone.io/](https://drone.io/)

开源地址：[https://github.com/drone](https://github.com/drone)

Drone是现代化的持续集成和持续交付平台，可让繁忙的团队自动化其构建，测试和发布工作流程。利用Drone的团队可以更频繁地发布软件，并且漏洞更少。Drone的0.8版本和1.0版本差距很大，包括界面、语法等都有很大差异，尽量使用新版。

优点：开源免费，现代化界面，支持ssh、docker、k8s等多种cicd工作方式。可以和github、gitea等多种git服务器进行集成。并且支持插件模式，官网提供了一些常用的服务插件，可以很好的进行CI/CD工作。

缺点：除了常用的CI/CD功能之外，附件功能太少，连成员授权都是基于git的。

### flow.ci

![flow.ci](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-17-18-18.png)

官网：[https://flow.ci/](https://flow.ci/)

开源地址：[https://github.com/flowci](https://github.com/flowci)

flow.ci 使用 Java 语言编写，开源跨平台，是国内目前在做的开源CI/CD。我没用过，不过看界面好像也挺好的。之前停更过一段时间，目前又开启了更新，继续观察吧。

### gocd

官网：[https://www.gocd.org/](https://www.gocd.org/)

开源地址：[https://github.com/gocd/gocd](https://github.com/gocd/gocd)

GoCD，一款先进的持续集成和发布管理系统,由ThoughtWorks开发。其前身为CruiseControl,是ThoughtWorks在做咨询和交付交付项目时自己开发的一款开源的持续集成工具。后来随着持续集成及持续部署的火热，ThoughtWorks专门成立了一个项目组，基于Cruise开发除了Go这款工具。ThoughtWorks开源持续交付工具Go。使用Go来建立起一个项目的持续部署pipeline是非常快的，非常方便。

优点：开源免费，使用PipeLineGroup，PipeLine，Stage，Job，Task 分级分层控制任务粒度和关联性、强大的用户，角色系统、go-server &lt;–&gt; go-agent 通信和管理模式、除了JRE 1.6+ 以外不依赖其它组件，对系统的冲击很小，方便部署

缺点：不支持一个PipeLine、Job在多个Agent上依次执行（对于大规模集群式部署的应用来说，这简直要命）、插件比较稀少、开源时间短，用户群还比较小。

## 团队协作

### YApi

![YApi](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-17-49-58.png)

官网：[https://hellosean1025.github.io/yapi/](https://hellosean1025.github.io/yapi/)

开源地址：[https://github.com/ymfe/yapi](https://github.com/ymfe/yapi)

去哪儿网开发的Api管理系统，旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API。在前后端分离的项目中，可以方便前端查看接口、mock数据，也有利于前后端协作。

### YDoc

![YDoc](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-17-52-48.png)

官网：[https://hellosean1025.github.io/ydoc/](https://hellosean1025.github.io/ydoc/)

开源地址：[https://github.com/ymfe/ydoc](https://github.com/ymfe/ydoc)

同YApi一样，是去哪儿网开发的文档管理系统，基于 markdown 轻松生成完整静态站点。

### Showdoc

![Showdoc](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-17-54-29.png)

官网：[https://www.showdoc.cc/](https://www.showdoc.cc/)

开源地址：[https://github.com/star7th/showdoc](https://github.com/star7th/showdoc)

ShowDoc也是一款适合IT团队在线共享文档的工具。它可以提高团队成员之间的沟通效率。

### pearProject

![pearProject](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-17-56-38.png)

官网：[https://home.vilson.xyz/](https://home.vilson.xyz/)

开源地址：[https://github.com/a54552239/pearProject](https://github.com/a54552239/pearProject)

轻量级的在线项目/任务协作系统，远程办公协作。相当于增强版的任务看板、工单系统。

## 在线IDE

### code-server

![code-server](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-18-01-29.png)

官网：[https://coder.com/](https://coder.com/)

开源地址：[https://github.com/cdr/code-server](https://github.com/cdr/code-server)

这个项目得益于VSCode的发展与开源，虽然微软官方推出了VSCode-Online，但是目前还不支持私有化部署，这个项目就是第三方基于VSCode发展来。旨在在任何地方的任何计算机上运行VS Code，然后在浏览器中对其进行访问。这样我们就可以随时随地打开浏览器进行项目开发啦。只需要有浏览器即可，环境都是一样的，妈妈再也不用担心我在其他地方没有开发环境的问题啦。

### jupyterlab

![jupyterlab](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-18-06-58.png)

官网：[https://jupyter.org/](https://jupyter.org/)

开源地址：[https://github.com/jupyterlab/jupyterlab](https://github.com/jupyterlab/jupyterlab)

一个基于Jupyter Notebook和Architecture的可扩展环境，用于交互式和可重复计算。目前为用户准备。JupyterLab是Jupyter 项目的下一代用户界面，它以灵活而强大的用户界面提供了经典Jupyter Notebook的所有熟悉的构造块（笔记本，终端，文本编辑器，文件浏览器，丰富的输出等）。JupyterLab最终将取代经典的Jupyter Notebook。

这个项目适合用Python来做科学计算的科研人员。

## 内网穿透

### frp

![frp](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-18-10-59.png)

开源地址：[https://github.com/fatedier/frp](https://github.com/fatedier/frp)

frp是一种快速反向代理，可帮助您将NAT或防火墙后面的本地服务器公开到Internet。到目前为止，它支持TCP和UDP以及HTTP和HTTPS协议，在这些协议中，请求可以通过域名转发到内部服务。frp还具有P2P连接模式。

平时我们自己本地写的项目如果要给别人预览的话是很麻烦的，毕竟本地一般都是内网，外网是没法直接访问的。因此，我们可以在公网上部署frp服务，然后利用公网服务器进行流量中转从而实现外网访问本地项目。特别是小程序的调试，这个是非常有用的。

### nps

![nps](https://blog.woodcoding.com/resources/images/2020-06-06/2020-06-06-18-14-58.png)

官网：[https://ehang.io/nps/documents](https://ehang.io/nps/documents)

开源地址：[https://github.com/ehang-io/nps](https://github.com/ehang-io/nps)

支持tcp，udp，socks5，http等几乎所有流量转发，可以访问内部网网站，本地支付接口调试，ssh访问，远程桌面的一个轻量级，高性能，功能强大的内网扩展代理服务器。 ，内网dns解析，内网socks5代理等等……，并具有功能强大的内网渗透代理服务器，并具有功能强大的网络管理终端。

功能和frp是一样的，但是frp只支持文件配置访问，而这个工具支持web界面配置访问。有时候文件配置比较舒服，毕竟就那么几行，但是对于新手来说，有个界面让你点点点也是很友好的，各自权衡吧。

内网穿透其实还有很多，这两个是目前比较流行的两个。

#### 参考资料：

[https://zh.wikipedia.org/](https://zh.wikipedia.org/)

[https://baike.baidu.com/](https://baike.baidu.com/)

[http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

[https://www.jianshu.com/p/67f345ebec9b](https://www.jianshu.com/p/67f345ebec9b)

