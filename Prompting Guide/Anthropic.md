# Anthropic Prompting Guide

## Prompt architecture (Anthropic Applied AI)
1. Task context: Define who the model is and the goal.
2. Tone context: Specify behavioral constraints (evidence-only, uncertainty rules, no hallucination).
3. Background data, documents, and images: Provide static invariants (form structure/meaning, Swedish terms/layout, etc.).
4. Detailed task description & rules: Provide the ordered workflow and decision rules.
5. Examples: Few-shot examples (form + sketch) paired with ideal output.
6. Conversation history: Include prior turns only if relevant; otherwise omit/keep empty.
7. Immediate task description or request: Provide the dynamic inputs for this case.
8. Thinking step by step / take a deep breath: Instruct internal step-by-step reasoning before producing final fields.
9. Output formatting: Enforce strict XML/JSON (all fields present; use "unknown" when needed).

Practical mapping: in many single-shot setups, blocks 1-5 live in the system prompt, block 6 comes from chat history (if any), and blocks 7-9 live in the current user message.

## What this guide is based on
Hannah and Christian (Anthropic Applied AI) iteratively build a robust single-shot prompt for analyzing Swedish car accident report forms (structured checkboxes) plus a hand-drawn sketch (images) to determine fault. The demo evolves from a weak baseline that hallucinates irrelevant details (e.g., a "skiing accident") into a reliable workflow by adding structure, uncertainty handling, and strict output formatting.

References: bulaev.net, youtube.com.

## Core principles
- Prompt engineering is iterative and empirical: test on real/varied cases, then refine.
- Prefer clear roles + static context in the system prompt.
- Use structured formats (especially XML tags) and logical sequencing.
- Provide few-shot examples when edge cases matter.
- Enforce strict, machine-parsable output formatting.

## Recommended architecture (API-style single-shot)
Use two parts: (1) static instruction in the system prompt, (2) dynamic data in the user message.

### System prompt: Task context / role (up front)
Define who the model is and its goal.
Example:
"You are an expert AI assistant helping a human claims adjuster reviewing car accident report forms in Swedish. Your job is to analyze the provided form and sketch to determine what happened and who is at fault."

### System prompt: Tone / style / confidence rules
Make uncertainty and evidence requirements explicit.
- Stay factual and confident only when evidence supports it.
- If the sketch/form is unclear, illegible, or missing key facts: say "I don't know" (or explicitly flag limitations). Do not hallucinate or invent details.
- Be objective: cite specific evidence (e.g., checkboxes/fields) and avoid speculation.

### System prompt: Background / static details / documents (keep invariant)
Put invariants here so they never change between runs.
- Full structure of the Swedish checkbox form (field meanings, layout, Swedish terms).
- Any rules for interpreting fields (e.g., how to decide which vehicle/event corresponds to each checkbox).
- Anything else that is constant across tasks.

This reduces tokens, improves consistency, and enables prompt caching.

### User message: Dynamic content + delimiters
Put only per-case data in the user message.
- The filled-in form values.
- The sketch description or sketch image-derived description.

Use clear delimiters like:
- <form_data>...</form_data>
- <sketch_description>...</sketch_description>

## Step-by-step reasoning (with structured outputs)
Tell the model to follow an explicit ordered process.
- Analyze the form (structured data) first.
- Then analyze the sketch.
- Then compare and decide fault.

Example phrasing (adapt as needed):
- "Carefully review each checkbox. Extract details about Vehicle A and B. Then compare the extracted form facts to the sketch. Conclude who is at fault only when the evidence matches." 

If you ask for step-by-step reasoning, also require that only the specified fields are output (do not dump internal scratch work unless you truly want it).

Explicitly state that the model should do the step-by-step work internally before writing the final structured fields.

## XML/structure: what to tag
Use XML tags to organize inputs/outputs so the model is easier to parse and hard to confuse.
Common pattern:
- <form_data>
- <sketch_description>
- <analysis>
- <final_verdict>
- <evidence>
- <uncertainty>

## Few-shot examples
Add 1+ example(s) pairing (form + sketch) with:
- ideal reasoning/output
- desired formatting
- correct handling of ambiguity

Few-shot is especially useful for teaching:
- edge cases
- the right failure mode ("I don't know" when evidence is missing)
- consistent output structure.

## Key reminders (repeat near the end)
Reinforce the rules that prevent hallucinations.
- Only base conclusions on visible evidence.
- Cite specific checkboxes/fields as justification.
- Do not invent details.
- If unsure, state limitations explicitly.

## Output format / prefilling
Enforce a strict, machine-parsable output format (XML or JSON). To reduce formatting drift:
- prefill the response with the start of the target structure (e.g., begin with <final_verdict>)
- keep the output schema small and deterministic
- make sure every field is always present (even if "unknown").

## Advanced tips
- Iteration & testing: start simple, then add elements one by one; validate on a diverse test set.
- Temperature / params: use low or zero temperature for factual consistency; allow enough max tokens for detailed structured outputs.
- Sequence matters: keep static context in the system prompt; keep dynamic data + case instructions in the user message.
- Avoid vague or overly conversational prompts. Prefer explicit, structured single-turn instructions suitable for production.

## Copyable prompt skeleton (edit placeholders)

### System prompt skeleton
```text
You are an expert AI assistant helping a human claims adjuster reviewing car accident report forms in Swedish.
Your job is to analyze a provided Swedish form (checkboxes/fields) plus a hand-drawn sketch to determine what happened and who is at fault.

MECE-style coverage: consider all relevant scenarios present in the form and sketch; do not overlap categories; do not leave gaps.

Confidence and uncertainty rules:
- Only make claims that you can support with explicit evidence from the form or sketch.
- If the sketch or any key form fields are unclear/illegible/missing, state "I don't know" and explain what is missing.
- Never invent details.
- Be objective. Cite the specific checkbox/field(s) you used.

Static form interpretation rules (invariants):
[PASTE THE FULL INVARIANT STRUCTURE OF THE 17-CHECKBOX FORM HERE, INCLUDING FIELD MEANINGS, LAYOUT, AND SWEDISH TERMS]

Output requirements (strict XML):
Return exactly one XML object with this schema:
<final_response>
  <final_verdict>...</final_verdict>
  <evidence>...</evidence>
  <uncertainty>...</uncertainty>
  <reasoning_summary>...</reasoning_summary>
</final_response>

Process (ordered):
1) Analyze <form_data> first and extract relevant facts about Vehicle A, Vehicle B, and the event.
2) Analyze <sketch_description> second.
3) Compare form facts vs sketch depiction.
4) Decide fault only when evidence aligns; otherwise mark uncertainty and/or "I don't know".
```

### User message skeleton
```text
<form_data>
[PASTE THE FILLED-IN FORM VALUES HERE, DELIMITED BY FIELD NAMES OR CHECKBOX IDS]
</form_data>

<sketch_description>
[PASTE THE SKETCH DESCRIPTION OR IMAGE-DERIVED NOTES HERE]
</sketch_description>

Task: Determine what happened and who is at fault, using only the provided evidence.
```
