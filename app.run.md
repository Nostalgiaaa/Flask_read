# 先从官方示例开始说起

## 从头说起

打开 examples/flaskr/flaskr.py 。首先是作者的一段说明：

```text
Flaskr
~~~~~~

A microblog example application written as Flask tutorial with
Flask and sqlite3.

flaskr是一个用 Flask 框架和 sqlite3 完成的 Flask 教程示例应用
```

接下来是一个 Python 比较有趣的特性：上下文管理器，以及一些库的引用。

```text
from __future__ import with_statement  # 上下文管理器
import sqlite3  # 基于文件的数据库
from contextlib import closing
from flask import Flask, request, session, g, redirect, url_for, abort, \
     render_template, flash
```

 一些配置文件

```text
# configuration
DATABASE = 'flaskr_db.db'  # sqlite3 的文件名称
DEBUG = True  # 是否开启 debug 模式
SECRET_KEY = 'development key'  
USERNAME = 'admin'
PASSWORD = 'default'
```

 接下来开始创建实例

```text
# create our little application :)
app = Flask(__name__)  # 实例化 app
# 此处详见 http 请求前流程
app.secret_key = SECRET_KEY  
app.debug = DEBUG
```

初始化数据库 

```text
def connect_db():
    """Returns a new connection to the database."""
    return sqlite3.connect(DATABASE)


def init_db():
    """Creates the database tables."""
    with closing(connect_db()) as db:
        # 此处详见上下文管理器
        with app.open_resource('schema.sql') as f:
            db.cursor().executescript(f.read())
        db.commit()
```

http请求前后的一些功能，功能是在请求前打开数据库，结束后关闭数据库

```text
@app.before_request
# 详见装饰器和 http 请求前流程
def before_request():
    """Make sure we are connected to the database each request."""
    g.db = connect_db()
    # 详见g


@app.after_request
def after_request(response):
    # 详见装饰器和 http 请求后流程
    """Closes the database again at the end of the request."""
    g.db.close()
    return response
```

 路由转发

```text
@app.route('/')
def show_entries():
    # 直接访问的界面，显示内容转发
    cur = g.db.execute('select title, text from entries order by id desc')
    entries = [dict(title=row[0], text=row[1]) for row in cur.fetchall()]
    return render_template('show_entries.html', entries=entries)


@app.route('/add', methods=['POST'])
def add_entry():
    # 增加文章
    if not session.get('logged_in'):
        abort(401)
        # 详见 http 请求码
    g.db.execute('insert into entries (title, text) values (?, ?)',
                 [request.form['title'], request.form['text']])
    g.db.commit()
    flash('New entry was successfully posted')
    return redirect(url_for('show_entries'))


@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != USERNAME:
            error = 'Invalid username'
        elif request.form['password'] != PASSWORD:
            error = 'Invalid password'
        else:
            session['logged_in'] = True
            flash('You were logged in')
            return redirect(url_for('show_entries'))
    return render_template('login.html', error=error)


@app.route('/logout')
def logout():
    session.pop('logged_in', None)
    flash('You were logged out')
    return redirect(url_for('show_entries'))
```

最后 启动app 

```text
if __name__ == '__main__':
    app.run()
```

  


 

