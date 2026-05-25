# Nexent macOS 从 0 到 1 本地部署、智能体构建、调试与 GitHub 提交完整教程

> **适用系统：macOS**  
> **实际验证环境：Mac + Docker Desktop + Nexent v2.1.1 + 本地 Docker 部署**  
> **适用对象：从未接触过 Nexent / 智能体 / Docker 的小白用户**  
> **最终目标：在 Mac 本地部署 Nexent，构建“数字信号处理期末复习智能体”，完成知识库入库、智能体发布、GitHub Discussions 提交。**  
> **安全说明：本文档不包含 API Key、Token、`.env` 文件内容、数据库密码、课程原始资料或其他敏感数据。**

> **脱敏说明：本版本已将本机用户名、GitHub 用户名、学校、学号、Discussion 编号等个人标识替换为占位符。提交课程作业时可按要求恢复学校与学号；公开发布教程时建议保持占位符。**

---

## 0. 本文档说明

这是一份基于本次完整实操记录整理的 **macOS 版一次通过教学文档**。

它不是单纯复制官方文档，而是把以下内容合并为一份完整流程：

1. Nexent 官方 Docker 部署流程。
2. Docker Desktop for Mac 安装流程。
3. 本次在 Mac 上实际部署 Nexent v2.1.1 的操作。
4. 硅基流动 API Key 获取与模型接入。
5. 大语言模型与向量模型配置。
6. 知识库创建、上传文件、入库失败排查与修复。
7. 智能体创建、工具绑定、保存、发布、开始问答测试。
8. 导出智能体 JSON。
9. GitHub Discussions 提交。
10. 重复 Discussion 的处理方式。
11. 后续重启、清理、删除部署内容。
12. 所有踩坑原因与解决办法。

本文档特别强调：

```text
这份教程是在 macOS 上使用 Docker Desktop 本地部署 Nexent 的教程。
Windows / Linux 用户可以参考思路，但命令路径和 Docker 安装方式可能不同。
```

---

## 1. 最终完成结果

本次最终成功完成了以下目标：

1. 在 Mac 本机通过 Docker 部署并运行 Nexent。
2. 访问 `http://localhost:3000`，进入 Nexent Web 页面。
3. 接入大语言模型 `Qwen3.5-9B`。
4. 接入向量模型 `bge-m3`。
5. 创建并成功入库“数字信号处理”知识库。
6. 上传 3 个课程相关 PPTX 文件，状态均为“已就绪”。
7. 创建“数字信号处理期末复习智能体”。
8. 配置 `knowledge_base_search` 工具，并绑定“数字信号处理”知识库。
9. 成功发布智能体。
10. 在“开始问答”页面完成测试，智能体能够基于知识库生成数字信号处理期末复习提纲。
11. 导出智能体 JSON 配置文件。
12. 在 GitHub Discussions 中完成智能体提交。
13. 发现重复 Discussion 后，确认普通提交者不一定具备删除权限，并给出作废 / 请求维护者删除的处理方式。

最终智能体信息：

```text
智能体名称：数字信号处理期末复习智能体
Agent Name: dsp_final_review_agent
课程方向：数字信号处理
知识库名称：数字信号处理
知识库文档数量：3
知识库分块数量：25
大语言模型：Qwen3.5-9B
向量模型：bge-m3
工具：knowledge_base_search
部署方式：macOS 本地 Docker 部署
访问地址：http://localhost:3000
Nexent 版本：v2.1.1
GitHub Discussion 标题：[Agent Submission] <your_university> + <your_student_id> - 数字信号处理期末复习智能体
```

---

## 2. 官方流程与本次实际流程的对应关系

### 2.1 Nexent 官方推荐流程

Nexent 官方快速配置流程可以概括为：

```text
模型管理
→ 知识库配置
→ 智能体开发
→ 发布智能体
→ 开始问答测试
```

也就是说，顺序不能乱：

1. 先确保模型可用。
2. 再让知识库能成功入库。
3. 再创建智能体。
4. 再给智能体绑定知识库检索工具。
5. 最后保存、发布、测试。

本次最后也是按照这个顺序完成的。

### 2.2 本次实际走过的完整流程

本次实际流程是：

```text
Mac 安装 Docker Desktop
→ 克隆 Nexent 仓库
→ 进入 docker 目录
→ 复制 .env.example 为 .env
→ 执行 deploy.sh
→ 打开 http://localhost:3000
→ 配置硅基流动 API Key
→ 添加大语言模型 Qwen3.5-9B
→ 添加向量模型 bge-m3
→ 创建知识库
→ 遇到入库失败
→ 查日志
→ 查 PostgreSQL
→ 修复 knowledge_record_t.embedding_model_id
→ 删除失败文件并重新上传
→ 成功入库“数字信号处理”知识库
→ 创建智能体
→ 遇到“生成智能体”模型不可用
→ 改为手动填写智能体
→ 绑定 knowledge_base_search 工具
→ 发布后开始问答无响应
→ 查 runtime 日志发现 index_names 为空
→ 重新配置工具 index_names 并重新发布
→ 开始问答成功
→ 导出 JSON
→ GitHub Discussions 提交
```

---

## 3. Mac 上安装 Docker Desktop

### 3.1 检查芯片类型

Mac 有两类：

