# 处理文件

## 定位文件
Project.file(java . lang . object) 方法
```groovy
// Using a relative path
File configFile = file('src/config.xml')


// Using a File object with a relative path
configFile = file(new File('src/config.xml'))

// Using an absolute path
configFile = file(configFile.absolutePath)
println configFile
```

`file()` 方法接收任何形式的对象参数.它会将参数值转换为一个绝对文件对象,一般情况下，你可以传递一个 `String` 或者一个 File 实例.如果传递的路径是个绝对路径,它会被直接构造为一个文件实例.否则,会被构造为项目目录加上传递的目录的文件对象.另外,`file()`函数也能识别URL,例如 `file:/some/path.xml`.

这个方法非常有用,它将参数值转换为一个绝对路径文件.所以请尽量使用 `new File(somePath)` , 因为 `file()` 总是相对于当前项目路径计算传递的路径,然后加以矫正.因为当前工作区间目录依赖于用户以何种方式运行 Gradle.

**总结：**
`file()` 方法尝试把一个值转换成一个绝对文件对象。

`注意：` 如果是一个相对路径转换成绝对路径，那么这个绝对路径将会是当前的项目目录



## 文件集合
获取一个文件集，常用的方法有 Project.files(java . lang . object[]) 方法

### 创建一个文件集合

```groovy
FileCollection collection = files('src/file1.txt',
                                  new File('src/file2.txt'),
                                  ['src/file3.txt', 'src/file4.txt'])
println collection.from  // 打印我们put的相对路径
println collection.getFiles() //获取这一组绝对文件对象
```
文件集合可以被迭代器,使用迭代操作能够将其转换为其他的一些类型.你可以使用 + 操作将两个文件集合合并,使用 - 操作能够对一个文件集合做减法.下面一些例子介绍如何操作文件集合.

### 使用文件集合

```groovy
// Iterate over the files in the collection
// 遍历集合中的文件
collection.each { File file ->
    println file.name
}

// Convert the collection to various types
// 转换为各种类型
Set set = collection.files
Set set2 = collection as Set
List list = collection as List
String path = collection.asPath
File file = collection.singleFile
File file2 = collection as File

// 需要注意的是：
* singleFile 和 as File : 集合中存在多个文件的话会抛出异常

// 增加或则减少文件，不会改变原来的集合对象，新生成对象
def union = collection + files('src/file5.txt')
def different = collection - files('src/file5.txt')

```
你也可以向 `files()` 方法专递一个闭合或者可回调的实例参数.当查询集合的内容时就会调用它,然后将返回值转换为一些文件实例.返回值可以是 `files()` 方法支持的任何类型的对象.下面有个简单的例子来演示实现 `FileCollection` 接口

## 文件树

可以使用files方法或则闭包返回可调用的实例，只要返回的是files支持的类型即可
```groovy
task list {
    doLast {
        File srcDir

        // 这里有一个特性，按照java中来看，srcDir.listFiles() 肯定空指针了，但是这里好像并没有被执行，闭包
        FileCollection collection = files { srcDir.listFiles() }
        
        srcDir = file('.idea')
        // 正真使用前再赋值
        collection.collect { relativePath(it) }.sort().each { println it }

        println "Contents of $srcDir.name"
        collection.collect { relativePath(it) }.sort().each { println it }
        
        srcDir = file('.gradle')
        println "Contents of $srcDir.name"
       collection.collect { it.absolutePath }.sort().each { println it }
       
       //上面的collection.collect...作用是返回一组指定的数据类型
    }
}
```
另外, files() 方法也接收其他类型的参数:

* FileCollection 内容损坏的文件包含在文件集合中.
* Task 任务的输出文件包含在文件集合中.
* TaskOutputs TaskOutputs 的输出文件包含在文件集合中

值得注意的是当有需要时文件集合的内容会被被惰性处理,就比如一些任务在需要的时候会创建一个 FileCollecion 代表的文件集合.

