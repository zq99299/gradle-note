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

## 使用存档的内容作为一个文件树
## 指定一组输入文件
## 复制文件
## 使用同步任务
## 创建档案




> #### type::本文学习官网文档
>
>https://docs.gradle.org/current/userguide/working_with_files.html




