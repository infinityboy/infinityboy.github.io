---
title: mac下安装node(强迫症版)
date: 2019-06-03 21:27:57
tags: 杂七杂八
categories: 
- Mac
---

>本篇博客仅以个人学习记录使用

mac 下拥有homebrew包管理器: 

1.直接使用brew install node 便可以安转最新版本的node

2.先使用brew search node 可以找到当前node的版本 然后再通过 brew install node@10这样可安装指定版本

从nodejs 官网下载pkg的nodejs安装包安装

但是通常在mac上安装都回通过homebrew包管理器去安装，如果这个时候单单node使用从安装包下载的方式安装就很有可能与mac的包管理器所安装的环境变量等有所冲突，所以推荐统一使用包管理器去安装，这样方便包管理器的统一管理 

那么如果之前并不知道包管理的存在，或者未使用包管理器安装，匆忙的使用了安装包去安装，但是后来又想通过包管理器去安装，这样产生的冲突怎么解决呢？

先直接一条命令brew uninstall node 卸载包管理器安装的node

通常在使用安装包的默认安装情况下：

```
先执行：
sudo rm -rf /usr/local/{bin/{node,npm},lib/node_modules/npm,lib/node,share/man//node.} 
将local中的node清除
```

但是往往仅用这一条命令是不够的：

1.删除/usr/local/lib中的所有node和node_modules  
2.删除/usr/local/lib中的所有node和node_modules的文件夹  
3.如果是从brew安装的, 运行brew uninstall node  
4.检查~/中所有的local, lib或者include文件夹, 删除里面所有node和node_modules  
5.在/usr/local/bin中, 删除所有node的可执行文件  
6.最后运行以下代码:(可能具体安装路径会有区别 ,find ~ -name "node"   可以找到所有  
```
    sudo rm /usr/local/bin/npm
    sudo rm /usr/local/share/man/man1/node.1
    sudo rm /usr/local/lib/dtrace/node.d
    sudo rm -rf ~/.npm
    sudo rm -rf ~/.node-gyp
    sudo rm /opt/local/bin/node
    sudo rm /opt/local/include/node
    sudo rm -rf /opt/local/lib/node_modules
```

注意使用rm命令的风险，尽量确定所将要删除的文件是安全的

这时可能仍然需要使用find ~ -name "node" 和 find ~ -name "npm"  查找缺漏的进行逐一删除

当以上操作全部做完，执行brew doctor 命令检查，当出现相关node警告或错误时，根据提示的文件指向再进行逐一清除。清除过后再次执行brew doctor 检查一遍 ，当出现提示 Your system is ready to brew. 这时便可以再次使用brew install node 命令重新安装node。 这时再出现The `brew link` step did not complete successfully 这个错误提示时，就按提示执行brew link --overwrite node 将link绑定到node上，即成功安装node。（如果在重装node之后提示'brew link' 下面的提示上最终提示文件指向node/xx，如：/usr/local/share/doc/node/gdbinit，直接将路径下node/文件夹rm掉，再重新安装即可）。