```text
Apple Silicon：M1 / M2 / M3 / M4
Intel：老款 Intel Mac
```

查看方式：

```text
左上角苹果图标
→ 关于本机
→ 芯片 / 处理器
```

下载 Docker Desktop 时要选择对应版本：

```text
Apple Silicon Mac：下载 Apple Silicon 版本
Intel Mac：下载 Intel 版本
```

### 3.2 安装 Docker Desktop

操作步骤：

1. 打开 Docker 官网。
2. 下载 Docker Desktop for Mac。
3. 双击 `Docker.dmg`。
4. 把 Docker 拖到 Applications。
5. 打开 Docker Desktop。
6. 第一次启动时接受协议。
7. 等 Docker Desktop 右上角状态变为 Running。

### 3.3 验证 Docker 是否可用

打开终端，执行：

```bash
docker --version
docker compose version
```

本次实际检查结果类似：

```text
Docker version 29.5.0
Docker Compose version v5.1.3
```

只要能显示版本，就说明 Docker 命令可用。

### 3.4 Mac 上常见 Docker 注意事项

1. **必须先打开 Docker Desktop**  
   只在终端里敲 Docker 命令不够，Docker Desktop 后台要运行。

2. **第一次启动可能很慢**  
   Docker Desktop、Elasticsearch、PostgreSQL 等服务启动都需要时间。

3. **内存建议至少 8GB，最好 16GB**  
   Nexent 会启动多个容器，8GB 以下容易卡。

4. **Mac 睡眠后容器可能停止或状态异常**  
   如果页面打不开，先看 Docker Desktop 是否还在运行。

---

## 4. 克隆 Nexent 仓库与部署目录

### 4.1 创建本地目录

本次操作目录为：

```text
/Users/<your_username>/Desktop/nexent
```

实际 Nexent 仓库目录为：

```text
/Users/<your_username>/Desktop/nexent/nexent
```

Docker 部署目录为：

```text
/Users/<your_username>/Desktop/nexent/nexent/docker
```

数据目录为：

```text
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data
```

### 4.2 官方部署命令

官方基础流程是：

```bash
git clone https://github.com/ModelEngine-Group/nexent.git
cd nexent/docker
cp .env.example .env
bash deploy.sh
```

本次曾遇到过：

```text
一开始 /Users/<your_username>/Desktop/nexent 是空目录，不是 Git 仓库。
第一次 git clone 因沙箱权限无法写入 .git 失败。
第二次授权后因为无法连接 GitHub 超时失败。
随后用户手动完成 clone。
```

实际仓库路径变为：

```text
/Users/<your_username>/Desktop/nexent/nexent
```

### 4.3 进入 Docker 部署目录

```bash
cd /Users/<your_username>/Desktop/nexent/nexent/docker
```

确认该目录包含：

```text
deploy.sh
.env.example
docker-compose.yml
docker-compose.dev.yml
docker-compose.prod.yml
docker-compose-supabase.yml
docker-compose-supabase.prod.yml
```

### 4.4 复制环境配置文件

```bash
cp .env.example .env
```

生成：

```text
/Users/<your_username>/Desktop/nexent/nexent/docker/.env
```

注意：

```text
.env 文件不能上传 GitHub。
.env 文件可能包含服务配置、数据库配置、模型密钥等敏感信息。
```

---

## 5. 执行 Nexent Docker 部署

### 5.1 本次使用的部署命令

本次使用的是开发模式 + speed 轻量版：

```bash
bash deploy.sh \
  --mode development \
  --version speed \
  --is-mainland N \
  --enable-terminal N \
  --enable-skills Y \
  --root-dir /Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data
```

参数含义：

| 参数 | 本次选择 | 含义 |
|---|---|---|
| `--mode` | `development` | 开发模式，暴露多个端口，方便调试 |
| `--version` | `speed` | 轻量版，适合个人使用 |
| `--is-mainland` | `N` | 不使用大陆镜像优化源 |
| `--enable-terminal` | `N` | 不启用终端工具 |
| `--enable-skills` | `Y` | 启用内置 skills |
| `--root-dir` | 本地 `nexent-data` | 指定数据持久化目录 |

### 5.2 部署生成的配置记录

部署脚本生成：

```text
/Users/<your_username>/Desktop/nexent/nexent/docker/deploy.options
```

记录内容包括：

```text
APP_VERSION="v2.1.1"
ROOT_DIR="/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data"
MODE_CHOICE="development"
VERSION_CHOICE="speed"
IS_MAINLAND="N"
ENABLE_SKILLS="Y"
ENABLE_TERMINAL="N"
```

### 5.3 创建的数据目录

部署创建 / 使用以下目录：

```text
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data/elasticsearch
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data/postgresql
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data/minio
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data/redis
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data/volumes
/Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data/scripts
```

另外还创建了：

```text
/Users/<your_username>/nexent
```

这个目录当前大小为 0B，但属于部署产生内容。

### 5.4 启动的 Docker 容器

部署成功后，运行容器包括：

```text
nexent-web
nexent-config
nexent-runtime
nexent-mcp
nexent-northbound
nexent-data-process
nexent-minio
nexent-postgresql
nexent-redis
nexent-elasticsearch
```

对应作用：

