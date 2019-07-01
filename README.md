# KLog

日志输出类

支持序列化到文件中

demo 的话请看 app的MainActivity

```kotlin
class Config {
   /**
     * Logcat tag
     */
    var logcatTag = "KLOG"

    /** 是否打印线程信息 */
    var showThreadInfo = true
    /** 堆栈方法数 */
    var methodCount = 1
    /**
     * 打印器,可自定义
     */
    var printer: IPrinter = LogPrinter()
    /**
     * 是否使用内置的打印器
     */
    var useDefaultType = true

    /**
     * 自定义的打印类型
     */
    var customTypes = listOf<ILogType>()
}
```

## 打印Log

```
KLog.recordE("msg",type = DefaultLogger.NAME , logcat = true)
//只打印logcat
Klog.e("msg")
```

```
//type : 是打印类型标识,默认为 logcat
//logcat :当type 为其他打印类型时,是否同时打印 logcat
fun e(
        msg: String?, error: Throwable? = null,
        type: String = LogCatLogger.NAME,
        logcat: Boolean = false
    )
```



## 支持自定义`TYPE`

```kotlin

interface ILogType {
    //名字作为唯一标识
    val typeName: String

    /** 保存到文件? */
    val saveToFile: Boolean

    /**
     * 可以处理其他类的消息?
     */
    fun handleType(name: String): Boolean

    /**
     * 最大保存行数
     * [saveToFile]为true 必须大于0
     *
     * 文件中存储的最大日志行数
     * 注意是行数,一个日志加上头尾,线程信息和方法堆栈,就已经7行了
     */
    val maxCount: Int
    /**
     * 当达到最大行数时,要删除多少条旧数据
     * [saveToFile]为true 必须大于0
     */
    val delCountOnMax: Int

    /**
     * 打印消息的一行
     */
    fun printLine(msg: String, log: LogModel)

    /**
     * 打印整个消息,每行已经用"\n"分割开
     */
    fun printFullMsg(msg: String, log: LogModel)

    fun logToFile(file: File)
}

```

默认实现请看`LogCatType`和`DefaultType`

在`Config.customTypes`中添加



## 日志转存到文件

```
 //要在子线程执行
 //@param groupByType true:会依次往下排,false,会按照时间顺序排列
 fun logToFile(
     path: String, type: Int = KLog.config.logLevel,
     groupByType: Boolean = false
 )

 fun logToFileAll(path: String, groupByType: Boolean = false)
 
 //删除某一个类型的log,不穿则删除全部
 fun clearLog(type: Int? = null)
```

## 依赖

`implementation com.gwmfc.android:klog:${version}`

如果集成后报错
`More than one file was found with OS independent path ‘META-INF/main.kotlin_module’`

请在 `app` 的 `android` 标签下加入以下代码

```gradle
packagingOptions {
    exclude 'META-INF/library_release.kotlin_module'
}
```


## 效果

- logcat

![](shapshot/30e5a519.png)

- 文件

![](shapshot/6b5d17ff.png)

## 版本记录

### 1.3.3

- 添加一个api  `getDbFile`获取日志存储db文件

查询语句,并将 time格式化
`select *,datetime(time/1000, 'unixepoch', 'localtime') from klog;`

### 1.3.2

- cursor 错误处理,某些地方忘记关闭了

### 1.3.1 

- API 修改

`logcat`: `Klog.e()`

需要打印到文件的:`KLog.recordE(...)`

### 1.3.0

- 更改大量API, type 不再使用code进行区分,直接用名字

### 1.2.2 

- KLog Api 调整

### 1.2.1

- KLog 方法 使用`@JvmStatic`注解
- `LogCatType`与`DefaultType``open`,可继承重写
- 加入`debug`配置,如果为false,那么logcat不会再有 输出

### 1.2.0

线程优化,日志改到子线程去打印
经测试,一次完整打印大概耗费`385938`纳秒

### 1.1.0

增加自定义Type功能

### 1.0.0

- 完成基本功能



## 参考

https://github.com/orhanobut/logger

其中有些类完全是复制该项目中的文件,修改而成,比如`LogPrinter`