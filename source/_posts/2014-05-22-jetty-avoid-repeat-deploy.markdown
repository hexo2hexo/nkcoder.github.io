---
layout: post
title: Jetty9避免重复部署
date: 2014-05-22 22:34:28 +0800
comments: true
tags: jetty
categories: Backend
---

jetty是一个轻量级的web容器，部署简单，同时高度可定制化。由于刚接触，对其约定和配置不太熟练，在自动部署时，跳过一次重复部署的坑，这里简要说明一下，希望给同样打算在项目中使用jetty9的朋友一个参考。

默认情况下，jetty会对根目录(也可以配置jetty.base)下webapps/目录下的内容实现自动部署，部署的规则如下：

1. 隐藏文件(.开头)和.d结尾的目录被忽略；
2. CVS目录如"CVS"和"CVSROOT"被忽略；
3. 任何war包都会被自动部署；
4. 任何xml描述文件被认为是可部署的；
5. 任何目录都被认为是可部署的；
6. 同名的war包和目录同时存在，目录不被部署，仅war包部署，且认为war包引用该目录；
7. 同名的war包和xml文件同时存在，war包不被部署，仅xml文件描述符被部署，且认为该xml文件引用该war包；
8. 同名的目录和xml文件同时存在，目录不被部署，xml文件被部署，且认为xml文件引用该目录；

关于更详细的说明，请参考官方文档的[这里](http://www.eclipse.org/jetty/documentation/current/deployment-architecture.html)。我主要提醒的是：在webapps目录中，如果存在同名的目录、war包和xml文件，它们会被当做同一个工程，部署的优先级是`xml文件>war包>目录`。一定要注意同名，如果不同名，在webapps下存在一个war包，同时存在一个引用该war包的xml文件，则会导致重复部署，这就是我跳的坑。

部署时，推荐的做法是，将xml描述文件放到自动部署的webapps目录下，里面定义war包的路径、上下文路径、是否解压、临时目录、日志文件等，然后将war包放在自定义的固定目录下，项目更新，只需要备份和替换war包，重启jetty即可。

参考：
1. [Deployment Architecture](http://www.eclipse.org/jetty/documentation/current/deployment-architecture.html)
