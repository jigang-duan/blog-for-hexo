---
title: Python多版本配置
date: 2017-07-15 15:47:10
tags:
- python
categories:
- python

---

![2Fban2.png](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510742685480&di=a6c8329698b90d882a5b3491ace56161&imgtype=0&src=http%3A%2F%2Fwww.njrst.com.cn%2Fbaiduzt%2Fpython%2Fimages%2Fban2.png)


Python易用，但用好却不易，其中比较头疼的就是包管理和Python不同版本的问题。为了解决这些问题，有不少发行版的Python，比如WinPython、Anaconda等，这些发行版将python和许多常用的package打包，方便pythoners直接使用，此外，还有virtualenv、pyenv等工具管理虚拟环境。

一般网上比较多的是使用pip + pyenv + virtualenv。
开始尝试pyenv，但安装过程中遇见很多的坑，最终选择了Anaconda。

<!-- more -->

## pyenv

经常遇到这样的情况：

* 系统自带的Python是2.x，自己需要Python 3.x，测试尝鲜；
* 系统是2.6.x，开发环境是2.7.x
* 由于Mac机器系统保护的原因，默认的Python中无法对PIP一些包升级，需要组建新的Python环境。
* 此时需要在系统中安装多个Python，但又不能影响系统自带的Python，即需要实现Python的多版本共存。pyenv就是这样一个Python版本管理器。

### 安装pyenv

```bash
brew install pyenv
```

### 查看当前激活的是那个版本的Python

```bash
$ pyenv version
system (set by /Users/jiang.duan/.pyenv/version)
```

### 查看已经安装了那些版本的Python

```bash
$ pyenv versions
* system (set by /Users/jiang.duan/.pyenv/version)
```

### 安装指定版本的Python

```bash
pyenv install 3.5.0
```

**遇到下面错误**：

```
Downloading Python-3.5.0.tar.xz...
-> https://www.python.org/ftp/python/3.5.0/Python-3.5.0.tar.xz
Installing Python-3.5.0...

BUILD FAILED (OS X 10.12.6 using python-build 20160602)

Inspect or clean up the working tree at /var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115092041.13510
Results logged to /var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115092041.13510.log

Last 10 log lines:
  File "/private/var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115092041.13510/Python-3.5.0/Lib/ensurepip/__main__.py", line 4, in <module>
    ensurepip._main()
  File "/private/var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115092041.13510/Python-3.5.0/Lib/ensurepip/__init__.py", line 209, in _main
    default_pip=args.default_pip,
  File "/private/var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115092041.13510/Python-3.5.0/Lib/ensurepip/__init__.py", line 116, in bootstrap
    _run_pip(args + [p[0] for p in _PROJECTS], additional_paths)
  File "/private/var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115092041.13510/Python-3.5.0/Lib/ensurepip/__init__.py", line 40, in _run_pip
    import pip
zipimport.ZipImportError: can't decompress data; zlib not available
make: *** [install] Error 1
```

原因：

```
zipimport.ZipImportError: can't decompress data; zlib not available
```

