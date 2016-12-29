# 生命周期

`Gradle`的核心是以基于依赖的编程语言，意味这你可以定义任何和任务之间的依赖关系。
`Gradle`保证执行这些任务的执行依赖顺序，且每个任务只执行一次。 这些任务形成一个 `有向无环图`。Gradle构建完整的依赖关系图之前或之后执行任何任务

## 构建阶段
Gradle构建有三个不同的阶段：

1. 初始化

 它支持单和多项目构建。在初始化阶段，它决定哪些项目要参与构建，并为每一个项目创建一个`Project`实例
2. 配置

  会去执行所有工程的build.gradle脚本，配置project对象，一个对象由多个任务组成，
  此阶段也会去创建、配置task及相关信息。
3. 运行/执行

  根据`gradle`命令传递过来的task名称，执行相关依赖任务


`建立文件注意：`使用Notepad++ 需要修改为 以`以UTF-8无BOM格式编码` 中文才不会乱码和报错

## 配置文件 settings.gradle 
` settings.gradle `默认约定配置文件(在以后的章节中会讲到怎么修改该默认值)

` settings.gradle `在初始化阶段执行，一个多项目构建必须有一个`settings.gradle `文件，因为它设置了哪些项目参与多项目构建，对于单一的项目构建，该文件是可选的。除了定义包含的项目，你可能需要添加库构建脚本类路径（原文：42章组织建立逻辑）

## 单一项目构建

settings.gradle
```groovy
println '这是在初始化阶段执行的。'
```

build.gradle
```groovy
println '这是在配置阶段执行。'

task configured {
    println '这也是在配置阶段执行。'
}

task test {
    doLast {
        println '这是在执行阶段执行。'
    }
}

task testBoth {
    doFirst {
      println '这在执行阶段首先执行。'
    }
    doLast {
      println '这是在执行阶段最后执行'
    }
    println '这是在配置阶段执行以及。'
}
```
```groovy
$ gradle testBoth
这是在配置阶段执行。
这是在配置阶段执行。
这也是在配置阶段执行。
这是在配置阶段执行以及。
:testBoth
这在执行阶段首先执行。
这是在执行阶段最后执行

BUILD SUCCESSFUL

Total time: 0.785 secs
```

在setting中也能使用方法访问属性，详细参考settings的API https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html

## 多项目构建
在一次build中对多个项目执行构建，需要把参与多项目构建的项目设置在`settings.gradle`文件中。在https://docs.gradle.org/current/userguide/multi_project_builds.html中有详细的多项目构建说明

### 项目的位置/定位
多项目构建总是用一个root树表示。树中每个元素标表示一个项目。每个项目也有一个位置路径。在大多数情况下，项目路径是与文件系统中的路径是一致的。但是此行为是可以配置的。该项目树中的`settings.gradle`文件中配置。默认情况下，假定`settings.gradle`文件所在位置就是根项目的位置。但是你可以重新设置根项目的位置。

### 构建树

在`settings.gradle`中，你可以使用一组方法来构建项目树。分层和平面布局。

#### 分层布局
settings.gradle
```groovy
include 'project1', 'project2:child', 'project3:child1'
```

include 采用项目路径作为参数。项目路径假设是相对物理文件系统的路径。例如：
`services:api` 映射到一个文件夹 `services/api(相对路径)`.
`services:hotels:ap` 将创建三个项目：`services`、`services:hotels`、`services:hotels:ap`

运行gradle project就能看到gradle为我们构建的项目层级关系。
```bash
$ gradle project
:projects

-------------------------------------
Root project
-------------------------------------

Root project 'gradle'
+--- Project ':project1'
+--- Project ':project2'
|    \--- Project ':project2:child'
\--- Project ':project3'
     \--- Project ':project3:child1'
```
`services:hotels:ap`生成的项目构建
```bash
$ gradle project
:projects

----------------------------------------------
Root project
----------------------------------------------

Root project 'gradle'
\--- Project ':services'
     \--- Project ':services:hotels'
          \--- Project ':services:hotels:ap'
```