**疑问：**
没有搞明白task的输入和输出是什么意思。


### collect 知识

collect 的源码注释，这个注释在idea中不知道是怎么搞出来的了。可以看到下面的示例。就知道该方法怎么使用了
```java
org.codehaus.groovy.runtime.DefaultGroovyMethods
public static <T> List<T> collect(@Nullable Object self,
                                  Closure<T> transform)
    /**
     * Iterates through this aggregate Object transforming each item into a new value using the
     * <code>transform</code> closure, returning a list of transformed values.
     * Example:
     * <pre class="groovyTestCase">def list = [1, 'a', 1.23, true ]
     * def types = list.collect { it.class }
     * assert types == [Integer, String, BigDecimal, Boolean]</pre>
     *
     * @param self      an aggregate Object with an Iterator returning its items
     * @param transform the closure used to transform each item of the aggregate object
     * @return a List of the transformed values
     * @since 1.0
     */
```

## 文件树
文件树就是一个按照层次结构分布的文件集合,例如,一个文件树可以代表一个目录树结构或者一个 `ZIP` 压缩文件的内容.它被抽象为 `FileTree` 结构,`FileTree` 继承自 `FileCollection`,所以你可以像处理文件集合一样处理文件树, Gradle 有些对象实现了`FileTree` 接口,例如 `source sets`. 使用 `Project.fileTree(java.util.Map) ` 方法可以得到 `FileTree` 的实例,它会创建一个基于基准目录的对象,然后视需要使用一些 Ant-style 的包含和去除规则.
```groovy
//以一个基准目录创建一个文件树
FileTree tree = fileTree(dir: '.idea')
// 添加包含和排除规则
tree.include '**/*.java'
tree.exclude '**/*.iml'
//排除和包含要先于使用，因为这里没有使用闭包
tree.collect { relativePath(it) }.sort().each { println it }

// 使用路径创建一个树
tree = fileTree('src').include('**/*.java')

// 使用闭合创建一个数
tree = fileTree('src') {
    include '**/*.java'
}

// 使用map创建一个树
tree = fileTree(dir: 'src', include: '**/*.java')
tree = fileTree(dir: 'src', includes: ['**/*.java', '**/*.xml'])
tree = fileTree(dir: 'src', include: '**/*.java', exclude: '**/*test*/**')
```

就像使用文件集合一样,你可以访问文件树的内容,使用 `Ant-style` 规则选择一个子树。

**使用文件树**
```groovy
// 遍历文件树
tree.each {File file ->
    println file
}
FileTree filtered = tree.matching {
    exclude '**/*.iml'
}

// 合并文件树A
FileTree sum = tree + fileTree(dir: '.gradle')

// 访问文件数的元素
tree.visit {element ->
    println "$element.relativePath => $element.file"
}
```

## 使用存档的内容作为一个文件树
你可以使用 `ZIP` 或者 `TAR` 等压缩文件的内容作为文件树,  `Project.zipTree(java.lang.object)` 和 `Project.tarTree(java.lang .object)`  方法返回一个 `FileTree` 实例, 你可以像使用其他文件树或者文件集合一样使用它.例如,你可以使用它去扩展一个压缩文档或者合并一些压缩文档.

```groovy
// 使用路径创建一个 ZIP 文件
FileTree zip = zipTree('someFile.zip')

// 使用路径创建一个 TAR 文件
FileTree tar = tarTree('someFile.tar')

//tar tree 能够根据文件扩展名得到压缩方式,如果你想明确的指定压缩方式,你可以使用下面方法
FileTree someTar = tarTree(resources.gzip('someTar.ext'))
```
## 指定一组输入文件
在 `Gradle` 中有一些对象的某些属性可以接收一组输入文件.例如,`JavaComplile` 任务有一个 `source` 属性,它定义了编译的源文件,你可以设置这个属性的值,只要 `files()` 方法支持. 这意味着你可以使用 `File , String , collection , FileCollection` 甚至是使用一个闭包去设置属性的值.

