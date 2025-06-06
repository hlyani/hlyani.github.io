# LangGraph

# 一、基础环境

```
uv venv -p 3.13
source .venv/Scripts/activate
```

```
uv pip install --upgrade "langgraph-cli[inmem]"
mkdir demo
langgraph new ./demo --template react-agent-python
cd demo
uv pip install -e .
touch .env
```



```
uv pip install langgraph langchain-community langchain-openai tavily-python
```

```
export TAVILY_API_KEY=""
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY=""
```

```
uv pip install langchain-deepseek
```

```
os.environ["DEEPSEEK_API_KEY"]
```

```
uv pip install "langgraph-cli[inmem]"
```



```
uv pip install dashscope
```