| 容器 | 作用 |
|---|---|
| `nexent-web` | 前端 Web 页面，访问入口 |
| `nexent-config` | 配置服务，管理模型、Agent、系统配置 |
| `nexent-runtime` | 智能体运行时 |
| `nexent-mcp` | MCP 工具服务 |
| `nexent-northbound` | 北向接口服务 |
| `nexent-data-process` | 文件解析、切片、向量化、入库 |
| `nexent-minio` | 对象存储 |
| `nexent-postgresql` | 关系型数据库 |
| `nexent-redis` | 缓存与队列 |
| `nexent-elasticsearch` | 搜索与向量索引服务 |

### 5.5 创建的 Docker 网络

```text
nexent_nexent
```

### 5.6 Docker 卷情况

本次检查：

```bash
docker volume ls --filter name=nexent
```

结果：

```text
没有发现 Nexent 命名 Docker volume。
数据主要落在本机 nexent-data 目录中。
```

### 5.7 暴露端口

开发模式下主要端口：

```text
3000  -> nexent-web
5010  -> nexent-config
5011  -> nexent-mcp
5012  -> nexent-data-process
5013  -> nexent-northbound
5014  -> nexent-runtime
5015  -> MCP management
5434  -> PostgreSQL
6379  -> Redis
8265  -> Ray dashboard
9010  -> MinIO API
9011  -> MinIO Console
9210  -> Elasticsearch HTTP
9310  -> Elasticsearch transport
5555  -> Flower
```

### 5.8 访问验证

部署脚本输出：

```text
Deployment completed successfully
You can now access the application at http://localhost:3000
```

浏览器打开：

```text
http://localhost:3000
```

即可访问 Nexent 页面。

曾验证：

```text
http://127.0.0.1:3000 返回 307，跳转到 /zh
nexent-web 容器内请求 /zh 返回 Next.js HTML
nexent-config 容器内请求 http://127.0.0.1:5010/docs 返回 200
```

---

## 6. 下次开机后如何启动 Nexent

如果 Mac 关机 / 重启后需要继续使用：

### 6.1 先启动 Docker Desktop

打开：

```text
Applications → Docker
```

等 Docker Desktop 状态变为 Running。

### 6.2 启动 Nexent 容器

```bash
docker start \
  nexent-elasticsearch \
  nexent-postgresql \
  nexent-minio \
  nexent-redis \
  nexent-web \
  nexent-config \
  nexent-runtime \
  nexent-mcp \
  nexent-northbound \
  nexent-data-process
```

### 6.3 等待 1 到 2 分钟

因为 Elasticsearch、PostgreSQL、后端服务需要启动时间。

### 6.4 查看状态

```bash
docker ps --filter name=nexent
```

### 6.5 打开页面

```text
http://localhost:3000
```

---

## 7. Docker 中 image、container、nexent-data 的关系

### 7.1 image

```text
image = 程序模板 / 安装包
```

例如：

```text
nexent/nexent-web:v2.1.1
nexent/nexent:v2.1.1
postgres:15-alpine
redis:alpine
```

### 7.2 container

```text
container = 从 image 创建出来的运行实例
```

例如：

```text
nexent-web
nexent-config
nexent-postgresql
nexent-redis
```

### 7.3 nexent-data

```text
nexent-data = 持久化数据目录
```

里面保存：

```text
PostgreSQL 数据
Elasticsearch 索引
MinIO 文件
Redis 数据
脚本和其他持久化内容
```

### 7.4 简单理解

```text
image -> 创建 container -> container 运行服务
nexent-data -> 保存你的数据
```

如果只是下次继续使用：

```text
不要删 image
不要删 nexent-data
只需要启动 containers
```

---

## 8. 获取硅基流动 API Key

### 8.1 API Key 是什么

API Key 是模型服务商给你的调用凭证。Nexent 需要它来调用大语言模型和向量模型。

它不是登录密码，但同样不能泄露。

### 8.2 获取步骤

1. 打开硅基流动平台。
2. 登录账号。
3. 进入控制台。
4. 找到 API 密钥 / API Keys。
5. 点击新建 API Key。
6. 名称可写：

```text
nexent-final-exam-agent
```

7. 创建后复制 `sk-...` 格式密钥。
8. 粘贴到 Nexent 模型管理页面。

### 8.3 安全要求

不要公开：

```text
API Key
sk- 开头的密钥
Bearer Token
模型服务商控制台截图
.env 文件
```

如果泄露，立即去模型服务商后台删除 / 重置密钥。

---

## 9. 模型管理配置

进入：

```text
左侧菜单 → 模型管理
```

---

### 9.1 大语言模型配置

最终可用配置：

```text
模型类型：大语言模型
模型名称：Qwen/Qwen3.5-9B 或页面显示为 Qwen3.5-9B
展示名称：Qwen3.5-9B
模型URL：https://api.siliconflow.cn/v1
最大Token数：4096
状态：绿色可用
```

本次页面中最终显示：

```text
Qwen3.5-9B
连通性验证：可用
```

如果不可用，检查：

```text
API Key 是否完整
模型名称是否真实存在
URL 是否为 https://api.siliconflow.cn/v1
账号是否有额度
是否需要实名认证
```

---

### 9.2 向量模型配置

最终可用配置以 Nexent 页面“连通性验证可用”和数据库实际记录为准。本次最终可用记录为：

