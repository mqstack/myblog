title: mac 下 python、virtualenv、django配置记录
date: 2017-06-21 16:16:36
tags:
- python
- django
---

# 配置环境
安装python3

brew install

安装virtualenv

pip3 install virtualenv

创建干净环境：

virtualenv --no-site-packages venv

创建python2环境：

virtualenv -p python2 --no-site-packages venv2

激活环境：

source venv/bin/activate

退出环境：

deactivate

# mac python3  No module named '_sqlite3'

mq@mq-Mac  ~  python3
Python 3.5.2 (default, Sep 28 2016, 15:42:37)
[GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.38)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> from pysqlite2 import dbapi2 as Database
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named 'pysqlite2'
>>> from sqlite3 import dbapi2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/Cellar/python3/3.5.2_1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/sqlite3/__init__.py", line 23, in <module>
    from sqlite3.dbapi2 import *
  File "/usr/local/Cellar/python3/3.5.2_1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/sqlite3/dbapi2.py", line 27, in <module>
    from _sqlite3 import *
ImportError: No module named '_sqlite3'
>>> exit()

https://github.com/Homebrew/homebrew-core/pull/3134