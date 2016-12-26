# 生命周期

`Gradle`的核心是以基于依赖的编程语言，意味这你可以定义任何和任务之间的依赖关系。
`Gradle`保证执行这些任务的执行依赖顺序，且每个任务只执行一次，

## 构建阶段
Gradle构建有三个不同的阶段：

1. 初始化

 它支持单和多项目构建。在初始化阶段，它决定哪些项目要参与构建，并为每一个项目创建一个`Project`实例
2. 配置

  会去执行所有工程的build.gradle脚本，配置project对象，一个对象由多个任务组成，
  此阶段也会去创建、配置task及相关信息。
3. 运行

  根据`gradle`命令传递过来的task名称，执行相关依赖任务

## 配置文件 settings.gradle 
` settings.gradle `默认约定配置文件(在以后的章节中会讲到怎么修改该默认值)

` settings.gradle `在初始化阶段执行，一个多项目构建必须有一个`settings.gradle `文件，对于单一的项目构建，该文件是可选的。除了定义包含的项目，你可能需要添加库构建脚本类路径（原文：42章组织建立逻辑）

## 单一项目构建

settings.gradle
```groovy
println 'This is executed during the initialization phase.'
```

build.gradle
```groovy
println 'This is executed during the configuration phase.'

task configured {
    println 'This is also executed during the configuration phase.'
}

task test {
    doLast {
        println 'This is executed during the execution phase.'
    }
}

task testBoth {
    doFirst {
      println 'This is executed first during the execution phase.'
    }
    doLast {
      println 'This is executed last during the execution phase.'
    }
    println 'This is executed during the configuration phase as well.'
}
```
