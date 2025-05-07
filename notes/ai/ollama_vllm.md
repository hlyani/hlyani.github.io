# Ollama and vLLM

Ollama适用于开发测试，vLLM适用于生产环境部署。

性能对比：

1. 吞吐量 ： vLLM通过连续批处理和内存优化，显著高于Ollama，尤其在高并发时差异更明显。
2. 资源占用 ： Ollama在单机环境下资源占用更低，启动更快；vLLM需要更多初始配置但能更好地利用多卡资源。
3. 延迟 ： Ollama在实时响应场景中延迟更低，而vLLM通过批处理优化可平衡延迟与吞吐。

# Ollama

## 一、部署

https://ollama.com

### 1.部署

```
curl -fsSL https://ollama.com/install.sh | sh
```

### 2.基本命令

```
Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       启动 Ollama 服务
  create      从 Modelfile 创建一个模型
  show        查看模型详细信息
  run         运行一个模型
  stop        停止正在运行的模型
  pull        从注册表拉取一个模型
  push        将一个模型推送到注册表
  list        列出所有可用的模型
  ps          列出当前正在运行的模型
  cp          复制一个模型
  rm          删除一个模型
  help        获取关于任何命令的帮助信息

Flags:
  -h, --help      helpfor ollama
  -v, --version   Show version information
```

### 3.拉取模型

https://ollama.com/library/deepseek-r1:7b

```
ollama pull deepseek-r1:7b
```

### 4.启动模型

```
ENV OLLAMA_HOST=0.0.0.0:11434 ollama serve
```

```
ollama run deepseek-r1:7b
```

# vLLM

## 一、部署

https://docs.vllm.ai

### 1.安装uv

https://docs.astral.sh/uv/getting-started/installation/

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2.安装vLLM

https://docs.vllm.ai/en/latest/getting_started/installation.html

```
uv venv vllm --python 3.12 --seed
source vllm/bin/activate
uv pip install vllm
```

### 3.启动

https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B?local-app=vllm

```
env OPENAI_API_KEY=123456  vllm serve "deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B"
```

### 4.访问

http://0.0.0.0:8000/docs

# 客户端工具

- **CherryStudio** (https://docs.cherry-ai.com/cherrystudio/download)
- **open-webui** (https://github.com/open-webui/open-webui)

## 一、open-webui

```
version: '3'

services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main # 镜像略大，下载情况根据你网速
    ports:
      - "3000:8080"
    environment:
      - OPENAI_API_KEY=123456  # api key
      - OPENAI_API_BASE_URL=http://你服务器地址:8000/v1/  # vllm服务地址
    volumes:
      - ./data:/app/backend/data  # 挂载数据卷（根据项目需求调整路径）
    restart: always
```

```
docker-compose up -d
```

http://0.0.0.0:3000