# Jupyter 内核操作

<show-structure depth="2"/>


## 1. 添加 Python 虚拟环境内核

将一个 Python 虚拟环境添加到 Jupyter 内核中，以便直接在 Notebook 中直接使用。

```Bash
# 进入到虚拟环境中
conda activate chatglm3
pip install jupyter ipykernel
python -m ipykernel install --user  --name chatglm3 --display-name "ChatGLM3"
```

添加完之后如果刷新页面没有出现 ChatGLM3 的内核，那可能需要重启下 Jupyter Notebook 的服务。

## 2. 卸载虚拟环境内核

使用 `jupyter kernelspec list` 查看当前 Jupyter 环境中的内核有哪些，确认需要移除的虚拟环境名称，再通过 `jupyter kernelspec remove` 移除相应的环境。值得注意的是，这个操作不仅可以移除 Python 虚拟环境内核，其他环境的内核同样也可以。 

```Bash
jupyter kernelspec remove chatglm3
```