```text
模型类型：向量模型
多模态：关闭
模型名称：bge-m3（部分平台或官方模型广场可能显示为 BAAI/bge-m3）
展示名称：bge-m3
模型URL：https://api.siliconflow.cn/v1/embeddings
文档切片大小：默认
单次请求切片量：10
状态：绿色可用
```

注意：如果你的硅基流动模型广场显示完整模型 ID 为 `BAAI/bge-m3`，应优先按平台显示的真实模型 ID 填写；如果验证失败，再以 Nexent 后台 `model_record_t` 中可用的 `model_name` 为准。

### 9.3 向量模型常见错误

#### 错误 1：把 bge-m3 当大语言模型

错误做法：

```text
模型类型：大语言模型
模型名称：BAAI/bge-m3
```

正确做法：

```text
模型类型：向量模型
模型名称：bge-m3 或页面验证可用的 embedding 模型名
```

#### 错误 2：以为没有多模态向量模型会导致入库失败

结论：

```text
不是。
上传 PPT / PDF / md / docx 这类文本资料，普通向量模型即可。
多模态向量模型主要用于图像 / 图文检索。
```

#### 错误 3：模型绿色就以为知识库一定能入库

结论：

```text
不一定。
模型绿色只是说明模型记录可用。
知识库还必须绑定到正确的 embedding_model_id。
```

---

## 10. 创建知识库

进入：

```text
左侧菜单 → 知识库
```

创建知识库：

```text
知识库名称：数字信号处理
```

上传资料：

```text
课程简介.pptx
总复习.pptx
数字信号处理学习疑问解答.pptx
```

最终状态：

```text
数字信号处理
3 文档
25 分块
bge-m3模型
文件状态：已就绪
```

---

## 11. 第一次重大困难：知识库入库失败

### 11.1 现象

上传文件后状态显示：

```text
入库失败
FORWARD_FAILED
PROCESS_FAILED
```

包括：

```text
13.pdf
13.md
Operating System Concepts_9th_ (1).pdf
```

页面显示：

```text
0 文档
0 分块
```

点失败原因显示：

```text
暂无错误原因
```

### 11.2 初步排查方向

曾怀疑：

```text
PDF 是扫描版
PDF 不能解析
md 文件格式不对
需要多模态向量模型
向量模型 URL 错
Elasticsearch 挂了
```

但后来发现：

```text
md 文档也失败
```

因此排除“单纯 PDF 格式问题”。

### 11.3 查看容器状态

```bash
docker ps --filter name=nexent
```

确认关键容器：

```text
nexent-data-process Up
nexent-config Up
nexent-elasticsearch healthy
nexent-postgresql Up
nexent-redis healthy
```

### 11.4 查看日志

```bash
docker logs --tail=200 nexent-data-process
docker logs --tail=200 nexent-config
docker logs --tail=200 nexent-elasticsearch
```

关键错误：

```text
Unexpected error when indexing documents: ElasticSearch service returned HTTP 500
```

进一步根因：

```text
Embedding API error: 'NoneType' object has no attribute 'get_embeddings'
```

### 11.5 错误含义

流程已经走到：

```text
文件上传
→ 文件解析
→ 文档切片
→ 准备生成 embedding
```

但实际：

```text
embedding_model = None
```

所以调用：

```text
get_embeddings
```

时失败。

### 11.6 查询模型表

```bash
docker exec nexent-postgresql psql -U root -d nexent -c "
select model_id, model_name, display_name, model_type, connect_status
from nexent.model_record_t
order by model_id;
"
```

确认存在可用模型：

```text
model_id = 4 或 5
model_name = bge-m3
display_name = bge-m3
model_type = embedding
connect_status = available
```

### 11.7 查询知识库表

```bash
docker exec nexent-postgresql psql -U root -d nexent -c "
select knowledge_id, knowledge_name, index_name, embedding_model_id, embedding_model_name
from nexent.knowledge_record_t
order by knowledge_id;
"
```

发现：

```text
embedding_model_id 为空
embedding_model_name 为空
```

### 11.8 根因

根因是：

```text
知识库记录没有绑定向量模型 ID。
```

具体来说：

```text
模型管理里有 bge-m3
但 knowledge_record_t 中没有 embedding_model_id
因此入库时拿不到 embedding 模型对象
最终 get_embeddings 报错
```

### 11.9 修复办法

把知识库绑定到当前可用的 bge-m3。

实际修复逻辑：

```sql
update nexent.knowledge_record_t
set embedding_model_id = 当前可用 bge-m3 的 model_id,
    embedding_model_name = 'bge-m3',
    update_time = CURRENT_TIMESTAMP;
```

最终绑定到：

```text
model_id = 5
embedding_model_name = bge-m3
```

### 11.10 重启关键容器

```bash
docker restart nexent-data-process
docker restart nexent-config
```

### 11.11 删除旧失败文件并重新上传

重要经验：

```text
旧的 FORWARD_FAILED / PROCESS_FAILED 文件不会自动重新入库。
```

所以必须：

```text
删除旧失败文件
重新上传 test.md
重新上传正式课程资料
```

最终成功：

```text
数字信号处理
3 文档
25 分块
文件状态：已就绪
```

---

## 12. 创建数字信号处理期末复习智能体

