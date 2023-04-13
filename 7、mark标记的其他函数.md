在前面的文章中，讲了mark的自定义标记以及参数化，fixture的手动调用。本章就主要介绍介绍剩下的内容。
# skip跳过用例
## 1、函数/方法级跳过
```python
import pytest

def test_01():
    print("--01--")

@pytest.mark.skip("无条件跳过")
def test_02():
    print("--02--")

def test_03():
    print("--03--")
"""
Case/test_a.py::test_01 --01--
PASSED
Case/test_a.py::test_02 SKIPPED (无条件跳过)
Case/test_a.py::test_03 --03--
PASSED
"""
```
## 2、模块级跳过
```python
"""test_a.py"""
import pytest

pytest.skip('这个模块不测了',allow_module_level=True)

def test_01():
    print("--01--")

def test_02():
    print("--02--")

def test_03():
    print("--03--")
```
```python
"""test_c.py"""
def test_01():
    print("--01--")
```
```python
Case/test_c.py::test_01 --01--
PASSED

======================== 1 passed, 1 skipped in 0.02s =========================
```
看到了吗，test_a.py直接被跳过了，只执行了test_c.py文件中的用例。
## 3、类级跳过
```python
import pytest

def test_01():
    print("--01--")

def test_02():
    print("--02--")

def test_03():
    print("--03--")

@pytest.mark.skip("这个类，我跳过了")
class Test_a:
    def test_method(self):
        print("--method--")
"""
Case/test_a.py::test_01 --01--
PASSED
Case/test_a.py::test_02 --02--
PASSED
Case/test_a.py::test_03 --03--
PASSED
Case/test_a.py::Test_a::test_method SKIPPED (这个类，我跳过了)
"""
```
# skipif条件跳过
## 1、类级跳过
```python
import pytest

def test_01():
    print("--01--")

def test_02():
    print("--02--")

def test_03():
    print("--03--")

@pytest.mark.skipif(1>0,reason="条件为真，这个类，我跳过了")
class Test_a:
    def test_method(self):
        print("--method--")
"""
Case/test_a.py::test_01 --01--
PASSED
Case/test_a.py::test_02 --02--
PASSED
Case/test_a.py::test_03 --03--
PASSED
Case/test_a.py::Test_a::test_method SKIPPED (条件为真，这个类，我跳过了)

======================== 3 passed, 1 skipped in 0.03s =========================
"""
```
## 2、方法/函数级跳过
```python
import pytest

def test_01():
    print("--01--")

def test_02():
    print("--02--")

@pytest.mark.skipif(1>0,reason="条件为真，这个函数，我跳过了")
def test_03():
    print("--03--")

class Test_a:
    def test_method(self):
        print("--method--")
"""
Case/test_a.py::test_01 --01--
PASSED
Case/test_a.py::test_02 --02--
PASSED
Case/test_a.py::test_03 SKIPPED (条件为真，这个函数，我跳过了)
Case/test_a.py::Test_a::test_method --method--
PASSED
"""
```
## 3、模块级跳过
```python
"""test_a.py"""
import pytest

pytestmark = pytest.mark.skipif(1>0,reason='这个模块不测了，跳过')

def test_01():
    print("--01--")

def test_02():
    print("--02--")

def test_03():
    print("--03--")

class Test_a:
    def test_method(self):
        print("--method--")
```
```python
"""test_c.py"""
def test_01():
    print("--01--")
```
```python
"""执行结果"""
Case/test_a.py::test_01 SKIPPED (这个模块不测了，跳过)
Case/test_a.py::test_02 SKIPPED (这个模块不测了，跳过)
Case/test_a.py::test_03 SKIPPED (这个模块不测了，跳过)
Case/test_a.py::Test_a::test_method SKIPPED (这个模块不测了，跳过)
Case/test_c.py::test_01 --01--
PASSED
```
#  xfail 预失败标志  
> pytest.mark.xfail(condition=None, *, reason=None, raises=None, run=True, strict=False)
> 参数
> - condition (bool or str) -- 将测试功能标记为xfail的条件 (True/False 或A condition string ). 如果是bool，还必须指定 reason 
> - reason (str) -- 测试函数标记为xfail的原因。
> - raises (Type[Exception]) -- 异常子类预期由测试函数引发；其他异常将使测试失败。
> - run (bool) -- 是否应实际执行测试功能。如果 False ，函数将始终Xfail且不会执行（如果函数是segfaulting，则很有用）。
> - strict (bool) --
> 
如果 False （默认）功能将在终端输出中显示为 xfailed 如果失败了， xpass 如果它通过。在这两种情况下，这不会导致整个测试套件失败。
> 如果 True ，功能将在终端输出中显示为 xfailed 如果它失败了，但是如果它意外地通过了，那么它将 fail 测试套件。这对于标记总是失败的函数特别有用，如果函数意外开始通过，则应该有明确的指示。

