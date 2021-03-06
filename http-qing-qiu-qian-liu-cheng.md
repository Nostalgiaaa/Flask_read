# 从第一行开始看起

从官方实例中， 我们可以看到这样一个import 以及实例化Flask类的操作，接下来就深入源码看一下 http 请求前，Flask 都干了什么。

```text
from flask import Flask
app = Flask(__name__)  # 参考Python相关/__name__是什么
app.secret_key = SECRET_KEY
app.debug = DEBUG
```

深入 Flask 类定义中来查看

```text
class Flask(object):
    """The flask object implements a WSGI application and acts as the central
    object.  It is passed the name of the module or package of the
    application.  Once it is created it will act as a central registry for
    the view functions, the URL rules, template configuration and much more.

    The name of the package is used to resolve resources from inside the
    package or the folder the module is contained in depending on if the
    package parameter resolves to an actual python package (a folder with
    an `__init__.py` file inside) or a standard module (just a `.py` file).

    For more information about resource loading, see :func:`open_resource`.

    Usually you create a :class:`Flask` instance in your main module or
    in the `__init__.py` file of your package like this::

        from flask import Flask
        app = Flask(__name__)
    """
    # 此处注释大概介绍了如何实例化一个 Flask 对象，介绍了 Flask 的是如何实现的。

    #: the class that is used for request objects.  See :class:`~flask.request`
    #: for more information.
    request_class = Request
    # 此处继承了 werkzeug 框架的 Request 类，把 http request 抽象成一个类

    #: the class that is used for response objects.  See
    #: :class:`~flask.Response` for more information.
    response_class = Response
    # 此处同上， 抽象的是 http response

    #: path for the static files.  If you don't want to use static files
    #: you can set this value to `None` in which case no URL rule is added
    #: and the development server will no longer serve any static files.
    static_path = '/static'
    # 静态资源文件路径

    #: if a secret key is set, cryptographic components can use this to
    #: sign cookies and other things.  Set this to a complex random value
    #: when you want to use the secure cookie for instance.
    secret_key = None
    # 加密 cookie 的设置

    #: The secure cookie uses this for the name of the session cookie
    session_cookie_name = 'session'
    # session 名称

    #: options that are passed directly to the Jinja2 environment
    jinja_options = dict(
        autoescape=True,
        extensions=['jinja2.ext.autoescape', 'jinja2.ext.with_']
    )
    # jinja2 设置

```

 接下来是 \_\_init\_\_ 方法

