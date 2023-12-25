# Python包管理

<show-structure for="chapter" depth="2"></show-structure>

## 1. PIP

### 1.1 镜像源加速

学习 Python 的过程中，你一定遇到过使用 `pip` 安装包时下载速度贼慢，心态被搞得很爆炸。这是因为默认情况下，Python 安装第三方库所使用的安装源是 PyPI，这个网站是在美国的，所以没有梯子下载速度慢很正常。 要解决这个问题的方法也非常简单，**只需要将 PyPi 安装源更换为国内即可**。

国内比较好用的 PIP 镜像源主要有三个，个人比较推荐豆瓣镜像源:
- **豆瓣**: pypi.doubanio.com/simple
- **清华**: pypi.tuna.tsinghua.edu.cn/simple
- **阿里云**: mirrors.aliyun.com/pypi/simple

镜像源的使用通常有 

> 有些库安装时可能提示找不到对应的版本，这是因为国内镜像源没有同步这些库，比如 cn2an。通常这种情况，我们直接使用 pypi.org



## 2. CONDA

## 3. Poetry

## 4. PDM

