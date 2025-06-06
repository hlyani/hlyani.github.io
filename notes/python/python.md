# Python 相关



# 一、安装

```
wget https://www.python.org/ftp/python/3.13.0/Python-3.13.0.tgz
./configure --enable-optimizations --with-ensurepip=install --with-openssl=/usr/local/openssl-1.1.1 --with-openssl-rpath=auto
make altinstall -j 20
```

# 二、查看 gpu 是否可用

```
python -c  "import torch; print(torch.cuda.is_available()); print(torch.cuda.device_count())"
python -c "import torch.distributed as dist; print(dist.is_nccl_available())"
```

# 三、代码片断

## 1、enumerate， zip

```
for i, v in enumerate(['tic', 'tac', 'toe']):

for q, a in zip(questions, answers):

>>> for x in range(1, 11):
...     print(repr(x).rjust(2), repr(x*x).rjust(3), end=' ')
...     # 注意前一行 'end' 的使用
...     print(repr(x*x*x).rjust(4))

for x in range(1, 11):
...     print('{0:2d} {1:3d} {2:4d}'.format(x, x*x, x*x*x))

```

## 2、remove_all_whitespace

```
def remove_all_whitespace(text: str) -> str:
    """
    移除字符串中的所有空白字符（包括空格、换行、制表符等）

    Args:
        text: 要处理的文本

    Returns:
        处理后的无空白字符文本
    """
    return re.sub(r'\s+', '', text)
```

## 3、collect_tools_from_modules

```
def collect_tools_from_modules(module_paths):
    from langchain_core.tools import BaseTool
    import importlib
    import inspect

    tools = []
    for path in module_paths:
        module = importlib.import_module(path)
        for _, obj in inspect.getmembers(module):
            if isinstance(obj, BaseTool):
                tools.append(obj)
    return tools
```

## 4、load_modules

```
def load_modules(package="src.agents"):
    for _, module_name, _ in pkgutil.iter_modules([package.replace(".", "/")]):
        # 排除 agents.py
        if module_name in ["agents"]:
            continue
        try:
            module = importlib.import_module(f"{package}.{module_name}")
            for attr_name in dir(module):
                if is_agent_node(attr_name):
                    describe = getattr(module, "Describe")

                    # 排除不需要描述的 node
                    if attr_name not in ["executor_general_node"]:
                        AgentDescribe[get_node_name(attr_name)] = describe

                    AgentExecutorNode[get_node_name(
                        attr_name)] = getattr(module, attr_name)
        except Exception as e:
            print(f"加载模块 {module_name} 失败: {e}")
```