```groovy
task compile(type: JavaCompile)
//使用一个 File 对象设置源目录
compile {
    source = file('src/main/java')
}

//使用一个字符路径设置源目录
compile {
    source = 'src/main/java'
}

// 使用一个集合设置多个源目录
compile {
    source = ['src/main/java', '../shared/java']
}

// 使用 FileCollection 或者 FileTree 设置源目录
compile {
    source = fileTree(dir: 'src/main/java').matching { include 'org/gradle/api/**' }
}

// 使用一个闭包设置源目录
compile {
    source = {
        // Use the contents of each zip file in the src dir
        file('src').listFiles().findAll {it.name.endsWith('.zip')}.collect { zipTree(it) }
    }
}

```
教你看源码实现：
```java
task compile(type: JavaCompile){
    source = 'src/main/java'
}
在idea中，点source就能看到了
SourceTask.java
    protected final List<Object> source = new ArrayList();
    public void setSource(Object source) {
        this.source.clear();
        this.source.add(source);
    }
```
在这里就能看出来了，由于源码使用object接收，所以source能接收字符串
```groovy
compile {
    // 使用字符路径添加源目录
    source 'src/main/java', 'src/main/groovy'

    // 使用 File 对象添加源目录
    source file('../shared/java')

    // 使用闭合添加源目录
    source { file('src/test/').listFiles() }
}
```

## 复制文件
你可以使用复制任务( `Copy` )去复制文件. 复制任务扩展性很强,能够过滤复制文件的内容, 映射文件名.

### 使用复制任务复制文件
使用复制任务时需要提供想要复制的源文件和一个目标目录,如果你要指定文件被复制时的转换方式，可以使用 复制规则. 复制规则被 `CopySpec` 接口抽象,复制任务实现了这个接口. 使用 `CopySpec.from(java.lang.object[])` 方法指定源文件.使用 `CopySpec.into(java.lang.object)` 方法指定目标目录.
```groovy
task copyTask(type: Copy) {
    from 'src/main/webapp'
    into 'build/explodedWar'
}
```
`from()` 方法接收任何 `files()` 方法支持的参数. 当参数被解析为一个目录时,在这个目录下的任何文件都会被递归地复制到目标目录(但不是目录本身).当一个参数解析为一个文件时,该文件被复制到目标目录中.当参数被解析为一个不存在的文件时,这个参数就会忽略.如果这个参数是一个任务,任务的输出文件(这个任务创建的文件)会被复制,然后这个任务会被自动添加为复制任务的依赖.
### 指定复制任务的源文件和目标目录
```groovy
task anotherCopyTask(type: Copy) {
    // 复制 src/main/webapp 目录下的所有文件
    from 'src/main/webapp'
    // 复制一个单独文件
    from 'src/staging/index.html'
    // 复制一个任务输出的文件
    from copyTask
    // 显式使用任务的 outputs 属性复制任务的输出文件
    from copyTaskWithPatterns.outputs
    // 复制一个 ZIP 压缩文件的内容
    from zipTree('src/main/assets.zip')
    // 最后指定目标目录
    into { getDestDir() }
}
```

你可以使用Ant-style 规则或者一个闭合选择要复制的文件.
### 选择要复制文件
```groovy
task copyTaskWithPatterns(type: Copy) {
    from 'src/main/webapp'
    into 'build/explodedWar'
    include '**/*.html'
    include '**/*.jsp'
    exclude { details -> details.file.name.endsWith('.html') &&
                         details.file.text.contains('staging') }
}
```

