> 官方文档说：测试也不必局限于单个装置。它们可以依赖于您想要的任意多个装置，装置也可以使用其他装置。这才是pytest的夹具系统真正闪耀的地方。
> 如果能让事情变得更干净，不要害怕拆散。

# 入门操作
```python
import pytest

@pytest.fixture()
def func():
    print("我是前置 func")
    yield 12
    print("我是后置 func")

def test_data(func):
    assert func == 12
    print("我是 test_data")

def test_func(func):
    print("我是func函数")
```
```python
(venv) D:\Python_test\pythonpp\pytest_>pytest -vs test_b.py
================================================================================= test session starts =================================================================================
platform win32 -- Python 3.9.5, pytest-7.2.0, pluggy-1.0.0 -- D:\Python_test\venv\Scripts\python.exe
cachedir: .pytest_cache
rootdir: D:\Python_test\pythonpp\pytest_, configfile: pytest.ini
plugins: ordering-0.6
collected 2 items

test_b.py::test_data 我是前置 func
我是 test_data
PASSED我是后置 func

test_b.py::test_func 我是前置 func
我是func函数
PASSED我是后置 func


================================================================================== 2 passed in 0.05s =======================
```
> 1. 与 setup、teardown 类似，提供了测试执行前和执行后的动作处理，但是相对来说又比 setup、teardown 好用
> 2. fixture 通过 yield 来区分前后置，前后置可以单独存在；fixture 如果有后置，都会执行后置(除非前置报错)
> 3. fixture 可用于封装数据，也可用于封逻辑动作，使用范围非常广
> 4. fixture 可用于代码模块化、数据处理、流程设计等

# fixture的执行方式
## 函数引用
```python
import pytest

@pytest.fixture()
def func():
    print("我是前置 func")
    yield 12
    print("我是后置 func")

def test_data(func):
    assert func == 12
    print("我是 test_data")
```
## 自动适配-所有用例
```python
import pytest

@pytest.fixture(autouse=True)
def func():
    print("我是前置 func")
    yield 12
    print("我是后置 func")

def test_func():
    print("我是func函数")
```
```python
test_b.py::test_func 我是前置 func
我是func函数
PASSED我是后置 func
```
##  usefixtures-手动调用
```python
import pytest

@pytest.fixture
def func():
    print("我是前置 func")
    yield 12
    print("我是后置 func")

@pytest.mark.usefixtures('func')
def test_func():
    print("我是func函数")
```
```python
test_b.py::test_func 我是前置 func
我是func函数
PASSED我是后置 func
```
### 在类中使用及几种其他方法
```python
"""conftest.py"""
import pytest

@pytest.fixture
def user_name():
    name = "清安"
    print("HELLO，清安")
    yield name
    print("BY !")
```
```python
import pytest

@pytest.mark.usefixtures("user_name")
class Test_user:

    def test_name(self):
        print("Hello")

    def test_user(self, user_name):
        print("user_name", user_name)
```
```python
test_a.py::Test_user::test_name HELLO，清安
Hello
PASSEDBY !

test_a.py::Test_user::test_user HELLO，清安
user_name  清安
PASSEDBY !
```
学会了吗。上述中你也可以指定多个usefixtures。
```python
@pytest.mark.usefixtures("user_name","user_age")
class Test_user:
	...
```
也可以写成这样：
```python
import pytest

# @pytest.mark.usefixtures("user_name")
class Test_user:
    pytestmark = pytest.mark.usefixtures("user_name")
	
    def test_name(self):
        print("Hello")

    def test_user(self,user_name):
        print(user_name)
```
最后一种方式就是将usefixtures写在pytest.ini配置文件中：
```python
[pytest]
usefixtures = user_name
```
```python
import pytest

# @pytest.mark.usefixtures("user_name")
class Test_user:
    # pytestmark = pytest.mark.usefixtures("user_name")
    def test_name(self):
        print("Hello")

    def test_user(self,user_name):
        print(user_name)
```
## 综合示例--注意点
上述例子中，皆可以在测试类中作用到，且可以同时使用：
```python
import pytest

@pytest.fixture(autouse=True)
def func_a():
    print("我是前置 func_a")
    yield 12
    print("我是后置 func_a")

@pytest.fixture()
def func_b():
    print("我是前置 func_b")
    yield 12
    print("我是后置 func_b")

@pytest.mark.usefixtures('func_b')
def test_func():
    print("我手动使用了--func_b--夹具")

class Test_A:
    @pytest.mark.usefixtures('func_b')
    def test_a(self):
        print("我手动使用了--func_b--夹具")

class Test_B:

    def test_b(self):
        print("我被--func_a--自动适配了")
```
```python
test_b.py::test_func 我是前置 func_a
我是前置 func_b
我手动使用了--func_b--夹具
PASSED我是后置 func_b
我是后置 func_a

test_b.py::Test_A::test_a 我是前置 func_a
我是前置 func_b
我手动使用了--func_b--夹具
PASSED我是后置 func_b
我是后置 func_a

test_b.py::Test_B::test_b 我是前置 func_a
我被--func_a--自动适配了
PASSED我是后置 func_a
```
:::warning
上述示例中，体现了一个不好的一点，就是自动适配的夹具会优先使用，且手动使用的夹具还会再使用一次，应该主动避免这种事情，除非特殊要求。
:::
# 夹具并不局限于一个
> 💥测试和夹具并不局限于 **请求** 一次一个固定装置。他们可以想要多少就要求多少。且可以重复使用。

