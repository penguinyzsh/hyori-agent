# Domain Docs（领域文档规范）

定义各工程技能在探查代码库时如何读取与维护本仓库的领域文档。

## 探查代码前优先阅读

- 仓库根目录下的 **`GLOSSARY.md`**，或
- 若根目录存在 **`GLOSSARY-MAP.md`**：它会指向各限界上下文（Context）对应的 `GLOSSARY.md`，请阅读与当前任务相关的词汇表。
- **`docs/adr/`**：阅读与当前修改区域相关的架构决策记录（ADR）。在多上下文仓库中，还需检查 `src/<context>/docs/adr/` 下特定上下文的决策。

若上述文件尚不存在，**静默继续执行任务**，不要报错，也不要主动提议提前创建。`/domain-modeling` 技能（由 `/grill-with-docs` 或 `/improve-codebase-architecture` 触发）会在术语或架构决策真正落定时按需惰性创建它们。

## 目录结构

单上下文仓库（Single-context，当前默认采用）：

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-xxx.md
│   └── 0002-xxx.md
└── src/
```

多上下文仓库（Multi-context，以根目录存在 `GLOSSARY-MAP.md` 为标志）：

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← 全局架构决策
└── packages/
    ├── omote/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← Omote 内部决策
    └── ura/
        ├── GLOSSARY.md
        └── docs/adr/
```

## 严格遵循词汇表（Glossary）术语

当你在输出中命名领域概念（无论是议题标题、重构提案、假设还是测试名称）时，必须使用 `GLOSSARY.md` 中定义的标准术语，不得自行漂移到词汇表明确排除的近义词。

若所需的领域概念尚未收录在词汇表中，说明可能存在两种情况：要么你在生造项目不使用的概念（请重新审视），要么发现了真实的领域模型缺口（请记录下来供 `/domain-modeling` 处理）。

## 显式标注 ADR 冲突

若你的方案或输出与现有 ADR 存在冲突，必须显式指出，严禁静默覆盖：

> _与 ADR-0007（xxx）冲突，但建议重新评估，原因如下：…_
