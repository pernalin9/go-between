# 传话筒 · go-between

让 AI 发挥出本来的水平：在你和执行代理之间加一个传话筒。它先弄清你要什么，删掉会把 AI 带偏的猜测，再把干净的任务交出去。

*A go-between for AI agents: it clarifies what you want, strips guesses that would anchor the model, and hands a clean prompt to a fresh sub-agent.*

## 它解决什么问题

> 我想把这张概念图做成头巾，Recraft Vector 可以做到吗？你觉得用什么效果最好？

提到 Recraft Vector，只是因为听说矢量图质量高。它其实完全不适合这张图，但 AI 仍然回答"Recraft Vector 可以参与，但……"，然后围绕它做了大量工作，最后产出无法使用的矢量纹样。

AI 并非不知道它不合适。问题在于 prompt 里提到的东西会锚定模型，叫它"忽略"也没用：在 [System 2 Attention](https://arxiv.org/abs/2311.11829) 的实验中，带有用户意见时准确率是 62.8%，加上"请忽略意见"是 71.7%，把意见直接删掉才回到 80.3%。最可靠的办法是让执行者根本看不到它。

传话筒会：

- 判断你提到的工具、方法、原因是**要求**还是**猜测**。猜测删掉，换成你真正想要的效果。
- 只问你才知道、而且会改变做法的问题，并给出默认答案，回一个"行"就能继续。
- 去掉专家人设、"请一步步思考"、套模板等无效内容，也不替你添加要求。
- 把整理好的 prompt 交给**不继承对话历史**的执行代理，再把结果原样带回来。
- 多数请求原样转交，不为了显得有用而改写。

## 使用

### 安装

适用于 Codex、Claude Code、Cursor、Gemini CLI、GitHub Copilot 等支持 [Agent Skills](https://agentskills.io) 的工具：

```bash
npx skills add pernalin9/go-between
```

或使用 GitHub CLI（2.90 及以上）：

```bash
gh skill install pernalin9/go-between go-between --pin v1.0.0
```

安装后在对话中启用：Codex 输入 `$go-between`，Claude Code 输入 `/go-between`。之后正常提需求即可，说"退出传话筒"恢复。

### 不安装，直接试用

在 Codex、Claude Code 这类能执行命令的工具里说：

```text
用 curl 下载 https://raw.githubusercontent.com/pernalin9/go-between/v1.0.0/skills/go-between/SKILL.md ，读完整原文，本次会话按它工作。
```

用 curl 而不是网页抓取工具，是因为有些抓取工具会先把页面交给另一个模型总结，拿到的就不是原文了。

### 聊天应用

在 ChatGPT、Claude 网页版等不能启动子代理的地方，把 [SKILL.md](skills/go-between/SKILL.md) 的内容作为第一条消息发送。传话筒会把整理好的 prompt 交给你，由你粘贴到一个新对话里。新对话没有之前的历史，效果相同。

## 注意

- **确认执行代理不继承对话历史。** 部分版本的 Codex 子代理默认会继承完整对话，执行者就会看到你的原话，过滤失效。启用后，可以让传话筒派一个执行代理，问它"你能看到之前的对话吗"。
- **固定版本。** 建议安装带版本号的发布（如 `v1.0.0`），而不是始终跟随 main 分支。任何 skill 都以 agent 的完整权限运行，安装前请先阅读 SKILL.md。

## 依据

- 先删掉意见再提问，效果明显好于叫模型"忽略意见"；原文和删改版同时给出也会变差：[System 2 Attention](https://arxiv.org/abs/2311.11829)（Weston & Sukhbaatar, 2023）
- 用户说出自己猜的答案，会把模型准确率拉低最多 27%：[Towards Understanding Sycophancy](https://arxiv.org/abs/2310.13548)（Sharma et al., ICLR 2024）
- 思维链、"忽略锚点"等提示都不能消除锚定效应：[Anchoring Bias in LLMs](https://arxiv.org/abs/2412.06593)（Lou et al., 2024）
- 叫模型"不要提 X"，并不能可靠地让它少提 X：[Suppressing Pink Elephants](https://arxiv.org/abs/2402.07896)（Castricato et al., 2024）
- 无关信息会明显降低推理表现：[GSM-IC](https://arxiv.org/abs/2302.00093)（Shi et al., ICML 2023）
- 信息分多轮给出时，平均表现下降 39%，早期的错误假设难以纠正：[LLMs Get Lost In Multi-Turn Conversation](https://arxiv.org/abs/2505.06120)（Laban et al., 2025）
- 漏写关键要求会降低表现，但把要求全写上同样会降低：[What Prompts Don't Say](https://arxiv.org/abs/2505.13360)（Yang et al., 2025）
- 专家人设不提高准确性：[Prompting Science Report 4](https://arxiv.org/abs/2512.05858)（Wharton, 2025）
- 对推理模型，"一步步思考"几乎没有收益，却增加 20%–80% 的耗时：[Prompting Science Report 2](https://arxiv.org/abs/2506.07142)（Wharton, 2025）
- 冲突的指令会让推理模型浪费推理去调和：[GPT-5 Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide)（OpenAI）
- 先自己弄清能弄清的，只问用户答得上来的问题：[Ambig-SWE](https://arxiv.org/abs/2502.13069)（Vijayvargiya et al., ICLR 2026）
- 委派任务要写清目标和边界，投入按任务难度分配：[How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)（Anthropic, 2025）
- 解释要求背后的原因、长材料放在任务前面：[Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)（Anthropic）