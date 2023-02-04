- [Jmap](#jmap)
- [Jstack](#jstack)
  - [Remote Connect JvisualVM](#remote-connect-jvisualvm)
  - [tomcat JMX Config](#tomcat-jmx-config)
  - [Jstack 找出佔用cpu最高的線程堆棧信息](#jstack-找出佔用cpu最高的線程堆棧信息)
  - [Jstack Find Out Highest CPU Usage of Thread Stack Info](#jstack-find-out-highest-cpu-usage-of-thread-stack-info)

# Jmap

通過 `Jmap` 可以用來查看 memory info，instance 個數以及佔用 memory 大小

```sh
jmap -histo process_id(jps) > ./log.txt
```

`log.txt`文件看上去是一個表格，表格里有四個 fields:

- num：序號
- instances：instance 數量
- bytes：佔用 memory 大小
- class name：類名稱
  - C is a char[]
  - S is a short[]
  - I is a int[]
  - B is a byte[]
  - I is a int[]

`Jmap` 查看 heap info：

```sh
jmap -heap process_id
```

heap memory dump：

```sh
jmap ‐dump:format=b,file=eureka.hprof process_id
```

也可以設置 memory overflow 自動導出 dump 檔案(memory 很大的時候，可能會失敗)

1. -XX:+HeapDumpOnOutOfMemoryError
2. -XX:HeapDumpPath=./ （path）

> 可以使用 `jvisualvm` 工具導入該 dump 檔案分析

# Jstack

用 `jstack` 加 process id 查找 deadlock，當然也可以用 `jvisualvm` 自動檢測 deadlock

- Thread-1: thread name
- prio=5: 優先級=5 
- tid=0x000000001fa9e000: thread id 
- nid=0x2d64: thread 對應的 local thread 標識 nid 
- java.lang.Thread.State: BLOCKED : thread status

## Remote Connect JvisualVM

啟動普通的 jar 程式 JMX port config：

```sh
 java ‐Dcom.sun.management.jmxremote.port=8888 ‐Djava.rmi.server.hostname=192.168.50.60 ‐Dcom.sun.management.jmxremot e.ssl=false ‐Dcom.sun.management.jmxremote.authenticate=false ‐jar microservice‐eureka‐server.jar
 ```

1. -Dcom.sun.management.jmxremote.port: 為遠端機器的 JMX port
2. -Djava.rmi.server.hostname 為遠程機器 IP



## tomcat JMX Config

> 在 `catalina.sh` 檔案裡的最後一個 `JAVA_OPTS` 的賦值語句下一行增加如下 config

```env
 JAVA_OPTS="$JAVA_OPTS ‐Dcom.sun.management.jmxremote.port=8888 ‐Djava.rmi.server.hostname=192.168.50.60 ‐Dcom.sun.ma nagement.jmxremote.ssl=false ‐Dcom.sun.management.jmxremote.authenticate=false"
```

## Jstack 找出佔用cpu最高的線程堆棧信息
## Jstack Find Out Highest CPU Usage of Thread Stack Info

1. 使用命令 `top -p  pid`，顯示 java process memory status，pid 為 java process id，如 19663
2. 按 H，獲取每個 thread memory status
3. 找到 memory 和 cpu 佔用最高的 thread tid，如 19664
4. 轉為十六進制得到 0x4cd0，此為 thread id 的十六進制表示
5. 執行 `jstack 19663|grep -A 10 4cd0`，得到 thread stack 資訊中 `4cd0` 這個 thread 所在行的後面10行，從 stack 中可以發現導致 cpu 飆高的調用方法
6. 查看對應的 stack info 找出可能存在問題的程式碼