```python
import pytest

@pytest.fixture
def first_func():
    return '拾贰'

@pytest.fixture()
def second_func():
    return '清安'

@pytest.fixture()
def name_func():
    return [first_func, second_func]

def test_func(name_func):
    assert name_func == [first_func, second_func]
```
```python
test_b.py::test_func PASSED
```
## 一个测试用例也可以使用多个夹具
```python
import pytest
@pytest.fixture()
def my_name():
    yield "清安"

@pytest.fixture()
def your_name():
    yield "拾贰"

def test_our(my_name,your_name):
    print("名字有：",my_name,your_name)
"""
test_a.py::test_our 名字有： 清安 拾贰
PASSED
"""
```
# 如果不明白夹具
```python
def first_name():
    return "清安"

def order(first_name):
    return [first_name]

def test_name(order):
    order.append("拾贰")
    assert order == ["清安", "拾贰"]

entry = first_name()
the_list = order(first_name=entry)
test_name(order=the_list)
```
> 如上代码近似于fixtrue的作用，如果还不明白，请回头学习函数。

# 注意点
```python
import pytest

@pytest.fixture
def first_func():
    return 12

def test_first_func():
    print(first_func)
    assert first_func == 12

def test_second_func(first_func):
    print(first_func)
    assert first_func == 12

@pytest.mark.usefixtures('first_func')
def test_func(first_func):
    print(first_func)
    assert first_func == 12
    print("我是func函数")
```
```python
test_b.py::test_first_func <function first_func at 0x0000027A41C64C10>
FAILED
test_b.py::test_second_func 12
PASSED
test_b.py::test_func 12
我是func函数
PASSED
```
> 🔴test_first_func，断言失败了，为什么，看打印信息是一个内存地址，并不是一个具体的值，所以会出现断言失败的情况。如何解决上述已经给到答案了，调用。


---

# 跨类、模块、包或会话共享fixture
|  function   |  每一个测试用例(包括函数和方法)执行前均会执行一次   |
| --- | --- |
|  class   |  每一个测试类/测试函数执行前都会执行一次   |
|  module   |  每一个 py 文件执行前都会执行一   |
|  package   |  每一个 python   包执行前会执行一次   |
|  session   |  整个测试前执行一次   |

> 1. 执行顺序遵循 sesstion->package->module->class->function
> 2. 每一类 fixture 可以是多个，同类按书写先后执行
> 3. 模块中的 fixture 对函数、方法均有效，class 中的 fixture 只对方法有效
> 4. 每一个函数前后均会执行模块中的 class
> 5. 在模块和类中有同名的 fixture 存在时：如果是 function，类中会覆盖模块中，其它的不会覆盖

