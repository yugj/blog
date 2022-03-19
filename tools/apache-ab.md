# Apache Ab Test

## 简介
单线程 http性能测试工具

docs: https://httpd.apache.org/docs/2.4/programs/ab.html

优势：相对jmeter更轻量、更快、脚本编写简单  
劣势：jmeter提供更细的统计结果&报表，如接口错误信息、单线程的请求时间，而AB则不支持；

## Demo: 
```shell script
ab -{option}
-n requests  请求数量
-c concurrency b并发数
-p  请问文件报文  搭配-T使用
-H 请求头
```

```shell script
ab -n 10 -c 10 http://localhost:9002/test/10
```

```shell script
ab -n 100000000 -c 100 -T "application/json" -H "api-tag: yugj" -p /Users/yugj/Desktop/ab.txt http://localhost:8088/demo/test/demo
```

## 实用参数：
```
-k
Enable the HTTP KeepAlive feature, i.e., perform multiple requests within one HTTP session. Default is no KeepAlive.
```

## 注意事项
* 无断言，无法感知结果正确性，一般配合监控或日志使用
* 单线程，无法利用CPU核数，一般开多进程压