```text
def __init__(self, package_name):
    # 此处package_name 就是你传入的 __name__ 也就是字符串 __name__ 
    # 下面这段英文注释大概意思是如果把 debug 模式打开，所有 debug 信息都会被弹出
    # 如果 app 文件有变更会自动重启。
    #: the debug flag.  Set this to `True` to enable debugging of
    #: the application.  In debug mode the debugger will kick in
    #: when an unhandled exception ocurrs and the integrated server
    #: will automatically reload the application if changes in the
    #: code are detected.
    self.debug = False

    #: the name of the package or module.  Do not change this once
    #: it was set by the constructor.
    # 大意：别改这个
    self.package_name = package_name

    #: where is the app root located?
    # 获取app的绝对路径，函数的实现方式是用os模块获取的 
    self.root_path = _get_package_path(self.package_name)

    #: a dictionary of all view functions registered.  The keys will
    #: be function names which are also used to generate URLs and
    #: the values are the function objects themselves.
    #: to register a view function, use the :meth:`route` decorator.
    # view_functions 是一个存放已注册的视图函数字典，字典的key是函数名(所以你同一个app下不可以有两个
    # 相同名字的函数，否则会报错), key也被用为生成url。字典的values就是视图函数本身。
    self.view_functions = {}

    #: a dictionary of all registered error handlers.  The key is
    #: be the error code as integer, the value the function that
    #: should handle that error.
    #: To register a error handler, use the :meth:`errorhandler`
    #: decorator.
    # 这个特性是不是之后去掉了或者很少用？不过也是一种捕捉错误的思路。大概意思是这个字典是存储了所有的
    # 错误处理。 key 是错误码，value 是错误处理函数。
    self.error_handlers = {}

    #: a list of functions that should be called at the beginning
    #: of the request before request dispatching kicks in.  This
    #: can for example be used to open database connections or
    #: getting hold of the currently logged in user.
    #: To register a function here, use the :meth:`before_request`
    #: decorator.
    # 这个 list 会存储所有 http 真正请求进来之前你想要执行的函数，这个例子里
    # 打开了数据库连接，或者保持用户的登录状态。
    self.before_request_funcs = []

    #: a list of functions that are called at the end of the
    #: request.  Tha function is passed the current response
    #: object and modify it in place or replace it.
    #: To register a function here use the :meth:`after_request`
    #: decorator.
    # 基本同上，区别就是请求之后你想执行的函数。
    self.after_request_funcs = []

    #: a list of functions that are called without arguments
    #: to populate the template context.  Each returns a dictionary
    #: that the template context is updated with.
    #: To register a function here, use the :meth:`context_processor`
    #: decorator.
    # 此处是一些常用的上下文，包括常见的 request 对象，g 对象， session 对象
    # 这里会单独拿出一段文章讲如何实现的全局变量。
    self.template_context_processors = [_default_template_ctx_processor]
    
    # 存储 url 和函数的匹配规则。注意，此处不是 Python 中的 Map 而是 werkzeug 中的 Map
    self.url_map = Map()
    
    if self.static_path is not None:
        # 判断你有没有建立存放静态资源的文件夹
        self.url_map.add(Rule(self.static_path + '/<filename>',
                              build_only=True, endpoint='static'))
        if pkg_resources is not None:
            target = (self.package_name, 'static')
        else:
            target = os.path.join(self.root_path, 'static')
        # 这里 SharedDataMiddleware 函数 是 werkzeug 里的函数， 功能是为你的app
        # 增加了静态资源。
        self.wsgi_app = SharedDataMiddleware(self.wsgi_app, {
            self.static_path: target
        })

    #: the Jinja2 environment.  It is created from the
    #: :attr:`jinja_options` and the loader that is returned
    #: by the :meth:`create_jinja_loader` function.
    # 此处是 Flask 选用的模板框架，jinja2 此处都是设置 jinja2
    self.jinja_env = Environment(loader=self.create_jinja_loader(),
                                 **self.jinja_options)
    self.jinja_env.globals.update(
        url_for=url_for,
        get_flashed_messages=get_flashed_messages
    )
```

 到此处 其实才算真正的执行完 

```text
app = Flask(__name__)
```

 继续往下看， connect\_db 和 init\_db 两个函数都是操作数据库的函数， 暂且不表。到了下一个函数before\_request 有一个装饰器 

```text
@app.before_request
```

```text
def before_request(self, f):
    # 这个装饰器的作用很简单，把被装饰的函数添加到在 __init__ 中创建的
    # self.before_request_funcs 中
    """Registers a function to run before each request."""
    self.before_request_funcs.append(f)
    return f

@app.after_request  # 这个实现方式类似，不多赘述。
```

 继续看实现路由的装饰器

```text
@app.route('/')
```

