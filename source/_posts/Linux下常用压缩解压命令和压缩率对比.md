---
title: Linux下常用压缩解压命令和压缩率对比
date: 2019-06-03 20:31:29
tags: Linux
categories: 
- 后端
---

>本篇博客仅以个人学习记录使用
>原文链接: https://www.cnblogs.com/joshua317/p/6170839.html


常用的格式有：
tar, tar.gz(tgz), tar.bz2,
不同方式，压缩和解压方式所耗CPU时间和压缩比率也差异也比较大。

1.tar ---- 只是打包动作，相当于归档处理，不做压缩；解压也一样，只是把归档文件释放出来。

(1)打包归档格式：

```
tar -cvf examples.tar files|dir
#说明：
-c, --create  create a new archive 创建一个归档文件
-v, --verbose verbosely list files processed 显示创建归档文件的进程
-f, --file=ARCHIVE use archive file or device ARCHIVE  后面要立刻接被处理的档案名,比如--file=examples.tar

#举例：
tar -cvf file.tar file1       #file1文件
tar -cvf file.tar file1 file2 #file1，file2文件
tar -cvf file.tar dir         #dir目录
```

(2)释放解压格式：

```
tar -xvf examples.tar （解压至当前目录下）
tar -xvf examples.tar  -C /path (/path 解压至其它路径)

#说明：
-x, --extract, extract files from an archive 从一个归档文件中提取文件

#举例：
tar -xvf file.tar
tar -xvf file.tar -C /temp  #解压到temp目录下
```

2.tar.gz tgz---- (tar.gz和tgz只是两种不同的书写方式，后者是一种简化书写，等同处理)

这种格式是Linux下使用非常普遍的一种压缩方式，兼顾了压缩时间（耗费CPU）和压缩空间（压缩比率）

其实这是对tar包进行gzip算法的压缩

(1)打包压缩格式：
```
tar -zcvf examples.tgz examples (examples当前执行路径下的目录)

说明：
-z, --gzip filter the archive through gzip 通过gzip压缩的形式对文件进行归档

举例：
tar -zcvf file.tgz dir #dir目录
```

(2)释放解压格式：

```
tar -zxvf examples.tar （解压至当前执行目录下）
tar -zxvf examples.tar  -C /path (/path 解压至其它路径)

举例：
tar -zcvf file.tgz
tar -zcvf file.tgz -C /temp
```

3.tar.bz ---- Linux下压缩比率较tgz大，即压缩后占用更小的空间，使得压缩包看起来更小。但同时在压缩，解压的过程却是非常耗费CPU时间。

(1)打包压缩格式:

```
tar -jcvf examples.tar.bz2 examples   (examples为当前执行路径下的目录)

说明：
-j, --bzip2 filter the archive through bzip2 通过bzip2压缩的形式对文件进行归档

举例：
tar -jcvf file.tar.bz2 dir #dir目录
```

(2)释放解压：

```
tar -jxvf examples.tar.bz2 （解压至当前执行目录下）
tar -jxvf examples.tar.bz2  -C /path (/path 解压至其它路径)

举例：
tar -jxvf file.tar.bz2
tar -jxvf file.tar.bz2 -C /temp
```

4.gzip

```
压缩：gzip -d examples.gz examples
解压：
gunzip examples.gz
````

5.zip-----zip 格式是开放且免费的，所以广泛使用在 Windows、Linux、MacOS 平台，要说 zip 有什么缺点的话，就是它的压缩率并不是很高，不如 rar及 tar.gz 等格式。

```
压缩：
zip -r examples.zip examples (examples为目录)
解压：
zip examples.zip
```

6 .rar

```
压缩：
rar -a examples.rar examples

解压：
rar -x examples.rar
```

### 压缩比率，占用时间对比

为了保证能够让压缩比率较为明显，需选取一个内容较多、占用空间较大的目录作为本次实验的测试。
找了一个大概有23G的目录来测试,首先要明确由于执行环境的变化，误差在所难免

首先明确一个概念：

压缩比率=原内容大小/压缩后大小，压缩比率越大，则表明压缩后占用空间的压缩包越小

.tar---

```
time tar -cvf test.tar /usr/test
时间：
real    3m20.709s
user    0m3.477s
sys     0m42.595s

大小：
打包前：23214680
打包后：22202984

耗时：3m20.709s
压缩比率：22202984/23214680


解压：
time tar -xvf test.tar

大小：
解压前：22202984
解压后：23211064

耗时:
real    2m47.548s
user    0m4.999s
sys     1m14.186s
```

.tgz----

```
打包压缩：
time tar -zcvf test.tgz /usr/test
时间：
real    16m30.767s
user    16m1.394s
sys     1m7.391s

大小：
打包前：23211064
打包后：18949032

耗时：
压缩比率：

解压：
tar -zxvf test.tar

大小：
解压前：18949032
解压后：23211064

耗时:
real    3m52.418s
user    2m46.325s
sys     1m21.442s
```

.tar.bz2---

```
打包压缩：
time tar -jcvf test.tar.bz2 /usr/test

时间：
real    80m39.422s
user    80m14.599s
sys     0m58.623s

大小：
打包前：23211064
打包后：18728904

耗时：80m39.422s
压缩比率：


解压：
time tar -jxvf test.tar.bz2

时间：
real    27m54.525s
user    27m44.108s
sys     1m43.645s

大小：
解压前：18728904
解压后：23211064
```

```
综上结果，初步结论：

综合起来，在压缩比率上： tar.bz2 > tgz > tar

占用空间与压缩比率成反比： tar.bz2 < tgz < tar

耗费时间（打包，解压）

打包：tar.bz2>tgz>tar

解压： tar.bz2>tar>tgz

从效率角度来说，当然是耗费时间越短越好

因此，Linux下对于占用空间与耗费时间的折衷多选用tgz格式，不仅压缩率较高，而且打包、解压的时间都较为快速，是较为理想的选择。

结论：

再一次印证了物理空间与时间的矛盾（想占用更小的空间，得到高压缩比率，肯定要牺牲较长的时间；反之，如果时间较为宝贵，要求快速，那么所得的压缩比率一定较小，当然会占用更大的空间了）。
```


