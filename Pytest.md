# 安装
```python
pip install -U pytest
```
# 测试安装
```python
import pytest

class Testlogin:
    def test01(self):
        print(1)

if __name__ == '__main__':
    pytest.main()
"""
testone.py::Testlogin::test01 PASSED                                     [100%]1
"""
```
或者
```python
pytest --version # 显示pytest是从哪里导入的
pytest --fixtures # 显示可用的内建函数参数
pytest -h | --help # 在命令行上显示帮助和配置文件选项
```
# 插件介绍
> pip install pytest-ordering 控制用例的执行顺序（重点） 
> pip install pytest-xdist 分布式并发执行测试用例（重点） 
> pip install pytest-dependency 控制用例的依赖关系 （了解） 
> pip install pytest-rerunfailures 失败重跑（了解） 
> pip install pytest-assume 多重较验（了解） 
> pip install pytest-random-order 用例随机执行（了解） 
> pip install pytest-html 测试报告（了解） 
> pip install allure-pytest 测试报告（重点） 
> pip install pytest-reportlog 测试报告（了解）  

