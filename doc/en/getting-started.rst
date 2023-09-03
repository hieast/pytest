.. _get-started:

开始使用
===================================

.. _`getstarted`:
.. _`installation`:

安装 ``pytest``
----------------------------------------

``pytest`` 需要: Python 3.8+ 或 PyPy3.

1. 在命令行中运行以下命令：

.. code-block:: bash

    pip install -U pytest

2. 检查你是否安装了正确的版本：

.. code-block:: bash

    $ pytest --version
    pytest 7.4.1

.. _`simpletest`:

创建你的第一个测试
----------------------------------------------------------

创建一个名为 ``test_sample.py`` 的新文件，其中包含一个函数和一个测试：

.. code-block:: python

    # test_sample.py 文件内容
    def func(x):
        return x + 1


    def test_answer():
        assert func(3) == 5

测试

.. code-block:: pytest

    $ pytest
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-7.x.y, pluggy-1.x.y
    rootdir: /home/sweet/project
    collected 1 item

    test_sample.py F                                                     [100%]

    ================================= FAILURES =================================
    _______________________________ test_answer ________________________________

        def test_answer():
    >       assert func(3) == 5
    E       assert 4 == 5
    E        +  where 4 = func(3)

    test_sample.py:6: AssertionError
    ========================= short test summary info ==========================
    FAILED test_sample.py::test_answer - assert 4 == 5
    ============================ 1 failed in 0.12s =============================

``[100%]`` 是指运行所有测试用例的整体进度。完成后，pytest 会显示失败报告，因为 ``func(3)`` 并未返回 ``5``。

.. note::

    你可以使用 ``assert`` 语句来验证测试的预期结果。pytest 的 :ref:`高级断言内省 <python:assert>` 会智能地报告断言表达式的中间值，以便你可以避免许多名为 :ref:`JUnit 遗留方法的名称 <testcase-objects>` 。

运行多个测试
----------------------------------------------------------

``pytest`` 会运行当前目录及其子目录中所有形如 test_*.py 或 \*_test.py 的文件。更一般地说，它遵循 :ref:`标准的测试发现规则 <test discovery>`。

断言某段代码会触发特定的异常
--------------------------------------------------------------

使用 :ref:`raises <assertraises>` 助手来断言某段代码会触发异常：

.. code-block:: python

    # test_sysexit.py 文件内容
    import pytest


    def f():
        raise SystemExit(1)


    def test_mytest():
        with pytest.raises(SystemExit):
            f()

以“静默”报告模式执行测试函数：

.. code-block:: pytest

    $ pytest -q test_sysexit.py
    .                                                                    [100%]
    1 passed in 0.12s

.. note::

    ``-q/--quiet`` 标志在此及后续示例中将输出保持简洁。

将多个测试组合到一个类中
--------------------------------------------------------------

.. regendoc:wipe

一旦你开发出多个测试，你可能希望将它们分组到一个类中。pytest 让你可以轻松创建包含多个测试的类：

.. code-block:: python

    # test_class.py 文件内容
    class TestClass:
        def test_one(self):
            x = "this"
            assert "h" in x

        def test_two(self):
            x = "hello"
            assert hasattr(x, "check")

``pytest`` 发现所有的测试都遵循其 :ref:`Python 测试发现的约定 <test discovery>`，因此它找到了两个以 ``test_`` 开头的函数。不需要继承任何东西，但一定要确保你的类名以 ``Test`` 开头，否则该类将被跳过。我们可以通过传递文件名来简单地运行模块：

.. code-block:: pytest

    $ pytest -q test_class.py
    .F                                                                   [100%]
    ================================= FAILURES =================================
    ____________________________ TestClass.test_two ____________________________

    self = <test_class.TestClass object at 0xdeadbeef0001>

        def test_two(self):
            x = "hello"
    >       assert hasattr(x, "check")
    E       AssertionError: assert False
    E        +  where False = hasattr('hello', 'check')

    test_class.py:8: AssertionError
    ========================= short test summary info ==========================
    FAILED test_class.py::TestClass::test_two - AssertionError: assert False
    1 failed, 1 passed in 0.12s

