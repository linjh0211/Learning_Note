### Q1:Python 的参数传递方式

{ 一共有3种参数传递方式，具体可以看python的官方文档https://docs.python.org/zh-cn/3/tutorial/index.html}

- #### 参数默认值

  直接在函数接口定义处写 argv = default 就可以

- #### 关键字参数

  调用时需要写成

  ```python
  fun(name = 'lily')
  ```

- #### 特殊参数

  ![image-20191128180432489](/home/linjinhao/.config/Typora/typora-user-images/image-20191128180432489.png)







### Q2:Python   写setup.py

{一般的python程序都会有一个setup.py文件，用于配置项目的python环境，具体地再查一下distutils使用}

- #### call、check_call和check_output 

用来写一些bash的命令

- #### 有几个点需要注意一下

  - Extension用于C/C++拓展库的配置

  - setup里

    - ext_modules关键字用于添加动态链接库

    - package_data 用于添加一些除了代码的其它文件 





### Q3:python3 setup.py install --user中的--user是什么意思？

你想要安装一个第三方包，但是没有权限将它安装到系统Python库中去。 或者，你可能想要安装一个供自己使用的包，而不是系统上面所有用户。

Python有一个用户安装目录，通常类似”~/.local/lib/python3.3/site-packages”。 要强制在这个目录中安装包，可使用安装选项“–user”。例如：

```python3 setup.py install --user```

### 

### Q4:python项目某个文件夹下有一个__init__.py文件，它的作用是什么 ？

- 标记这是一个python的模块包
- 可以使用from 文件夹名 import （.so .py文件）