进入：

```text
左侧菜单 → 智能体开发
```

创建智能体：

```text
智能体名称：数字信号处理期末复习智能体
智能体变量名：dsp_final_review_agent
作者：Default User
大语言模型：Qwen3.5-9B
最大步骤数：5
```

---

## 13. 第二次重大困难：点击“生成智能体”报模型不可用

### 13.1 现象

在智能体开发页面点击：

```text
生成智能体
```

提示：

```text
模型不可用，请检查模型状态后重试
```

### 13.2 排查

回到模型管理：

```text
Qwen3.5-9B 连通性验证：可用
```

说明模型本身可用。

### 13.3 处理方式

不再依赖“生成智能体”。

改为手动填写：

```text
智能体名称
智能体变量名
智能体描述
使用要求
大语言模型
```

经验：

```text
生成智能体只是辅助功能，不是必需流程。
手动填写也可以完整创建智能体。
```

---

## 14. 智能体使用要求 Prompt

最终使用要求大致如下：

```text
你是一个面向大学生“数字信号处理”课程期末复习的智能助教。

你的主要任务是基于“数字信号处理”知识库中的课程简介、总复习资料和学习疑问解答资料，帮助学生进行期末复习。

回答数字信号处理相关问题时，必须优先检索“数字信号处理”知识库。

如果知识库中有相关内容，要优先依据知识库资料回答。

如果知识库中没有检索到明确依据，要说明“资料中未检索到明确依据”，不要胡编。

当用户问某个知识点时，按照以下结构回答：
一、通俗解释
二、考试表述
三、关键公式
四、典型例题
五、易错点

当用户要求复习某一章或某一部分时，按照以下结构回答：
一、知识框架
二、核心概念
三、重点难点
四、常考题型
五、复习建议

当用户问题目时，要先判断考查知识点，再给出详细解题步骤，最后总结解题套路。

当用户要求制定复习计划时，要根据剩余天数、资料内容和复习目标，生成按天划分的复习安排。

当用户要求考前冲刺时，要输出高频考点、必背公式、典型题型、易错点和最后一天复习建议。

回答风格要像耐心的大学助教，适合基础一般的学生理解。默认使用清晰的小标题和分点说明。
```

---

## 15. 配置 knowledge_base_search 工具

### 15.1 选择工具

在智能体开发页面中选择：

```text
knowledge_base_search
```

### 15.2 点击齿轮配置

配置参数：

```text
top_k: 3
index_names: 数字信号处理
search_mode: hybrid
rerank: 关闭
```

如果 index_names 显示为索引：

```text
3-63d9fa655219435fac407a20afe28301
```

也可以，因为它对应“数字信号处理”知识库。

### 15.3 保存顺序

必须按这个顺序：

```text
保存工具配置
→ 保存智能体
→ 发布智能体
```

不要只保存工具不保存智能体。

---

## 16. 第三次重大困难：开始问答没有反应

### 16.1 现象

进入：

```text
开始问答
```

选择：

```text
数字信号处理期末复习智能体
```

输入：

```text
请根据知识库，生成一份数字信号处理期末复习总提纲。
```

页面没有正常回复。

### 16.2 查看日志

查看 `nexent-runtime` 日志。

关键错误：

```text
Embedding model is required for knowledge_base_search but index_names is empty
```

### 16.3 根因

发布版本里的工具参数为空：

```json
{
  "top_k": null,
  "index_names": null,
  "search_mode": null,
  "rerank": null,
  "rerank_model_name": null
}
```

也就是说：

```text
智能体启用了 knowledge_base_search
但发布版本没有保存知识库绑定
```

### 16.4 解决办法

回到智能体开发：

```text
打开数字信号处理期末复习智能体
→ 点击 knowledge_base_search 齿轮
→ 选择 index_names = 数字信号处理
→ top_k = 3
→ search_mode = hybrid
→ rerank 关闭
→ 保存工具
→ 保存智能体
→ 重新发布
```

### 16.5 结果

重新发布后，开始问答成功。

智能体能够生成：

```text
数字信号处理期末复习总提纲
```

---

## 17. 发布与开始问答测试

### 17.1 发布

在智能体开发页面：

```text
保存
→ 发布
```

版本说明可写：

```text
v1.0：数字信号处理期末复习智能体初版，支持基于知识库的知识点整理、题目讲解、复习计划和考前冲刺。
```

如果是修复工具绑定后的版本：

```text
v1.1：修复 knowledge_base_search 知识库绑定。
```

### 17.2 开始问答测试

进入：

```text
开始问答
```

选择：

```text
数字信号处理期末复习智能体
```

测试：

```text
请根据知识库，生成一份数字信号处理期末复习总提纲。
```

成功后显示：

```text
课程概述
知识框架
重点难点
常考内容
复习建议
```

---

## 18. 导出智能体 JSON

导出文件：

```text
agent_dsp_final_review_agent_1779708591498.json
```

建议改名：

```text
dsp_final_review_agent_<your_student_id>.json
```

### 18.1 JSON 中包含的内容

```text
agent_id
agent_info
智能体名称
智能体显示名
智能体描述
业务描述
作者
max_steps
工具列表
knowledge_base_search 参数
model_id
model_name
```

关键内容：

