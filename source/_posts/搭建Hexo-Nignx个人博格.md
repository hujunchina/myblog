---
title: 搭建Hexo+Nginx个人博客
date: 2020-10-22 18:03:54
categories: 工具
tags: 
	- NGINX
	- HEXO
---

今天从零搭建一个博客，在Centos上使用Nginx作为服务器，并使用强大的Hexo博客系统。在搭建的过程中，遇到了很多问题，先填个坑。



### 配置文件的引用ejs版

配置文件分为全局的和主题的，都是 `config.yml`文件，只是文件位置不同。

在html中：`<%= config.subtitle %>`

在JS中：	  `<%= config.root %>`

在CSS中：   `<%= config.root %>`



### 添加访问统计

使用[腾讯移动分析](https://mta.qq.com/)统计访问次数等信息，网站会自动产生几行JS脚本文件，需要加入项目中。

由于不同的`themes`使用不同的语言，我这里使用的是`ejs`。

找到 `script.ejs`文件，并把代码复杂到最后即可。



### Hexo 添加图片

在source下建立images文件夹，并放入图片，该文件夹会在执行生成命令时，生成一个根目录下文件夹。

所以在文中引用图片：`![](/images/myphoto.jpg)` 即可。



### Hexo 添加JS和CSS

```
<%- css(['styles/atom-one-light.min.css']) %>
```

资源文件放在themes的source文件夹下即可。



### Hexo 的 Front Matter

```markdown
title: Java 笔记
date: 2020-10-22 21:11:37
categories:
	- 笔记
tags: 
	- Java
	- 编程
```

上面是正确格式，注意hexo采用markdown格式保存文档。



### 有关yaml文件格式小结：

#### 语法特点

- 大小写敏感
- 通过缩进表示层级关系
- **禁止使用tab缩进，只能使用空格键** （个人感觉这条最重要）
- 缩进的空格数目不重要，只要相同层级左对齐即可
- 使用#表示注释

#### 数据结构

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
- 纯量（scalars）：单个的、不可再分的值

```
value: this is a string
car:{color:red, brand:JL}			#对象
arr:
  - 1
  - 2
  - 3
arr2: [1,2,3]
```



# 搜索引擎优化

新建的网站无法被各大搜索引擎检索到，需要进行简单的设置，让搜索引擎知道自己的网站并建立索引。

## 1. Google搜索引擎优化

Google还是比较方便的，只需要生成token，并把token加入DNS解析中即可。

### 1.1 生成token

进入[Google搜索控制台](https://search.google.com/search-console)，点击新增资源，输入自己URL即可获得token。

![image-20201025110302310](/images/dns01.png)

复制好token，并导入到DNS中。

### 1.2 生成DNS记录

添加新的DNS解析记录，这里记录类型要选择TXT，并把token复制到记录值中。

![image-20201025110957110](/images/dns02.png)

等待几分钟，让域名记录生效。

### 1.3 完成

当在Google控制台点击验证，并通过后，就完成了。还需要等待一天，才能看到检索信息。

## 2. Bing搜索引擎优化

### 2.1 迁移Google信息

登录[Bing搜索控制台](https://www.bing.com/webmasters)，添加新资源，并选择从Google搜索控制台导入即可完成。

![image-20201025111721293](/images/dns03.png)

## 3. So搜索引擎优化

### 3.1 添加DNS记录

进入[360站长平台](http://zhanzhang.so.com/sitetool/site_manage)，添加新域名后，进行验证操作。

![image-20201025112805633](/images/dns04.png)

选择CNAME验证，并在DNS记录中加入一条记录，类型为CNAME，值为zz.zhanzhang.so.com即可。

![image-20201025112945698](/images/dns05.png)

## 4. Baidu搜索引擎优化

### 4.1 添加站点

登录[百度站长平台](https://ziyuan.baidu.com/site/index)，按照提示添加域名，并到第三步，如选择CNAME验证方式同So搜索引擎优化。

这里选择HTML标签验证方式。

直接在head文件中加入`<meta name="baidu-site-verification" content="code-d5Nue4S4rk" />`即可。