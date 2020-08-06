# Java 常见问题排查
## 1、排查 CPU 100%
?> **面试题：** 在开发过程中遇到过 CPU 100% 的情况吗？怎么排查的？
<!-- tabs:start -->

#### **参考回答**

- 首先使用 `top -c` 找到 CPU 占用最高的进程；
- 然后使用 `top -Hp PID` 找到进程下 CPU 占用最高的线程 PID，并且将十进制 PID 转换成十六进制；
- 之后使用 `jstack` 命令导出进程快照；
- 最后用 `cat` 命令结合 `grep` 命令对十六进制线程 PID 进行过滤，可以定位到出现问题的代码。

### 参考资料
- [敖丙我把线上CPU打到100%，三歪吓尿了【三太子敖丙】](https://mp.weixin.qq.com/s/roEMz-5tzBZvGxbjq8NhOQ)

#### **源码详解**

### 1）找到 CPU 占用最高的进程
- 使用`top -c`命令得到进程运行列表，然后按`P`按照 CPU 占用率排序；
- 得到排在第一的进程的 `PID：18412`。
![](http://images.intflag.com/cpu100-01.png)

### 2）找到 CPU 占用最高的线程
- 使用`top -Hp 18412`命令得到这个进程下的线程列表，然后按`P`按照 CPU 占用率排序；
- 得到排在第一的线程的 `PID：18413`；
![](http://images.intflag.com/cpu100-02.png)

- 将十进制 PID 转换为十六进制得到`47ED`。
![](http://images.intflag.com/cpu100-03.png)

### 3）导出进程快照并定位问题
- 使用`jstack`命令导出进程快照；
```bash
jstack -l 18412 > ./18412.stack
```
- 根据第二部得到的小写的十六进制线程 PID 使用 grep 命令进行查找；
```bash
cat 18412.stack | grep '47ed' -C 8
```
- 从查询得到的堆栈信息中即可定位到有问题的代码；
![](http://images.intflag.com/cpu100-04.png)
![](http://images.intflag.com/cpu100-05.png)

<!-- tabs:end -->


## 2、排查 OOM
?> **面试题：** 工作中遇到过 OOM 吗？你是怎么排查的？
<!-- tabs:start -->

#### **参考回答**

- `-XX:+HeapDumpOnOutOfMemoryError` 参数可以在发生 OOM 时自动进行 dump；
- `jmap` 命令可以手动 dump；
- 可以使用 JDK 自带的工具 jvisualvm.exe 进行分析。

### 参考资料
- [老公：怎么排查堆内存溢出啊？【三太子敖丙】](https://mp.weixin.qq.com/s/7XGD-Z3wrThv5HyoK3B8AQ)

#### **源码详解**


### 1）写程序模拟内存耗尽从而发生 OOM
```java
public class OomBugsHandlerImpl extends LoggerHandler implements BugsHandler {

    private static final Integer K = 1024;

    @Override
    public void bug() {
        int size = K * K * 10;
        List<byte[]> list = new ArrayList<>();
        for (Integer i = 0; i < K; i++) {
            logger.info("JVM 写入数据 {} M", (i + 1) * 10);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(new byte[size]);
        }
    }
}
```

### 2）启动程序并开启 OOM 发生时自动 dump
- `-XX:+HeapDumpOnOutOfMemoryError` 参数可以在发生 OOM 时自动进行 dump；
```java
java -XX:+HeapDumpOnOutOfMemoryError -jar Bugs-App-v1.0-jar-with-dependencies.jar oom
```
- 发生 OOM 自动 dump。
![](http://images.intflag.com/oom-07.png)

- 手动 dump
```bash
jmap -dump:format=b,file=<dumpfile.hprof> <pid>
```

### 3）使用 jvisualvm.exe 进行实时分析
- 工具位于：JDK安装目录\bin 下，同时保证安装了 Visual GC 插件；
![](http://images.intflag.com/oom-01.png)
![](http://images.intflag.com/oom-04.png)

- 同时可以在监视窗口下查看堆使用情况；
![](http://images.intflag.com/oom-02.png)

- 直到发生 OOM。
![](http://images.intflag.com/oom-03.png)

### 4）使用 jvisualvm.exe 进行离线分析
- 将 dump 后的文件导入工具中进行分析；
![](http://images.intflag.com/oom-05.png)
- 通过分析得知是由于 byte 数组太多导致内存溢出。
![](http://images.intflag.com/oom-06.png)

### 5）使用其他工具分析
- MAT 工具

<!-- tabs:end -->