#### 平面布局

settings.gradle
```groovy
includeFlat 'project3', 'project4'
```
`includeFlat` 以目录名作为参数，这些目录作为根目录中的项目。
```bash
+--- Project ':project3'
\--- Project ':project4'
```

我是这样理解的：该布局有点类似 eclipse中的workset。布局下的项目都是独立的项目。独立项目下还可以有分层结构。

### 修改项目树的元素

在`settings`文件中，是以项目描述符来创建多项目树的，您可以随时修改设置文件中的这些描述符。要访问一个描述符，你可以做：

使用这个描述可以更改名称，项目目录，并建立项目的文件。

* 打印信息
settings.gradle
```groovy
println rootProject.name
println project(':projectA').name
```

* 修改项目的目录 和 默认的 build.gradle文件指向
settings.gradle
```groovy
rootProject.name = 'main'
project(':projectA').projectDir = new File(settingsDir, '../my-project-a')
project(':projectA').buildFileName = 'projectA.gradle'
```

以下是测试脚本
settings.gradle

```groovy
includeFlat 'projectA', 'projectB'

println "=== 修改前"
println rootProject.name
println project(':projectA').name

rootProject.name = 'main'
project(':projectA').projectDir = new File(settingsDir, '../my-project-a')
project(':projectA').buildFileName = 'projectA.gradle'

println "=== 修改后"
println rootProject.name
println project(':projectA').name
```
gradle build 输出
```bash
$ gradle build
=== 修改前
gradle
projectA
=== 修改后
main
projectA
```

这里更改的名称，名相当于是一个别名和变量。。

> #### API文档详细信息
>
> https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/ProjectDescriptor.html


## 初始化

`gradle`是如何知道是单项目还是多项目构建的？如果在`settings.gradle`所在目录进行构建，很容易。但是也允许在任何一个子项目中执行构建。`Gradle`会查按以下顺序查找`settings.gradl`文件，并读取它进行配置构建

1. 当前目录
2. 如果没有找到，则搜索父目录
3. 如果没有找到，则作为单一的项目构建执行
4. - u 命令禁止在父目录查找
5. 以上只适合include布局，includeFlat布局都会从当前目录寻找。https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:running_partial_build_from_the_root API详细
Gradle为每个项目创建一个project对象参与构建。 多项目构建这些项目设置中指定的对象(+根 项目)。 每个项目对象有默认名称等于它的顶层目录的名称, 和每一个项目除了根项目都有父项目。 任何项目可能有孩子的项目。


测试如下：
```groovy
创建目录和文件结构：
|-gradle
|-build.gradle
|-settings.gradle
  |--projectB
	|--build.gradle
  	|--projectC
```

根目录 gradle 的配置
```groovy
build.gradle 里面为空，什么都不写
settings.gradle：

include 'projectA','projectB'
println "gradle settings 文件被访问"
```

projectB 的配置
```groovy
build.gradle :

println "projectB---" + rootProject.name

settings.gradle：

//include 'projectC'
println "projectC settings 文件被访问"
```

projectC 的配置
```groovy
build.gradle :

println "projectC---" + rootProject.name
```

1. 在根目录`gradle`目录中运行

```bash
$ gradle 
gradle settings 文件被访问
projectB---gradle
```
可见项目b参与了构建

2. 在projectB中运行

```bash
$ gradle 
projectC settings 文件被访问
projectB---projectB
```
可见项目b项目的根目录是自己，因为在当前目录找到了setting文件。

3. 在projectC中运行
```bash
$ gradle 
projectC settings 文件被访问
projectC---projectC
```
可见去找了父类的配置文件，但是父类中 `//include 'projectC'`被注释掉了，所以自己就不是父类中的子项目。自己目录中也没有`settings`文件，所以执行单项目的构建。

