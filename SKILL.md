---
name: codex-gpt-pro-dialogue
description: Use when the user wants Codex to operate their logged-in ChatGPT/GPT Pro conversation as a research-advice channel: connect, optionally upload a sanitized evidence bundle, send the user's custom prompt, wait without interrupting active generation, monitor, extract, and summarize the answer while preserving strict project claim boundaries.
---

# Codex GPT Pro Dialogue

Use this skill when the user asks Codex to operate Chrome/ChatGPT/GPT Pro as an external research-advice channel. The skill is about the operational chain between Codex and GPT Pro, not about fixed prompt templates.

## Core Use Cases

- Continue an existing ChatGPT/GPT Pro conversation in Chrome.
- Send the user's custom research prompt to GPT Pro, using English by default when Codex drafts the wording.
- Upload a sanitized evidence bundle for GPT Pro to review when the user asks for file-backed analysis.
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
4. Upload an evidence bundle if the user requested file-backed review.
5. Send the prompt the user wants, with only minimal safety or clarity additions when appropriate.
6. Wait and monitor until GPT Pro completes.
7. Extract the result and summarize it for the user.

When the user gives only a high-level instruction, Codex may draft a one-off prompt for that exact request. Keep it task-specific and do not rely on reusable templates.

## Required Setup

Use the Chrome plugin skill (`chrome:control-chrome`) and follow its bootstrap exactly. Prefer an existing ChatGPT tab when the user is continuing a conversation.

Before sending any prompt:

1. Claim the current ChatGPT tab from `browser.user.openTabs()`.
2. Check the page is idle: no active `stop-button`, no unfinished `Pro thinking`, and no `organizing answer`, unless the user explicitly wants to wait.
3. If the previous answer is still running, wait unless the user explicitly asks to stop or replace it.
4. Do not reload a ChatGPT conversation tab unless the user requests it; reloading can lose UI state.

## Evidence Bundle Uploads

Use this section when the user asks Codex to upload files or a compact evidence bundle for GPT Pro analysis.

Do not hard-code local paths, usernames, project names, browser window titles, run IDs, model IDs, operating-system language, or one machine's directory layout. Discover them from the current task and use placeholders in reusable notes, such as `<ABSOLUTE_BUNDLE_PATH>`, `<CHAT_URL>`, `<VISIBLE_FILE_DIALOG_TITLE>`, and `<PROJECT_LEDGER_PATH>`.

Before uploading:

1. Build a compact bundle from an explicit whitelist, not the full project tree.
2. Include only the current blocker, status/theory or claim-boundary docs, key summaries/metrics, relevant scripts, and reproduction notes needed for the review.
3. Compute and record the absolute bundle path valid on the current machine, entry count, byte size, and SHA256.
4. Scan for likely secrets, API keys, passwords, credentials, suspicious filenames, and unrelated personal data.
5. Write a short prompt telling GPT Pro which files to read, what repeated suggestions to avoid, the exact question to answer, and the claim boundaries to preserve.

Browser upload flow:

1. Open or reclaim the target ChatGPT/GPT Pro tab.
2. Open the composer attachment menu.
3. Try the visible add-file item or the shortcut shown by the current UI.
4. If browser automation exposes a `filechooser` event, try setting `<ABSOLUTE_BUNDLE_PATH>` through automation.
5. If `fileChooser.setFiles(...)` fails with `Not allowed`, treat this as an extension/browser permission boundary, not a path error.
6. If the operating-system file picker is visible to the user but not to browser automation, use GUI automation only as a fallback:
   - copy `<ABSOLUTE_BUNDLE_PATH>` to the system clipboard;
   - activate the visible file picker using the current OS/window manager;
   - focus the filename/path field using the current OS convention;
   - paste the absolute path and confirm;
   - return to ChatGPT and confirm the attachment chip or filename appears.

Platform examples are templates only:

- Windows: PowerShell can use `System.Windows.Forms.Clipboard`, `WScript.Shell.AppActivate(<VISIBLE_FILE_DIALOG_TITLE>)`, and SendKeys. Replace all placeholders; do not assume English dialog titles or a fixed shortcut.
- macOS: copy the absolute path, activate the file picker, use the OS path-entry convention such as `Cmd+Shift+G` when available, paste, and confirm.
- Linux desktop: copy the absolute path, activate the file picker, use the dialog's path-entry shortcut such as `Ctrl+L` when available, paste, and confirm.

Before sending:

1. Confirm the composer shows the uploaded filename.
2. Confirm the send button is enabled.
3. Ask GPT Pro to explicitly state whether it can read the attachment and whether prior advice changes after reading it.
4. Ask GPT Pro to identify missing files rather than pretending to have read them.

After the response:

1. Save a concise response summary in the current workspace only if the task calls for durable records.
2. If the project has a status ledger, update it only when the user's current instructions allow it.
3. Do not write upload workflow notes or skill-maintenance notes into a project status ledger unless the user explicitly requests that.

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

- Once GPT Pro has started generating a response, do not press `stop-button`, stop responding, cancel, reload, or otherwise terminate generation unless the user explicitly reverses this rule in the current conversation.
- If the user says to wait indefinitely or not stop, treat that as a strict restriction.
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
