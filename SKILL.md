---
name: chatgpt-browser-image-loop
description: Run planned image-generation tasks through the logged-in ChatGPT web app in Codex's in-app browser, paste local reference images through the clipboard, submit Codex-written prompts, download generated images, and iterate with evidence-based prompt corrections until the task-specific QA contract passes or a stop condition occurs. Use when a plan requires ChatGPT web image generation, reference-guided image editing, automatic local image QA, or a generate-review-revise loop.
---

# ChatGPT Browser Image Loop

Execute image jobs from a Codex plan in the ChatGPT web interface, then bring each result back into Codex for local QA and iterative correction.

## Protect authentication data

Treat “save the login cookie” as “reuse the in-app browser's managed persistent profile.” Let the user sign in through the visible ChatGPT page once, then reuse that browser binding and profile while the session remains valid.

Never inspect, read, export, print, copy, serialize, commit, or otherwise handle cookies, local storage, passwords, tokens, or browser session files. Never include authentication data in the skill, repository, logs, prompts, or generated artifacts. If the session is signed out or challenged, ask the user to complete sign-in in the in-app browser and resume afterward.

## Build an image-job contract

Before opening ChatGPT, turn every planned image task into a separate job with:

- a unique job ID and one requested output;
- the ordered local reference-image paths, if any;
- the generation or edit prompt;
- hard requirements and forbidden elements;
- a task-specific QA rubric and passing threshold;
- a candidate output directory and non-destructive filename pattern;
- optional user-defined iteration or cost limits.

Process jobs one at a time. Use a fresh ChatGPT conversation for each unrelated image job so references and instructions do not leak between tasks. Never batch multiple deliverables into a collage unless the plan explicitly requests one.

Read [references/qa-loop-contract.md](references/qa-loop-contract.md) before executing the first job.

## Operate the in-app browser

Use `browser:control-in-app-browser` and follow its current instructions completely. Select the in-app browser explicitly, initialize its runtime from the installed Browser plugin path, read the browser's complete runtime documentation, and reuse the resulting browser binding across the job loop. Do not hardcode a plugin version, browser node ID, or undocumented API.

Navigate to `https://chatgpt.com/` and inspect visible page state. Confirm that the user is signed in. If authentication, CAPTCHA, account confirmation, or a consent screen blocks the task, pause and request the user's visible interaction; do not bypass it or switch browsers.

Use the image-generation mode currently exposed by ChatGPT. Do not hardcode a model name because the web product may rename or replace the image model.

## Paste references and prompt

Prefer clipboard paste when the upload button or file chooser is unreliable:

1. Re-read the visible DOM and find the current composer; node IDs are dynamic.
2. Focus the composer.
3. Read one local reference image, write it to the browser clipboard with its correct MIME type, and press the platform paste shortcut.
4. Verify the attachment appears before pasting the next reference.
5. Preserve the job contract's reference order.
6. Paste or type the complete prompt only after all required references are attached.

Submit exactly once. A slow browser call is not proof of failure; inspect visible state before retrying so the same job is not submitted twice.

## Retrieve and inspect the result

Wait for ChatGPT to finish. Identify the new generated image through the browser's documented page-asset or download capability. Distinguish generated outputs from uploaded references. Save only the intended generated asset to the candidate directory without overwriting references or previously accepted files.

Open the local candidate with the available image-inspection tool. Compare it against every hard requirement, forbidden element, reference image, and scored criterion in the job contract. Record concrete evidence, not impressions alone.

Never claim a pass when the image cannot be retrieved or inspected.

## Iterate with targeted corrections

If the candidate fails:

1. List the failed hard gates and the largest scoring deductions.
2. Preserve everything that already passed.
3. Write a delta prompt that changes only the failed attributes and explicitly locks passed attributes.
4. Continue in the same ChatGPT conversation when its image context is intact; otherwise start a clean conversation and paste the necessary references plus the latest candidate.
5. Submit once, retrieve the next candidate, and run the full QA contract again.
6. Track iteration number, prompt, output path, score, hard-gate failures, and correction rationale.

Continue until the image passes or a stop condition occurs. Do not repeat an unchanged prompt after a failure.

## Stop conditions

Stop the loop and report the current best candidate when any of these occurs:

- the QA contract passes;
- the user stops or changes the task;
- an explicit iteration, time, or cost limit is reached;
- sign-in, CAPTCHA, rate limits, policy refusal, or browser unavailability blocks progress;
- three consecutive iterations fail for substantially the same reason, indicating prompt-only correction is no longer making progress;
- required reference files or acceptance criteria are missing and cannot be inferred safely.

The no-progress stop prevents an uncontrolled infinite loop. Do not weaken the acceptance standard merely to finish.

## Complete the job

For a pass, return the accepted candidate path, final prompt, iteration count, QA score, and evidence that all hard gates passed. For a stop without a pass, return the best candidate, remaining failures, attempted corrections, and the exact blocker.

Treat automated QA as final only when the job contract contains explicit, testable criteria. Mark subjective brand or artistic approval as requiring user sign-off even if the technical threshold passes.