详情看这里：[issues/25](https://github.com/yyuu/pyenv/issues/25)

执行下面的代码即可解决这个问题：

```bash
CFLAGS="-I$(xcrun --show-sdk-path)/usr/include" pyenv install -v 3.5.0
```

**遇到下面错误**：
```
warning: xz not found; consider installing `xz` package
/var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115122939.26443/Python-3.5.0 /var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115122939.26443 ~/WorkSpace/js/hexo/jigang-duan.github.io/hexo/jigang
checking build system type... x86_64-apple-darwin16.7.0
checking host system type... x86_64-apple-darwin16.7.0
checking for --enable-universalsdk... no
checking for --with-universal-archs... no
checking MACHDEP... darwin
checking for --without-gcc... no
checking for gcc... clang
checking whether the C compiler works... no
configure: error: in `/var/folders/ks/yhszh_sx2qzb22rn46b145xr0000gn/T/python-build.20171115122939.26443/Python-3.5.0':
configure: error: C compiler cannot create executables
See `config.log' for more details
make: *** No targets specified and no makefile found.  Stop.
```

详情看这里：[issues/843](https://github.com/pyenv/pyenv/issues/843)

从中发现了Anaconda, pyenv坑多，所以放弃，改用Anaconda。


## Anaconda

[介绍](http://www.jianshu.com/p/169403f7e40c)

是不是在开始时就遇到各种麻烦呢？

* 到底该装 Python2 呢还是 Python3 ？
* 为什么安装 Python 时总是出错？
* 怎么安装工具包呢？
* 为什么提示说在安装这个工具前必须先安装一堆其他不明所以的工具？

### Anaconda的安装

Anaconda的下载页参见[官网](https://www.continuum.io/downloads)下载，Linux、Mac、Windows均支持。

下载命令行版本 ：

[macOS installer](https://www.anaconda.com/downloads#macos)

安装：

```bash
bash ~/Downloads/Anaconda2-5.0.1-MacOSX-x86_64.sh
```

对于Mac、Linux系统，Anaconda安装好后，实际上就是在主目录下多了个文件夹（~/anaconda）而已，Windows会写入注册表。安装时，安装程序会把bin目录加入PATH（Linux/Mac写入~/.bashrc，Windows添加到系统变量PATH），这些操作也完全可以自己完成。以Linux/Mac为例，安装完成后设置PATH的操作是

```bash
# 将anaconda的bin目录加入PATH，根据版本不同，也可能是~/anaconda3/bin
echo 'export PATH="~/anaconda3/bin:$PATH"' >> ~/.bashrc
# 更新bashrc以立即生效
source ~/.bashrc
```

检查是否正确

```bash
$ conda --version
conda 4.3.30
$ python --version
Python 3.6.3 :: Anaconda, Inc.
```

### Conda的环境管理

创建一个名为python27的环境，指定Python版本是2.7（不用管是2.7.x，conda会为我们自动寻找2.7.x中的最新版本）

```bash
$ conda create --name py27 python=2.7

Package plan for installation in environment /Users/jiang.duan/anaconda3/envs/py27:

```

显示所有的环境：

```bash
$ conda env list
# conda environments:
#
py27                     /Users/jiang.duan/anaconda3/envs/py27
root                  *  /Users/jiang.duan/anaconda3
```

安装好后，使用activate激活某个环境:

```bash
$ source activate py27
```

查看环境：

```bash
(py27) $ conda env list
# conda environments:
#
py27                  *  /Users/jiang.duan/anaconda3/envs/py27
root                     /Users/jiang.duan/anaconda3
(py27) $ python --version
Python 2.7.14 :: Anaconda, Inc.
```

退出当前环境：

```bash
$ source deactivate
```

删除名为 py27 的环境：

```bash
$ conda env remove -n py27

Package plan for package removal in environment /Users/jiang.duan/anaconda3/envs/py27:

```

查看环境：

```bash
jiangduandeiMac:py jiang.duan$ conda env list
# conda environments:
#
root                  *  /Users/jiang.duan/anaconda3
```

用户安装的不同python环境都会被放在目录`~/anaconda/envs`下，可以在命令中运行`conda info -e`查看已安装的环境，当前被激活的环境会显示有一个星号或者括号。

### Conda的包管理

Conda的包管理就比较好理解了，这部分功能与pip类似。

conda的一些常用操作如下：

```bash
# 查看当前环境下已安装的包
conda list
 
# 查看某个指定环境的已安装包
conda list -n python34
 
# 查找package信息
conda search numpy
 
# 安装package
conda install -n python34 numpy
# 如果不用-n指定环境名称，则被安装在当前活跃环境
# 也可以通过-c指定通过某个channel安装
 
# 更新package
conda update -n python34 numpy
 
# 删除package
conda remove -n python34 numpy
```

conda将conda、python等都视为package，因此，完全可以使用conda来管理conda和python的版本，例如

```bash
# 更新conda，保持conda最新
conda update conda
 
# 更新anaconda
conda update anaconda
 
# 更新python
conda update python
# 假设当前环境是python 3.4, conda会将python升级为3.4.x系列的当前最新版本
```


> 补充：如果创建新的python环境，比如2.7，运行conda create -n python27 python=2.7之后，conda仅安装python 2.7相关的必须项，如python, pip等，如果希望该环境像默认环境那样，安装anaconda集合包，只需要：


```bash
# 在当前环境下安装anaconda包集合
conda install anaconda
 
# 结合创建环境的命令，以上操作可以合并为
conda create -n python34 python=3.4 anaconda
# 也可以不用全部安装，根据需求安装自己需要的package即可
```

#### 分享环境

当分享代码的时候，同时也需要将运行环境分享给大家，执行如下命令可以将当前环境下的 package 信息存入名为 environment 的 YAML 文件中。

```bash
conda env export > environment.yaml
```

同样，当执行他人的代码时，也需要配置相应的环境。这时你可以用对方分享的 YAML 文件来创建一摸一样的运行环境。

```bash
conda env create -f environment.yaml
```

