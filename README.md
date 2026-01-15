# AF - 基于 OpenAgents 的金融风险事件点评项目

专注于微观风险信号（如逾期新闻）的金融视角点评，基于行业标准和专家经验对风险信号的影响进行推演和研判，自动化“可证伪假设提出“”尽调取证“”证据链梳理”等流程，从而降低研判成本、提升推演准确率。适配 [OpenAgents](https://github.com/openagents-org/openagents) 框架。

- 目标用户：股民/风控分析师/信用审批人/权益分析师/财经记者/学者
  - 使用场景：
    530. 早期预警系统（EWS）报警泛洪下的快速分诊（triage）与调查研判（investigation）
    531. 关注新闻、公告、传闻等“弱信号”对关切问题（如股价、债权安全性、公司表现）的潜在影响
    532. 监管要求从业者提供可追溯、可复核、可审计的研判记录
    533. 基于直觉判断客体存在问题，但找不到客观具体的证据

## 功能特性

- 🤖 **引入“证伪主义” (Falsificationism)**：针对 LLM 容易产生的“证实偏差”（即倾向于寻找支持性证据），引入卡尔·波普尔的哲学思想。系统鼓励生成具备可观测性、可操作性的风险假设和取证问题。其中，取证环节聚焦对推翻原假设有关键作用的问题。
- 📊 **黑板机制**：黑板允许智能体异步领取任务信息和存储结果。黑板机制使得智能体的子任务分包成为可能。
- 🌐 **OpenAgents 网络**：支持多代理松耦合协作，理论上可以接入任意数量的外部通用智能体作为“外包调查员”
- 🔄 **基于专家经验的上下文约束**：引入基于历史案例的事件模型库缓解RAG噪声。系统不直接进行广域搜索，而是先定位到相似的历史案例，在案例的“思维框架”内进行定向取证。

## 快速开始

### 1. 安装依赖

```bash
# 创建虚拟环境（推荐）
conda create -n AF python=3.12
conda activate AF

# 安装依赖
pip install -r requirements.txt

# 如果使用 Gemini，还需安装：
pip install google-generativeai
```

### 2. 配置环境变量

```bash
# 数据库配置 （可选）
export DAILY_DATABASE="your_database"
export DAILY_USER="your_user"
export DAILY_PASSWORD="your_password"
export DAILY_HOST="your_host"
export DAILY_PORT="5432"

# LLM API 密钥（选择其中一种方式）

# 方式一：OpenAgents 标准
export DEFAULT_LLM_API_KEY="your_api_key"

# 方式二：Gemini 特定（推荐）
export GEMINI_API_KEY="your_api_key"
# 或者
export GOOGLE_API_KEY="your_api_key"

# 方式三：OpenAI 特定
export OPENAI_API_KEY="your_api_key"

# 配置 LLM 提供商和模型
export OPENAGENTS_LLM_PROVIDER="gemini"  # 可选: openai, gemini, claude, deepseek 等
export OPENAGENTS_LLM_MODEL="gemini-2.5-flash"

# 兼容旧版：支持以 Y 或 MILITAI 开头的环境变量
export Y1="your_api_key_1"
```

### 3. 启动 OpenAgents 网络

```bash
# 初始化并启动网络
openagents network start ./network
```

### 4. 访问 OpenAgents Studio

在浏览器中打开 http://localhost:8050 即可与分析代理交互。

或者使用独立的 Studio：
```bash
openagents studio -s
```
### 5. 启动分析代理

**使用 Python 代理（完整功能）**
```bash
python ./network/agents/analysis_agent.py
```

## 项目结构

```
AF/
├── original_script.py                    # 独立脚本（使用 OpenAgents 全局 API）
├── requirements.txt           # Python 依赖
├── .env.example            # 环境变量
├── README.md                  # 项目文档
├── LICENSE                    # 许可证
└── network/                   # OpenAgents 网络配置
    ├── network.yaml           # 网络配置文件
    └── agents/
        └── analysis_agent.py  # Python 分析代理（完整功能）
```

## 使用方式

### 原脚本模式（独立脚本不依赖OpenAgents框架）

直接运行独立脚本输入信息进行分析：

```bash
python original_script.py
```

### OpenAgents 模式

启动网络和代理后，可以通过以下方式与分析代理交互：

1. **OpenAgents Studio**：通过 Web 界面发送消息

2. **编程方式**：使用 OpenAgents 客户端连接到网络

```python
from openagents.core.client import AgentClient

client = AgentClient()
client.connect(host="localhost", port=8700)
# 发送消息给分析代理
```

## 分析流程

1. **输入解析**：LLM 分析用户输入，抽取要素
2. **数据查询**：根据提取的信息查询数据库获取财务数据和事件模型
3. **综合分析**：LLM 基于上下文进行分析
4. **结果输出**：提出假设和调查问题
5. **结果存储**：将分析结果保存到数据库表 "I"（黑板）
6. 

## 分析报告格式

```
摘要：[简要总结]

关键要点：
  • [要点1]
  • [要点2]
  ...

风险评估：[风险评估内容]

建议：
  • [建议1]
  • [建议2]
  ...

置信度：[0.0-1.0]
```

## 支持的 LLM 提供商

通过 OpenAgents 全局 API，本项目支持以下 LLM 提供商：

| 提供商 | 环境变量 | 示例模型 |
|--------|----------|----------|
| OpenAI | `OPENAI_API_KEY` | gpt-4, gpt-3.5-turbo |
| Google Gemini | `GEMINI_API_KEY` 或 `GOOGLE_API_KEY` | gemini-2.5-flash, gemini-pro |
| Anthropic Claude | `ANTHROPIC_API_KEY` | claude-3-opus, claude-3-sonnet |
| DeepSeek | `DEEPSEEK_API_KEY` | deepseek-chat |
| Azure OpenAI | `AZURE_OPENAI_API_KEY` | gpt-4 (Azure) |
| 更多... | 参见 OpenAgents 文档 | - |

## 技术栈

- **OpenAgents**：AI 代理网络框架（包含统一的 LLM 提供商 API）
- **PostgreSQL**：关系型数据库
- **Python 3.10+**：编程语言


