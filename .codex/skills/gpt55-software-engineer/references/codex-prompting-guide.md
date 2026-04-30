# Codex Prompting Guide Notes

Source: https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide
Published: 2026-02-25
Target adaptation: GPT-5.5 software engineering skill.

## Core Takeaways

- Start from Codex-style autonomy: gather context, implement, test, refine, finish with outcome.
- Use `medium` reasoning as general interactive default; reserve `high`/`xhigh` for hardest tasks.
- Avoid instructions that force upfront plans/status messages for every task; they can delay useful action.
- Most important prompt areas: autonomy/persistence, repo exploration, tool use, frontend quality.
- Prefer `rg`/`rg --files` for search; batch independent file reads/searches.
- Preserve user changes in dirty worktrees; never revert unrelated edits.
- Use `apply_patch` for manual edits; model is optimized for this diff style.
- Keep shell tool simple: command string plus `workdir`; request escalation only when needed.
- Tool response truncation should preserve beginning and end when output is too large.
- Preambles should be short, human, outcome-oriented, and not log-like.
- Pragmatic personality: terse, direct, high action density; useful when user values throughput.

## Skill Adaptation

This skill is not an API harness prompt. It is a reusable Codex behavior guide for software engineering turns. Keep it short enough to load often, then read this reference only when exact source rationale matters.

For GPT-5.5, preserve the guide's coding-agent defaults while avoiding version-specific API claims unless verified against current official docs.
