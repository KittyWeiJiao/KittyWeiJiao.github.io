##

### 1. 查看gc情况

jstat -gcutil [pid] [count]

### 2. 初步判断内存使用

jmap -histo:live [pid]

### 3. profiler工具
> https://github.com/uber-common/jvm-profiler uber开源

> https://www.jianshu.com/p/9364028cca4e async-profiler  火焰图

### 4. dump内存

jmap -dump:format=b,file=dump_file_name pid

### 5. dump线程栈

jstack pid > dump_file_name


### 附录: 常见问题处理

```shell

jstack on alpine：Unable to get pid of LinuxThreads manager thread
在docker 中使用jstack 报错：
bash-4.3# jps
112 Jps
7 jar
bash-4.3# jstack 7
7: Unable to get pid of LinuxThreads manager thread
解决方法：尝试把你的dockerfile 中启动java的方式改为以下方法：
ENTRYPOINT ["/bin/bash", "-c", "set -e && java -Xmx100m -jar /demo.jar"]

```