第一个测试通过，第二个失败。你可以轻松查看断言中的中间值，以帮助你理解失败的原因。

将测试组合在类中可以有以下优点：

 * 测试组织
 * 只在特定类中共享 fixtures
 * 在类级别应用 marks，并将其隐式应用于所有测试

当你将测试组合在类中时，需要注意的是，每个测试都有类的唯一实例。
如果每个测试共享同一个类实例，那将对测试隔离非常不利，并且会提倡不良的测试实践。
这在下面有详细说明：

.. regendoc:wipe

.. code-block:: python

    # test_class_demo.py 文件内容
    class TestClassDemoInstance:
        value = 0

        def test_one(self):
            self.value = 1
            assert self.value == 1

        def test_two(self):
            assert self.value == 1


.. code-block:: pytest

    $ pytest -k TestClassDemoInstance -q
    .F                                                                   [100%]
    ================================= FAILURES =================================
    ______________________ TestClassDemoInstance.test_two ______________________

    self = <test_class_demo.TestClassDemoInstance object at 0xdeadbeef0002>

        def test_two(self):
    >       assert self.value == 1
    E       assert 0 == 1
    E        +  where 0 = <test_class_demo.TestClassDemoInstance object at 0xdeadbeef0002>.value

    test_class_demo.py:9: AssertionError
    ========================= short test summary info ==========================
    FAILED test_class_demo.py::TestClassDemoInstance::test_two - assert 0 == 1
    1 failed, 1 passed in 0.12s

注意，在类级别添加的属性是*类属性*，因此它们将在测试之间共享。

请求一个唯一的临时目录进行功能测试
--------------------------------------------------------------

``pytest`` 提供 :std:doc:`Builtin fixtures/function arguments <builtin>` 来请求任意资源，比如一个唯一的临时目录：

.. code-block:: python

    # test_tmp_path.py 文件内容
    def test_needsfiles(tmp_path):
        print(tmp_path)
        assert 0

在测试函数签名中列出名为 ``tmp_path`` 的名称，``pytest`` 将查找并调用一个 fixture 工厂在执行测试函数调用之前创建资源。在测试运行之前，``pytest`` 创建一个对每次测试调用都唯一的临时目录：

.. code-block:: pytest

    $ pytest -q test_tmp_path.py
    F                                                                    [100%]
    ================================= FAILURES =================================
    _____________________________ test_needsfiles ______________________________

    tmp_path = PosixPath('PYTEST_TMPDIR/test_needsfiles0')

        def test_needsfiles(tmp_path):
            print(tmp_path)
    >       assert 0
    E       assert 0

    test_tmp_path.py:3: AssertionError
    --------------------------- Captured stdout call ---------------------------
    PYTEST_TMPDIR/test_needsfiles0
    ========================= short test summary info ==========================
    FAILED test_tmp_path.py::test_needsfiles - assert 0
    1 failed in 0.12s

有关临时目录处理的更多信息，请查看 :ref:`临时目录和文件 <tmp_path handling>`。

查看哪些内置的 :ref:`pytest fixtures <fixtures>` 存在，可以使用命令：

.. code-block:: bash

    pytest --fixtures   # 显示内置和自定义的 fixtures

注意，这个命令会省略带有前导 ``_`` 的 fixtures，除非添加 ``-v`` 选项。

继续阅读
-------------------------------------

查看额外的 pytest 资源以帮助你定制你的独特工作流：

* ":ref:`usage`" 标注命令行调用示例
* ":ref:`existingtestsuite`" 用于处理已存在的测试
* ":ref:`mark`" 提供关于 ``pytest.mark`` 机制的信息
* ":ref:`fixtures`" 为你的测试提供基础功能
* ":ref:`plugins`" 用于管理和编写插件
* ":ref:`goodpractices`" 虚拟环境和测试布局的好实践