4. -u 命令，不从父目录寻找；在在projectC中运行
```groovy
$ gradle -u
projectC---projectC
``` 
可见父目录的打印语句没有被触发

5. 我们打开c中注释的语句，在projectC中运行
```groovy
$ gradle
projectC settings 文件被访问
projectB---projectB
projectC---projectB
```
c被参与了多项目构建。


## 配置和执行一个单项目的构建

对于单项目的构建流程初始化后的阶段是非常简单的。构建脚本在初始化阶段创建project对象。然后`Gradle寻找任务名称作为命令参数传递。如果这些任务名称存在，他们将会按照执行顺序执行

## 构建中的生命周期
构建的进度可以在构建脚本中接收到通知。这些通知一般采取两种形式：
* 可以实现特定的监听器接口
* 也可以提供一个闭包执行。

下面使用闭包的例子。有关如何使用监听器接口的详细信息，请参阅API文档。


### 项目评估
从文档注释来看：当该项目的生成文件已被执行时，会收到该侦听器的通知.
https://docs.gradle.org/current/dsl/org.gradle.api.Project.html 中 afterEvaluate(closure) api 文档中说道。
build.gradle
```groovy
allprojects {
    afterEvaluate { project ->
        if (project.hasTests) {
            println "Adding test task to $project"
            project.task('test') {
                doLast {
                    println "Running tests for $project"
                }
            }
        }
    }
}

allprojects {
    ext.hasTests = false
}
```
settings.gradle
```groovy
include 'projectA', 'projectB'

project(':projectA').buildFileName = '../projectA.gradle'  //注意这里，更改了projectA的默认构建文件为 projectA.gradle 。在 projectA.gradle中直接写 hasTests = true 才不会报错。
```
projectA.gradle
```groovy
hasTests = true
```

输出结果
```bash
$ gradle -q
Adding test task to project ':projectA'
```

`注意：`https://docs.gradle.org/3.0/userguide/build_lifecycle.html#sec:project_evaluation
官文这里的例子是不能运行成功的。因为文档不全。

这个示例使用方法 Project.afterEvaluate() 添加闭包执行 项目后评估。

在项目评估时也可以接收通知，下面演示了，打印了项目评估是否通过
### 通知
build.gradle
```groovy
gradle.afterProject {project, projectState ->
    if (projectState.failure) {
        println "Evaluation of $project FAILED"
    } else {
        println "Evaluation of $project succeeded"
    }
}

task test

```
projectB.gradle
```groovy
throw new RuntimeException('broken')
```
settings.gradle
```groovy
include 'projectA', 'projectB'

project(':projectB').buildFileName = '../projectB.gradle'
```

输出
```bash
$ gradle -q
Evaluation of root project 'buildProjectEvaluateEvents' succeeded
Evaluation of project ':projectA' succeeded
Evaluation of project ':projectB' FAILED

FAILURE: Build failed with an exception.
```
### 任务创建
您可以在任务添加到项目之后立即收到通知.。这可以用来设置一些默认值或添加行为之前的任务是可在建文件。
```groovy
build.gradle
tasks.whenTaskAdded { task ->
    task.ext.srcDir = 'src/main/java'
}

task a

println "source dir is $a.srcDir"
```
输出
```bash
> gradle -q a
source dir is src/main/java
```
上面的功能就类似这个
```groovy
task a{
  ext.srcDir = 'src/main/java'
}
```

### 记录每个任务执行的开始和结束
build.gradle
```groovy
task ok

task broken(dependsOn: ok) {
    doLast {
        throw new RuntimeException('broken') // 该任务抛出异常，执行失败
    }
}

// 任务执行前
gradle.taskGraph.beforeTask { Task task ->
    println "executing $task ..."
}

//任务执行后
gradle.taskGraph.afterTask { Task task, TaskState state ->
    if (state.failure) {
        println "FAILED"
    }
    else {
        println "done"
    }
}
```

输出
```bash
> gradle -q broken
executing task ':ok' ...
done
executing task ':broken' ...
FAILED
```