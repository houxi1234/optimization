# optimization


### 安装测试工具 ```siege```

1. 下载最新版本的siege：
   <https://www.joedog.org/siege-home/>

2. 安装解压：tar -zxvf siege-*.tar.gz

3. 进入到解压后的目录：

   ```shell
   ./configure
   make && make install
   ```

4. 安装完成



### Tomcat优化

- Tomcat 版本
- Spring Boot 版本



- 默认参数

  命令```siege -c4000 -t5 -b  http://10.197.236.76:11001/getTest```

```shell

Transactions:		       25500 hits
Availability:		      100.00 %
Elapsed time:		       13.28 secs
Data transferred:	        1.22 MB
Response time:		        0.12 secs
Transaction rate:	     1920.18 trans/sec
Throughput:		        0.09 MB/sec
Concurrency:		      236.37
Successful transactions:       25500
Failed transactions:	           0
Longest transaction:	        3.09
Shortest transaction:	        0.01

```

​	命令```siege -c2000 -t5 -b  http://10.197.236.76:11001/getTest```

```shell
Transactions:		       67957 hits
Availability:		       98.38 %
Elapsed time:		       38.26 secs
Data transferred:	        3.45 MB
Response time:		        0.13 secs
Transaction rate:	     1776.19 trans/sec
Throughput:		        0.09 MB/sec
Concurrency:		      237.68
Successful transactions:       67957
Failed transactions:	        1118
Longest transaction:	        3.16
Shortest transaction:	        0.01
```

```

```

```
CPU 6核12线程
```

单应用-undertow

```siege -c1000 -t5 -b  http://10.197.236.76:8081/test```

```
Transactions:		      574542 hits
Availability:		      100.00 %
Elapsed time:		      299.15 secs
Data transferred:	        8.22 MB
Response time:		        0.13 secs
Transaction rate:	     1920.58 trans/sec
Throughput:		        0.03 MB/sec
Concurrency:		      243.90
Successful transactions:      574542
Failed transactions:	           0
Longest transaction:	       15.35
Shortest transaction:	        0.02
```

siege -c2000 -t5 -b  http://10.197.236.76:8081/test

```
Transactions:		      619559 hits
Availability:		      100.00 %
Elapsed time:		      299.90 secs
Data transferred:	        8.86 MB
Response time:		        0.12 secs
Transaction rate:	     2065.89 trans/sec
Throughput:		        0.03 MB/sec
Concurrency:		      253.97
Successful transactions:      619559
Failed transactions:	           0
Longest transaction:	        3.17
Shortest transaction:	        0.01
```

```siege -c2500 -t5 -b  http://10.197.236.76:8081/test```

```
Transactions:		      601743 hits
Availability:		      100.00 %
Elapsed time:		      299.20 secs
Data transferred:	        8.61 MB
Response time:		        0.13 secs
Transaction rate:	     2011.17 trans/sec
Throughput:		        0.03 MB/sec
Concurrency:		      253.00
Successful transactions:      601743
Failed transactions:	           0
Longest transaction:	        3.15
Shortest transaction:	        0.01

```



###### 上述：TPS 2000达到瓶颈

单应用-tomcat

```siege -c2500 -t5 -b  http://10.197.236.76:8081/test```

```
Transactions:		      605622 hits
Availability:		      100.00 %
Elapsed time:		      299.67 secs
Data transferred:	        8.66 MB
Response time:		        0.12 secs
Transaction rate:	     2020.96 trans/sec
Throughput:		        0.03 MB/sec
Concurrency:		      243.51
Successful transactions:      605622
Failed transactions:	           0
Longest transaction:	        3.15
Shortest transaction:	        0.02
```



###### HTTP2支持

<https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#howto-configure-http2>

undertow 对HTTP2天然支持

```siege -c1000 -t5 -b http://localhost:11001/getTest```



springcloud性能优化

报错

```shell
org.springframework.web.util.NestedServletException: Request processing failed; nested exception is feign.RetryableException: Address already in use: connect executing
```

原因：http连接耗尽

优化：替换`URLConnection` 使用http连接池

Feign在默认情况下使用的是JDK原生的`URLConnection`发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用HTTP的`persistence connection`

添加httpclient连接池

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

```yml
feign:
  httpclient:
    enabled: false
  okhttp:
    enabled: true
```

`siege -c4000 -t5 -b http://10.197.236.76:11001/getTest`

```
Transactions:		      551065 hits
Availability:		      100.00 %
Elapsed time:		      299.58 secs
Data transferred:	       26.28 MB
Response time:		        0.14 secs
Transaction rate:	     1839.46 trans/sec
Throughput:		        0.09 MB/sec
Concurrency:		      253.13
Successful transactions:      551065
Failed transactions:	           0
Longest transaction:	        3.19
Shortest transaction:	        0.01
```

添加okhttp连接池

```xml
<dependency>
	<groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

```yml
feign:
  httpclient:
    enabled: false
  okhttp:
    enabled: true
```

`siege -c4000 -t5 -b http://10.197.236.76:11001/getTest`

```
Transactions:		      535127 hits
Availability:		      100.00 %
Elapsed time:		      299.36 secs
Data transferred:	       25.52 MB
Response time:		        0.14 secs
Transaction rate:	     1787.57 trans/sec
Throughput:		        0.09 MB/sec
Concurrency:		      254.17
Successful transactions:      535127
Failed transactions:	           0
Longest transaction:	        3.28
Shortest transaction:	        0.03

```



HTTP2 替换 HTTP 

okhttp连接池

```
Transactions:		      554457 hits
Availability:		      100.00 %
Elapsed time:		      299.96 secs
Data transferred:	       26.44 MB
Response time:		        0.14 secs
Transaction rate:	     1848.44 trans/sec
Throughput:		        0.09 MB/sec
Concurrency:		      254.52
Successful transactions:      554457
Failed transactions:	           0
Longest transaction:	        3.16
Shortest transaction:	        0.02

```

httpclient连接池

```
Transactions:		      522554 hits
Availability:		      100.00 %
Elapsed time:		      299.51 secs
Data transferred:	       24.92 MB
Response time:		        0.15 secs
Transaction rate:	     1744.70 trans/sec
Throughput:		        0.08 MB/sec
Concurrency:		      254.67
Successful transactions:      522554
Failed transactions:	           0
Longest transaction:	        3.18
Shortest transaction:	        0.03
```

