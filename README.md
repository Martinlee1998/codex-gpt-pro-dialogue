# Codex GPT Pro Dialogue

`codex-gpt-pro-dialogue` is a Codex skill for using a logged-in Chrome ChatGPT/GPT Pro session as a research dialogue channel.

It handles the browser side of the handoff: find the right ChatGPT tab, send the user's prompt, wait for long GPT Pro runs, extract the latest assistant answer, keep the original conversation open, and report the result back through Codex.

## Why

For research work, I often want GPT Pro as a second opinion: route selection, literature review, research-gap checks, project critique, or backup idea generation.

The prompt changes every time, so this project deliberately does not ship prompt templates. The part worth standardizing is the Chrome-to-GPT-Pro loop:

- do not refresh away the current ChatGPT state;
- do not send a new prompt while GPT Pro is still generating;
- wait through long reasoning, search, and answer-finalization runs;
- continue generation when ChatGPT asks for it;
- extract the latest assistant answer instead of accidentally copying the user's prompt;
- keep the live GPT Pro conversation available for later inspection;
- summarize the answer without turning GPT Pro's advice into experimental evidence.

That is the whole scope: **Codex -> Chrome -> GPT Pro -> Codex summary**.

## Scope

Use this for:

- continuing an existing GPT Pro conversation from Codex;
- asking GPT Pro for literature reviews, research-gap reviews, project judgments, or new research ideas;
- waiting until GPT Pro actually finishes;
- preserving the ChatGPT tab as part of the deliverable;
- summarizing the result with clear claim boundaries.

Do not use this as:

- a fixed prompt-template library;
- a review-bundle generator;
- a replacement for the Chrome control plugin;
- a way to upload secrets, API keys, private logs, browser history, or full repository dumps;
- evidence that a local research project has succeeded.

## Difference From Review-Bundle Projects

Review-bundle projects usually package a plan into a compact markdown artifact for GPT Pro to review.

This project is about the live dialogue itself. It leaves prompt wording to the user and focuses on the operational details that are easy to get wrong: tab selection, waiting, continuation, extraction, tab preservation, and final summarization.

## Installation

Clone the repository, then copy the skill folder into your Codex skills directory.

### Windows PowerShell

```powershell
$skillName = "codex-gpt-pro-dialogue"
$repoUrl = "https://github.com/Martinlee1998/codex-gpt-pro-dialogue.git"
$tmp = Join-Path $env:TEMP $skillName
$dst = Join-Path $env:USERPROFILE ".codex\skills\$skillName"

if (Test-Path $tmp) { Remove-Item -Recurse -Force $tmp }
git clone $repoUrl $tmp

New-Item -ItemType Directory -Force -Path (Split-Path $dst) | Out-Null
if (Test-Path $dst) { Remove-Item -Recurse -Force $dst }
Copy-Item -Recurse -Path $tmp -Destination $dst
```

The installed skill should be here:

```text
%USERPROFILE%\.codex\skills\codex-gpt-pro-dialogue\SKILL.md
```

### macOS / Linux

```bash
skill_name="codex-gpt-pro-dialogue"
repo_url="https://github.com/Martinlee1998/codex-gpt-pro-dialogue.git"
tmp_dir="${TMPDIR:-/tmp}/${skill_name}"
dst_dir="${HOME}/.codex/skills/${skill_name}"

rm -rf "$tmp_dir"
git clone "$repo_url" "$tmp_dir"

mkdir -p "$(dirname "$dst_dir")"
rm -rf "$dst_dir"
cp -R "$tmp_dir" "$dst_dir"
```

The installed skill should be here:

```text
$HOME/.codex/skills/codex-gpt-pro-dialogue/SKILL.md
```

Restart or refresh Codex after installation so the skill list reloads.

## Usage

Make sure Chrome is logged in to ChatGPT/GPT Pro and the Codex Chrome plugin is available. Then ask Codex to use the skill:

```text
Use codex-gpt-pro-dialogue. Continue the current GPT Pro conversation in Chrome, send my prompt below, wait until GPT Pro finishes, then summarize the result.
```

The target ChatGPT tab can already be open, or the user can ask Codex to open a new conversation first.
