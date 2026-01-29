# clawdbot moltbot

# 一、安装 

nodejs > 22

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
nvm install 25
nvm -v
```

```
docker pull node:25.5.0-alpine3.23
```

```
npm install -g clawdbot@latest

npm install -g moltbot@latest
```

# 二、初始化

```
moltbot onboard
```

配置

~/.clawdbot/clawdbot.json

```
{
  "meta": {
    "lastTouchedVersion": "2026.1.24-3",
    "lastTouchedAt": "2026-01-29T01:42:18.692Z"
  },
  "wizard": {
    "lastRunAt": "2026-01-29T01:13:03.344Z",
    "lastRunVersion": "2026.1.24-3",
    "lastRunCommand": "onboard",
    "lastRunMode": "remote"
  },
  "models": {
    "mode": "merge",
    "providers": {
      "local": {
        "baseUrl": "https://api.scnet.cn/api/llm/v1",
        "apiKey": "APIKEY",
        "api": "openai-completions",
        "models": [
          {
            "id": "Qwen3-30B-A3B",
            "name": "Qwen3-30B-A3B",
            "reasoning": false,
            "input": [
              "text"
            ],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 196608,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "local/Qwen3-30B-A3B"
      },
      "models": {
        "local/Qwen3-30B-A3B": {}
      },
      "workspace": "C:\\Users\\Admin\\clawd",
      "compaction": {
        "mode": "safeguard"
      },
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  },
  "messages": {
    "ackReactionScope": "group-mentions"
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto"
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "controlUi": {
      "allowInsecureAuth": true
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    },
    "remote": {
      "url": "ws://0.0.0.0:18789",
      "token": "aaa"
    }
  }
}
```

# 三、启动

```
clawdbot gateway --token aaa
```

```
http://localhost:18789/?token=aaa
```

# 四、其他

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
export NODE_OPTIONS="--use-env-proxy"
```

```
npm config set proxy http://127.0.0.1:7890
npm config set https-proxy http://127.0.0.1:7890
```

```
npm config set registry https://registry.npmmirror.com
```

```
npm uninstall -g moltbot
npm install -g moltbot@latest
npm list -g --depth=0
```

```
clawdbot config set models.providers.local.apiKey "aa"
```

```
clawdbot reset --scope full --yes
```

```
ssh -R 9999:localhost:8888 user@你的本机IP
```

```
FROM node:25.5.0-alpine3.23

ARG http_proxy=http://127.0.0.1:7890
ENV http_proxy=$http_proxy
ENV https_proxy=$http_proxy
ENV HTTP_PROXY=$http_proxy
ENV HTTPS_PROXY=$http_proxy

RUN apk update && apk add git && npm config set registry https://registry.npmmirror.com && npm i -g clawdbot
```

```
docker run -it --rm \
-e HTTP_PROXY=http://127.0.0.1:7890 \
-e HTTPS_PROXY=http://127.0.0.1:7890 \
-e NODE_OPTIONS="--use-env-proxy" \
-v /hl/clawdbot/clawdbot_conf:/root/.clawdbot \
-p 18789:18789 \
clawdbot_base:1.0.0 sh
```

```
帮我安装插件，指令是：clawdbot plugins install @m1heng-clawd/feishu
```

