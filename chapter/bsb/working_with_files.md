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
file 方法尝试把一个值转换成一个绝对文件对象。

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
### 使用文件集合

## 文件树
## 使用存档的内容作为一个文件树
## 指定一组输入文件
## 复制文件
## 使用同步任务
## 创建档案




> #### type::本文学习官网文档
>
>https://docs.gradle.org/current/userguide/working_with_files.html




