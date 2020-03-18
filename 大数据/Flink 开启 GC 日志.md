# Flink 开启 GC 日志

## 位置 客户端的 `conf/flink-conf.yaml`
```bash
env.java.opts: -Xloggc:<LOG_DIR>/gc.log -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDetails -XX:-OmitStackTraceInFastThrow -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=20 -XX:GCLogFileSize=20M -XX:+PrintPromotionFailure -XX:+PrintGCCause
```

## 参数解读

`-XX:+PrintGCApplicationStoppedTime` 打印 GC 导致程序停顿的时间

`-XX:+PrintGCDetails` 开启打印 GC 详情

`-XX:+PrintGCTimeStamps` 打印 GC 时间戳（相对 JVM 启动时候的时间）

`-XX:+PrintGCDateStamps` 打印 GC 日期戳（get time of day）

`-XX:+UseGCLogFileRotation` 开启滚动日志

`-XX:NumberOfGCLogFiles=20` 设置滚动日志的数量

`-XX:GCLogFileSize=20M` 设置单个滚动日志文件的文件大小阈值，如果当前写入的日志文件大于该值则进行日志切割。

`-Xloggc:<LOG_DIR>/gc.log` 设置日志路径
- `<LOG_DIR>` 变量的值是 Job/ Task manager 的日志路径，如果用 yarn 部署就是 container 的 log 路径。
- 如果没设置该参数，GC日志会输出在进程的标准输出里。
- 打开或关闭GC日志滚动记录功能，必须设置 `-Xloggc` 参数。
- Flink 文档中这个变量的值是 `{FLINK_LOG_PREFIX}.gc.log`，但是我弄了半天没打印出日志。

`-XX:-OmitStackTraceInFastThrow` 关闭 JIT 对热点异常的优化，保证打印详细的异常堆栈信息

`-XX:+PrintPromotionFailure` 打印新生代对象晋升老年代失败的附加信息
`-XX:+PrintGCCause` 打印 GC 原因