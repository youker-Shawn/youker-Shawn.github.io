---
layout: post
title:  "Python环境"
date:   2022-12-23 21:56:26 +0800
categories: Python
---

若无特殊说明，默认指Linux环境下，默认指`redhat/centos`。

# macOS下Python环境

## 系统自带

~~系统自带的python27：`/System/Library/Frameworks/Python.framework/Versions/Current/bin/python`~~

macOS 12.3版本，已经把自带的**Python 2**给删了！

系统自带的python37：`/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions`

建议：不要直接用系统自带的python。

因为系统本身会用到这些自带的Python，如果自己的修改出了问题，可能引起系统崩溃！

推荐方式是，**使用macOS平台的第三方工具`homebrew`，去下载管理新版本的Python。**



## hombrew 安装Python3

可以安装多版本的Python3。

如下查看可以安装的版本：

```shell
~ ❯ brew search python
==> Formulae
app-engine-python         micropython               python-lsp-server         python-tk@3.9             python@3.7                pythran
boost-python3             ptpython                  python-markdown           python-typing-extensions  python@3.8                jython
bpython                   python-build              python-tabulate           python-yq                 python@3.9                cython
gst-python                python-gdbm@3.11          python-tk@3.10            python@3.10               reorder-python-imports
ipython                   python-launcher           python-tk@3.11            python@3.11 ✔             wxpython
```

我这里直接安装最新版本：

```sh
~ ❯ brew install python@3.11
...
```



接下来做额外的操作，为的是：

- 输入`python`或`python3` 都直接指向新安装的`python3.11`，避免调用系统默认的python3。
- 简化输入。

`pip`工具同理。

设置过程如下：

```sh
~ ❯ ln -s /usr/local/bin/python3.11 /usr/local/bin/python3
~ ❯ ln -s /usr/local/bin/python3 /usr/local/bin/python
~ ❯ ln -s /usr/local/bin/pip3.11 /usr/local/bin/pip3
~ ❯ ln -s /usr/local/bin/pip3 /usr/local/bin/pip
```

为了让自己安装的命令优先于系统自带版本的命令`/usr/bin/`，需要调整「环境变量`$PATH`」的搜索路径顺序！

```sh
~ ❯ sudo vim /etc/paths
```

将`/usr/local/bin` 放到第一行！保存退出，重启终端即可生效。

配置完成后进入的就是新安装的Python3.11了。

```sh
~ ❯ python -V 
Python 3.11.0

~ ❯ python3 -V
Python 3.11.0

~ ❯ python3.11 -V
Python 3.11.0

~ ❯ pip -V
pip 22.3.1 from /usr/local/lib/python3.11/site-packages/pip (python 3.11)

~ ❯ pip3 -V   
pip 22.3.1 from /usr/local/lib/python3.11/site-packages/pip (python 3.11)

~ ❯ pip3.11 -V
pip 22.3.1 from /usr/local/lib/python3.11/site-packages/pip (python 3.11)

```

`pip`管理的包放在`/usr/local/lib/pythonX.X/site-packages`下。



如果用`which`以及`ll`命令查看，就会发现这些都是软连接，真正指向的是`homebrew`安装在`Cellar/python@3.11`目录下的可执行程序。



## hombrew 安装Python2

Python2.x，官方不再继续维护，所以homebrew团队也不支持直接安装了！

但是参与一些老旧项目的维护时，可能还是需要搭建Python2的环境来调试，因此还是有必要了解下安装方案。

