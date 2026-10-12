# go-between

[中文](README.md)

Let AI work at its real level: put a go-between between you and the agent that does the work. It works out what you actually want, removes the guesses that would steer the AI wrong, and hands off a clean task.

## The problem

> I want to turn this concept art into a headscarf. Can Recraft Vector do it? What do you think would give the best result?

Recraft Vector came up only because I had heard vector art is high quality. It was completely wrong for this image, yet the AI answered "Recraft Vector can play a part, but…", built a lot of work around it, and ended up with vector patterns that could not be used.

The AI was not unaware that it was a poor fit. Whatever a prompt mentions anchors the model, and telling it to ignore that does not work: in the [System 2 Attention](https://arxiv.org/abs/2311.11829) experiments, accuracy was 62.8% with the user's opinion in the question, 71.7% after adding "ignore the opinion", and 80.3% only once the opinion was removed. The reliable fix is to make sure the executor never sees it.

## What it does

- Decides whether each tool, method, or cause you mention is a **requirement** or a **guess**. Guesses are removed and replaced with the result you actually want.
- Asks only what only you know and what would change the approach, with a default answer, so "OK" is enough to continue.
- Strips expert personas, "think step by step", templates, and other filler, and never adds requirements of its own.
- Hands the cleaned prompt to an executor agent that **does not inherit the conversation**, and brings the result back unchanged.
- Passes most requests through as they are, instead of rewriting them to look useful.

It talks to you in your language, so this edition works whatever language you use. A Chinese edition is also available (see below).

## Usage

### Codex: install in one sentence

Tell Codex:

```text
Install skills/go-between from the pernalin9/go-between repo, then read its SKILL.md and follow it for the rest of this conversation.
```

Codex installs it into `~/.codex/skills/go-between` with its built-in skill installer. Installing needs network access, so Codex will ask you to approve once. Reading SKILL.md right after installing makes it take effect in the same conversation. In later conversations, just type `$go-between`.

### Try it without installing

In Codex, Claude Code, or another agent that can run commands, say:

```text
Use curl to download https://raw.githubusercontent.com/pernalin9/go-between/main/skills/go-between/SKILL.md, read the full text, and follow it for the rest of this conversation.
```

Use curl rather than a web-fetch tool: some fetch tools pass the page through another model to summarize it, so the agent would not get the original text.

### Other agents

For Claude Code, Cursor, Gemini CLI, GitHub Copilot, and other tools that support [Agent Skills](https://agentskills.io):

```bash
npx skills add pernalin9/go-between
```

It will ask whether to install the English edition `go-between` or the Chinese edition `go-between-zh`. Or use GitHub CLI 2.90 or later:

```bash
gh skill install pernalin9/go-between go-between
```

Then activate it, for example with `/go-between` in Claude Code. Say "exit go-between" to go back to normal.

### Chat apps

In ChatGPT, Claude on the web, and other places that cannot start sub-agents, send the contents of [SKILL.md](skills/go-between/SKILL.md) as your first message. The go-between will give you the cleaned prompt to paste into a new conversation. A new conversation has none of the earlier history, so the effect is the same.

### Chinese edition

Replace `skills/go-between` with `skills/go-between-zh` and `go-between` with `go-between-zh` above. Its instructions are at [skills/go-between-zh/SKILL.md](skills/go-between-zh/SKILL.md).

## Notes

- **Make sure executors don't inherit the conversation.** Some versions of Codex start sub-agents with the full conversation by default, which shows the executor your original words and defeats the filtering. After activating, have the go-between start an executor and ask it "Can you see the earlier conversation?"
- **All of the above use the latest version on the main branch.** Every skill runs with your agent's full permissions, so read SKILL.md before installing.

## Research

- Removing an opinion before asking works much better than telling the model to ignore it, and giving the original alongside the cleaned version also hurts: [System 2 Attention](https://arxiv.org/abs/2311.11829) (Weston & Sukhbaatar, 2023)
- A user stating their own guess at the answer cut model accuracy by up to 27%: [Towards Understanding Sycophancy](https://arxiv.org/abs/2310.13548) (Sharma et al., ICLR 2024)
- Neither chain-of-thought nor "ignore the anchor" prompts removed anchoring: [Anchoring Bias in LLMs](https://arxiv.org/abs/2412.06593) (Lou et al., 2024)
- Telling a model "don't mention X" does not reliably make it mention X less: [Suppressing Pink Elephants](https://arxiv.org/abs/2402.07896) (Castricato et al., 2024)
- Irrelevant information markedly degrades reasoning: [GSM-IC](https://arxiv.org/abs/2302.00093) (Shi et al., ICML 2023)
- Spreading information over several turns lowers performance by 39% on average, and early wrong assumptions are hard to recover from: [LLMs Get Lost In Multi-Turn Conversation](https://arxiv.org/abs/2505.06120) (Laban et al., 2025)
- Leaving out key requirements hurts, but writing every requirement down hurts too: [What Prompts Don't Say](https://arxiv.org/abs/2505.13360) (Yang et al., 2025)
- Expert personas do not improve accuracy: [Prompting Science Report 4](https://arxiv.org/abs/2512.05858) (Wharton, 2025)
- For reasoning models, "think step by step" adds almost nothing but takes 20–80% longer: [Prompting Science Report 2](https://arxiv.org/abs/2506.07142) (Wharton, 2025)
- Conflicting instructions make reasoning models waste effort reconciling them: [GPT-5 Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide) (OpenAI)
- Find out what you can yourself, and ask users only what they can answer: [Ambig-SWE](https://arxiv.org/abs/2502.13069) (Vijayvargiya et al., ICLR 2026)
- Delegated tasks need a clear objective and boundaries, with effort scaled to difficulty: [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) (Anthropic, 2025)
- Explain the reasons behind instructions, and put long material before the task: [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) (Anthropic)