```text
name: dsp_final_review_agent
display_name: 数字信号处理期末复习智能体
model_name: Qwen3.5-9B
tool: knowledge_base_search
top_k: 3
index_names: 3-63d9fa655219435fac407a20afe28301
search_mode: hybrid
```

### 18.2 JSON 安全检查

检查后未发现：

```text
API Key
sk-
Bearer Token
password
secret
.env
课程原始资料
数据库文件
```

因此可以上传到 GitHub Discussion。

---

## 19. GitHub Discussions 提交

### 19.1 提交地址

```text
https://github.com/ModelEngine-Group/nexent/discussions
```

### 19.2 分类

选择：

```text
Show and tell
```

### 19.3 标题

```text
[Agent Submission] <your_university> + <your_student_id> - 数字信号处理期末复习智能体
```

### 19.4 正文模板

```markdown
## Agent Name

数字信号处理期末复习智能体  
Digital Signal Processing Final Review Agent

## Author

- University: <your_university>
- Student ID: <your_student_id>
- GitHub: @<your_github_username>

## Overview

This is a Nexent-based final review agent for the Digital Signal Processing course. It helps students review uploaded course materials, summarize key concepts, explain difficult topics, generate practice questions, and create exam preparation plans.

中文说明：这是一个基于 Nexent 构建的“数字信号处理”期末复习智能体。它可以结合课程资料知识库，帮助完成知识点梳理、重点难点总结、题目讲解、复习计划制定和考前冲刺复习。

## Key Features

- Course knowledge base retrieval
- Chapter-level key point summaries
- Explanation of DSP concepts and formulas
- Practice question generation
- Step-by-step problem solving
- Final exam review planning
- Self-test and review mode

## Knowledge Base

The agent uses a local Nexent knowledge base named `数字信号处理`.

For privacy and copyright reasons, the original course files are not included in this public submission.

## Tools Used

- `knowledge_base_search`
- Text/document analysis capability from Nexent

## Demo Questions

```text
帮我总结数字信号处理的期末复习重点。
```

```text
请解释离散时间傅里叶变换和 Z 变换的区别。
```

```text
根据知识库内容，给我出 5 道数字信号处理期末复习题。
```

```text
我还有 3 天考试，请帮我制定数字信号处理复习计划。
```

## Screenshots

Please add screenshots here:

1. Nexent agent configuration page
2. Knowledge base page showing the `数字信号处理` knowledge base
3. Start Chat page showing the agent answering a review question

## Deployment

- Platform: Nexent v2.1.1
- Deployment: Local Docker deployment on macOS
- Demo access: Local demo only
- Local URL: `http://localhost:3000`

## Attached Files

Agent configuration JSON is attached. Sensitive data such as API keys, tokens, `.env` files, database files, and private course materials are not included.

## Notes

No API keys, tokens, `.env` files, database files, or private course materials are included in this submission.
```

### 19.5 上传附件

可以上传：

```text
dsp_final_review_agent_<your_student_id>.json
知识库页面截图
智能体开发页面截图
开始问答成功截图
```

不要上传：

```text
API Key
Token
.env
docker/nexent-data
PostgreSQL 数据
Elasticsearch 数据
课程原始 PPT/PDF
模型管理密钥截图
终端敏感信息截图
```

### 19.6 提交后

提交后保存链接，格式类似：

```text
https://github.com/ModelEngine-Group/nexent/discussions/<discussion_id>
```

---

## 20. 重复 Discussion 的处理

本次提交过程中出现过重复 Discussion：

```text
一个标题较规范：[Agent Submission] <your_university> + <your_student_id> - 数字信号处理期末复习智能体
另一个标题较简单：<your_university>+<your_student_id>-数字信号处理期末复习智能体
```

### 20.1 是否能删除

打开 Discussion 后，点击 `...` 菜单，只看到：

```text
Copy link
Copy Markdown
Quote reply
Reference in new issue
Edit
```

没有：

```text
Delete discussion
```

结论：

```text
虽然是发帖人，但普通提交者不一定拥有删除仓库 Discussion 的权限。
```

### 20.2 处理方式

如果没有删除权限，可以：

1. 保留正确版本。
2. 把错误版本编辑为 Deprecated。
3. 或评论请求维护者删除。

作废说明可写：

```markdown
> This discussion was submitted by mistake and has been superseded by the corrected submission.
>
> 此 Discussion 为误提交版本，已作废。请以新的提交版本为准。
```

请求维护者删除可写：

```markdown
Hi maintainers, I submitted this discussion by mistake and have created a corrected version. Could you please help delete this duplicate discussion? Thank you.
```

---

## 21. 清理和删除部署

如果需要彻底删除本次部署，谨慎执行。

### 21.1 停止并删除容器

```bash
docker stop \
  nexent-web \
  nexent-config \
  nexent-runtime \
  nexent-mcp \
  nexent-northbound \
  nexent-data-process \
  nexent-minio \
  nexent-postgresql \
  nexent-redis \
  nexent-elasticsearch

docker rm \
  nexent-web \
  nexent-config \
  nexent-runtime \
  nexent-mcp \
  nexent-northbound \
  nexent-data-process \
  nexent-minio \
  nexent-postgresql \
  nexent-redis \
  nexent-elasticsearch
