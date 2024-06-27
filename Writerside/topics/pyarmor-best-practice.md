# 应用实践

<show-structure depth="3"/>


## 2.4 打包加密项目为 Wheel

这里我创建一个 ImpalaFlow 的项目，核心库功能为 imflow，文件结构如下:

```Bash
imflow
├── __init__.py
├── db
│   ├── __init__.py
│   ├── hive.py
│   └── impala.py
└── decorators
    ├── __init__.py
    └── common.py
```

接下来，我们就可以使用 `pyarmor gen` 命令来给项目文件进行加密。加密完成后可以在 dist 文件夹看到一个 imflow 文件夹，文件结构与加密前基本一致，并且里面的脚本文件都进行了加密。值得注意的是，imflow 文件夹还多出来一个 `pyarmor_runtime_000000` 文件，里面有一个核心文件 `pyarmor_runtime.so`。

```Bash
pyarmor gen -r -i imflow
```

接下来我们在加密后 imflow 同级目录下新建一个 setup.py 文件，内容如下:

```Python
from setuptools import setup, find_packages, Extension


VERSION = '0.0.1'
setup(
    name="imflow",
    version=VERSION,
    platforms=["any"],
    packages=find_packages(),
    url='https://github.com/mingminyu/ImpalaFlow',
    license='None',
    author='mingminyu',
    author_email='yu_mingm623@163.com',
    description='Impala/Hive 执行 SQL 工具',
    zip_safe=False,
    include_package_data=True,
    classifiers=[
        "Development Status :: 5 - Production/Stable",
        "Environment :: Console",
        "Intended Audience :: Developers",
        "Operating System :: Microsoft :: Windows",
        "Intended Audience :: System Administrators",
        "License :: OSI Approved :: Apache Software License",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
        "Programming Language :: Python :: 3.12",
        "Topic :: System :: Machine Learning",
        ],
    install_requires=[
        "loguru",
        "impyla",
        "pandas",
        "retrying",
    ],
    package_data={
        "imflow": [
            "*.py", "*.so", "db/*.py", 
            "decorators/*.py",
            "pyarmor_runtime_000000/*.py",
            "pyarmor_runtime_000000/*.so"
        ],
    }
)
```
{collapsible="true"}

值得注意的是，我们需要将 `include_package_data` 设置为 `True`，且 `package_data` 需要将 `pyarmor_runtime_000000` 文件夹下的所有文件都包含进去。


现在我们就可以使用 `python setup.py bdist_wheel`，会在当前目录下再次另外生成一个 dist 目录，里面有我们打包好的 Wheel 文件，使用 `pip install dist/imflow-0.0.1-py3-none-any.whl` 进行安装使用。


> 将在 Mac 上打包后的 whl 文件放在 Linux 上能够正常安装，但是导入时却发现 pyarmor_runtime.so 加载时报错 invalid ELF Header。
> 
{style="warning"}

> 导致该问题的主要原因还是跨平台导致的，我们需要在 Linux 服务器上重新执行上述操作。


<seealso>
<category ref="ref_docs">
    <a href="https://pyarmor.readthedocs.io/zh/latest/how-to/wheel.html">2.4 打包加密脚本为 Wheel</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/dashingsoft/pyarmor">PyArmor</a>
</category>
<category ref="ref_issues">
    <a href="https://github.com/dashingsoft/pyarmor/issues/969">969: invalid ELF header</a>
</category>
</seealso>