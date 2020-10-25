---
title: 编程中的日期和时间
date: 2020-10-23 11:52:06
categories:
	- 基础知识
tags:
	- Linux
	- Java
---

# 1. Unix 时间戳

**UNIX时间**，或称**POSIX时间**是[UNIX](https://zh.wikipedia.org/wiki/UNIX)或[类UNIX](https://zh.wikipedia.org/wiki/類UNIX)系统使用的时间表示方式：从[UTC](https://zh.wikipedia.org/wiki/協調世界時)1970年1月1日0时0分0秒起至现在的总秒数，不考虑[闰秒](https://zh.wikipedia.org/wiki/閏秒)[[1\]](https://zh.wikipedia.org/wiki/UNIX时间#cite_note-2)。 在多数Unix系统上Unix时间可以透过`date +%s`指令来检查。

UNIX时间戳的0按照ISO 8601规范为 ：1970-01-01T00:00:00Z。

## 1.1 数据库的时间戳

MySQL 提供了 三种存储时间的方式：Datetime、Timestamp、Unix timestamp。

|                | 存储格式      | 内存占用 | 能否跨时区 |
| -------------- | ------------- | -------- | ---------- |
| Datetime       | “2020-06-25…" | 8字节    | 不能       |
| Timestamp      | “2020-06-25…" | 4字节    | 能         |
| Unix timestamp | 15298832      | 4字节    | 能         |

使用Unix timestamp 需要在读取数据是转换，但是好处是很多的，比如对全球化查询友好，可以跨时区，格式显示可以自定义等。

## 1.2 Unix timestamp 转数字

数据库把时间精确到毫秒保存的，1000 毫秒 = 1 秒。

所以做时间判断时，需要先/1000

```java
if (System.currentTimeUtils() - sqlTimeStamp > 60 * 1000 ) {}
```

表示 如果当前时间超过了记录时间 1分钟。

## 1.3 Java 时间转换

```java
SimpleDateFormat sdf = New SimpleDateFormat("%Y%M%D %h%m%s");
Date date = new Date();	// 结果是 Unix timestamp
sdf.format(date);
```

## 1.4 前端时间转换

```javascript
Date date = new Date(15298832);		//将Unix时间戳作为参数
date.toLocalTime();								//转为常规时间
var date = new Date()
date.format("yyyy年mm月dd日") // 需要导入包，格式化时间
data.getFullYear() + date.getMouth()+1 + date.getDate() // 或手动格式化年-月-日
```

## 1.5 Python 时间转换

```python
timeUnix = datetime.now()
timestring = datetime.now().strftime('%Y%m%d_%H%M%S')	//获取时间并格式化
```

## 1.6 Golang 时间转换

```go
t := time.Now()   //2019-07-31 13:55:21.3410012 +0800 CST m=+0.006015601
fmt.Println(t.Format("20060102150405"))    //当前时间戳
t1 := time.Now().Unix()  //1564552562
fmt.Println(time.Now().Format("2006-01-02"))  //2019-07-31
fmt.Println(time.Now().Format("2006-01-02 15:04:05"))  //2019-07-31 13:57:52
```

