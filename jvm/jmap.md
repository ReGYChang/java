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