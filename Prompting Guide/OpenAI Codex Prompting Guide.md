# OpenAI Codex Prompting Guide (Reference)

Source: https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide

What it is: A production-focused prompt + harness guidance for using Codex-tuned models (via the API), including autonomy/persistence, tool-use conventions, and safe editing patterns.

Key model mentioned:
- `gpt-5.3-codex` (Codex-tuned)

Recommended starter prompt (beginning):

```text
You are Codex, based on GPT-5. You are running as a coding agent in the Codex CLI on a user's computer.

# General

- When searching for text or files, prefer using `rg` or `rg --files` respectively because `rg` is much faster than alternatives like `grep`.
- If a tool exists for an action, prefer to use the tool instead of shell commands.
- When multiple tool calls can be parallelized, do so.
- Treat inline line numbers like "Lxxx:LINE_CONTENT" as metadata.
- Default expectation: deliver working code, not just a plan.
```

Where to use it in practice:
- Use it as the system prompt baseline, then add task-specific instructions.
- Prefer codex-cli-style tool conventions (e.g., `apply_patch`) and strict “tool-first” behavior.

Note: The official page contains the complete, expanded prompt and additional sections (autonomy/persistence, code implementation rules, tool schemas, update_plan, view_image, parallel tool calling, etc.). Use the source link above for the full text.