```

### 21.2 删除 Docker 网络

```bash
docker network rm nexent_nexent
```

### 21.3 删除数据目录

```bash
rm -rf /Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data
```

### 21.4 删除配置文件

```bash
rm -f /Users/<your_username>/Desktop/nexent/nexent/docker/.env
rm -f /Users/<your_username>/Desktop/nexent/nexent/docker/deploy.options
```

### 21.5 删除用户工作目录

```bash
rm -rf /Users/<your_username>/nexent
```

### 21.6 删除整个仓库

```bash
rm -rf /Users/<your_username>/Desktop/nexent
```

注意：

```text
删除 nexent-data 会删除 PostgreSQL、Elasticsearch、MinIO、Redis 等本地数据。
如果以后还要继续用，不要删 nexent-data。
```

---

## 22. 常用排查命令

### 22.1 查看 Nexent 容器

```bash
docker ps --filter name=nexent
```

### 22.2 查看数据处理服务日志

```bash
docker logs --tail=200 nexent-data-process
```

### 22.3 查看配置服务日志

```bash
docker logs --tail=200 nexent-config
```

### 22.4 查看运行时日志

```bash
docker logs --tail=200 nexent-runtime
```

### 22.5 查看 MCP 日志

```bash
docker logs --tail=200 nexent-mcp
```

### 22.6 查看模型表

```bash
docker exec nexent-postgresql psql -U root -d nexent -c "
select model_id, model_name, display_name, model_type, connect_status
from nexent.model_record_t
order by model_id;
"
```

### 22.7 查看知识库表

```bash
docker exec nexent-postgresql psql -U root -d nexent -c "
select knowledge_id, knowledge_name, index_name, embedding_model_id, embedding_model_name
from nexent.knowledge_record_t
order by knowledge_id;
"
```

### 22.8 重新绑定知识库到最新可用 bge-m3

> 高风险操作：该 SQL 会修改 PostgreSQL 中的知识库记录。只在确认日志出现 `embedding_model is None` 或 `get_embeddings` 相关错误、且模型表中确实存在可用 embedding 模型时执行。执行前建议先查询备份结果。

```bash
docker exec nexent-postgresql psql -U root -d nexent -c "
update nexent.knowledge_record_t
set embedding_model_id = (
    select model_id
    from nexent.model_record_t
    where model_name = 'bge-m3'
      and model_type = 'embedding'
      and connect_status = 'available'
    order by model_id desc
    limit 1
),
embedding_model_name = 'bge-m3',
update_time = CURRENT_TIMESTAMP;
"
```

### 22.9 重启关键容器

```bash
docker restart nexent-data-process
docker restart nexent-config
docker restart nexent-runtime
```

---

## 23. 一次通过版本：最短正确路线

下面是给下次使用的精简路线。

### 23.1 Mac 环境准备

```text
安装 Docker Desktop
启动 Docker Desktop
确认 docker --version 和 docker compose version 正常
```

### 23.2 部署 Nexent

下面写法假设你先创建了一个父目录 `~/Desktop/nexent`，仓库会被 clone 到 `~/Desktop/nexent/nexent`：

```bash
mkdir -p /Users/<your_username>/Desktop/nexent
cd /Users/<your_username>/Desktop/nexent
git clone https://github.com/ModelEngine-Group/nexent.git
cd nexent/docker
cp .env.example .env
bash deploy.sh \
  --mode development \
  --version speed \
  --is-mainland N \
  --enable-terminal N \
  --enable-skills Y \
  --root-dir /Users/<your_username>/Desktop/nexent/nexent/docker/nexent-data
