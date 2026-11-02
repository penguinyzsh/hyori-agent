# Hyori

极简本地多 Agent 框架，分为表层基础运行环境（Omote）与后台认知编排层（Ura）。

## Language

**Hyori**:
整个 Agent 系统的总称，包含 Omote 和 Ura 两部分。

**Omote**:
基础运行与交互层，负责对接聊天平台（Web、Telegram 等）、运行基础 Agent 循环，以及提供插件运行的沙盒环境。
_Avoid_: Frontend, Bot Adapter

**Ura**:
后台多 Agent 编排层，负责分析用户真实意图、维护 Agent 独立性格、执行后台自主任务（如主动搜索、编写插件）并管理记忆。
_Avoid_: Backend, Router

**Channel Action（通道动作）**:
聊天平台原生的交互动作（由 Omote 直接提供），包括 Telegram 的已读状态、正在输入提示（typing）、消息表情回应（reaction）、贴纸（sticker）收发和分段发消息。
_Avoid_: Body Reflex, 身体反射

**Intent Analysis（意图分析）**:
不直接按字面意思处理用户输入，而是结合上下文、历史记录和关系状态，判断反话、试探或真实情绪的过程。
_Avoid_: Subtext Decoding, 潜台词解码, Sentiment Analysis

**Sandbox（沙盒）**:
基于 `@anthropic-ai/sandbox-runtime` 的本地轻量级隔离环境。Ura 编写的插件在沙盒中直接运行，无需人工审批；若运行出错或损坏，仅重置沙盒目录，不影响 Omote 主进程。
_Avoid_: Blast-Radius Sandbox, 可爆破沙盒, Docker Container
