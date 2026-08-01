# codex-image2free

> Current release: **v0.2.0-beta.1**

A Codex Skill that routes planned image-generation or editing jobs through the logged-in ChatGPT web app, pastes local references, retrieves candidates, and iterates with local QA.

## Install

Copy this repository folder into a Codex Skill directory, keeping `SKILL.md` at the folder root. Invoke it with:

```text
$codex-image2free
```

The Skill is self-contained: `SKILL.md` includes the job manifest, QA rubric, correction loop, browser-performance rules, and stop conditions. No external `references/` file is required.

## What's new in v0.2 beta

- Standardizes the public Skill and repository name as `codex-image2free`.
- Embeds the QA loop contract directly in `SKILL.md`, fixing the broken dependency on the missing `references/qa-loop-contract.md` file.
- Reuses one browser binding, tab, and ChatGPT conversation for each job.
- Reuses pasted references across correction rounds and submits short delta prompts instead of repeating the full brief.
- Uses DOM and page-asset deltas with adaptive polling to reduce slow browser automation and duplicate submissions.
- Keeps authentication inside the visible managed browser profile; it never reads or exports cookies or credentials.

This is a beta release. ChatGPT UI changes can temporarily affect browser automation, so validate the first run in your environment before using it for unattended batches.

## Authentication

Sign in through the visible ChatGPT page. The Skill reuses the in-app browser's managed profile and never reads or exports cookies, tokens, passwords, local storage, or browser profile files.

## Limits

The name does not guarantee free or unlimited usage. ChatGPT account limits, policies, rate limits, and subscription terms still apply. Automated QA is final only for explicit, testable criteria; subjective approval can still require user sign-off.