## 执行顺序测试
```python
"""conftest.py"""
import pytest

@pytest.fixture(scope="session")
def order():
    return []

@pytest.fixture
def func(order):
    order.append("function")

@pytest.fixture(scope="class")
def cls(order):
    order.append("class")

@pytest.fixture(scope="module")
def mod(order):
    order.append("module")

@pytest.fixture(scope="package")
def pack(order):
    order.append("package")

@pytest.fixture(scope="session")
def sess(order):
    order.append("session")
```
```python
"""test_b.py"""
class TestClass:
    def test_order(self, func, cls, mod, pack, sess, order):
        assert order == ["session", "package", "module", "class", "function"]
```
可以自己执行试试，assert断言中可以调换顺序运行，看是否属实。
## 实践小说明
:::warning
 实践使用中，建议重点使用 sesstion、module、function 级，越简单越好  
:::
## 小例子-fixture范围
### conftest说明
回到conftest.py文件中。当然，不写conftest也是可以的，不过多个模块使用相同的fixture的时候写道conftest还是比较方便的。
Conftest是一个Python模块，它用于管理pytest插件的参数。它允许您在项目的不同部分定义和重用测试插件参数，而无需在代码中进行任何特殊调用。使用conftest.py文件可以让您的测试代码更加模块化，并且更容易维护。
:::warning
 1、文件名称默认为 conftest.py 
2、conftest.py 可以有多个，搜索优先级自底而上，下层会覆盖上层(特别注意) 
3、conftest.py 中的 fixture 支持函数直接引用、手动调用，也可以自动适配  
:::
#### 示例
```python
"""目录层级"""
pythonpp
	pytest_
		conftest.py
		test_a.py
	conftest.py
	test_b.py
```
```python
"""示例代码"""

"""pytest_
		conftest.py
"""
@pytest.fixture
def my_name():
    dic = {"name":"清安"}
    print("你猜我叫什么？")
    yield dic['name']
    print(f"我叫{dic['name']}")
"""test_a.py"""
def test_get_name(my_name):
    print(my_name)
```
```python
"""pythonpp
		conftest.py
"""
import pytest

@pytest.fixture
def my_name():
    dic = {"name":"拾贰"}
    print("你猜我叫什么？")
    yield dic['name']
    print(f"我叫{dic['name']}"
		  
"""test_b.py"""
def test_get_name(my_name):
    print(my_name)
```
:::warning
注意目录层级下的各py文件代码，代码中皆有标注，按照指定文件复制即可。
首先我们fixture不传参，运行看看，先运行test_a.py
:::
```python
test_a.py::test_get_name 你猜我叫什么？
清安
PASSED我叫清安
```
再运行test_b.py:
```python
test_b.py::test_get_name 你猜我叫什么？
拾贰
PASSED我叫拾贰
```
注意cd ..到pythonpp层级下，命令运行。
然后将pytest_层级下的conftest.py内容注释掉。再运行test_b.py：
```python
test_a.py::test_get_name 你猜我叫什么？
拾贰
PASSED我叫拾贰

pytest_/test_a.py::test_my_fruit_in_basket PASSED
pytest_/test_b.py::test_get_name 你猜我叫什么？
拾贰
PASSED我叫拾贰
```
:::warning
看，pythonpp层级下的conftest.py的夹具作用到了子目录下。如果不注释pytest_下的conftest.py，在pythonpp层级下运行pytest -vs命令，那么它们将各自作用于各自的领域。
:::
```python
test_a.py::test_get_name 你猜我叫什么？
拾贰
PASSED我叫拾贰

pytest_/test_a.py::test_my_fruit_in_basket PASSED
pytest_/test_b.py::test_get_name 你猜我叫什么？
清安
PASSED我叫清安
```
### 范围示例-class为例
:::warning
前面说了作用范围，此处以class为例，举一反三
:::
```python
"""conftest.py"""
import pytest

@pytest.fixture(scope="class")
def my_name():
    dic = {"name":"清安"}
    print("你猜我叫什么？")
    yield dic['name']
    print(f"我叫{dic['name']}")
```
```python
"""test_b.py"""
class TestA:
    def test1_get_name(self,my_name):
        print(f"{__class__.__name__}",my_name)

    def test2_get_name(self,my_name):
        print(f"{__class__.__name__}",my_name)

class TestB:
    def test1_get_name(self,my_name):
        print(f"{__class__.__name__}",my_name)

    def test2_get_name(self, my_name):
        print(f"{__class__.__name__}", my_name)
```
```python
test_b.py::TestA::test1_get_name 你猜我叫什么？
TestA 清安
PASSED
test_b.py::TestA::test2_get_name TestA 清安
PASSED我叫清安

test_b.py::TestB::test1_get_name 你猜我叫什么？
TestB 清安
PASSED
test_b.py::TestB::test2_get_name TestB 清安
PASSED我叫清安
```
:::warning
可以看到，夹具直接作用到了类，并没有直接作用到每个测试用例，仅仅只是将值给到了每一个用例。这也是class与默认function的区别。以此类推，根据上述的文字描述，可以尝试module以及更高层的。
:::
# fixture销毁方式
 我们把 fixture 后置处理又叫作 fixture 的销毁，fixture 的销毁有两种方  
