# 文档地址
https://docs.pytest.org/en/latest/getting-started.html#install-pytest

# 测试用例的识别与运行
- 当前目录及其子目录:已test_开头或结尾
    - test_*.py
    - *_test.py
- 用例识别
    - 文件以test开头、类以Test开头，方法以test_开头或以_test结尾
    - Test*类包含的所有test_*的方法(测试类不能含有_init__方法)
    - 不在class中的所有test_*方法
- 可执行unittest用例

# pytest终端执行
- pytest
    * 搜索当前目录下所有已test_开头或以_test结尾的.py文件
- -s
    * 在控制台输出详情
- -v
    * 打印详情运行日志信息(什么文件下的什么类下的什么方法)
- pytest file.py
    * 执行file.py文件进行执行
- pytest file.py :: 类名
    * 指定file.py文件中的类明进行执行
- pytest file.py :: 类名 :: 方法名
    * 执行测试方法
- pytest -k file.py "类名 and not 方法名"
    * 跳过运行某一个用例
- pytest -m[标记名]
    * @pytest.mark[标记名] 运行执行标记名的用例
- pytest -x file.py
    * 运行报错就停止运行
- pytest --maxfail=[num]
    * 错误达到num的时候就停止运行

# Pytest执行失败重新运行
- 测试失败后重新运行N次，要在重新运行之间添加延迟时间
- 1、使用pytest-rerunfailures
- 2、pip install pytest-rerunfailures
- 3、执行
    * pytest --reruns 3 -v -s test_file.py
    * pytest -v reruns 5 --reruns-delay 2
        * --reruns-delay 2 ： 表示间隔2秒

# 多条断言有失败也都运行
- 一个方法中国写多条断言，通常第一条报错后面的断言就不执行。
- 1、使用pytest-assume
- 2、pip install pytest-assume
- 3、pytest.assume(表达式)
    * 断言方法不使用assert使用pytest.assume(表示式)

# pytest框架结构
import pytest 类似setip,teardown同样更灵活
- 模块级(setup_module / teardown_module)模块始末，全局(优先最高)
- 函数级(setup_function / teardown_function)只对函数用例生效(不在类中)
- 类级(setup_class / teardown_class)只再类中前后运行(在类中)
- 方法级(setup_method / teardown_mothod)开始于方法始末(在类中)
- 类里面的(setup / teardown)运行在条用方法前后

``` python3

#!/usr/bin/python3
#-*- coding: utf-8 -*-

import pytest


def setup_module():
    print("setup_module")


def teardown_module():
    print("teardown_module")


def setup_function():
    print("setup_function")


def teardown_function():
    print("teardown_function")


def test_login():
    print("\n 这是一个外包方法")


class TestDemo():
    def setup_class(self):
        print("setup_class")

    def setup_method(self):
        print("setup_method")

    def setup(self):
        print("setup")

    def teardown_class(self):
        print("teardown_class")

    def teardown_method(self):
        print("teardown_method")

    def teardown(self):
        print("teardown")

    def test_one(self):
        print("\n run test_one")
        assert(1==1)

    def test_two(self):
        print("\n run test_two")
        assert(2==2)


执行结果：

test_demo.py::test_login setup_module
setup_function

 这是一个外包方法
PASSEDteardown_function

test_demo.py::TestDemo::test_one setup_class
setup_method
setup

 run test_one
PASSEDteardown
teardown_class

test_demo.py::TestDemo::test_two setup_class
setup

 run test_two
PASSEDteardown
teardown_method
teardown_class
teardown_module

```


# pytest.fixture
被pytest.fixtur()装饰的方法，通过方法名可直接调用
- K

``` python3
#!/usr/bin/python3
#-*- coding: utf-8 -*-

import pytest
import random


@pytest.fixture()
def get_token():
    token = random.randint(1, 100)
    print("\n登录获取token" + str(token))
    return token


class TestDemo():

    def test_one(self, get_token):
        print("\n run test_one")
        assert(1==1)

    def test_two(self):
        print("\n run test_two")
        assert(2==2)

执行结果：

test_demo.py::TestDemo::test_one
登录获取token91

 run test_one
PASSED
test_demo.py::TestDemo::test_two
 run test_two
PASSED

```



- conftest.py 
    * 通过conftest.py文件进行数据共享，放在不同的位置起着不同的范围共享作用

