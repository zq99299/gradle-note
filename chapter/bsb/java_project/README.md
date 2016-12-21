# JAVA构建入门

* [3.1. 基础项目](chapter/java_project/basic_project.md)
* [3.2. 多项目](chapter/java_project/multi-project.md)

## Java插件
Gradle 是一种多用途的构建工具. 它可以在你的构建脚本里构建任何你想要实现的东西. 但前提是你必须先在构建脚本里加入代码, 不然它什么都不会执行.

大都数 Java 项目是非常相像的: 
* 你需要编译你的 Java 源文件
* 运行一些单元测试,
* 同时创建一个包含你类文件的 JAR. 
如果你可以不需要为每一个项目重复编写这些, 我想你会非常乐意的.

Gradle 通过使用插件解决了这个问题. 插件是 Gradle 的扩展, 它会通过某种方式配置你的项目, 典型的有加入一些预配置任务. Gradle 自带了许多插件, 你也可以很简单地编写自己的插件并和其他开发者分享它. Java 插件就是一个这样的插件. 这个插件在你的项目里加入了许多任务， 这些任务会编译和单元测试你的源文件, 并且把它们都集成一个 JAR 文件里.

Java 插件是基于合约的. 这意味着插件已经给项目的许多方面定义了默认的参数, 比如 Java 源文件的位置. 如果你在项目里遵从这些合约, 你通常不需要在你的构建脚本里加入太多东西. 如果你不想要或者是你不能遵循合约, Gradle 也允许你自己定制你的项目. 事实上, 因为对 Java 项目的支持是通过插件实现的, 如果你不想要的话, 你一点也不需要使用这个插件来构建你的项目.