你也可以使用 `Project.copy()` 方法复制文件,它的工作方式有一些限制:
- 首先该方法不是增量的,请参考 第 14.9节 跳过最新的任务.
- 第二,当一个任务被用作复制源时(例如 `from()` 方法的参数), `copy()` 方法不能够实现任务依赖,因为它是一个普通的方法不是一个任务.因此,如果你使用 `copy()`方法作为一个任务的一部分功能,你需要显式的声明所有的输入和输出以确保获得正确的结果.


###  复制文件使用copy()方法没有最新的检查
```groovy
task copyMethod {
    doLast {
        copy {
            from 'src/main/webapp'
            into 'build/explodedWar'
            include '**/*.html'
            include '**/*.jsp'
        }
    }
}
```


### 复制文件使用copy()方法和最新的检查
```groovy
task copyMethodWithExplicitDependencies{
    // up-to-date check for inputs, plus add copyTask as dependency
    inputs.file copyTask
    outputs.dir 'some-dir' // up-to-date check for outputs
    doLast{
        copy {
            // Copy the output of copyTask
            from copyTask
            into 'some-dir'
        }
    }
}
```
建议尽可能的使用复制任务,因为它支持增量化的构建和任务依赖推理，而不需要去额外的费力处理这些.不过 `copy()` 方法可以用作复制任务实现的一部分.即该 方法被在自定义复制任务中使用,请参考 第60章 编写自定义任务.在这样的场景下，自定义任务应该充分声明与复制操作相关的输入/输出。

### 重命名文件
在复制时重命名文件
```groovy
task rename(type: Copy) {
    from 'src/main/webapp'
    into 'build/explodedWar'
    // 使用一个闭包映射文件名
    rename { String fileName ->
        fileName.replace('-staging-', '')
    }
    // 使用正则表达式映射文件名
    rename '(.+)-staging-(.+)', '$1$2'
    rename(/(.+)-staging-(.+)/, '$1$2')
}
```

### 过滤文件
在复制时过滤文件,过滤的时候替换掉占位符。
```groovy

-file ：build.gradle

import org.apache.tools.ant.filters.FixCrLfFilter
import org.apache.tools.ant.filters.ReplaceTokens

task filter(type: Copy) {
    from 'src/main/resources'
    into 'build/explodedWar'
    // 在文件中替代属性标记，这里我没有测试出是个什么意思
//    expand(copyright: '2009', version: '2.3.1')
//    expand(project.properties)
    // 使用 Ant 提供的过滤器
    filter(FixCrLfFilter)
    filter(ReplaceTokens, tokens: [copyright: '2009', version: '2.3.1'])
    // 用一个闭包来过滤每一行
    filter { String line ->
        "$line"
    }
    // 使用闭包来删除行
    filter { String line ->
        line.startsWith('-') ? null : line
    }
    // 闭包中也可以这样写，其实就是遍历每一行，叫你返回处理的值
    filter { String line ->
        if(line.startsWith('-')){
            return null
        }else{
            return line
        }
    }
     // 指定过滤时读取文件的编码格式
     filteringCharset = 'UTF-8'
}

-file ：src/main/resources/test.txt

@copyright@
@version@
-我是被删除的

-file ：build/explodedWar/test.txt

2009
2.3.1


可以看到，上面的占位符已经被替换了

```
在源文件中扩展和过滤操作都会查找的某个标志 `token`,如果它的名字是 `tokenName` , 它的格式应该类似于 `@tokenName@`.

### 使用 CopySpec 类
复制规范来自于层次结构,一个复制规范继承其目标路径,包括模式,排除模式，复制操作,名称映射和过滤器.

嵌套复制规范
```groovy
task nestedSpecs(type: Copy) {
    into 'build/explodedWar'
    exclude '**/*.txt'
    from('src/dist') {
        exclude '**/*.html'
    }
    into('libs') {
        from configurations.runtime
    }
}
```
也就是说：`from('src/dist')`的时候，里面如果有.txt文件的话，将被继承规则（外围定义了过滤该类型文件）给过滤掉