## 1、yield
前面写过一部分关于yield的代码。也在迭代器与生成器中讲过。此处不做多的讲解。它与return近似而不同于return。那么它如何具有销毁的作用呢，或者说，怎么样让它可以帮助我们达到销毁的一个作用
```python
"""conftest.py"""
@pytest.fixture(scope="function")
def user():
    print("create success!")
    yield "清安"
    print("delete success!")
```
模拟一下添加一个用户以及删除用户的功能，print处可以写sql语句以及各种语句。
那么调用后呢？
```python
def test_user(user):
    print(user)
"""
test_b.py::test_user create success!
清安
PASSED delete success!
"""
```
看到了吗，先成功后，然后自动销毁，也就是删除的操作。
## 2、 addfinalizer  
addfinalizer要比yield复杂一些。大致作用是一样的。
```python
"""conftest.py"""
@pytest.fixture(scope="function")
def user(request):
    print("create success!")
    def clear():
        print("delete success!")
    request.addfinalizer(clear)
    return '清安'
```
```python
def test_user(user):
    print(user)
"""
test_b.py::test_user create success!
清安
PASSEDdelete success!
"""
```
:::warning
两种方式的实现效果，我们可以认为是一样，无论哪一种方式，均遵循以下规则

1. 用例如果发生异常，不影响前后置动作的执行
2. 如果前置动作失败，那么后置动作不会执行
:::
## fixture标志传参-marker
```python
"""conftest.py"""
import pytest

@pytest.fixture()
def user_name(request):
    maker = request.node.get_closest_marker("name")
    if maker is not None:
        data = maker.args[0]
    else:
        data = None
    return data
```
```python
"""test_b.py"""
@pytest.mark.name("清安")
def test_get_user(user_name):
    print(user_name)
    assert user_name == '清安'
"""
test_b.py::test_get_user 清安
PASSED
"""
```
这样的方式会有一个warning提示，说没有注册。问题不大。
这样的方式其实就是通过mark标记后，通过request进行参数获取，并返回值，测试用例这边通过夹具机制获取到值。比较新的知识点就是获取值的方式了。
# fixture参数化
```python
"""conftest.py"""

import pytest

datas = ('清安','拾贰','詹姆斯')
@pytest.fixture(params=datas,ids=[1,2,3])
def user_name(request):
    print("user_name参数：",request.param)
    return request.param
```
```python
"""test_b.py"""
def test_user(user_name):
    print("我是：",user_name)
"""
test_b.py::test_user[1] user_name参数： 清安
我是： 清安
PASSED
test_b.py::test_user[2] user_name参数： 拾贰
我是： 拾贰
PASSED
test_b.py::test_user[3] user_name参数： 詹姆斯
我是： 詹姆斯
PASSED
"""
```
在contest.py中ids参数处也可以写成列表推导式：ids=[i for i in range(1,4)]。ids就是一个标志，起一个提示的作用，告诉你用例运行到了第几个参数，也可称之为取别名。此处的传参不难理解，主要还是通过request进行，也就是代码中的request.param。
:::warning
1. fixture 可以通过设计 params，让依赖该 fixture 的用例迭代执行 
2. params 数据可以为[列表]，(元组)，{集合}，{字典} 
3. params 数据要 fixture 中通过固定 request 变量来接收  
:::
## 参数化夹具与标记的结合使用
```python
import pytest

@pytest.fixture(params=[0,1,2,pytest.param(3,marks=pytest.mark.skip)])
def user_data(request):
    return request.param

def test_user(user_data):
    print(user_data)
"""
test_a.py::test_user[0] 0
PASSED
test_a.py::test_user[1] 1
PASSED
test_a.py::test_user[2] 2
PASSED
test_a.py::test_user[3] SKIPPED (unconditional skip)
"""
```
看到了吗，这样的方式，可以帮助你更好的参数化与实现想要的功能。
# fixture工厂模式
工厂模式有助于在单个测试中多次需要夹具结果的情况下。夹具不直接返回数据，而是返回一个生成数据的函数。然后可以在测试中多次调用此函数。简单的说就是便于重复造轮子。
例如，我想在一个测试用例里面重复注册多个用户：
```python
"""conftest.py"""
import pytest


@pytest.fixture()
def user_func():
    def use_func(name):
        return {"name":name}
    return use_func
```
```python
"""test_b.py"""
def test_user(user_func):
    user1 = user_func("清安")
    user2 = user_func("拾贰")
    user3 = user_func("安安")
    print("注册用户：",user1['name'],user2['name'],user3['name'])
"""
test_b.py::test_user 注册用户： 清安 拾贰 安安
"""
```
看着没感觉？那改一改改成有断言的：
## pytest-assume插件
简单的说就是即使断言失败了，不影响后续代码的运行。接上述:
```python
def test_user(user_func):
    user1 = user_func("清安")
    # assert user1
    pytest.assume(user1)
    user2 = user_func("拾贰")
    # assert user2 == '11'
    pytest.assume(user2 == '11')
    user3 = user_func("安安")
    # assert user2
    pytest.assume(user3)
    print("注册用户：",user1['name'],user2['name'],user3['name'])
	
________________________________assume_________________________________________

E               AssertionError: assert False

..\..\venv\lib\site-packages\six.py:718: FailedAssumption
注册用户： 清安 拾贰 安安
=============================== short test summary info ======================== 
PASSED test_a.py::test_my_fruit_in_basket
============================== 1 failed, 1 passed in 0.18s =====================
__________________________________assert_______________________________________
E       AssertionError: assert {'name': '拾贰'} == '11'

test_b.py:116: AssertionError
=============================== short test summary info =====================
PASSED test_a.py::test_my_fruit_in_basket
============================= 1 failed, 1 passed in 0.15s ====================
```
:::warning
看到了吗，assert失败后是不会继续运行后续的代码，而assume插件是可以跑的。上述只是一个小示例，也可以运用到每个测试用例中。
:::
# 模块化：使用fixture中的fixture
:::warning
除了在测试函数中使用fixture之外，fixture函数还可以使用其他fixture本身。这有助于夹具的模块化设计，并允许在许多项目中重复使用特定于框架的装置。
:::
这跟上述中的夹具并不局限于1个有些类似，不过用法上有些不同：
```python
"""contest.py"""
import pytest

class Create:
    def __init__(self,name):
        self.name = name


@pytest.fixture(scope='module')
def create(Str='清安'):
    print("CREATE SUCCESS")
    return Create(Str)

"""test_a.py"""
import pytest

class Delete:
    def __init__(self,name):
        self.name = name

@pytest.fixture(scope='module')
def delete(create):
    print("DELETE SUCCESS")
    return Delete(create)

def test_user(delete):
    assert delete.name
    print("SUCCESS")
```
```python
test_a.py::test_user CREATE SUCCESS
DELETE SUCCESS
SUCCESS
PASSED
```
:::warning
注意上述代码，fixture中使用fixture，添加一个用户，随后删除。此处不局限于与module，也可以是是class亦或者session。
💥注意：conftest中的scope的范围不能小于test_a.py中的fixture范围，否则ScopeMismatch: You tried to access the module scoped fixture create with a session scoped request object, involved factories:
:::
# 小结
> 1. fixture 的基础用法，前后置结构
> 2. fixture 的调用方式，包括函数引用、自动引用、手动调用
> 3. fixture 的 scope 范围，包括 function、class、module、package、session，了解每一种的使用及执行顺序
> 4. conftest.py 管理共享的 fixture
> 5. fixture 的安全销毁
> 6. fixture 的参数内容，包括标志传参，参数化(数据驱动)
> 7. fixture 工厂模式的使用场景
> 8. 利用 fixture

至于fixture是否必须要写在conftest中，我的回答是不是必须的，看情况来，如果你想多模块一起使用的情况，可以写在conftest中，这样便于关于，并且同级目录下也能便于使用。此外，fixture是pytest中使用频率相当高的一个装饰器。
