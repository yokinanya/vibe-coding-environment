这个仓库的配置仅供参考，可以按自己需要改，主要用于 Codex 和 Github Copilot

### Github Copilot Prompts 配置

- 通用指令模板：[.github/user--globallobal.md](.github/user--globallobal.md)
- 使用方式：将模板内容复制到你自己的 VS Code prompts 指令文件中
- 用户级路径示例：`~/.copilot/instructions/*.instructions.md`

### Codex 配置

- Codex 全局规则文件：[.codex/AGENTS.md](.codex/AGENTS.md)
- 作用：定义 Codex 代理在本仓库中的全局行为约束（如技能扫描、Python 任务使用 `uv`、`ruff` 用于代码检查）

## Skills 配置
这些skills单纯是按我习惯加的，你可以自己去 [skills.sh](https://skills.sh/) 上找你自己需要的
```bash
npx skills add https://github.com/HSSkyBoy/apk-reverse --global
npx skills add https://github.com/yhongm/material-design-skill --global
npx skills add https://github.com/softaworks/agent-toolkit --global --skill mermaid-diagrams
npx skills add https://github.com/trailofbits/skills --global --skill modern-python

# Web 设计
npx skills add https://github.com/vercel-labs/agent-skills --global --skill web-design-guidelines
# Next.js
npx skills add vercel/next.js --global --skill next-cache-components-optimizer
npx skills add vercel/next.js --global --skill next-dev-loop
# React / Next.js 性能优化
npx skills add vercel-labs/agent-skills --global --skill react-best-practices
# shadcn/ui
npx skills add shadcn/ui --global --skill shadcn
# 动效设计
npx skills add mblode/agent-skills --global --skill ui-animation
```