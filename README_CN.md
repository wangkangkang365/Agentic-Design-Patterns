# 📚 智能体设计模式 - 构建智能系统的实践指南

> **Fork 自 [evoiz/Agentic-Design-Patterns](https://github.com/evoiz/Agentic-Design-Patterns)**

[![书籍](https://img.shields.io/badge/书籍-预售-blue)](https://www.amazon.com/Agentic-Design-Patterns-Hands-Intelligent/dp/3032014018/)
[![作者](https://img.shields.io/badge/作者-Antonio%20Gulli-green)](https://www.linkedin.com/in/searchguy/)
[![慈善](https://img.shields.io/badge/版税-捐赠儿童救助会-red)](https://www.savethechildren.org/)
[![许可证](https://img.shields.io/badge/许可证-教育用途-yellow)]()

## 📖 关于本仓库

本仓库 Fork 自原作者 **Antonio Gulli** 的项目，包含 **《智能体设计模式：构建智能系统的实践指南》** 的完整学习材料。包含 PDF 格式的所有章节以及配套的代码笔记本，用于实践学习。

> **注意**：所有作者版税均捐赠给 Save the Children（儿童救助会）💝

## 🎯 学习内容

本指南共 21 章和 7 个附录，全面介绍如何构建智能 AI 智能体系统，包括：

- **基础模式**：提示链、路由、并行化
- **高级技术**：反思、工具使用、规划、多智能体系统
- **记忆与学习**：记忆管理、适应、目标设定
- **生产模式**：异常处理、人在回路、RAG
- **优化**：资源感知模式、推理技术、护栏
- **实际应用**：从 GUI 到真实世界环境

## 📁 仓库结构

```
.
├── 📄 README.md                              # 英文说明文件
├── 📄 README_CN.md                           # 中文说明文件（本文件）
├── 📚 Agentic_Design_Patterns_Complete.pdf   # 完整书籍（424 页）
├── 💻 chapter_notebooks/                     # 章节代码笔记本
│   ├── chapter_01_prompt_chaining/           # 第1章：提示链
│   │   ├── code_example.ipynb
│   │   ├── json_example.ipynb
│   │   └── summary.md
│   ├── chapter_02_routing/                   # 第2章：路由
│   │   ├── google_adk.ipynb
│   │   ├── langgraph.ipynb
│   │   └── openrouter.ipynb
│   ├── chapter_03_parallelization/           # 第3章：并行化
│   ├── chapter_04_reflection/                # 第4章：反思
│   ├── chapter_05_tool_use/                  # 第5章：工具使用
│   ├── chapter_06_planning/                  # 第6章：规划
│   ├── chapter_07_multi_agent/               # 第7章：多智能体
│   ├── chapter_08_memory/                    # 第8章：记忆管理
│   ├── chapter_09_adaptation/                # 第9章：适应
│   ├── chapter_10_mcp/                       # 第10章：MCP
│   ├── chapter_11_goal_setting/              # 第11章：目标设定
│   ├── chapter_12_exception_handling/        # 第12章：异常处理
│   ├── chapter_13_human_in_the_loop/         # 第13章：人在回路
│   ├── chapter_14_knowledge_retrieval/       # 第14章：知识检索
│   ├── chapter_15_inter_agent/               # 第15章：智能体间通信
│   ├── chapter_16_resource_optimization/     # 第16章：资源优化
│   ├── chapter_17_reasoning/                 # 第17章：推理
│   ├── chapter_18_guardrails/                # 第18章：护栏
│   ├── chapter_19_evaluation/                # 第19章：评估
│   ├── chapter_20_prioritization/            # 第20章：优先级
│   ├── chapter_21_exploration/               # 第21章：探索
│   └── appendix/                             # 附录
│       ├── advanced_prompting_techniques.ipynb
│       ├── ai_agentic_gui_to_real_world.ipynb
│       ├── quick_overview_agentic_frameworks.ipynb
│       └── ...
└── 📄 demo.py                                # 演示脚本
```

## 📚 目录

### 前言与基础
- 献词
- 致谢
- 序言
- 思想领袖视角：权力与责任
- 引言
- 什么让 AI 系统成为"智能体"？

### 第一部分：核心模式（103 页）
1. **第 1 章**：提示链 - 顺序任务分解
2. **第 2 章**：路由 - 动态路径选择
3. **第 3 章**：并行化 - 并发处理
4. **第 4 章**：反思 - 自我改进机制
5. **第 5 章**：工具使用 - 外部能力集成
6. **第 6 章**：规划 - 战略任务管理
7. **第 7 章**：多智能体 - 协作系统

### 第二部分：高级模式（61 页）
8. **第 8 章**：记忆管理 - 状态持久化
9. **第 9 章**：学习与适应 - 动态改进
10. **第 10 章**：模型上下文协议（MCP）- 标准化接口
11. **第 11 章**：目标设定与监控 - 目标跟踪

### 第三部分：生产模式（34 页）
12. **第 12 章**：异常处理与恢复 - 健壮的错误管理
13. **第 13 章**：人在回路 - 人机协作
14. **第 14 章**：知识检索（RAG）- 信息访问模式

### 第四部分：企业模式（114 页）
15. **第 15 章**：智能体间通信（A2A）- 智能体网络
16. **第 16 章**：资源感知优化 - 高效资源使用
17. **第 17 章**：推理技术 - 高级决策制定
18. **第 18 章**：护栏/安全模式 - 风险缓解
19. **第 19 章**：评估与监控 - 性能跟踪
20. **第 20 章**：优先级排序 - 任务管理
21. **第 21 章**：探索与发现 - 自主学习

### 附录（74 页）
- **附录 A**：高级提示技术
- **附录 B**：AI 智能体：从 GUI 到真实世界环境
- **附录 C**：智能体框架快速概览
- **附录 D**：使用 AgentSpace 构建智能体
- **附录 E**：CLI 上的 AI 智能体
- **附录 F**：深入底层：推理引擎
- **附录 G**：编码智能体

### 结论与参考
- 结论
- 术语表
- 术语索引

## 🚀 快速开始

### 前置要求

```bash
# 需要 Python 3.8 或更高版本
python --version

# 安装 Jupyter 以运行笔记本
pip install jupyter notebook

# 安装常用依赖
pip install -r requirements.txt
```

### 安装步骤

1. **克隆仓库**
```bash
git clone https://github.com/evoiz/Agentic-Design-Patterns.git
cd Agentic-Design-Patterns
```

2. **设置虚拟环境**（推荐）
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

3. **安装依赖**
```bash
pip install jupyter notebook
pip install pandas numpy matplotlib openai langchain
```

4. **启动 Jupyter Notebook**
```bash
jupyter notebook
```

## 💻 运行代码

每个章节都包含一个带有实践示例的 Jupyter 笔记本：

1. 进入 `chapter_notebooks/` 目录
2. 打开所需的章节笔记本
3. 按照每个笔记本内的说明操作
4. 按顺序运行单元格以获得最佳学习体验

### 示例：运行第 1 章
```bash
# 进入笔记本目录
cd chapter_notebooks

# 启动特定笔记本
jupyter notebook chapter_01_prompt_chaining/code_example.ipynb
```

## 📖 如何使用本仓库

### 自学
1. 阅读 PDF 中的每个章节
2. 打开对应的笔记本
3. 运行代码示例
4. 尝试修改和实验
5. 完成练习

### 教学
1. 将章节用作讲座材料
2. 将笔记本布置为实验作业
3. 基于模式创建自定义示例
4. 使用多个模式构建项目

### 研究
1. 参考实现模式
2. 对不同方法进行基准测试
3. 扩展模式以适应新用例
4. 贡献改进

## 🤝 贡献

欢迎贡献！请参阅我们的 [贡献指南](CONTRIBUTING.md)。

### 贡献方式
- 🐛 报告错误和问题
- 💡 建议新功能或模式
- 📝 改进文档
- 🔧 提交代码改进
- 🌍 翻译材料

### 贡献流程
1. Fork 仓库
2. 创建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m '添加某个很棒的功能'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开 Pull Request

## 📚 附加资源

### 原始项目
- 📁 [原始项目仓库](https://github.com/evoiz/Agentic-Design-Patterns)
- 📖 [预购书籍](https://www.amazon.com/Agentic-Design-Patterns-Hands-Intelligent/dp/3032014018/)
- 👨‍💼 [作者 LinkedIn](https://www.linkedin.com/in/searchguy/)
- 📁 [原始 Google Drive 材料](https://drive.google.com/drive/u/0/folders/1Y3U3IrYCiJ3E45Z8okR5eCg7OPnWQtPV)

### 相关框架
- [LangChain](https://langchain.com/)
- [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)
- [OpenAI Assistants](https://platform.openai.com/assistants)
- [Microsoft AutoGen](https://github.com/microsoft/autogen)
- [CrewAI](https://www.crewai.com/)
- [Google ADK](https://github.com/google/A2A)
- [FastMCP](https://github.com/jlowin/fastmcp)

### 学习路径
1. **初学者**：从第 1-7 章开始（核心模式）
2. **中级**：继续第 8-14 章（高级与生产）
3. **高级**：掌握第 15-21 章（企业模式）
4. **专家**：探索附录中的前沿技术

## 📊 笔记本框架使用

各章节笔记本使用不同的框架：

| 框架 | 使用章节 | 导入方式 |
|------|---------|---------|
| LangChain | 第 1-3 章、第 5 章 | `langchain_openai`, `langchain_google_genai` |
| Google ADK | 第 2-4 章、第 7-8 章 | `google.adk.agents` |
| CrewAI | 第 5 章、第 7 章 | `crewai` |
| FastMCP | 第 10 章 | `fastmcp` |
| OpenAI | 部分章节 | `openai` |

## 🔑 API 密钥配置

笔记本需要 API 密钥作为环境变量：

```bash
# OpenAI/LangChain 示例
export OPENAI_API_KEY="your-api-key"

# Gemini/ADK 示例
export GOOGLE_API_KEY="your-api-key"
```

> **注意**：本仓库不包含 `.env` 文件或密钥管理。请在运行前设置密钥。

## ⚖️ 许可证

本仓库用于教育用途。请尊重作者的版权和知识产权。

- **书籍内容**：© Antonio Gulli - 保留所有权利
- **代码示例**：MIT 许可证（见 LICENSE 文件）
- **教育用途**：允许使用，需注明归属

## 🙏 致谢

- **Antonio Gulli** - 作者兼 AI 思想领袖
- **Save the Children** - 所有书籍版税的受益方
- **贡献者** - 所有帮助改进这些材料的人
- **社区** - 推进 AI 智能体的学习者和实践者

## 📞 联系与支持

- **问题反馈**：[GitHub Issues](https://github.com/evoiz/Agentic-Design-Patterns/issues)
- **讨论交流**：[GitHub Discussions](https://github.com/evoiz/Agentic-Design-Patterns/discussions)
- **作者**：[LinkedIn](https://www.linkedin.com/in/searchguy/)

## 🌟 支持项目

如果您觉得本仓库有用，请考虑给我们一个星标 ⭐

---

<p align="center">
  <strong>一次一个模式，构建 AI 的未来 🚀</strong>
</p>

<p align="center">
  为 AI 社区用心制作 ❤️
</p>
