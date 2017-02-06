# 依赖管理

## 介绍
构建依赖关系管理是一个关键的特性，`Gradle`容易理解和兼容`mavn`的方式。对依赖关系管理的支持有：
* 传递依赖管理：它让你完全控制你的项目的依赖关系树
* 支持不受管理的依赖性：如果你的依赖仅仅是文件版本控制或共享驱动器，它提供了强大的功能支持这种需求
* 支持制定一依赖项的定义：它的模块依赖关系使您能构建脚本描述依赖关系的层次结构
* 完全与Maven和ivy的兼容性：无缝兼容
* 继承与现有管理基础设置的Mave和Ivy：兼容这两种管理的仓库

# 什么是依赖管理
粗略的讲：
- dependencies(依赖项) ：
Gradle 需要了解你的项目需要构建或运行的东西, 以便找到它们. 我们称这些传入的文件为项目的 dependencies(依赖项)
- publications(发布项) ：Gradle 需要构建并上传你的项目产生的东西. 我们称这些传出的项目文件为 publications(发布项)

又引申出了以下几个术语：

1. dependency resolution(依赖解析)

    大多数项目都会引用第三方包，这些第三方包构成项目的依赖。依赖关系可能需要从远程的 Maven 或者 Ivy 仓库中下载, 也可能是在本地文件系统中, 或者是通过多项目构建另一个构建。这个过程就叫 依赖解析。
2. transitive dependencies(依赖传递)
    
    一个第三方包包会依赖其他的第三方包， 举个例子, 运行 Hibernate 的核心需要其他几个类库在 classpath 中. 因此, Gradle 在为你的项目运行测试的时候, 它会找到这些依赖关系, 并使其可用. 
3. publication(发布)

    大部分项目的主要目的是要建立一些文件, 在项目之外使用. 比如, 你的项目产生一个 Java 库,你需要构建一个jar, 可能是一个 jar 和一些文档, 并将它们发布在某处.
    
    `发布`具体是看你的需求，可以发布到任何地方，如maven仓库，本地目录等
    
# 依赖关系管理最佳实践    
## 文件名中加入版本
 `commons-beanutils-1.3.jar` 或一组文件名称 `spring.jar`
您觉得哪种更直观呢？

## 版本冲突
一般的项目都会引用大量的第三方jar包，那么这个时候就会出现版本冲突，然而Gradle提供了以下几种策略

* 最新的：使用最新版本的依赖，默认策略
* 失败：一个版本冲突导致构建失败。需要显示的配置  [ResolutionStrategy](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ResolutionStrategy.html?_ga=1.13926064.663550861.1483336010   ) 中如何配置

上面的两种足以解决大多数的场景需求了。还提供了以下更细粒度的机制来解决版本冲突
* 强制配置第一级的依赖：例子:D[ependencyHandler ](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html?_ga=1.77181586.663550861.1483336010)
* 配置任何以来项的传递：
* 配置依赖决议：
* xxxx

`gradle dependencies` 查看依赖树。`gradle :您的项目:dependencies` 来查看子项目的依赖树

## 使用动态版本和修改模块
* 使用一个最新的：`latest.integration`
* 某一个版本的最新的：`2.+`

默认情况下Gradle缓存24小时的动态版本和修改模块，可以使用[命令行选项](https://docs.gradle.org/current/userguide/dependency_management.html?_ga=1.114054784.663550861.1483336010#sec:cache_command_line_options) 和参考 [调整控制依赖缓存](https://docs.gradle.org/current/userguide/dependency_management.html?_ga=1.184662830.663550861.1483336010#sec:controlling_caching)

# 依赖配置
Gradle的依赖关系配置，可以被扩展，许多插件会添加预定义的配置，例如`java`配置了各种资源路径。API:  [ConfigurationContainer ](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ConfigurationContainer.html?_ga=1.114512384.663550861.1483336010)。

### 定义配置
```groovy
configurations {
    compile
}
```
### 访问配置
```java
println configurations.compile.name
println configurations['compile'].name
```

### 配置一个配置
```groovy
configurations {
    compile {
        description = 'compile classpath'
        transitive = true
    }
    runtime {
        extendsFrom compile
    }
}
configurations.compile {
    description = 'compile classpath'
}
```

# 如何声明你的依赖
## 依赖类型
 依赖类型

| 类型 | 描述 |
|----------------------|------------------
| 外部模块的依赖	     | 在一些存储库依赖于外部模块。
| 项目依赖项	     | 在相同的构建依赖于另一个项目。
| 文件的依赖	     | 依赖于一组文件在本地文件系统。
| 客户端模块的依赖    | 依赖于外部模块,工件位于一些库,但模块的元数据 是由当地指定的构建。 你使用这种依赖当你想覆盖模块的元数据。
| Gradle API依赖     | 依赖于当前Gradle版本的API。 你使用这种依赖,当你开发定制Gradle插件和任务类型。
| 当地Groovy的依赖    | 依赖于当前使用的Groovy版本Gradle版本。 你使用这种依赖,当你开发定制Gradle插件和任务类型。

## 外部依赖
让我们看一下一些依赖的声明. 下面是一个基础的构建脚本:
声明依赖
```groovy
build.gradle
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```
这里发生了什么? 这个构建脚本声明 Hibernate core 3.6.7.最终 被用来编译项目的源代码. 言外之意是, 在运行阶段同样也需要 Hibernate core 和它的依赖. 构建脚本同样声明了需要 junit >= 4.0 的版本来编译项目测试. 它告诉 Gradle 到 Maven 中央仓库里找任何需要的依赖. 接下来的部分会具体说明.

> 官方文档 https://docs.gradle.org/3.0/userguide/dependency_management.html

# 发布 artifacts
> 官方文档 https://docs.gradle.org/current/userguide/artifact_management.html

