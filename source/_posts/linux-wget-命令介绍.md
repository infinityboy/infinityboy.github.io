---
title: linux wget 命令介绍
date: 2019-06-03 20:41:01
tags: Linux
categories: 
- 后端
---


>本篇博客仅以个人学习记录使用
>原文链接：https://www.cnblogs.com/ginvip/p/6350189.html

Linux wget是一个下载文件的工具，它用在命令行下。对于Linux用户是必不可少的工具，尤其对于网络管理员，经常要下载一些软件或从远程服务器恢复备份到本地服务器。如果我们使用虚拟主机，处理这样的事务我们只能先从远程服务器下载到我们电脑磁盘，然后再用ftp工具上传到服务器。这样既浪费时间又浪费精力，那不没办法的事。而到了Linux VPS，它则可以直接下载到服务器而不用经过上传这一步。wget工具体积小但功能完善，它支持断点下载功能，同时支持FTP和HTTP下载方式，支持代理服务器和设置起来方便简单。下面我们以实例的形式说明怎么使用wget。

1.使用wget下载单个文件，以下的例子是从网络下载一个文件并保存在当前目录
```
1.wget http://cn.wordpress.org/wordpress-3.1-zh_CN.zip
2.wget -P /tools http://cn.wordpress.org/wordpress-3.1-zh_CN.zip
-P 指定下载到哪个目录
在下载的过程中会显示进度条，包含（下载完成百分比，已经下载的字节，当前下载速度，剩余下载时间）
```
2、使用wget -O下载并以不同的文件名保存，wget默认会以最后一个符合”/”的后面的字符来命令，对于动态链接的下载通常文件名会不正确。
```
错误：下面的例子会下载一个文件并以名称download.php?id=1080保存
1.wget https://www.centos.bz/download?id=1
即使下载的文件是zip格式，它仍然以download.php?id=1080命令。
```
```
正确：为了解决这个问题，我们可以使用参数-O来指定一个文件名
1.wget -O wordpress.zip https://www.centos.bz/download.php?id=1080
```
3.使用wget –limit -rate限速下载，当你执行wget的时候，它默认会占用全部可能的宽带下载。但是当你准备下载一个大文件，而你还需要下载其它文件时就有必要限速了
```
1.wget --limit-rate=300k http://cn.wordpress.org/wordpress-3.1-zh_CN.zip
```
4.使用wget -c断点续传，使用wget -c重新启动下载中断的文件:
```
wget -c http://cn.wordpress.org/wordpress-3.1-zh_CN.zip
对于我们下载大文件时突然由于网络等原因中断非常有帮助，我们可以继续接着下载而不是重新下载一个文件。需要继续中断的下载时可以使用-c参数
```
5.使用wget -b后台下载，对于下载非常大的文件的时候，我们可以使用参数-b进行后台下载。
```
1.wget -b http://cn.wordpress.org/wordpress-3.1-zh_CN.zip
2.Continuing in background, pid 1840.
3.Output will be written to `wget-log'.

你可以使用以下命令来察看下载进度
tail -f wget-log
```

6.伪装代理名称下载
```
有些网站能通过根据判断代理名称不是浏览器而拒绝你的下载请求。不过你可以通过–user-agent参数伪装。
wget --user-agent="Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.16 (KHTML, like Gecko) Chrome/10.0.648.204 Safari/534.16" 下载链接
```
7.使用wget –spider测试下载链接
```
当你打算进行定时下载，你应该在预定时间测试下载链接是否有效。我们可以增加–spider参数进行检查
1.wget --spider URL

如果下载链接正确，将会显示
wget --spider URL
Spider mode enabled. Check if remote file exists.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Remote file exists and could contain further links,
but recursion is disabled -- not retrieving.


这保证了下载能在预定的时间进行，但当你给错了一个链接，将会显示如下错误
wget --spider url
Spider mode enabled. Check if remote file exists.
HTTP request sent, awaiting response... 404 Not Found
Remote file does not exist -- broken link!!!

你可以在以下几种情况下使用spider参数:
1.定时下载之前进行检查
2.间隔检测网站是否可用
3.检查网站页面的死链接
```
8.使用wget –tries增加重试次数
```
如果网络有问题或下载一个大文件也有可能失败。wget默认重试20次连接下载文件。如果需要，你可以使用–tries增加重试次数
wget --tries=40 URL
```
9.使用wget -i下载多个文件
```
首先，保存一份下载链接文件

1.cat > filelist.txt
2.url1
3.url2
4.url3
5.url4

接着使用这个文件和参数-i下载
wget -i filelist.txt
```
10.使用wget –mirror镜像网站
```
下面的例子是下载整个网站到本地

wget --mirror -p --convert-links -P ./LOCAL URL
1.–miror:开户镜像下载
2.-p:下载所有为了html页面显示正常的文件
3.–convert-links:下载后，转换成本地的链接
4.-P ./LOCAL：保存所有文件和目录到本地指定目录
```
11.使用wget –reject过滤指定格式下载
```
你想下载一个网站，但你不希望下载图片，你可以使用以下命令。

wget --reject=gif url
```
12.使用wget -o把下载信息存入日志文件
```
你不希望下载信息直接显示在终端而是在一个日志文件，可以使用以下命令
wget -o download.log URL
```
13.使用wget -Q限制总下载文件大小
```
当你想要下载的文件超过5M而退出下载，你可以使用以下命令:
wget -Q5m -i filelist.txt

注意：这个参数对单个文件下载不起作用，只能递归下载时才有效。
```
14.使用wget -r -A下载指定格式文件
```
可以在以下情况使用该功能
1.下载一个网站的所有图片
2.下载一个网站的所有视频
3.下载一个网站的所有PDF文件
wget -r -A.pdf url
```
15.使用wget FTP下载,你可以使用wget来完成ftp链接的下载。
```
1.使用wget匿名ftp下载
wget ftp-url
2.使用wget用户名和密码认证的ftp下载
wget --ftp-user=USERNAME --ftp-password=PASSWORD url
```
