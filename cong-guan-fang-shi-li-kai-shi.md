# 代码结构

## 版本

用source tree 切到了tag 0.1 上，这是作者第一个发布版本， 从这个版本以后就有多人协作了，也就是说这个版本其中的代码风格完全是作者的个人风格， 非常值得一看。

## 结构

### 字符示意图

```text
├─ LICENSE
├─ Makefile
├─ README
├─ artwork
│    └─ logo-full.svg
├─ docs
│    ├─ .gitignore
│    ├─ Makefile
│    ├─ _static
│    │    ├─ debugger.png
│    │    ├─ flask.png
│    │    ├─ flaskr.png
│    │    └─ logo-full.png
│    ├─ _templates
│    │    ├─ sidebarintro.html
│    │    └─ sidebarlogo.html
│    ├─ _themes
│    │    └─ flasky
│    ├─ api.rst
│    ├─ becomingbig.rst
│    ├─ conf.py
│    ├─ deploying.rst
│    ├─ flaskext.py
│    ├─ foreword.rst
│    ├─ index.rst
│    ├─ installation.rst
│    ├─ make.bat
│    ├─ patterns.rst
│    ├─ quickstart.rst
│    ├─ testing.rst
│    └─ tutorial.rst
├─ examples
│    ├─ .DS_Store
│    ├─ flaskr
│    │    ├─ .DS_Store
│    │    ├─ README
│    │    ├─ flaskr.py
│    │    ├─ flaskr_db.db
│    │    ├─ flaskr_tests.py
│    │    ├─ schema.sql
│    │    ├─ static
│    │    └─ templates
│    └─ minitwit
│           ├─ README
│           ├─ minitwit.py
│           ├─ minitwit_tests.py
│           ├─ schema.sql
│           ├─ static
│           └─ templates
├─ flask.py
├─ setup.py
├─ tests
│    ├─ flask_tests.py
│    ├─ static
│    │    └─ index.html
│    └─ templates
│           ├─ context_template.html
│           └─ escaping_template.html
└─ website
       ├─ index.html
       └─ logo.png
```

其中核心组件就是 flask.py  只有664行。

## 官方examples

其中examples文件夹下有两个例子 一个是 flaskr，另一个是 minitwit。我们从flaskr开始入手

