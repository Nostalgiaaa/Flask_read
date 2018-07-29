# \_\_name\_\_ 是什么

想要知道 \_\_name\_\_是什么东西， 我们来做个简单的实验来直观的理解下他是干嘛用的。详情见我的配套项目 Example

目录结构：

```text
python_related
    ├── __init__.py
    └── what_is_main
        ├── __init__.py
        ├── folder_0
        │   ├── __init__.py
        └── folder_1
            └── __init__.py

```

```python
在 folder_1 中的 __init__.py 写入这样一行话：
print(__name__)
运行 folder_1 下的 __init__.py
返回 __main__

在 folder_0 中的 __init__.py 写入 
from python_related.what_is_main import folder_1
运行 folder_0 下的 __init__.py
返回 python_related.what_is_main.folder_1

Python是一种动态语言，会逐行把你的代码翻译成机器能看懂的内容去执行，所以这两种的顺序
如下

运行folder_1 : 
直接执行 print(__name__) 因为我是程序的主入口，我的名字 Python 规定为 __main__

运行folder_0 : 
首先我想引入 folder_1 开始翻译 folder_1 的内容 -> 发现语句 print(__name__) 
我不是主入口，显示我的包结构 也就是 python_related.what_is_main.folder_1
```

\_\_name\_\_ 是一个 Python的 内置变量，如果这个文件是程序入口的话，\_\_name\_\_ 的值就是 \_\_main\_\_。如果不是的话，值为包结构。

大概讲了下 \_\_main\_\_是什么东西， 接下来继续，一般Python文件中有很多类似的代码：

```python
if __name__ == "__main__":
    # dosomething
    pass
```

现在可以理解是什么含义了， 只有我这个脚本文件是主入口，才去执行 if 缩进里面的内容，被 import 的时候不执行里面的内容，所以这里面一般会写一些测试用的脚本，而实际去在服务器运行的时候这段代码就无用了。

顺便一提，Python在运行的时候会从你的主入口脚本的第一行开始执行。

