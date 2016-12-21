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
 
## 依赖添加
`官网文档：` https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html

```groovy
repositories {
    mavenLocal()
    mavenCentral()
    maven{ url "https://maven.repository.redhat.com/ga/"}
}

dependencies {
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testCompile group: 'junit', name: 'junit', version: '4.+'
} 
```

## 依赖检查
```bash
$ gradle dependencies
:dependencies

----------------------------------------------------------
Root project
----------------------------------------------------------

archives - Configuration for archive artifacts.
No dependencies

compile - Compile classpath for source set 'main'.
\--- commons-collections:commons-collections:3.2

default - Configuration for default artifacts.
\--- commons-collections:commons-collections:3.2

runtime - Runtime classpath for source set 'main'.
\--- commons-collections:commons-collections:3.2

testCompile - Compile classpath for source set 'test'.
+--- commons-collections:commons-collections:3.2
\--- junit:junit:4.+ -> 4.12
     \--- org.hamcrest:hamcrest-core:1.3

testRuntime - Runtime classpath for source set 'test'.
+--- commons-collections:commons-collections:3.2
\--- junit:junit:4.+ -> 4.12
     \--- org.hamcrest:hamcrest-core:1.3

BUILD SUCCESSFUL

Total time: 4.493 secs
```
可以查看当前项目的依赖关系，具体是啥意思，自己领悟吧。我也看不太明白


## 定制项目
```groovy
sourceCompatibility = 1.5
version = '1.0'
jar {
	// 定制 MANIFEST.MF 文件的内容，一个attributes 占一行
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
    }
}
```

* sourceCompatibility : 源码编译JDK版本
* version ：项目版本/JAR包版本
* jar ： jar包构建定制
 
 官网文档：https://docs.gradle.org/current/dsl/org.gradle.api.tasks.bundling.Jar.html

## 测试阶段加入一个系统属性
怎么使用这个属性 还不知道
```groovy
test {
    systemProperties 'propertyssssssssssssssssssss': 'value'
}
```

## 获取项目中所有的属性
```base
$ gradle properties 
```
使用以下命令来列出项目的所有属性，包括自己添加的，里面有的都可以直接引用
比如这样引用属性：
```groovy

task demo{
	println libsDir 
	println "这是libs存放路径$libsDir"
}

```

## 发布 JAR 文件
```groovy
uploadArchives {
    repositories {
       flatDir {
           dirs 'repos' // 发布jar到当前目录的repos中
       }
    }
}
$ gradle tasks 中能看到增加了一个 task：uploadArchives    

```
疑问就是，为什么这里增加这个配置，就多出了一个task呢？


## 完成的build.gradle 文件
```groovy
apply plugin: 'java'

sourceCompatibility = 1.5
version = '1.0'

jar {
	// 定制 MANIFEST.MF 文件的内容，一个attributes 占一行
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
    }
}


repositories {
    mavenCentral()
}

dependencies {
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}

uploadArchives {
    repositories {
       flatDir {
           dirs 'repos' // 当前目录
       }
    }
}
```

## 创建 Eclipse 项目
```groovy
apply plugin: 'eclipse'

$ gradle eclipse 命令来生成 Eclipse 的项目文件
```
## 创建 Idea项目
```groovy
apply plugin: 'idea'

$ gradle idea 命令来生成 idea 的项目文件
```









