---
title: 根据JetBrains Fleet文档创建maven项目失败的解决办法
date: 2022-10-26 16:46:26
tags:
	- 根据JetBrains
	- Fleet
	- maven
categories:
	- 开发工具
---



打算尝试一下Fleet编辑器，根据[JetBrains Fleet文档](https://www.jetbrains.com/help/fleet/getting-started-with-java-in-fleet.html#open-workspace)创建maven项目.

首选确认本地机器上已经安装并配置了maven

按照文档提示，通过`CTRL+ALT+T` 打开 terminal 窗口，在编辑器的下方

粘贴并执行  `mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false`

创建失败，提下如下


{% asset_image  QQ截图20221026170547.png %}

<!-- more -->


然后google之，在stackoverflow上发现了这篇文章
[https://stackoverflow.com/questions/16348459/error-the-goal-you-specified-requires-a-project-to-execute-but-there-is-no-pom](https://stackoverflow.com/questions/16348459/error-the-goal-you-specified-requires-a-project-to-execute-but-there-is-no-pom)


{% asset_image QQ截图20221026170831.png %}


{% asset_image UFqqh.png %}

简而言之：*必须用引号括起所有参数*

于是重新修改了mvn命令并重新执行

`mvn archetype:generate "-DgroupId=com.mycompany.app" "-DartifactId=my-app" "-DarchetypeArtifactId=maven-archetype-quickstart" "-DarchetypeVersion=1.4" "-DinteractiveMode=false"  `

于是maven项目创建成功

{% asset_image QQ截图20221026171206.png %}

{% asset_image QQ截图20221026171222.png %}