```text
def route(self, rule, **options):
    """A decorator that is used to register a view function for a
    given URL rule.  Example::

        @app.route('/')
        def index():
            return 'Hello World'

    Variables parts in the route can be specified with angular
    brackets (``/user/<username>``).  By default a variable part
    in the URL accepts any string without a slash however a different
    converter can be specified as well by using ``<converter:name>``.

    Variable parts are passed to the view function as keyword
    arguments.

    The following converters are possible:

    =========== ===========================================
    `int`       accepts integers
    `float`     like `int` but for floating point values
    `path`      like the default but also accepts slashes
    =========== ===========================================

    Here some examples::

        @app.route('/')
        def index():
            pass

        @app.route('/<username>')
        def show_user(username):
            pass

        @app.route('/post/<int:post_id>')
        def show_post(post_id):
            pass       
    # 重点一： 可以在 url 中加入变量, 这个变量会以参数的形式传给被装饰的函数
    An important detail to keep in mind is how Flask deals with trailing
    slashes.  The idea is to keep each URL unique so the following rules
    apply:

    1. If a rule ends with a slash and is requested without a slash
       by the user, the user is automatically redirected to the same
       page with a trailing slash attached.
    2. If a rule does not end with a trailing slash and the user request
       the page with a trailing slash, a 404 not found is raised.

    This is consistent with how web servers deal with static files.  This
    also makes it possible to use relative link targets safely.
    # 重点二： 如果你写 url 规则以 '/' 结尾，用户即使请求 url 结尾没加 '/' 也是可以请求到
    # 如果你 url 规则结尾本身就没有 '/' 那么用户请求 url 结尾加了 '/' 则返回 404

    The :meth:`route` decorator accepts a couple of other arguments
    as well:

    :param rule: the URL rule as string
    :param methods: a list of methods this rule should be limited
                    to (``GET``, ``POST`` etc.).  By default a rule
                    just listens for ``GET`` (and implicitly ``HEAD``).
    # 此处函数常用，通过函数可以选择监听哪种 http 请求。如果只监听了 GET 请求
    # 用户请求的是 POST 方法，这个请求会被拒绝。               
    
    :param subdomain: specifies the rule for the subdoain in case
                      subdomain matching is in use.
    :param strict_slashes: can be used to disable the strict slashes
                           setting for this rule.  See above.
    :param options: other options to be forwarded to the underlying
                    :class:`~werkzeug.routing.Rule` object.
    """
    def decorator(f):
        # 见下文
        self.add_url_rule(rule, f.__name__, **options)
        self.view_functions[f.__name__] = f
        return f
    return decorator
```

 详细看下 add\_url\_rule

```text
def add_url_rule(self, rule, endpoint, **options):
    """Connects a URL rule.  Works exactly like the :meth:`route`
    decorator but does not register the view function for the endpoint.

    Basically this example::

        @app.route('/')
        def index():
            pass

    Is equivalent to the following::

        def index():
            pass
        app.add_url_rule('index', '/')
        app.view_functions['index'] = index

    :param rule: the URL rule as string
    :param endpoint: the endpoint for the registered URL rule.  Flask
                     itself assumes the name of the view function as
                     endpoint
    :param options: the options to be forwarded to the underlying
                    :class:`~werkzeug.routing.Rule` object
    """
    # 举个例子，比如 show_entries 函数的 options 是：
    # {'endpoint': 'show_entries', 'methods': ('GET',)}
    # 其实就是把所有的信息装入 dict 中
    options['endpoint'] = endpoint
    options.setdefault('methods', ('GET',))
    # 此处是使用 Map 对象的 add 方法 把 Rule 中所有的转发规则绑定。
    # 引用的都是 werkzeug 中的方法，属于 Flask 的再底层。所以先标为 TODO
    self.url_map.add(Rule(rule, **options))
```

综上，Flask 会通过装饰器的形式，把你所写的  url 转发规则以及函数一一映射并且存储起来。 

定义完了路由之后，来启动 app 吧~ 

```text
app.run()
```

 

```text
def run(self, host='localhost', port=5000, **options):
    """Runs the application on a local development server.  If the
    :attr:`debug` flag is set the server will automatically reload
    for code changes and show a debugger in case an exception happened.

    :param host: the hostname to listen on.  set this to ``'0.0.0.0'``
                 to have the server available externally as well.
    :param port: the port of the webserver
    :param options: the options to be forwarded to the underlying
                    Werkzeug server.  See :func:`werkzeug.run_simple`
                    for more information.
    """
    # 此处没什么好具体说明的，是否开启 debug 模式， 改变文件重启模式， 调用 werkzeug
    # 中的 run_simple 方法来启动一个简单的 server
    from werkzeug import run_simple
    if 'debug' in options:
        self.debug = options.pop('debug')
    options.setdefault('use_reloader', self.debug)
    options.setdefault('use_debugger', self.debug)
    return run_simple(host, port, self, **options)
```

 

