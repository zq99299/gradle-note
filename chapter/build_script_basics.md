# 脚本构建基础

## Projects 和 tasks

Gradle 里的任何东西都是基于这两个基础概念:

* projects ( 项目 )
* tasks ( 任务 )

### projects
每一个构建都是由一个或多个 projects 构成的. 一个 project 到底代表什么依赖于你想用 Gradle 做什么. 

举个例子, 一个 project 可以代表一个 JAR 或者一个网页应用. 它也可能代表一个发布的 ZIP 压缩包, 这个 ZIP 可能是由许多其他项目的 JARs 构成的. 但是一个 project 不一定非要代表被构建的某个东西. 它可以代表一件**要做的事, 比如部署你的应用.

### tasks
每一个 project 是由一个或多个 tasks 构成的.

 一个 task 代表一些更加细化的构建. 可能是编译一些 classes, 创建一个 JAR, 生成 javadoc, 或者生成某个目录的压缩文件.


## Hello Word
```groovy
task hello{
	doLast {
        println 'Hello world!'
    }
}

// gradle -q hello


task hello2 << {
    println 'Hello world!'
}

task upper << {
    String someString = 'mY_nAmE'
    println "Original: " + someString
    println "Upper case: " + someString.toUpperCase()
}

```

## 任务依赖,可以不提前声明。
```groovy
task taskX(dependsOn: 'taskY') << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}
// 依赖的另外一种写法
//taskX.dependsOn taskY
```
> 不提前声明 :指y在文档中可以定义在x后面



## 循环
```groovy
task count << {
    4.times { print "$it " }
}

```

## 动态创建任务
```groovy
// counter -> $it
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
```

## 通过API访问一个任务 - 加入行为
```groovy
task hello3 << {
    println 'Hello Earth'
}
hello3.doFirst {
    println 'Hello Venus'
}
hello3.doLast {
    println 'Hello Mars'
}
hello3 << {
    println 'Hello Jupiter'
}
```

* doFirst ： 在什么之前执行
* doLast  : 在什么之后执行

需要搞清楚的一个点就是`“什么”` 是个什么东西？,这个来说，每个任务都一个默认的执行体，这个执行体在后面讲自定义任务类型的时候会讲到。这里的任务执行体相当于是空。


## 短标记法
```groovy
// $ 引用task，task也作为project中的一个属性
task hello4 << {
    println 'Hello world!'
}
hello4.doLast {
    println "Greetings from the $hello4.name task."
}

```

## 自定义任务属性
```groovy
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties << {
	logger.info("sss");
    println myTask.myProperty
	
}
```

## 默认任务
```groovy
defaultTasks 'clean', 'run','other'  // 运行gradle 运行时会默认执行这个task

task clean << {
    println 'Default Cleaning!'
}

task run << {
    println 'Default Running!'
}

task otherss << {
    println "I'm not a default task!"
}
```

## 使用方法组织脚本逻辑
```groovy
task useMethod(){
    doLast{
        println getMsg("使用方法");
    }
}
String getMsg(String msg){
    return "msg:" + msg;
}
```


