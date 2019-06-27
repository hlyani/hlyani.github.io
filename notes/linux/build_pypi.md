# 发布pypi软件包

> 发布pypi python包，打包软件包时，主要依赖setuptools。

```
pip install setuptools
```

## 一、注册pypi账号

[pypi register](https://pypi.python.org/pypi)

##### 1、点击register

##### 2、填写名字，密码，邮件

> 记住自己的用户名和密码，后面上传的时候需要输入

## 二、准备自己的python源码

##### 略

## 三、准备setup文件（全部放在源码根目录）

##### 1、准备setup.cfg

```
[bdist_wheel]
universal = 1
```

##### 2、准备README.rst，具体语法可以参考http://rest-sphinx-memo.readthedocs.io/en/latest/ReST.html

```
========
autopep8
========

.. image:: https://img.shields.io/pypi/v/autopep8.svg
    :target: https://pypi.org/project/autopep8/
    :alt: PyPI Version

.. image:: https://travis-ci.org/hhatto/autopep8.svg?branch=master
    :target: https://travis-ci.org/hhatto/autopep8
    :alt: Build status
.. contents::


Installation
============

From pip::

    $ pip install --upgrade autopep8

Consider using the ``--user`` option_.

.. _option: https://pip.pypa.io/en/latest/user_guide/#user-installs


Requirements
============

autopep8 requires pycodestyle_.

.. _pycodestyle: https://github.com/PyCQA/pycodestyle
```

##### 3、准备setup.py

* version：包版本，更新包时更新版本号
* name：包名称
* long_description：必须是rst（reStructuredText）格式，这个内容会显示在pypi包首页，具体语法可以参考http://rest-sphinx-memo.readthedocs.io/en/latest/ReST.html
* install_requires：申明依赖包。安装包是pip会自动安装
* packages = find_packages()，这个参数是导入目录下的所有\__init__.py包

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""Setup for autopep8."""

import ast
import io

from setuptools import setup


INSTALL_REQUIRES = (
    ['pycodestyle >= 2.5.0']
)


def version():
    """Return version string."""
    with io.open('autopep8.py') as input_file:
        for line in input_file:
            if line.startswith('__version__'):
                return ast.parse(line).body[0].value.s


with io.open('README.rst') as readme:
    setup(
        name='hlautopep8',
        version=version(),
        description='A tool that automatically formats Python code to conform '
                    'to the PEP 8 style guide',
        long_description=readme.read(),
        license='Expat License',
        author='tmp',
        author_email='tmp@gmail.com',
        url='https://tmp/autopep8',
        classifiers=[
            'Development Status :: 5 - Production/Stable',
            'Environment :: Console',
            'Intended Audience :: Developers',
            'License :: OSI Approved :: MIT License',
            'Operating System :: OS Independent',
            'Programming Language :: Python',
            'Programming Language :: Python :: 2',
            'Programming Language :: Python :: 2.7',
            'Programming Language :: Python :: 3',
            'Programming Language :: Python :: 3.4',
            'Programming Language :: Python :: 3.5',
            'Programming Language :: Python :: 3.6',
            'Programming Language :: Python :: 3.7',
            'Topic :: Software Development :: Libraries :: Python Modules',
            'Topic :: Software Development :: Quality Assurance',
        ],
        keywords='automation, pep8, format, pycodestyle',
        install_requires=INSTALL_REQUIRES,
        test_suite='test.test_autopep8',
        py_modules=['autopep8'],
        zip_safe=False,
        entry_points={'console_scripts': ['autopep8 = autopep8:main']},
    )
```

## 四、开始打包（到源码根目录打包，成功执行打包命令会生成一个dist文件夹）

##### 1、将源码打包为wheels(whl)格式

```
python setup.py bdist_wheel --universal
```

> 可以使用pip命令进行安装

```
pip install xx.whl
```

##### 2、将源码打包为tar.gz格式

```
python setup.py sdist build
```
##### 3、将源码打包为egg格式

```
python setup.py bdist_egg
```

## 五、上传生成的包

##### 1、使用twine上传，先安装twine

```
pip install twine
```

##### 2、上传，会提示输入用户名和密码

```
twine upload dist/*
```

##### 3、上传成功可以到自己的pypi仓库查看

## 六、使用

```
pip search xx

pip install xx
```

## 七、其他

```
#!/usr/bin/env python
# coding: utf-8

from setuptools import setup

setup(
    name='jujube_pill',
    version='0.0.1',
    author='xlzd',
    author_email='what@the.f*ck',
    url='https://zhuanlan.zhihu.com/p/26159930',
    description=u'吃枣药丸',
    packages=['jujube_pill'],
    install_requires=[],
    entry_points={
        'console_scripts': [
            'jujube=jujube_pill:jujube',
            'pill=jujube_pill:pill'
        ]
    }
)
```

> install\_requires 是这个库所依赖的其它库，当别人使用 pip 等工具安装你的包时，会自动安装你所依赖的包。console_scripts 是这个包所提供的终端命令，比如我希望在安装这个包后可以使用「 jujube 」和「 pill 」两个命令，则按照 setup 文件的写法，当我在终端输入「 jujube 」的时候，将会执行 jujube_pill 包下（__init__ 中）的 jujube 函数。
>
> __init__.py 文件如下：

```
#!/usr/bin/env python
# encoding=utf-8


def jujube():
    print u'吃枣'
 

def pill():
    print u'药丸'
```