## 使用同步任务
同步任务 ( `Sync` ) 任务继承自复制任务 ( `Copy` ) , 当它执行时,它会复制源文件到目标目录中,然后从目标目录中的删除所有非复制的文件,这种方式非常有用,比如安装一个应用,创建一个文档的副本,或者维护项目的依赖关系副本.

下面有一个例子,维护 build/libs 目录下项目在运行时的依赖
```groovy
使用 Sync 任务复制依赖关系

task libs(type: Sync) {
    from configurations.runtime
    into "$buildDir/libs"
}
```
总结：测试出来的结果，根本看不明白这个是什么意思，是删除什么文件？

## 创建归档文件

一个项目可以有很多 JAR 文件,你可以向项目中添加 WAR , ZIP 和 TAR 文档,使用归档任务可以创建这些文档: Zip , Tar , Jar , War 和Ear. 它门都以同样的机制工作.

### 创建一个 ZIP 文档

```groovy
apply plugin: 'java'

task zip(type: Zip) {
    from 'src/dist'
    into('libs') {
        from configurations.runtime
    }
}
```
所有的归档任务的工作机制和复制任务相同,它们都实现了 `CopySpec` 接口,和 `Copy` 任务一样,使用 `from()` 方法指定输入文件,可以选择性的使用 `into()` 方法指定什么时候结束.你还可以过滤文件内容,重命名文件等等,就如同使用复制规则一样.

### 归档文件的命名
默认生成方式：projectName-vsersion.type
```groovy
apply plugin: 'java'

version = 1.0

task myZip(type: Zip) {
    from 'somedir'
}

println myZip.archiveName
println relativePath(myZip.destinationDir)
println relativePath(myZip.archivePath)
```
gradle -q myZip 输出：
```groovy
gradle-1.0.zip
build\distributions
build\distributions\gradle-1.0.zip
```
可以看出，zip有一些默认的配置，比如上面的文档名，路径等。
可以通过设置项目属性 archivesBaseName 的值来 修改生成文档的默认名.当然,文档的名称也可以通过其他方法随时更改.下面是一些配置属性：

### zip属性配置列表
| 属性名 |	类型	| 默认值	 | 描述
|-------|---------------|-------|-------
| archiveName	| String	| baseName-appendix-version-classifier.extension,如果其中任何一| 个都是空的,则不添加名称	| 归档文件的基本文件名
| archivePath	| File	| destinationDir/archiveName	| 生成归档文件的绝对路径。
| destinationDir	| File	| 取决于文档类型, JAR 和 WAR 使用| project.buildDir/distributions. project.buildDir/libraries.ZIP 和 TAR	归档文件的目录
| baseName	| String	| project.name	| 归档文件名的基础部分。
| appendix	| String	| null	| 归档文件名的附加部分。
| version	| String	| project.version	| 归档文件名的版本部分。
| classifier	| String	| null	| 归档文件名的分类部分
| extension	| String	| 取决于文档类型和压缩类型: zip, jar, war, tar, tgz 或者 tbz2.	| 归档文件的扩展名

### 配置归档文件-自定义文档名

```groovy
apply plugin: 'java'
version = 1.0

task myZip(type: Zip) {
    from 'somedir'
    baseName = 'customName'
}

println myZip.archiveName
```

```groovy
使用 gradle -q myZip 命令进行输出:

> gradle -q myZip
customName-1.0.zip
```

更多配置:


### 配置归档任务- 附加其他后缀

```groovy
apply plugin: 'java'
archivesBaseName = 'gradle'
version = 1.0

task myZip(type: Zip) {
    appendix = 'wrapper'
    classifier = 'src'
    from 'somedir'
}

println myZip.archiveName
```

使用 gradle -q myZip 命令进行输出:
```groovy
> gradle -q myZip
gradle-wrapper-1.0-src.zip
```

> #### type::本文学习官网文档
>
>https://docs.gradle.org/current/userguide/working_with_files.html




