# Issue tracker: GitHub

本仓库的议题（Issues）与规格说明（Specs）均托管在 GitHub Issues 中。所有操作统一使用 `gh` CLI。

## 操作约定

- **创建议题**：`gh issue create --title "..." --body "..."`。多行内容请使用 heredoc 传递。
- **读取议题**：`gh issue view <number> --json number,title,body,labels,comments`。
- **列出议题**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，并按需附加 `--label` 与 `--state` 过滤条件。
- **将议题设为父议题的子议题（Sub-issue）**：创建时使用 `gh issue create --parent <parent> ...`，或事后执行 `gh issue edit <parent> --add-sub-issue <child>`（需 `gh` 2.94+）。旧版 `gh` 可调用：`gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`（使用下方 **阻塞依赖** 中说明的数值型 database id）。若环境不支持原生子议题，则在子议题正文顶部标注 `Part of #<parent>`。
- **阻塞依赖（Blocking）**：优先使用 GitHub **原生议题依赖（native issue dependencies）**。可通过 `gh issue create --blocked-by <n>,<n> ...` 或 `gh issue edit <child> --add-blocked-by <blocker>`（`gh` 2.94+）添加阻塞边。旧版 `gh` 可调用：`gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`，其中 `<blocker-db-id>` 为阻塞议题的数值型 **database id**（通过 `gh api repos/<owner>/<repo>/issues/<n> --jq .id` 获取，而非 `#number` 或 `node_id`）。若原生依赖不可用，则在子议题正文顶部回退使用 `Blocked by: #<n>, #<n>`。当所有前置阻塞议题均关闭时，该工单即解除阻塞。
- **统计未关闭的阻塞项数量**：`gh api repos/<owner>/<repo>/issues/<n> --jq .issue_dependencies_summary.blocked_by`（仅统计未关闭的阻塞项；为 0 表示可开始执行）。若使用 `gh issue view <n> --json blockedBy`，需过滤掉已关闭项：`--jq '[.blockedBy.nodes[] | select(.state == "OPEN")] | length'`。
- **评论议题**：`gh issue comment <number> --body "..."`
- **添加 / 移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭议题**：先通过 `gh issue comment <number> --body "..."` 发布说明，再执行 `gh issue close <number>`。注意：对已关闭的议题直接执行 `gh issue close --comment` 会丢弃评论且返回码仍为 0。

仓库地址由 `git remote -v` 自动推断；在克隆目录内执行 `gh` 时会自动识别。

所有议题仅允许提交至用户自己的仓库，严禁提交至 `mattpocock/skills`。在派生仓库（Fork）中，`gh` 默认可能指向游标上游仓库，因此在每次会话首次执行 `gh issue create` 前，需先通过 `gh repo view --json nameWithOwner` 校验目标仓库。

## 将 Pull Requests 作为分拣入口

**PRs as a request surface: no.** _（若本仓库将外部 PR 视为功能请求入口，可将其设为 `yes`；`/triage` 会读取此开关。）_

设为 `yes` 时，PR 遵循与议题相同的标签和状态流转，并使用对应的 `gh pr` 命令：

- **读取 PR**：依次执行 `gh pr view <number>`、`gh pr view <number> --comments`（非 TTY 环境下 `--comments` 仅输出评论而不含正文）以及 `gh pr diff <number>`。
- **列出待分拣的外部 PR**：`gh api --paginate 'repos/{owner}/{repo}/pulls?state=open' --jq '.[] | select(.author_association | IN("OWNER","MEMBER","COLLABORATOR") | not) | {number, title, author: .user.login, author_association, labels: [.labels[].name]}'`。
- **评论 / 标签 / 关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 的 Issue 与 PR 共用同一编号空间，单独的 `#42` 可能是其中任意一种：可先尝试 `gh pr view 42`，失败时回退至 `gh issue view 42`。

## 当技能要求“发布到 issue tracker”时

创建一个 GitHub Issue。

## 当技能要求“获取相关工单（ticket）”时

按上方 **读取议题** 的方式读取。

## Wayfinding（探路导航）操作约定

供 `/wayfinder` 技能使用。其中 **Map（全局路线图）** 为单个父议题，各具体任务工单作为其 **Child（子议题）**。

- **Map**：带有 `wayfinder:map` 标签的单个议题，正文维护 Notes / Decisions-so-far / Fog 三部分。创建命令：`gh issue create --label wayfinder:map`。
- **Child ticket**：作为 GitHub 子议题关联到 Map（参见 **将议题设为父议题的子议题**）。若未启用原生子议题，则将其加入 Map 正文的任务列表，并在子议题正文顶部标注 `Part of #<map>`。标签格式：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。认领后将工单指派给当前开发者。
- **Blocking**：参见上方 **阻塞依赖**。
- **Frontier query（前沿可执行工单查询）**：列出 Map 下所有未关闭的子议题，剔除仍存在未关闭阻塞项或已有指派人的工单，按 Map 中的排列顺序取第一个。
- **Claim（认领）**：`gh issue edit <n> --add-assignee @me`，作为会话的首次写操作。
- **Resolve（结项）**：先执行 `gh issue comment <n> --body "<answer>"`，再执行 `gh issue close <n>`，最后将结论摘要与链接追加至 Map 的 Decisions-so-far 区域。
