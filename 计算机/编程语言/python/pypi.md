# pypi

## 推送到pypi

### 项目组织结构

```
<project_name>/                 # 项目根目录
|-- <package_name>              # package
|   |-- __init__.py
|   `-- <files> ....          # 代码模块
|-- README.md
|-- LICENSE
|-- setup.cfg
|-- setup.py
```

setup.cfg是一个配置信息文件，运行setup.py程序打包的时候会用到里面的配置，作为setup.py的命令行参数。

setup.py用来描述我们的项目，之后打包的时候会用到这个文件。它告诉PyPI我们的项目叫什么名字，是什么版本，依赖哪些库，支持哪些操作系统，可以在哪些版本的Python上运行，等等。

### setup.cfg

```python
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="PACKAGE_NAME",
    version="VERSION",
    author="AUTHOR_NAME",
    author_email="AUTHOR_EMAIL",
    description="DESCRIPTION",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="GITHUB_URL",
    packages=setuptools.find_packages(),
    install_requires=['LIB_NAME>=5.1.0', 'LIB_NAME==1.14.4'],
    entry_points={
        'console_scripts': [
            'SCRIPT_NAME=PACKAGE_NAME:main'
        ],
    },
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)
```

- name - 项目的名称
- version - 项目的版本。需要注意的是，PyPI上只允许一个版本存在，如果后续代码有了任何更改，再次上传需要增加版本号
- author和author_email - 项目作者的名字和邮件
- description - 项目的简短描述
- long_description - 项目的详细描述，会显示在PyPI的项目描述页面。上面的例子里直接用了README.md中的内容做详细描述
- long_description_content_type - 用于指定long_description的markup类型，上面的例子是markdown
- url - 项目主页的URL，一般给出代码仓库的链接
- packages - 指定最终发布的包中要包含的packages。上面的例子中find_packages() 会自动发现项目根目录下所有的packages，当然也可以手动指定package的名字
- install_requires - 项目依赖哪些库，这些库会在pip install的时候自动安装
- entry_points - 上面的例子中entry_points用来自动创建脚本，上面的例子在pip install安装成功后会创建douyin_image这个命令，直接可以在命令行运行，很方便
- classifiers - 其他信息，一般包括项目支持的Python版本，License，支持的操作系统。上面的例子中，我们指定项目只能在Python 3上运行，使用MIT License，不依赖操作系统。关于classifiers的完整列表，可参考 [https://pypi.org/classifiers/](https://link.zhihu.com/?target=https%3A//pypi.org/classifiers/)。

### 打包项目

安装`setuptools`和`wheel`库

```bash
pip install setuptools
pip install wheel
```

运行以下命令打包项目

```bash
python3 setup.py sdist bdist_wheel
```

上面的命令会在dist/目录下生成一个tar.gz的源码包和一个.whl的Wheel包。

```
dist/
  douyin_image-0.1-py3-none-any.whl
  douyin_image-0.1.tar.gz
```

打包完之后，可以从本地安装库，来验证项目能否被成功安装，如下

```bash
pip3 install dist/douyin_image-0.1-py3-none-any.whl
```

### 上传项目

使用twine上传项目，先安装twine

```bash
pip3 install twine
```

安装完之后，运行下面的命令将库上传

```bash
twine upload dist/*
```