```python
import pytest

@pytest.mark.xfail()
def test_01():
    raise Exception("主动异常")

@pytest.mark.xfail(reason='函数级xfail标志')
def test_02():
    raise Exception("主动异常")

@pytest.mark.xfail(raises=RuntimeError)
def test_03():
    raise RuntimeError("主动异常")

@pytest.mark.xfail(run=False,reason='PASS')
def test_04(self):
    raise Exception("主动异常")

@pytest.mark.xfail(struct=False)
def test_05():
    pass

def test_06():
    pytest.xfail(reason="函数级另一种标志")

@pytest.mark.xfail(1<0,reason='函数级另一种标志')
def test_07():
    pass
```
```python
"""运行结果"""
Case/test_a.py::test_01 XFAIL
Case/test_a.py::test_02 XFAIL (函数级xfail标志)
Case/test_a.py::test_03 XFAIL
Case/test_a.py::test_04 XFAIL ([NOTRUN] PASS)
Case/test_a.py::test_05 XPASS
Case/test_a.py::test_06 XFAIL (函数级另一种标志)
Case/test_a.py::test_07 PASSED

=================== 1 passed, 5 xfailed, 1 xpassed in 0.18s ===================
```
> 上述中的condition ，可以理解为条件判断，如果1>0的时候，则为XPASS，反之如上述所示为PASSED。预失败标志，即使失败了也不会抛出异常，但是会抛出标志。

# 警告
```python
import warnings

def v1():
    warnings.warn(UserWarning("当前版本为V1，请检查版本信息"))

def test_01():
    assert v1() == 1.1
```
```python
============================== warnings summary ===============================
Case/test_a.py::test_01
  D:\pytest_\Case\test_a.py:262: UserWarning: 当前版本为V1，请检查版本信息
    warnings.warn(UserWarning("当前版本为V1，请检查版本信息"))
```
你会得到这样一串信息，正常，自定义的warnings生效了。
## 警告分类
| 警告 | 描述 |
| --- | --- |
|  Warning   |  这是所有警告的基类。它是 Exception 的子类   |
|  UserWarning   |  警告的默认类别   |
| DeprecationWarning   |  有关已弃用功能的警告的基本类别   |
|  SyntaxWarning   |  静态/语法的警告   |
|  RuntimeWarning   |  动态/运行时警告   |
|  FutureWarning   |  将来语义可能会改变的警告   |
|  PendingDeprecationWarning   |  有关将来不推荐使用的功能的警告的基本类别   |
|  ImportWarning   |  导入模块过程中触发的警告   |
|  UnicodeWarning   |  与 Unicode 相关的警告   |
|  BytesWarning   |  bytes 与和相关的警告   |
|  ResourceWarning   |  与资源使用相关的警告   |

## 警告过滤器
 警告过滤器由 5 块信息组件：
 action(过滤器动作):message(信息):category(类别):module(模块):line(行号,默认匹配所有)  
常用示例：

|  default   |  显示所有警告   |
| --- | --- |
|  ignore   |  忽略所有警告   |
|  error   |  将警告转换为错误   |
|  error::ResourceWarning   |  将 ResourceWarning 警告转换为错   |
|  default::DeprecationWarning   |  显示 DeprecationWarning 警告信   |
|  ignore,default::mymodule   |  只展示由 mymodule 模块触发的警告   |
|  error::mymodule   |  将警告转换为错误(在模块 mymodule 中  ） |

### 配置过滤警告
```python
pytest.main(["-vs", "-W error::UserWarning"])
```
```python
[pytest]
filterwarnings =
	ignore
	error::UserWarning
```
##  filterwarnings  
鉴于上述代码中的一处示例，如果不做过滤的情况下是有warning信息的。那么我们就来过滤一下
```python
import pytest
import warnings

def v1():
    warnings.warn(UserWarning("当前版本为V1，请检查版本信息"))
	return 1

@pytest.mark.filterwarnings("ignore::UserWarning")
def test_01():
    assert v1() == 1.1
```
```python
=========================== short test summary info ===========================
FAILED Case/test_a.py::test_01 - assert None == 1.1
```
除了assert断言错误信息之外，只有简短的信息了。以及成功的过滤掉了。
当然你可以可以作用在整个模块亦或者类中，一起看看：
```python
import pytest
import warnings

@pytest.mark.filterwarnings("ignore::UserWarning")
class Test_A:
    def v1(self):
        warnings.warn(UserWarning("当前版本为V1，请检查版本信息"))
        return 1

    def test_01(self):
        assert self.v1() == 1.1
```
```python
import pytest
import warnings

pytestmark = pytest.mark.filterwarnings("ignore")

class Test_A:
    def v1(self):
        warnings.warn(UserWarning("当前版本为V1，请检查版本信息"))
        return 1

    def test_01(self):
        assert self.v1() == 1.1
```
# 小结
警告，我们在平常的使用中见的还是比较的多的，实际自身使用的其实并不算太多。我们会用logging亦或者python自带的异常捕捉直接那捏住。
