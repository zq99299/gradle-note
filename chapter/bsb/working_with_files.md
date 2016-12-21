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

## 使用文件树
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

## 复制文件
## 使用同步任务
## 创建档案




> #### type::本文学习官网文档
>
>https://docs.gradle.org/current/userguide/working_with_files.html




