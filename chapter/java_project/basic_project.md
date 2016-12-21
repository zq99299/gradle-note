# 基础项目构建

## 添加插件
```groovy
apply plugin: 'java'

该插件基于以下约定目录默认配置：
src
 └──main
  ├── java  
  ├── resources
  └── webapp
 └──test
  ├── java
  └── resources
build
  └──libs            -生成的java.jar将被放在这里
```

## 执行构建
```groovy
$ gradle build
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 4.298 secs
```
部分任务解说：

* **clean**

 删除 build 生成的目录和所有生成的文件.

* **assemble**

 编译并打包你的代码, 但是并不运行单元测试. 其他插件会在这个任务里加入更多的东西. 举个例子, 如果你使用 War 插件, 这个任务将根据你的项目生成一个 WAR 文件.

* **check**

 编译并测试你的代码. 其他的插件会加入更多的检查步骤. 举个例子, 如果你使用 checkstyle 插件, 这个任务将会运行 Checkstyle 来检查你的代码.