这个仓库的配置仅供参考，可以按自己需要改，主要用于 Codex 和 Github Copilot

### Github Copilot Prompts 配置

- 通用指令模板：[.github/user--globallobal.md](.github/user--globallobal.md)
- 结束前提问规则模板：[.github/ask-before-end.md](.github/ask-before-end.md) （可选，用于GitHub Copilot 节省用量）
- 使用方式：将模板内容复制到你自己的 VS Code prompts 指令文件中
- 用户级路径示例：`~/.copilot/instructions/*.instructions.md`

### Codex 配置

- Codex 全局规则文件：[.codex/AGENTS.md](.codex/AGENTS.md)
- 作用：定义 Codex 代理在本仓库中的全局行为约束（如技能扫描、Python 任务使用 `uv`、`ruff` 用于代码检查）

## Skills 配置
这些skills单纯是按我习惯加的，你可以自己去 [skills.sh](https://skills.sh/) 上找你自己需要的
```bash
npx skills add https://github.com/vercel-labs/agent-eval --global  --skill frontend-design
npx skills add https://github.com/softaworks/agent-toolkit --global --skill mermaid-diagrams
npx skills add https://github.com/vercel-labs/next-skills --global --skill next-best-practices
npx skills add https://github.com/laurigates/claude-plugins --global --skill ruff-linting
npx skills add https://github.com/wshobson/agents --global --skill uv-package-manager
npx skills add https://github.com/vercel-labs/agent-skills --global --skill vercel-react-best-practices
npx skills add https://github.com/vercel-labs/agent-eval --global --skill web-design--globaluidelines
npx skills add https://github.com/wshobson/agents --global --skill tailwind-design-system
```