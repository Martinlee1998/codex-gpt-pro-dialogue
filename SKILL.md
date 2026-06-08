---
name: codex-gpt-pro-dialogue
description: Use when the user wants Codex to operate their logged-in Chrome ChatGPT/GPT Pro conversation as an English-first research-advice channel: connect, send the user's custom prompt, wait, monitor, extract, and summarize the answer while preserving strict project claim boundaries.
---

# Codex GPT Pro Dialogue

Use this skill when the user asks Codex to operate Chrome/ChatGPT/GPT Pro as an external research-advice channel. The skill is about the operational chain between Codex and GPT Pro, not about fixed prompt templates.

## Core Use Cases

- Continue an existing ChatGPT/GPT Pro conversation in Chrome.
- Send the user's custom research prompt to GPT Pro, using English by default when Codex drafts the wording.
- Ask GPT Pro for project critique, literature review, research-gap analysis, route selection, or new idea generation.
- Wait through long GPT Pro reasoning/search runs without interrupting them.
- Extract and summarize the completed answer while keeping the original tab available.

Do not use this skill for normal web browsing, generic ChatGPT prompting, or local implementation work.

## Operating Principle

The user owns the prompt. Do not force a canned template.

Default language is English. When Codex drafts a prompt, writes status notes into GPT Pro, or summarizes GPT Pro's answer, use English unless the user explicitly requests another language. If the user supplies non-English wording, preserve the user's wording instead of translating it silently.

Codex should handle the chain:

1. Gather and compress local context when needed.
2. Connect to the user's logged-in Chrome session.
3. Claim the correct ChatGPT tab.
4. Send the prompt the user wants, with only minimal safety or clarity additions when appropriate.
5. Wait and monitor until GPT Pro completes.
6. Extract the result and summarize it for the user.

When the user gives only a high-level instruction, Codex may draft a one-off prompt for that exact request. Keep it task-specific and do not rely on reusable templates.

## Required Setup

Use the Chrome plugin skill (`chrome:control-chrome`) and follow its bootstrap exactly. Prefer an existing ChatGPT tab when the user is continuing a conversation.

Before sending any prompt:

1. Claim the current ChatGPT tab from `browser.user.openTabs()`.
2. Check the page is idle: no active `stop-button`, no unfinished `Pro thinking`, and no `organizing answer`, unless the user explicitly wants to wait.
3. If the previous answer is still running, wait unless the user explicitly asks to stop or replace it.
4. Do not reload a ChatGPT conversation tab unless the user requests it; reloading can lose UI state.

## Prompt Handling

Do not maintain or inject a fixed prompt template. If the user supplies wording, use it directly unless it would transmit sensitive data.

If Codex needs to formulate the prompt from the user's intent:

- Make it specific to this turn.
- Write it in English unless the user explicitly requests another language.
- Include only compact project context needed for GPT Pro to answer.
- Preserve the user's requested stance, such as "must web search", "wait until finished", "research gap only", or "do not evaluate yet".
- Include explicit claim boundaries when the project has mixed evidence.
- Avoid generic boilerplate and avoid overlong repo dumps.

## Data Safety

Only send information the user has authorized for GPT Pro:

- Project summaries, metrics, status ledgers, claim boundaries, and sanitized prompts are okay.
- Do not upload or paste secrets, API keys, private credentials, full logs with secrets, browser history, local memory files, or unrelated personal data.
- Avoid dumping whole repos. Prefer compact evidence bundles and direct questions.

Treat GPT Pro output as external advice, not verified fact. If it cites literature or current facts and accuracy matters, preserve the source list and note uncertainty if sources are weak.

## Waiting Policy

Respect the user's waiting preference.

- If the user says to wait indefinitely or not stop, do not press `stop-button`.
- Poll the page periodically and report short status updates.
- Typical states:
  - `Pro thinking`: still thinking/searching.
  - `organizing answer`: search/thinking likely done, answer being assembled.
  - `completed Xm Ys ago`: answer completed.
  - `continue generating`: click it and continue waiting.
  - `stop responding`: active generation is still in progress.

If GPT Pro searches unrelated sites, do not interrupt unless the user told you to actively correct it. Let it finish, then summarize any source-quality issue.

## Extracting the Answer

Prefer DOM extraction of the latest assistant turn when copy buttons are ambiguous.

Checklist:

1. Identify the latest `conversation-turn-*`.
2. Confirm it is an assistant answer, not the user's prompt.
3. If using a copy button, verify the clipboard starts with GPT Pro's answer, not the prompt.
4. If copy returns the wrong text, extract the latest assistant turn text from the DOM.
5. Preserve the ChatGPT tab with `browser.tabs.finalize({ keep: [{ tab, status: "deliverable" }] })`.

## Summarizing to User

Final user summary should be in English unless the user requests otherwise.

Include:

- ChatGPT conversation link.
- GPT Pro's final verdict or central conclusion.
- Ranked ideas or routes, if requested.
- Pass gates and stop gates, if relevant.
- Most dangerous adjacent work / research-gap risks, if relevant.
- A clear distinction between GPT Pro's opinion and Codex's own synthesis when both are present.

Keep the summary compact. Do not paste a full long GPT Pro response unless the user asks for it.

## Research Claim Discipline

When discussing research projects, preserve these distinctions:

- End-to-end benchmark/task success.
- Learned policy improvement.
- LLM/semantic causal contribution.
- Verifier/controller/readout/system success.
- Offline oracle or smoke/frontier evidence.
- Negative result with mechanism insight.

Never promote smoke, offline labels, expected actions, controller cleanup, or postprocessing into full benchmark success.

## Typical Conversation Chain

When the user is using GPT Pro as a research adviser, a common sequence is:

1. Ask for a route or idea judgment.
2. Ask for literature review.
3. Ask specifically for research gaps and dangerous adjacent work.
4. Ask separately whether the project is worth doing.
5. Ask for other ideas and backup routes.
6. Summarize the final ranking and next execution priorities.

Do not merge steps 3 and 4 if the user wants an independent project judgment after the research-gap review.

## Finalization

At the end of Chrome work:

1. Keep the ChatGPT tab as a deliverable unless the user asks to close it.
2. Release browser control with `browser.tabs.finalize({ keep: [{ tab, status: "deliverable" }] })`.
3. In the final response, include the conversation link and a concise status/result summary.
