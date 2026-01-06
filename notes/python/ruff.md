# ruff

# 一、配置

pyproject.yaml

```
[build-system]
requires = ["hatchling>=1.27.0"]
build-backend = "hatchling.build"

[project]
name = "aaa"
version = "1.0.0"
description = "project"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "mcp>=1.13.1",
]

[dependency-groups]
dev = [
  "black>=25.11.0",
  "ruff>=0.8.0",
  "pre-commit>=4.5.0",
  "hatchling>=1.27.0",
  "pytest",
  "pytest-cov",
  "pytest-asyncio"
]

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = "-v --cov=src --cov-report=term-missing"
filterwarnings = ["ignore::DeprecationWarning", "ignore::UserWarning"]

[tool.coverage.report]
fail_under = 25

[tool.hatch.build.targets.wheel]
packages = ["src"]

# === 代码质量工具配置 ===

[tool.black]
line-length = 88
target-version = ["py312"]
include = '\.pyi?$'
extend-exclude = '''
^/build/
^/dist/
^/.venv/
^/venv/
'''

[[tool.uv.index]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true

[tool.ruff]
target-version = "py312"
line-length = 120

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors (代码错误)
    "W",    # pycodestyle warnings (代码警告)
    "F",    # pyflakes (未使用导入、未定义变量等)
    "I",    # isort (导入排序)
    "B",    # flake8-bugbear (潜在bug)
    "C4",   # flake8-comprehensions (列表推导式优化)
    "UP",   # pyupgrade (Python升级建议)
    "SIM",  # flake8-simplify (代码简化)
    # "T20",  # flake8-print (print语句检查)
    "RET",  # flake8-return (返回语句优化)
    # "PTH",  # flake8-use-pathlib (pathlib使用建议)
]
ignore = [
    "E501",  # 行长度由Black处理
    "B008",  # 函数默认参数中不允许函数调用
    "C901",  # 函数复杂度过高
    "RET504", # 不必要的赋值后再返回
    "SIM108",   
    "SIM112",   
    "SIM115",
    "E741",  # 避免使用l、O、I作为变量名
    "B006",
    "F811",
]
fixable = [
    "E", "W", "F", "I", "UP", "SIM", "C4", "RET", "F841"
]

# 文件级忽略规则
per-file-ignores = { "__init__.py" = ["F401"], "agent_manager.py" = ["F401"] }

# 导入排序配置
[tool.ruff.lint.isort]
known-first-party = ["src"]
known-third-party = ["langchain", "langgraph", "pydantic", "fastapi"]
section-order = ["future", "standard-library", "third-party", "first-party", "local-folder"]

# 规则特定配置
[tool.ruff.lint.flake8-bugbear]
extend-immutable-calls = ["fastapi.Depends", "fastapi.Query"]

[tool.ruff.lint.pyupgrade]
keep-runtime-typing = true


```

setup-precommit.py

```
#!/usr/bin/env python3
"""
Pre-commit钩子安装脚本
基于现代pre-commit框架
"""

import subprocess
import sys
from pathlib import Path


def install_precommit():
    """安装pre-commit钩子"""
    print("🚀 安装pre-commit钩子...")

    try:
        # 安装pre-commit
        subprocess.run([sys.executable, "-m", "pip", "install", "pre-commit"], check=True)

        # 安装钩子
        subprocess.run(["pre-commit", "install"], check=True)

        # 运行一次检查配置
        subprocess.run(["pre-commit", "run", "--all-files"], check=True)

        print("✅ pre-commit钩子安装完成！")
        print("💡 钩子将在每次git commit时自动运行")
        print("💡 手动运行: pre-commit run --all-files")

    except subprocess.CalledProcessError as e:
        print(f"❌ 安装失败: {e}")
        sys.exit(1)


def main():
    """主函数"""
    if not Path(".pre-commit-config.yaml").exists():
        print("❌ 未找到.pre-commit-config.yaml文件")
        sys.exit(1)

    install_precommit()


if __name__ == "__main__":
    main()

```

.pre-commit-config.yaml

```
# 现代pre-commit配置
# 基于Ruff的单一工具链
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.14.10
    hooks:
      # Ruff代码规范检查和自动修复
      - id: ruff
        args: [--fix, --unsafe-fixes, --exit-non-zero-on-fix]
      
      # 导入排序
      - id: ruff-format
        args: [--check]

  # Ruff-format 替代 Black，避免冲突
  - repo: local
    hooks:
      # 自定义检查：确保没有函数级导入
      - id: no-function-imports
        name: Check for function-level imports
        entry: bash -c 'ruff check --select=E402 src/ || (echo "❌ 发现函数级导入问题" && exit 1)'
        language: system
        pass_filenames: false
        always_run: true
```

# 二、使用

## 1、安装

```
# 安装Ruff和pre-commit
uv add ruff pre-commit

# 安装git hooks（首次使用）
mkdir -p .git/hooks
pre-commit install

# 更新
pre-commit autoupdate
```

## 2、使用

```
# 直接运行钩子脚本（如当前脚本）
./.git/hooks/pre-commit
```

```
# 对所有文件运行 pre-commit 检查
pre-commit run --all-files

# 对暂存区的文件运行 pre-commit 检查
pre-commit run
```

```
ruff format .
```

```
# 跳过检查（紧急情况）
git commit --no-verify -m "紧急提交"
git commit -n -m "紧急提交"
```

