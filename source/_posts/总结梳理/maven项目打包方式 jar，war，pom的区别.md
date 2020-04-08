---
title: maven项目打包方式 jar，war，pom的区别
date: 2020-04-08 11:28:50
tags: 
 - Maven
 - 总结梳理
categories:
 - 总结梳理
comments: true
keywords: 
description: 
---

***

### 打包文件类型的具体含义

1. JAR 文件（扩展名为. Jar，Java Application Archive）包含Java类的普通库、资源（resources）、辅助文件（auxiliary files）等。为 J2EE 应用程序创建的 JAR 文件是 EAR 文件（企业 JAR 文件）。
2. WAR 文件（扩展名为.War，Web Application Archive）包含全部 Web 应用程序。在这种情形下，一个 Web 应用程序被定义为单独的一组文件、类和资源，用户可以对 WAR 文件进行封装，并把它作为小型服务程序（Servlet）来访问。
3. EAR 文件（扩展名为.Ear，Enterprise Application Archive）包含全部企业应用程序。在这种情形下，一个企业应用程序为多个 JAR 文件、资源、类和Web应用程序的集合。EAR 文件包括整个项目，内含多个ejb module（JAR文件）和 web module（WAR文件）

***

	那么问题来了，maven项目的三种打包方式: Jar，War，pom 有何区别呢？

  1. JavaSE 程序可以打包成 JAR 包，默认的打包方式，用于存放一些其他工程都会使用的类、工具类。我们可以在其他工程的pom文件中去引用它。
	JAR 文件格式以流行的 ZIP 文件格式为基础。与 ZIP 文件不同的是，JAR 文件不仅用于压缩和发布，而且还用于部署和封装库、组件和插件程序，并可被像编译器和 JVM 这样的工具直接使用。在 JAR 中包含特殊的文件，如 manifests 和部署描述符，用来指示工具如何处理特定的 JAR。

  2. JavaWeb 程序可以打包成 WAR 包，然后把 WAR 发布到 Tomcat 的 webapps 目录下， Tomcat 会在启动时自动解压 WAR 包。

  3. pom：用在父级工程或聚合工程中，用来做 JAR 包的版本控制，必须指明这个聚合工程的打包方式为pom。

***

  - 当一个 Web 应用程序的目录和文件非常多时，我们将这个Web应用程序部署到另一台机器上，并不是很方便。此时，我们可以将 Web 应用程序打包成 Web 归档（WAR）文件，这个过程和把 Java类 文件打包成 Java 归档 (JAR) 文件 的过程类似。利用 WAR 文件，可以把Servlet类文件和相关的资源集中在一起进行发布。在这个过程中，Web应用程序就不是按照目录层次结构来进行部署了，而是把 WAR 文件作为部署单元来使用。

  - 一个 War 文件就是一个 Web 应用程序，建立 WAR 文件，就是把整个 Web 应用程序（不包括Web应用程序层次结构的根目录）压缩起来，指定一个.war 扩展名。

  - 要注意的是，虽然 WAR 文件和 JAR 文件的文件格式是一样的，并且都是使用 jar 命令来创建，但就其应用来说，WAR 文件和 JAR 文件是有根本区别的。JAR 文件的目的是把类和相关的资源封装到压缩的归档文件中，而对于 WAR 文件来说，一个 WAR 文件代表了一个 Web 应用程序，它可以包含 Servlet、HTML页面、Java类、图像文件，以及组成 Web 应用程序的其他资源，而不仅仅是类的归档文件。

  - 我们什么时候应该使用 WAR 文件呢？在开发阶段不适合使用 WAR 文件，因为在开发阶段，经常需要添加或删除 Web 应用程序的内容，更新 Servlet 类文件，而每一次改动后，重新建立 WAR 文件将是一件浪费时间的事情。因此，在产品发布阶段，我们使用WAR文件是比较合适的，因为在这个时候，几乎不需要再做什么改动了。

  - 在开发阶段，我们通常将 Servlet 源文件放到 Web 应用程序目录的 src 子目录下，以便和 Web 资源文件区分。在建立 WAR 文件时，只需要将 src 目录从 Web 应用程序目录中移走，就可以打包了。