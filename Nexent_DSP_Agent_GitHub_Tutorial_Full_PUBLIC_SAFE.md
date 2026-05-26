# Nexent 数字信号处理期末复习智能体：macOS 本地部署、知识库、联网搜索、多模态识别与 DSP Python MCP 全流程教程


> **Public-safe version:** 本文已将学校、学号、用户名、本地绝对路径、知识库索引、API Key、Token、数据库密码等内容统一替换为占位符。发布到 GitHub 前，请再次确认没有真实密钥、账号或本地隐私路径。

> 适用场景：希望在 macOS 上从 0 到 1 部署 Nexent，并构建一个“数字信号处理期末复习智能体”的用户。  
> 本文覆盖：Docker 本地部署、模型配置、知识库入库、智能体构建、联网搜索、图片识别、自定义 DSP Python MCP、GitHub Discussion 提交、Issue 提交、隐私检查与常见问题排查。  
> 本文所有密钥、API Key、Token、数据库密码均使用占位符表示，请勿把真实密钥上传到 GitHub。

---

## 目录

1. [项目最终效果](#1-项目最终效果)
2. [环境准备](#2-环境准备)
3. [Nexent 本地部署](#3-nexent-本地部署)
4. [SiliconFlow 模型与 API Key 配置](#4-siliconflow-模型与-api-key-配置)
5. [Nexent 模型管理配置](#5-nexent-模型管理配置)
6. [知识库创建与入库](#6-知识库创建与入库)
7. [创建数字信号处理期末复习智能体](#7-创建数字信号处理期末复习智能体)
8. [配置 knowledge_base_search 知识库检索工具](#8-配置-knowledge_base_search-知识库检索工具)
9. [配置 linkup_search 联网搜索工具](#9-配置-linkup_search-联网搜索工具)
10. [配置视觉语言模型与图片题目识别](#10-配置视觉语言模型与图片题目识别)
11. [创建并注册 DSP Python MCP 工具](#11-创建并注册-dsp-python-mcp-工具)
12. [发布 v1.6 并测试](#12-发布-v16-并测试)
13. [常见问题与排查](#13-常见问题与排查)
14. [GitHub Discussion 提交模板](#14-github-discussion-提交模板)
15. [GitHub Issue 提交流程](#15-github-issue-提交流程)
16. [Safari / Chrome 长截图方法](#16-safari--chrome-长截图方法)
17. [隐私与安全检查清单](#17-隐私与安全检查清单)
18. [如何删除 DSP MCP](#18-如何删除-dsp-mcp)
19. [版本演进记录](#19-版本演进记录)

---

## 1. 项目最终效果

最终构建的智能体名称：

```text
数字信号处理期末复习智能体
Digital Signal Processing Final Review Agent
```

最终能力：

1. 基于“数字信号处理”课程资料知识库进行问答。
2. 生成期末复习总提纲、重点难点、常考题型和复习计划。
3. 解释采样定理、混叠、DFT、DTFT、Z 变换、卷积、LTI 系统等知识点。
4. 调用 `linkup_search` 联网搜索公开资料，补充通俗解释。
5. 调用视觉语言模型识别题目截图、公式截图、PPT 截图、手写笔记图片。
6. 调用自定义 DSP Python MCP 工具进行：
   - DFT 计算；
   - 线性卷积计算；
   - DFT 幅度谱图生成；
   - 卷积结果图生成；
   - FIR 频率响应图生成。
7. 支持“知识库内容 + 联网搜索补充 + 图片识别 + MCP 计算绘图”的综合问答模式。

最终使用模型：

```text
LLM：Qwen3.5-9B
Embedding：bge-m3
Vision-Language Model：Qwen3-VL-8B-Instruct
```

最终主要工具：

```text
knowledge_base_search
linkup_search
analyze_image
calculate_dft
calculate_convolution
plot_dft_magnitude
plot_convolution
plot_frequency_response
```

---

## 2. 环境准备

### 2.1 操作系统

本文基于：

```text
macOS
```

### 2.2 必要软件

建议提前安装：

```text
Docker Desktop
Git
Homebrew
Python 3.12
浏览器：Safari / Chrome
代码编辑器：VS Code
```

### 2.3 安装 Homebrew

如果尚未安装 Homebrew，可先安装。安装方式以 Homebrew 官网为准。

安装完成后检查：

```bash
brew --version
```

### 2.4 安装 Python 3.12

macOS 自带 Python 版本可能较低，MCP SDK 需要 Python 3.10+。建议安装 Python 3.12：

```bash
brew install python@3.12
```

检查：

```bash
python3.12 --version
```

---

## 3. Nexent 本地部署

### 3.1 克隆 Nexent 仓库

```bash
cd ~/Desktop
git clone https://github.com/ModelEngine-Group/nexent.git
cd nexent
```

### 3.2 启动 Docker Desktop

确保 Docker Desktop 已经启动，并且终端可以运行：

```bash
docker ps
```

### 3.3 启动 Nexent 容器

根据项目文档选择对应的 compose 启动方式。常见形式如下：

```bash
docker compose up -d
```

如果项目提供了特定启动脚本或文档，请以项目文档为准。

### 3.4 检查容器状态

```bash
docker compose ps
```

应看到类似容器处于 `Up` 状态：

```text
nexent-web
nexent-config
nexent-runtime
nexent-mcp
nexent-postgresql
nexent-redis
nexent-elasticsearch
nexent-minio
nexent-data-process
```

### 3.5 打开 Nexent

浏览器访问：

```text
http://localhost:<nexent_web_port>
```

如果页面暂时打不开，等待 1~2 分钟后刷新，因为 Elasticsearch、后端服务、数据处理服务可能需要时间初始化。

---

## 4. SiliconFlow 模型与 API Key 配置

### 4.1 获取 SiliconFlow API Key

操作步骤：

1. 打开 SiliconFlow 官网。
2. 登录账号。
3. 进入 API Key / 密钥管理页面。
4. 创建一个新的 API Key。
5. 复制 API Key。

注意：

```text
不要把 API Key 发到 GitHub。
不要把 API Key 截图上传。
不要把 API Key 写进公开 README。
```

### 4.2 本项目使用的模型

推荐配置：

```text
大语言模型：Qwen/Qwen3.5-9B
向量模型：BAAI/bge-m3
视觉语言模型：Qwen/Qwen3-VL-8B-Instruct
```

模型 URL：

```text
LLM / VLM URL:
https://api.siliconflow.cn/v1

Embedding URL:
https://api.siliconflow.cn/v1/embeddings
```

---

## 5. Nexent 模型管理配置

进入：

```text
Nexent 左侧栏 → 模型管理
```

### 5.1 添加大语言模型

点击：

```text
添加模型
```

填写：

```text
模型类型：大语言模型
模型名称：Qwen/Qwen3.5-9B
展示名称：Qwen3.5-9B
模型URL：https://api.siliconflow.cn/v1
API Key：填自己的 SiliconFlow API Key
最大Token数：4096
```

点击：

```text
连通性验证
```

如果显示绿色“可用”，点击保存。

### 5.2 添加向量模型

填写：

```text
模型类型：向量模型
多模态：关闭
模型名称：BAAI/bge-m3
展示名称：bge-m3
模型URL：https://api.siliconflow.cn/v1/embeddings
API Key：填自己的 SiliconFlow API Key
文档切片大小：默认
单次请求切片量：10
```

点击连通性验证。

如果验证成功，保存。

### 5.3 添加视觉语言模型

填写：

```text
模型类型：大语言模型 或 多模态模型
模型名称：Qwen/Qwen3-VL-8B-Instruct
展示名称：Qwen3-VL-8B-Instruct
模型URL：https://api.siliconflow.cn/v1
API Key：填自己的 SiliconFlow API Key
最大Token数：4096
```

如果 Nexent 页面有单独的视觉语言模型位置，则在视觉语言模型位置选择该模型。

---

## 6. 知识库创建与入库

进入：

```text
左侧栏 → 知识库
```

### 6.1 创建知识库

点击：

```text
创建
```

填写：

```text
知识库名称：数字信号处理
向量模型：bge-m3
```

如果页面没有显示“向量模型选择”，后续要特别检查知识库是否绑定了 embedding 模型。

### 6.2 上传课程资料

上传示例：

```text
<course_intro>.pptx
<final_review_material>.pptx
<qa_material>.pptx
```

上传后等待入库完成。

成功状态应显示：

```text
已就绪
```

并看到：

```text
文档数量：3
分块数量：25
向量模型：bge-m3
```

### 6.3 如果入库失败

如果页面显示：

```text
入库失败
```

不要反复乱传文件。先排查后端日志。

常见日志：

```text
Embedding API error: 'NoneType' object has no attribute 'get_embeddings'
```

这通常说明：

```text
知识库记录没有绑定 embedding_model_id
```

可让 Codex 或终端检查数据库：

```bash
docker exec nexent-postgresql psql -U root -d nexent -c "
select knowledge_id, knowledge_name, index_name, embedding_model_id, embedding_model_name
from nexent.knowledge_record_t
order by knowledge_id;
"
```

如果看到：

```text
embedding_model_id = null
embedding_model_name = null
```

说明知识库没有绑定向量模型。

临时修复方式示例：

```sql
update nexent.knowledge_record_t
set embedding_model_id = 5,
    embedding_model_name = 'bge-m3',
    update_time = CURRENT_TIMESTAMP
where embedding_model_id is null
   or embedding_model_name is null;
```

注意：

```text
model_id 不一定是 5，请以你本机模型表实际查询结果为准。
```

查询模型表：

```bash
docker exec nexent-postgresql psql -U root -d nexent -c "
select model_id, model_name, display_name, model_type, connect_status
from nexent.model_record_t
order by model_id;
"
```

修复后：

1. 删除之前失败的文件；
2. 重新上传；
3. 等待入库；
4. 确认状态为“已就绪”。

---

## 7. 创建数字信号处理期末复习智能体

进入：

```text
左侧栏 → 智能体开发
```

点击：

```text
创建智能体
```

建议手动填写，不依赖“生成智能体”按钮。

### 7.1 智能体基本信息

```text
智能体名称：数字信号处理期末复习智能体
智能体变量名：dsp_final_review_agent
作者：Default User 或自己的展示名
大语言模型：Qwen3.5-9B
```

### 7.2 智能体描述

```text
你是一个面向大学生“数字信号处理”课程期末复习的智能助教。
你的主要任务是基于“数字信号处理”知识库中的课程资料，帮助学生梳理知识点、总结重点难点、讲解典型题目、制定复习计划，并在需要时结合联网搜索和 DSP Python MCP 工具完成补充解释、数值计算和图像生成。
回答风格要像耐心的大学助教，适合基础一般的学生理解。
```

---

## 8. 配置 knowledge_base_search 知识库检索工具

### 8.1 选择工具

在智能体开发页面中部：

```text
配置智能体能力 → 选择智能体的工具
```

点击：

```text
刷新工具
```

找到：

```text
knowledge_base_search
```

选中。

### 8.2 配置工具参数

点击工具右侧齿轮，设置：

```text
top_k：3
index_names：选择“数字信号处理”知识库
search_mode：hybrid
rerank：关闭
```

保存工具配置。

### 8.3 保存与发布顺序

必须按顺序：

```text
保存工具配置
保存智能体
发布新版本
```

否则可能出现前端看起来配置了，但发布版本里 `index_names = null` 的问题。

### 8.4 常见错误

如果开始问答无响应，日志出现：

```text
Embedding model is required for knowledge_base_search but index_names is empty
```

说明：

```text
发布版本中的 knowledge_base_search 没有绑定知识库
```

解决方式：

1. 回智能体开发；
2. 打开 `knowledge_base_search` 配置；
3. 重新选择知识库；
4. 保存工具配置；
5. 保存智能体；
6. 重新发布。

---

## 9. 配置 linkup_search 联网搜索工具

### 9.1 获取 Linkup API Key

打开 Linkup 官网，注册登录后进入 API Key 页面，复制 API Key。

注意：

```text
不要公开 Linkup API Key。
不要上传包含 linkup_api_key 的 JSON。
```

### 9.2 配置 linkup_search

进入智能体开发：

```text
配置智能体能力 → 选择工具
```

找到：

```text
linkup_search
```

点击齿轮，填写：

```text
linkup_api_key：填自己的 Linkup API Key
max_results：3
image_filter：根据需要开启或关闭
```

工具测试输入：

```text
查找一下采样定理
```

如果返回结果，说明配置成功。

### 9.3 使用规则

在“使用要求”里加入：

```text
【联网搜索使用规则】

当用户明确提出以下需求时，可以调用 linkup_search：
1. 查找最新资料、公开教程、网上例题、教材推荐、学习网站；
2. 对知识库内容进行网络补充解释；
3. 查询某个数字信号处理概念的通俗类比、公开课程解释或外部参考资料；
4. 用户明确说“联网搜索”“网上查一下”“找公开资料”。

回答时必须区分：
一、本地知识库内容
二、联网搜索补充内容
三、我的复习建议

如果联网搜索结果与知识库内容不一致，要说明差异，不要混在一起。
```

---

## 10. 配置视觉语言模型与图片题目识别

### 10.1 在模型管理中确认 VLM 可用

确认视觉语言模型：

```text
Qwen3-VL-8B-Instruct
```

处于绿色可用状态。

### 10.2 在智能体中配置图片识别规则

在“使用要求”中追加：

```text
当用户上传题目截图、公式截图、PPT截图或手写笔记照片时，先识别图片中的文字、公式和题目条件，再判断考查的数字信号处理知识点，随后结合“数字信号处理”知识库进行讲解，最后给出详细解题步骤、易错点和同类题解题套路。

如果图片不清晰，要提醒用户重新上传更清晰的图片。
```

### 10.3 测试图片识别

测试提示：

```text
请识别这张图片中的题目，并结合我的数字信号处理知识库进行讲解。
```

可测试内容：

```text
采样与混叠
DFT
LTI 系统与卷积
```

---

## 11. 创建并注册 DSP Python MCP 工具

### 11.1 为什么需要 MCP

Calculator MCP 只能做简单表达式计算，不适合画图。

DSP Python MCP 可以实现：

```text
calculate_dft
calculate_convolution
plot_dft_magnitude
plot_convolution
plot_frequency_response
```

并自动生成：

```text
DFT 幅度谱图
卷积结果图
频率响应图
```

### 11.2 Nexent v2.1.1 的 MCP 现状

在 v2.1.1 中，左侧：

```text
MCP 工具
```

页面可能显示：

```text
MCP 工具管理即将推出
```

这并不代表后端不支持 MCP。

排查结果：

```text
支持通过后端 API /mcp/add 注册远程 SSE MCP 服务。
```

推荐方案：

```text
宿主机启动 DSP Python SSE MCP 服务
Nexent 通过 http://host.docker.internal:<mcp_port>/sse 访问
```

### 11.3 创建目录

```bash
mkdir -p ~/Desktop/dsp_mcp_tool
mkdir -p ~/Desktop/dsp_mcp_outputs
cd ~/Desktop/dsp_mcp_tool
```

### 11.4 创建 Python 3.12 虚拟环境

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install mcp numpy matplotlib scipy
```

### 11.5 创建 DSP MCP Server

创建文件：

```bash
nano dsp_mcp_server.py
```

写入示例代码：

```python
from __future__ import annotations

import json
import os
import uuid
from typing import List

import matplotlib
matplotlib.use("Agg")

import matplotlib.pyplot as plt
import numpy as np
from scipy import signal
from mcp.server.fastmcp import FastMCP


mcp = FastMCP("DSP Python MCP")

OUTPUT_DIR = os.path.expanduser("~/Desktop/dsp_mcp_outputs")
os.makedirs(OUTPUT_DIR, exist_ok=True)


def parse_sequence(sequence: str) -> np.ndarray:
    try:
        data = json.loads(sequence)
    except json.JSONDecodeError as exc:
        raise ValueError("Input must be a JSON list string, for example: [1,2,1,0]") from exc

    if not isinstance(data, list):
        raise ValueError("Input must be a list.")

    return np.array(data, dtype=complex)


def complex_array_to_dict(arr: np.ndarray) -> List[dict]:
    result = []
    for value in arr:
        result.append({
            "real": float(np.real(value)),
            "imag": float(np.imag(value)),
            "magnitude": float(np.abs(value)),
            "phase_rad": float(np.angle(value)),
        })
    return result


@mcp.tool()
def calculate_dft(sequence: str) -> str:
    x = parse_sequence(sequence)
    X = np.fft.fft(x)

    output = {
        "input_sequence": sequence,
        "N": int(len(x)),
        "dft": complex_array_to_dict(X),
        "magnitude": [float(v) for v in np.abs(X)],
        "phase_rad": [float(v) for v in np.angle(X)],
    }

    return json.dumps(output, ensure_ascii=False, indent=2)


@mcp.tool()
def calculate_convolution(x_sequence: str, h_sequence: str) -> str:
    x = parse_sequence(x_sequence)
    h = parse_sequence(h_sequence)
    y = np.convolve(x, h)

    output = {
        "x_sequence": x_sequence,
        "h_sequence": h_sequence,
        "y_length": int(len(y)),
        "y_real": [float(np.real(v)) for v in y],
        "y_imag": [float(np.imag(v)) for v in y],
    }

    return json.dumps(output, ensure_ascii=False, indent=2)


@mcp.tool()
def plot_dft_magnitude(sequence: str) -> str:
    x = parse_sequence(sequence)
    X = np.fft.fft(x)
    mag = np.abs(X)
    k = np.arange(len(x))

    filename = f"dft_magnitude_{uuid.uuid4().hex[:8]}.png"
    path = os.path.join(OUTPUT_DIR, filename)

    plt.figure(figsize=(8, 4.5))
    plt.stem(k, mag)
    plt.xlabel("n")
    plt.ylabel("|X[k]|")
    plt.title("DFT Magnitude Spectrum")
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig(path, dpi=180)
    plt.close()

    return json.dumps({
        "image_path": path,
        "N": int(len(x)),
        "magnitude": [float(v) for v in mag],
    }, ensure_ascii=False, indent=2)


@mcp.tool()
def plot_convolution(x_sequence: str, h_sequence: str) -> str:
    x = parse_sequence(x_sequence)
    h = parse_sequence(h_sequence)
    y = np.convolve(x, h)
    n = np.arange(len(y))

    filename = f"convolution_{uuid.uuid4().hex[:8]}.png"
    path = os.path.join(OUTPUT_DIR, filename)

    plt.figure(figsize=(8, 4.5))
    plt.stem(n, np.real(y))
    plt.xlabel("n")
    plt.ylabel("y[n]")
    plt.title("Linear Convolution Result")
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig(path, dpi=180)
    plt.close()

    return json.dumps({
        "image_path": path,
        "y_real": [float(np.real(v)) for v in y],
    }, ensure_ascii=False, indent=2)


@mcp.tool()
def plot_frequency_response(b_coefficients: str) -> str:
    b = parse_sequence(b_coefficients)
    w, h = signal.freqz(b, worN=512)
    magnitude = np.abs(h)

    filename = f"frequency_response_{uuid.uuid4().hex[:8]}.png"
    path = os.path.join(OUTPUT_DIR, filename)

    plt.figure(figsize=(8, 4.5))
    plt.plot(w, magnitude)
    plt.xlabel("Normalized frequency (rad/sample)")
    plt.ylabel("|H(e^jw)|")
    plt.title("FIR Frequency Response")
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig(path, dpi=180)
    plt.close()

    return json.dumps({
        "image_path": path,
        "frequency_axis_rad": [float(v) for v in w],
        "magnitude": [float(v) for v in magnitude],
    }, ensure_ascii=False, indent=2)


if __name__ == "__main__":
    mcp.run(transport="sse", host="0.0.0.0", port=<mcp_port>)
```

### 11.6 启动 MCP 服务

```bash
cd ~/Desktop/dsp_mcp_tool
source .venv/bin/activate
python dsp_mcp_server.py
```

服务地址：

```text
宿主机：http://localhost:<mcp_port>/sse
Nexent 容器：http://host.docker.internal:<mcp_port>/sse
```

如果需要后台运行，可用：

```bash
nohup ~/Desktop/dsp_mcp_tool/.venv/bin/python ~/Desktop/dsp_mcp_tool/dsp_mcp_server.py > ~/Desktop/dsp_mcp_tool/dsp_mcp.log 2>&1 &
```

查看进程：

```bash
ps aux | grep dsp_mcp_server.py
```

停止进程：

```bash
kill <PID>
```

### 11.7 注册到 Nexent

推荐让 Codex 先查看 `/mcp/add` 的真实 API 格式，不要直接猜 payload。

注册信息：

```text
服务名：<your_mcp_service_name>
URL：http://host.docker.internal:<mcp_port>/sse
```

注册成功后检查：

```text
/mcp/list 显示 <your_mcp_service_name> status:true
/tool/scan_tool 扫描成功
/tool/list 出现 5 个工具
/tool/validate 可以调用 calculate_dft
```

### 11.8 在智能体中选择 MCP 工具

进入：

```text
智能体开发 → 数字信号处理期末复习智能体
```

点击：

```text
刷新工具
```

选择：

```text
calculate_dft
calculate_convolution
plot_dft_magnitude
plot_convolution
plot_frequency_response
```

保留原有工具：

```text
knowledge_base_search
linkup_search
analyze_image
```

### 11.9 添加 v1.6 使用规则

在“使用要求”末尾追加：

```text
【v1.6 DSP Python MCP 计算与绘图规则】

当用户要求计算 DFT、幅度谱、卷积、频率响应、滤波器响应或自动生成计算图时，应优先调用 DSP Python MCP 工具，而不是只靠语言模型心算。

可调用工具包括：
1. calculate_dft：计算有限长序列的 DFT；
2. calculate_convolution：计算两个离散序列的线性卷积；
3. plot_dft_magnitude：生成 DFT 幅度谱图；
4. plot_convolution：生成卷积结果图；
5. plot_frequency_response：生成 FIR 滤波器频率响应图。

回答计算题时必须包含：
一、题目条件识别；
二、调用的工具名称；
三、计算结果；
四、图像路径或图像说明；
五、对应的数字信号处理知识点；
六、易错点；
七、同类题套路。

如果工具返回 image_path，应告诉用户该图像已生成，并解释图像表示的含义。

当用户要求“画图”“生成幅度谱”“生成卷积图”“生成频率响应图”“自动生图”时，应优先调用 plot_dft_magnitude、plot_convolution 或 plot_frequency_response。
```

---

## 12. 发布 v1.6 并测试

### 12.1 发布

按顺序：

```text
保存工具配置
保存智能体
发布
```

版本说明：

```text
v1.6：增加 DSP Python MCP，用于 DFT、卷积、频率响应计算与图像生成。
```

### 12.2 测试 DFT 计算

```text
请调用 DSP Python MCP 计算 x[n]=[1,2,1,0] 的 4 点 DFT，并给出幅度谱。
```

期望结果：

```text
X[0] = 4
X[1] = -2j
X[2] = 0
X[3] = 2j
|X[k]| = [4, 2, 0, 2]
```

### 12.3 测试 DFT 幅度谱图

```text
请调用 DSP Python MCP 计算 x[n]=[1,2,1,0] 的 4 点 DFT，并生成幅度谱图。
```

期望返回：

```text
image_path: ~/Desktop/dsp_mcp_outputs/dft_magnitude_xxxxxxxx.png
```

### 12.4 测试卷积

```text
请调用 DSP Python MCP 计算 x[n]=[1,2,1] 和 h[n]=[1,-1] 的线性卷积，并生成卷积结果图。
```

期望结果：

```text
y[n] = [1, 1, -1, -1]
```

### 12.5 测试频率响应

```text
请调用 DSP Python MCP 为 FIR 系数 b=[1,-1] 生成频率响应幅度图，并解释它代表什么。
```

---

## 13. 常见问题与排查

### 13.1 页面显示模型可用，但生成智能体时报“模型不可用”

解决方式：

```text
不要依赖自动生成智能体。
手动填写智能体名称、变量名、描述、使用要求。
```

### 13.2 入库失败

重点检查：

```text
knowledge_record_t.embedding_model_id
knowledge_record_t.embedding_model_name
```

### 13.3 问答无响应

重点检查：

```text
knowledge_base_search.index_names
```

### 13.4 linkup_search 报 FieldInfo / get_secret_value

先检查是否填写了 Linkup API Key。

如果填写后仍失败，再查工具字段定义或后端日志。

### 13.5 MCP 工具刷新后看不到

检查：

```text
DSP MCP 服务是否还在运行
/mcp/list 是否 status:true
/tool/scan_tool 是否执行成功
/tool/list 是否出现 5 个工具
Nexent 容器是否能访问 http://host.docker.internal:<mcp_port>/sse
```

### 13.6 Mac 内存查看

活动监视器：

```text
Command + Space → 搜索“活动监视器” → 内存
```

重点看：

```text
内存压力
Docker
python
Chrome / Safari
```

终端：

```bash
top -o mem
```

---

## 14. GitHub Discussion 提交模板

建议标题：

```text
[Agent Submission] <your_university> + <your_student_id> - 数字信号处理期末复习智能体
```

正文可包含：

```markdown
## <your_university> + <your_student_id>

### Agent Name / 智能体名称

数字信号处理期末复习智能体  
Digital Signal Processing Final Review Agent

## 项目简介 / Overview

本项目基于 Nexent 平台构建了一个面向大学生“数字信号处理”课程期末复习场景的智能体。该智能体结合课程资料知识库、联网搜索工具、视觉语言模型能力和自定义 DSP Python MCP 工具，能够帮助学生完成知识点梳理、重点难点总结、常考题型整理、题目讲解、截图识别、复习计划制定、考前冲刺，以及 DFT、卷积和频率响应等数字信号处理任务的计算与图像生成。

## 使用模型 / Models

- 大语言模型 / LLM：Qwen3.5-9B
- 向量模型 / Embedding Model：bge-m3
- 视觉语言模型 / Vision-Language Model：Qwen3-VL-8B-Instruct

## 工具配置 / Tools

- `knowledge_base_search`
- `linkup_search`
- `analyze_image`
- `<your_mcp_service_name>`
  - `calculate_dft`
  - `calculate_convolution`
  - `plot_dft_magnitude`
  - `plot_convolution`
  - `plot_frequency_response`

## 当前能力 / Key Features

1. 基于知识库进行数字信号处理复习问答。
2. 联网搜索公开资料进行补充解释。
3. 识别题目截图、公式截图和 PPT 截图。
4. 调用 DSP Python MCP 完成 DFT、卷积和频率响应计算。
5. 自动生成 DFT 幅度谱图、卷积结果图和 FIR 频率响应图。

## 隐私说明 / Privacy

No API keys, tokens, `.env` files, database files, Docker data directories, or private course materials are included.
```

### 14.1 附图建议

建议附两张图：

1. Nexent 问答页面截图；
2. DSP MCP 生成的 DFT Magnitude Spectrum 图。

说明文字：

```markdown
The first screenshot shows the agent running in the Nexent chat interface, and the second image shows a DFT magnitude spectrum generated by the custom DSP Python MCP tool.
```

中文：

```markdown
第一张截图展示了智能体在 Nexent 问答页面中的运行效果，第二张图片展示了自定义 DSP Python MCP 工具生成的 DFT 幅度谱图。
```

### 14.2 不建议上传未脱敏 JSON

如果 JSON 中包含：

```text
linkup_api_key
api_key
token
secret
password
Bearer
sk-
```

必须删除或打码。

推荐：

```json
"linkup_api_key": "REMOVED"
```

如果不确定，宁可不上传 JSON，只发正文和截图。

---

## 15. GitHub Issue 提交流程

### 15.1 已提交 Issue：配置未持久化

标题：

```text
[Bug] Knowledge base embedding_model_id and tool index_names are not persisted
```

核心问题：

```text
knowledge_record_t.embedding_model_id 为空
knowledge_base_search.index_names 为空
```

### 15.2 Issue 表单字段

如果 GitHub Issue 表单只有：

```text
Problem Description
Reproduction Steps
Additional Information
```

则每个框分别填写。

#### Problem Description

```markdown
During local deployment and agent creation, I encountered cases where the frontend showed that configuration was completed, but the backend database or the published agent version did not actually contain the required configuration.

The two main issues were:

1. A knowledge base was created from the frontend, but `knowledge_record_t.embedding_model_id` and `embedding_model_name` were empty. As a result, document indexing failed even though the embedding model `bge-m3` was shown as available in Model Management.

2. The agent frontend showed `knowledge_base_search` as selected/configured, but the published agent version still had `index_names = null`. As a result, the chat page could not correctly use the knowledge base.

Relevant backend errors:

```text
Embedding API error: 'NoneType' object has no attribute 'get_embeddings'
```

```text
Embedding model is required for knowledge_base_search but index_names is empty
```

This made it difficult to determine whether the problem came from file parsing, embedding model configuration, knowledge base binding, tool configuration, or published agent version persistence.
```

#### Reproduction Steps

```markdown
1. Deploy Nexent v2.1.1 locally with Docker on macOS.
2. Add an embedding model in Model Management.
3. Confirm the embedding model `bge-m3` is shown as available.
4. Create a new knowledge base from the frontend.
5. Upload PDF / Markdown / PPT files.
6. The files may fail to index if the knowledge base record does not contain `embedding_model_id`.
7. Check the database and observe that `knowledge_record_t.embedding_model_id` and `embedding_model_name` are empty.
8. Create a new agent.
9. Add `knowledge_base_search` as a tool.
10. Configure the tool to use the knowledge base.
11. Save the tool configuration.
12. Save and publish the agent.
13. Open the chat page and select the published agent.
14. Send a knowledge-base-related query.
15. The agent may fail if the published tool config still has `index_names = null`.

Temporary workaround used locally:

1. Manually bind the knowledge base record to the available `bge-m3` embedding model.
2. Delete the failed files and re-upload them.
3. Reopen the `knowledge_base_search` tool configuration.
4. Explicitly set `index_names`, `top_k`, `search_mode`, and `rerank`.
5. Save tool config.
6. Save the agent.
7. Publish a new version.

After these steps, document indexing and agent chat worked correctly.
```

#### Additional Information

```markdown
Expected behavior:

1. When creating or using a knowledge base, Nexent should either:
   - automatically bind the current available embedding model;
   - or require the user to select an embedding model;
   - or block file upload with a clear message if no embedding model is bound.

2. When publishing an agent, if `knowledge_base_search` is selected but `index_names` is empty, Nexent should:
   - block publishing;
   - show a clear validation error such as `Please select a knowledge base for knowledge_base_search`;
   - or automatically validate the published tool configuration before publishing.

3. The system should not allow publishing an agent that fails at runtime because a required tool parameter is missing.

4. The UI should make the difference clearer between:
   - Save tool config
   - Save agent
   - Publish version

5. It would be helpful to add a debug-friendly display of the effective published tool configuration.

Environment:

- Nexent version: v2.1.1
- Deployment: Local Docker deployment on macOS
- Browser: Chrome / Safari
- Model provider: SiliconFlow
- LLM: Qwen3.5-9B
- Embedding model: bge-m3

No API keys, `.env` files, database files, or private course materials are included in this issue.
```

---

## 16. Safari / Chrome 长截图方法

### 16.1 Chrome 长截图

1. 打开页面。
2. 按：

```text
Option + Command + I
```

3. 按：

```text
Command + Shift + P
```

4. 搜索：

```text
screenshot
```

5. 选择：

```text
Capture full size screenshot
```

### 16.2 Safari 保存长页面

Safari 没有像 Chrome 一样稳定的内置 full page screenshot 命令。

推荐方法：

#### 方法 A：导出 PDF

```text
Safari → 文件 → 导出为 PDF
```

适合保存完整页面，但 GitHub 展示不如图片直观。

#### 方法 B：打印为 PDF

```text
Command + P → PDF → 存储为 PDF
```

#### 方法 C：换 Chrome 截图

如果要发 GitHub 图片，建议临时用 Chrome 打开同一个本地页面并使用 full size screenshot。

#### 方法 D：使用第三方长截图工具

可以使用长截图浏览器插件或 macOS 截图工具类软件，但要确认不会上传隐私数据到第三方平台。

---

## 17. 隐私与安全检查清单

发布到 GitHub 前，必须确认不包含：

```text
API Key
SiliconFlow Key
Linkup Key
OpenAI Key
Bearer Token
sk-
.env
数据库密码
PostgreSQL 密码
MinIO 密码
Docker 数据目录
完整本地路径中的用户名
私有课程资料原文件
未脱敏 JSON
```

重点检查 JSON：

```text
linkup_api_key
api_key
token
secret
password
```

建议公开说法：

```text
Sensitive data is not included.
API keys and tokens have been removed or masked.
Private course materials are not uploaded.
```

---

## 18. 如何删除 DSP MCP

### 18.1 停止 MCP 进程

查找进程：

```bash
ps aux | grep dsp_mcp_server.py
```

停止：

```bash
kill <PID>
```

### 18.2 在智能体中取消工具

进入：

```text
智能体开发 → 数字信号处理期末复习智能体
```

取消选择：

```text
calculate_dft
calculate_convolution
plot_dft_magnitude
plot_convolution
plot_frequency_response
```

保存并重新发布。

### 18.3 删除本地 MCP 目录

```bash
rm -rf ~/Desktop/dsp_mcp_tool
rm -rf ~/Desktop/dsp_mcp_outputs
```

### 18.4 不要删除 Nexent 数据目录

不要删除：

```text
~/Desktop/nexent/nexent/docker/nexent-data
```

该目录可能包含：

```text
数据库
知识库
向量数据
对象存储数据
```

---

## 19. 版本演进记录

### v1.0

基础知识库复习智能体。

### v1.1

修复 `knowledge_base_search` 工具绑定知识库问题。

### v1.2

新增 `linkup_search` 联网搜索能力。

### v1.4

新增视觉语言模型图片识别能力。

### v1.5

优化联网搜索和图片识别规则。

### v1.6

新增 DSP Python MCP：

```text
calculate_dft
calculate_convolution
plot_dft_magnitude
plot_convolution
plot_frequency_response
```

实现：

```text
DFT 计算
卷积计算
频率响应计算
DFT 幅度谱图生成
卷积图生成
FIR 频率响应图生成
```

---

## 总结

通过以上步骤，可以在 macOS 本地从 0 到 1 完成 Nexent 智能体构建，并逐步扩展到知识库、联网搜索、多模态图片理解和 DSP Python MCP 计算绘图能力。

最终形成的智能体不仅能完成课程复习问答，还能结合外部工具完成数字信号处理中的实际计算与可视化，适合作为课程复习、实验辅助和考试冲刺场景下的本地智能助教。