```

打开：

```text
http://localhost:3000
```

### 23.3 配置模型

大语言模型：

```text
模型类型：大语言模型
模型名称：Qwen/Qwen3.5-9B
展示名称：Qwen3.5-9B
模型URL：https://api.siliconflow.cn/v1
API Key：自己的硅基流动 API Key
```

向量模型：

```text
模型类型：向量模型
多模态：关闭
模型名称：bge-m3
展示名称：bge-m3
模型URL：https://api.siliconflow.cn/v1/embeddings
API Key：自己的硅基流动 API Key
```

### 23.4 创建知识库

```text
知识库名称：数字信号处理
上传课程资料
等待全部文件已就绪
```

如果入库失败，检查：

```text
knowledge_record_t.embedding_model_id 是否为空
```

### 23.5 创建智能体

```text
智能体名称：数字信号处理期末复习智能体
变量名：dsp_final_review_agent
大语言模型：Qwen3.5-9B
工具：knowledge_base_search
绑定知识库：数字信号处理
top_k：3
search_mode：hybrid
rerank：关闭
保存
发布
```

### 23.6 测试

```text
开始问答
选择：数字信号处理期末复习智能体
提问：请根据知识库，生成一份数字信号处理期末复习总提纲。
```

---

## 24. 本次所有困难与解决办法总表

| 阶段 | 问题 | 现象 | 根因 | 解决办法 |
|---|---|---|---|---|
| Git clone | clone 失败 | 权限或网络错误 | 沙箱权限 / GitHub 连接超时 | 手动 clone 仓库 |
| Docker 命令 | `docker compose ps` 报错 | no configuration file provided | 当前目录不是 docker 目录 | `cd /Users/<your_username>/Desktop/nexent/nexent/docker` |
| 模型配置 | bge-m3 不可用 | 验证失败 | 模型类型、名称、URL 混乱 | 确认类型为向量模型，最终使用可用 bge-m3 |
| 模型理解 | 怀疑缺少多模态向量模型 | md/pdf 入库失败 | 错误判断 | 文本入库不需要多模态向量模型 |
| 知识库入库 | 文件入库失败 | FORWARD_FAILED / PROCESS_FAILED | 知识库没有 embedding_model_id | 绑定到可用 bge-m3 model_id |
| 失败任务 | 修复后旧文件仍失败 | 状态仍为失败 | 旧任务不会自动重跑 | 删除失败文件，重新上传 |
| 智能体生成 | 模型不可用 | 生成智能体失败 | 自动生成接口不稳定 | 手动创建智能体 |
| 工具配置 | 开始问答没反应 | 发送后卡住 | 发布版本 index_names 为空 | 重新绑定知识库并重新发布 |
| 提交 | 不知道是否上传本地部署 | 担心要上传 Docker/数据库 | 提交只需展示 | 只上传正文、截图、JSON |
| JSON | 担心泄露密钥 | 不确定能否上传 | 需要检查内容 | 确认无 API Key / token 后上传 |
| Discussion | 想删重复帖子 | 无 Delete discussion | 无仓库删除权限 | 编辑为 Deprecated 或请求维护者删除 |

---

## 25. 自审检查记录

本文档生成后，已按以下标准自审：

### 25.1 macOS 适配性

已明确标注：

```text
适用系统：macOS
使用 Docker Desktop
路径采用 /Users/<your_username>/...
下次开机需先启动 Docker Desktop
```

通过。

### 25.2 官方流程覆盖

已覆盖：

```text
git clone
cd nexent/docker
cp .env.example .env
bash deploy.sh
http://localhost:3000
模型管理
知识库配置
智能体开发
发布
开始问答
```

通过。

### 25.3 本次聊天记录覆盖

已覆盖本次对话中的主要事件：

```text
部署 Nexent
模型配置
API Key 获取
向量模型问题
知识库入库失败
PostgreSQL 修复 embedding_model_id
旧失败文件重新上传
智能体手动创建
knowledge_base_search 绑定
index_names empty 问题
开始问答成功
GitHub Discussions 提交
JSON 安全检查
重复 Discussion 删除权限问题
```

通过。

### 25.4 安全性

已明确禁止上传：

```text
API Key
Token
.env
docker/nexent-data
数据库文件
Elasticsearch 数据
课程原始 PPT/PDF
模型密钥截图
```

通过。

### 25.5 可复现性

已提供：

```text
部署命令
容器启动命令
日志查看命令
数据库查询命令
知识库修复 SQL
智能体配置参数
GitHub 提交正文模板
```

通过。

### 25.6 一次通过程度

已整理“最短正确路线”，避免再次踩：

```text
不要把 bge-m3 当 LLM
不要认为必须配置多模态向量模型
不要只看模型绿色
知识库必须绑定 embedding_model_id
knowledge_base_search 必须绑定 index_names
工具保存后还要保存智能体并重新发布
```

通过。

---

## 26. 本次逻辑与隐私审查结论

### 26.1 逻辑审查结论

整体流程是闭环的：macOS 环境准备 → Docker 部署 → 模型接入 → 知识库入库 → 智能体创建 → 工具绑定 → 发布测试 → GitHub 提交 → 后续重启与清理。核心故障链路也完整：

```text
入库失败：knowledge_record_t.embedding_model_id 为空
问答无响应：发布版本中的 knowledge_base_search.index_names 为空
```

需要特别注意的逻辑点：

1. `生成智能体` 是辅助功能，失败时可手动创建智能体。
2. `knowledge_base_search` 需要保存工具配置、保存智能体、重新发布三步都完成。
3. 数据库修复 SQL 不是常规步骤，只应作为故障排查后的补救措施。
4. 向量模型名称可能因平台展示和 Nexent 存储不同而有差异，应以“连通性验证可用”和数据库实际可用记录共同确认。
5. 删除部署内容属于高风险操作，执行 `rm -rf` 前必须确认路径。

### 26.2 隐私审查结论

本脱敏版已经将以下内容替换为占位符：

```text
Mac 本机用户名
GitHub 用户名
学校名称
学号
Discussion 编号
本地绝对路径中的个人用户名
```

文档没有包含以下高风险敏感信息：

```text
API Key
sk- 开头密钥
Bearer Token
.env 文件内容
数据库密码
课程原始 PPT/PDF 内容
Docker 数据目录压缩包
PostgreSQL / Elasticsearch 数据文件
```

仍需注意：如果你要把文档公开发布，建议继续保持占位符；如果只是交课程作业，可以按老师要求恢复学校和学号。

---

## 26. 最终一句话总结

本次在 macOS 上通过 Docker Desktop 本地部署 Nexent v2.1.1，接入 Qwen3.5-9B 和 bge-m3，修复知识库未绑定 `embedding_model_id` 导致的入库失败问题，构建并发布“数字信号处理期末复习智能体”，最终通过 `knowledge_base_search` 成功检索课程知识库并完成开始问答测试，同时完成 GitHub Discussions 提交与 JSON 安全检查。