## conftest.py
- conftest文件名是不能换的
- conftest.py与运行的用例在同一个package下，并且有__init__.py文件
- 不需要import导入conftest.py,pytest用例会自动查找
- 全局的配置和前期工作都可以写在这里，放在某个包下，就是这个包数据共享的地方


## yieId
测试完毕后销毁清除数据
- 通过yieId关键字
- 第一次调用返回结果、第一次执行它下面的语句返回
- @pytest.fixture(scope=module)
- 没有返回值，希望返回使用addfinalizer
- 作用域
    - scope=“module”：  模块
    - scope="class"：   类
    - scope="session"： 全部
    - 优先级：
        - session-->module–->class–->function


``` python3
#!/usr/bin/python3
#-*- coding: utf-8 -*-

import pytest
import random


@pytest.fixture()
def get_token():
    print("yieId 关键字前")
    yield   # 在这个位置先return None,第二次的时候再从这个位置的下语句开始
    print("yieId后")


class TestDemo():

    def test_one(self, get_token):
        print("\n run test_one")
        assert(1==1)

    def test_two(self):
        print("\n run test_two")
        assert(2==2)

if __name__ == '__main__':
    pytest.main()

执行结果：
test_demo.py::TestDemo::test_one yieId 关键字前

 run test_one
PASSEDyieId后

test_demo.py::TestDemo::test_two
 run test_two
PASSED

```

## autouse=True
把pytest.fixture修复的方法应用到所以的测试用例当中

``` python3
#!/usr/bin/python3
#-*- coding: utf-8 -*-

import pytest
import random


@pytest.fixture(autouse=True)
def get_token():
    print("yieId 关键字前")
    yield
    print("yieId后")


class TestDemo():

    def test_one(self):
        print("\n run test_one")
        assert(1==1)

    def test_two(self):
        print("\n run test_two")
        assert(2==2)

if __name__ == '__main__':
    pytest.main("-s -v")

执行结果：
collected 2 items

test_demo.py::TestDemo::test_one yieId 关键字前

 run test_one
PASSEDyieId后

test_demo.py::TestDemo::test_two yieId 关键字前

 run test_two
PASSEDyieId后
```


## 参数化 pytest.mark.parametrize(argnames, argvalues)
- argnames:要参数的变量，string(逗号分开),list,tuple
- argnames: 参数化的值，list,list[tuple]
- pytest.mark.parametrize("a","b")
- 参数组合
- pytest.mark.parametrize(*,*,indirect=True)
    * indirect=True 开启传入的参数可作为函数处理，也就是可直接传入函数名


``` python3

#!/usr/bin/python3
#-*- coding: utf-8 -*-

import pytest


class TestDemo():

    @pytest.mark.parametrize("username, pwd", [("zhangsan", "Ab123456"), ("lise", "Aa123456")])
    def test_one(self, username, pwd):
        print("\n run test_one")
        print("\n" + username)
        assert(1==1)

if __name__ == '__main__':
    pytest.main("-s -v")


执行结果：

collected 2 items

test_demo.py::TestDemo::test_one[zhangsan-Ab123456]
 run test_one

zhangsan
PASSED
test_demo.py::TestDemo::test_one[lise-Aa123456]
 run test_one

lise
PASSED

```

## yaml参数化
``` python3
# 1、yaml表示二维数组
''' yamlBasic.yml 文件
-
 - by:id
 - locator:name
 - action:clickpwd

-
 - by:class
 - locator:tag
 - action:clickusername
'''

print(yaml.safe_load(open('yamlBasicData.yml')))

# [['by:id', 'locator:name', 'action:clickpwd'], ['by:class', 'locator:tag', 'action:clickusername']]


# 2、yaml表示字典


```



## pytest.mark.skip()
- 无条件跳过，season=“跳过问文本”

## pytest.mark.ifskip()
- 第一个参数为表达式，season为注解

## pytest.mark.xfail()

## 使用自定义标记mrak只执行某部分用例
- 在测试用例方法上加@pytest.mark.webtest(webtest可以自己定义)
- 执行
    - -s参数：输出所有测试用的print信息
    - -m:执行自定义标记相关用例 pytest -s test_mrak_zi_09.py

# 多线程并行与分布式执行
- pytest分布式执行插件：pytest-xdist.
- pytest -n 3: 并行3个线程

# pytest-html
作用：生成测试报告
- pytest -v -s --html=report.html --self-contailned-html


# 断言预期内的异常
``` python3
def test_zero_dicision():
    with pytest.raises(ZeroDivisionError):
        1 / 0
```



