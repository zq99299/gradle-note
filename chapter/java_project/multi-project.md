# 多项目构建

项目结构如下：
```groovy
multiproject/
  api/				// 打成jar包，给客户使用
  webservice/         // 网络服务
  shared/		     // 包含 api 和 webservice的共享代码 和 自己的代码
```

## 创建项目root目录 multiproject

1. 编写build.gradle:
这里只定义了三个子项目的公共配置
```groovy
subprojects {
    apply plugin: 'java'
    apply plugin: 'eclipse-wtp'

    repositories {
       mavenCentral()
    }

    dependencies {
        testCompile 'junit:junit:4.11'
    }

    version = '1.0'

    jar {
        manifest.attributes provider: 'gradle'
    }
}  
```

2. 编写settings.gradle
```groovy
简单布局，使用 gradle projects 能查看到当前的 所有 项目
include "shared", "api", "webservice"
```

3. 构建项目
```groovy
$ gradle build
```
可以看到我们定义的三个子项目目录已经被自动生成了。各个项目里面还有build文件夹。

## shared 的编写
由于这里只是演示，api和 webservice是空项目。只编写shared的build.gradle，用来打成一个zip包
```groovy
// 项目之间的依赖
dependencies {
	compile project(':api')
	compile project(':webservice')
}

// zip包打包
task dist(type: Zip) {
    dependsOn jar     // 该任务依赖jar 任务
    from 'dist'  // 把项目中dist目录中的内容也打到zip中
    into('libs') {
        from jar.archivePath  //本项目生成的jar包路径
        from configurations.runtime   // 运行依赖api和webservice项目，所以这里会得到两个jar
    }
}
// 这个配置我暂时没有搞明白是什么。效果就是在build的时候会自动调用 dist任务。
// 从而会在shared\build\distributions 中生成 shared-1.0.zip 包
artifacts {
   archives dist
}
```