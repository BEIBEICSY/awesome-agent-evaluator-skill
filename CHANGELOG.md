# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-04-30

### Added
- Initial release of Agent Evaluator Skill
- Seven milestone evaluation framework (感知层、大脑层、协议层、行动层、记忆层、工程化、安全性)
- SOTA benchmarking system with mandatory WebSearch validation
- Reference documentation for major Agent frameworks:
  - Anthropic Claude (MCP architecture)
  - OpenAI (Function Calling architecture)
  - LangGraph (StateGraph architecture)
  - Manus (autonomous execution + sandbox)
  - Hermes (self-evolving memory)
  - OpenClaw (open source architecture)
  - Trae (local agent framework)
  - CodeBuddy (coding agent)
  - Qoder (production agent)
- Evaluation standards documentation with design principles
- Three sample evaluation reports:
  - Simple ReAct Agent (1/7 milestones)
  - Multi-Agent CrewAI system (0/7 milestones)
  - Production MCP Agent (7/7 milestones)
- README.md with quick start guide
- MIT License
- Contributing guidelines (planned for next version)

### Core Features
- **SOTA 对标**: 强制搜索最新信息,拒绝过时知识
- **七大里程碑**: 基于 Agent = LLM + System + Environment 的本质定义
- **生产级验证**: 参考经过大规模验证的方案(Anthropic、OpenAI、LangGraph)
- **第一性原理**: 每个评价都有设计原理支撑,拒绝"魔法 Prompt"

### Design Principles
- 每次评价必须对标当前年份的行业最高水平(2026 SOTA)
- 搜索年份动态调整(根据当前日期)
- 主流大厂优先(Anthropic、OpenAI、LangGraph)
- 强制先检索(在评价前必须执行 WebSearch)
- reference/ 与 WebSearch 边界明确(静态缓存 vs 最新事实)

### Documentation Structure
```
agent-evaluator/
├── README.md                    # 项目核心文档
├── LICENSE                      # MIT 开源许可证
├── CHANGELOG.md                 # 版本历史
├── SKILL.md                     # Skill 定义文件
├── reference/                   # SOTA 参考文档
│   ├── evaluation-standards.md  # 七大里程碑详解
│   ├── anthropic.md             # Anthropic MCP 架构
│   ├── openai.md                # OpenAI Function Calling
│   ├── langgraph.md             # LangGraph StateGraph
│   ├── manus.md                 # Manus 自主执行
│   ├── hermes.md                # Hermes 自进化记忆
│   ├── openclaw.md              # OpenClaw 开源架构
│   ├── trae.md                  # Trae 本地框架
│   ├── codebuddy.md             # CodeBuddy 编程 Agent
│   └── qoder.md                 # Qoder 生产 Agent
└── examples/                    # 示例文档
    └── sample-reports/          # 评估报告示例
        ├── simple-react-agent-report.md
        ├── multi-agent-report.md
        └── production-mcp-agent-report.md
```

### Target Users
- Agent 开发者: 需要深度理解架构差距,改进自己的项目
- Agent 学习者: 需要通过评估学习最佳实践
- 技术决策者: 需要评估团队采用的 Agent 方案是否达标

### Known Limitations
- reference/ 文档需要定期更新(超过 3 个月必须用 WebSearch 验证)
- 评估报告依赖 WebSearch 的实时信息,网络不稳定时可能影响准确性
- 目前仅支持英文和中文双语

### Future Plans
- [ ] CONTRIBUTING.md - 贡献指南
- [ ] docs/evaluation-methodology.md - 评估方法论深度解释
- [ ] docs/sota-benchmarks.md - SOTA 基准选择逻辑
- [ ] .github/ISSUE_TEMPLATE/ - GitHub Issue 模板
- [ ] 更多示例报告(不同类型的 Agent 项目)
- [ ] 自动化评测脚本(批量评估多个项目)

---

## Version History Summary

| Version | Date | Key Changes |
|:---|:---|:---|
| 1.0.0 | 2026-04-30 | Initial release with seven milestone framework and SOTA benchmarking |

---

## Upgrade Guide

### From 0.x to 1.0.0

This is the initial release, no upgrade needed.

### Future Versions

When upgrading to future versions:
1. Check CHANGELOG.md for breaking changes
2. Update reference/ documents if last_updated > 3 months
3. Re-run WebSearch to validate SOTA standards
4. Review evaluation reports with new milestone criteria

---

## Contributing to Changelog

When adding new entries:
1. Follow [Keep a Changelog](https://keepachangelog.com/) format
2. Use semantic versioning for version numbers
3. Group changes by type: Added, Changed, Deprecated, Removed, Fixed, Security
4. Include date in ISO format (YYYY-MM-DD)
5. Reference related issues/PRs when applicable

---

## References

- [Keep a Changelog](https://keepachangelog.com/)
- [Semantic Versioning](https://semver.org/)
- [GitHub Changelog Best Practices](https://github.com/rhythmus/README.md-spec)