参考问答：[macos - How to reinstall python@2 from Homebrew? - Stack Overflow](https://stackoverflow.com/questions/60298514/how-to-reinstall-python2-from-homebrew#)

建议用pyenv来装python2版本。

下面使用`wget`下载ruby源码`fomula`文件再brew安装`python@2.rb`的方法已经失效了，会踩很多坑。



## pip镜像源

临时使用（比如这里的清华镜像源）：

```sh
$ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple XXX
```

可能碰到问题：

```sh
Could not fetch URL https://pypi.tuna.tsinghua.edu.cn/simple/requests/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.tuna.tsinghua.edu.cn', port=443): Max retries exceeded with url: /simple/requests/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.")) - skipping
```

解决方法是，加`--trusted-host`参数获取ssl证书，比如：

```sh
$ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.douban.com XXX  
```



默认生效新的镜像源：

创建`~/.pip/pip.conf`，内容如下。

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

或者，执行脚本创建，sh脚本内容为。

```sh
mkdir ~/.pip/
cat > ~/.pip/pip.conf <<EOF
[global]
index-url = http://pypi.douban.com/simple/ 
[install]
pypi.douban.com
EOF
```





# Python虚拟环境方案

个人推荐的方式：以下2种方案配合使用。

- `venv` ：内置方便，和项目本身关联强。

- `virtualenv` + `virtualenvwrapper` ：第三方，可以统一管理虚拟环境，方便创建和切换；可以指定虚拟环境所使用的Python版本（先安装过对应版本的Python）



##  `virtualenv` + `virtualenvwrapper` 安装和使用

使用pip安装`virtualenv` + `virtualenvwrapper`：

```sh
~ ❯ pip3.11 install virtualenv
...
Successfully installed distlib-0.3.6 filelock-3.8.2 platformdirs-2.6.0 virtualenv-20.17.1

~ ❯ pip3.11 install virtualenvwrapper
...
Successfully installed pbr-5.11.0 stevedore-4.1.1 virtualenv-clone-0.5.7 virtualenvwrapper-4.8.4

```



再配置virtualenvwrapper，添加如下。macOS下使用 zsh 终端，所以应该是`.zshrc` 文件，而不是`.bash_profile`。

```sh
# python3 虚拟环境 virtualenv + virtualenvwrapper
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python  # 一般用软连接指向了指定的版本
export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv
export WORKON_HOME='~/.virtualenvs'
source /usr/local/bin/virtualenvwrapper.sh
```

`/usr/local/bin/virtualenv`可执行程序在安装完`virtualenv` 就有了；`virtualenvwrapper.sh`脚本是在安装`virtualenvwrapper` 创建的。



执行`source ～/.zshrc`，就能看到创建了`~/.virtualenvs`目录以及一系列文件。

```sh
~ ❯ source ～/.zshrc
...
~ ❯ ls ~/.virtualenvs/
get_env_details  postdeactivate   postmkvirtualenv preactivate      premkproject     prermvirtualenv
initialize       postactivate     postmkproject    postrmvirtualenv predeactivate    premkvirtualenv
```



并且`workon` 等命令可用了。

```sh
~ ❯ mkvirtualenv lab_venv --python==python3.11
...

  lab_venv ~ ❯ deactivate

~ ❯

~ ❯ workon
lab_venv

~ ❯ workon lab_venv

  lab_venv ~ ❯
```

`mkvirtualenv`命令的参数`--python`就可以用来指定虚拟环境所使用的本机环境已安装的Python版本。



这样公共的Python3.11环境就不要再去安装各种库里，保持干净。不同项目根据需要创建虚拟环境和安装相关库即可。



## Linux下2、3版本共存时的虚拟环境设置

参与一些老项目和系统维护时，总会碰到python2的默认环境。此时又可能用到了python3，那么共存时，虚拟环境的配置需要注意个问题：

yum命令，可能依赖的是Python2版本，如果改动默认的python指向python3，会导致`yum`使用异常！

- 方案一：由Python2安装`virtualenv` + `virtualenvwrapper`。

- 方案二：Python3安装`virtualenv` + `virtualenvwrapper`。但是需要修改`virtualenvwrapper.sh`脚本，不去使用默认的`python`，避免调用python2，导致报错。替换如下：

```sh
# VIRTUALENVWRAPPER_PYTHON="$(command \which python)"
VIRTUALENVWRAPPER_PYTHON="$(command \which python3)"
```

- 方案三：使用`venv`为python3项目单独创建虚拟环境。python2则安装`virtualenv` + `virtualenvwrapper`进行使用。

个人认为方案三可能比较合适。避免不必要的改动，减少对系统影响。

当然，Linux下一般都是公共环境了，不太会出现个人需要统一管理虚拟环境的情况。所以`venv`使用的场景更多。



# Python的MySQL操作库

安装MySQL操作库之前，注意安装依赖：

```sh
$ yum -y install mysql-devel gcc gcc-devel python-devel
```

否则会碰到如下问题：

- python依赖问题`_mysql.c:37:20: fatal error: Python.h: No such file or directory`
    - 解决：执行`yum install -y python3-devel `安装依赖
- gcc缺失`unable to execute 'gcc': No such file or directory`
    - 解决：执行`yum install -y gcc` 安装依赖
- mysql_config缺失

```shell
File "setup_posix.py", line 25, in mysql_config
    raise EnvironmentError("%s not found" % (mysql_config.path,))`

EnvironmentError: mysql_config not found

# 或者

    File "/tmp/pip-install-dmuvzmv3/mysqlclient/setup_posix.py", line 31, in mysql_config
     raise OSError("{} not found".format(_mysql_config_path))
  OSError: mysql_config not found
```

- - 解决：执行`yum install -y mysql-devel`安装依赖



## 「推荐」`mysqlclient`

- MySQL-python的**Fork版本，完全兼容！**
- 使用也是：`import MySQLdb`
- **支持 Python3.x！**
- **Django ORM的依赖该工具**
- **支持使用原生 SQL** 来操作数据库

**推荐使用该库！**

pip安装:

```sh
$ pip install mysqlclient
```

可能出现问题:`ModuleNotFoundError: No module named '_ctypes'`，换如下的源码`setup`安装方式。

源码安装：

```sh
$ tar xvf mysqlclient-1.3.14.tar.gz
$ cd mysqlclient-1.3.14
$ python setup.py install
```



## 「推荐」`pymysql`

若不追求速度，可以用这个库

- 安装方便
- 调用方式不同：`import pymysql`
- **纯 Python 实现**，速度上比不上`mysqlclient`
- 也兼容 MySQL-python， **需要执行`pymysql.install_as_MySQLdb()`**

安装：

```sh
$ pip install pymysql
```



## ORM方式

- `SQLAlchemy` 既支持原生 SQL，也支持 ORM 的工具。它并不提供底层的数据库操作，而是要借助于`MySQLdb`、`PyMySQL`等第三方库来完成。
- `Django's ORM`



## 「建议不用」`MySQL-python`

- 别名：`MySQLdb`、`Python-MySQL`
- 使用：`import MySQLdb`
- 基于C开发，仅支持 Python2.x！
- 安装非常不友好，已停止维护。

**不推荐！**已经有很多衍生版本更易用。

pip安装：

```sh
$ pip install MySQLdb
# 或者
$ pip install mysql-python
```

yum方式：

```sh
$ yum install -y MySQL-python
```

 安装到了系统默认python的site-packages目录下。**一般用于 puppet 管理机器时，执行该库的统一安装。**

**安装这个库会出现非常多问题，容易踩坑。**



# Linux下的Python环境









# 第三方库的安装

**pip安装 「目前最常用」**



源码方式手动安装。

基本步骤如下：

1. wget 源码包
2. tar 解压，换root，cd 进入源码目录
3. `python setup.py install`



`easy_install`

这是Python setuptools系列工具中的一个，可以用来自动查找、下载、安装、升级依赖包。

需要先安装过`setuptool`
