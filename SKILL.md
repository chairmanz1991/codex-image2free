---
name: codex-image2free
description: Run planned image-generation and image-editing tasks through the logged-in ChatGPT web app in Codex's in-app browser, paste local reference images through the clipboard, submit Codex-written prompts, retrieve generated images, and iterate with local QA until explicit acceptance criteria pass or a stop condition occurs. Use for ChatGPT web image generation, reference-guided image editing, automatic visual QA, or a generate-review-revise loop that should minimize browser overhead.
---

# codex-image2free

Route image jobs from a Codex plan through ChatGPT's current web image generator, retrieve each result, and review it locally before accepting or revising it.

The repository name does not guarantee zero cost or unlimited generation. ChatGPT account limits, rate limits, policies, and subscription terms still apply.

## Protect authentication

Reuse the in-app browser's managed persistent profile after the user signs in visibly. Never inspect, export, print, copy, serialize, or commit cookies, local storage, passwords, tokens, or browser profile files. If authentication expires or a challenge appears, pause and ask the user to sign in through the visible page.

## Build one job contract per output

Before opening ChatGPT, define:

```yaml
job_id: image-001
task: "One requested output"
mode: generate | edit
references:
  - path: /absolute/path/to/reference.png
    role: "identity, product, composition, or style"
prompt: "Full prompt written by Codex"
hard_requirements:
  - "Condition that must pass"
forbidden_elements:
  - "Element whose presence is an automatic failure"
output_directory: /absolute/path/to/candidates
qa_threshold: 85
user_signoff_required: false
max_iterations: null
```

Use runtime paths from the current task; never hardcode one user's paths in the Skill. Process unrelated jobs in separate conversations so references do not leak between tasks.

## Use the fast browser path

Use `browser:control-in-app-browser` and follow its current instructions. Optimize browser work as follows:

1. Initialize the browser runtime only once per fresh execution session.
2. Read the selected browser's complete runtime documentation only once per browser binding.
3. Reuse the same in-app browser binding, tab, and ChatGPT conversation for all iterations of one job.
4. Open a new conversation only when starting an unrelated job or when the existing context is corrupted.
5. Do not re-upload reference images during corrections while the same conversation still retains them.
6. Prefer DOM and page-asset operations over screenshots and coordinate-based clicking.
7. Re-read the full visible DOM only after navigation, a major UI change, or a stale-node error. Reuse a verified composer node while the page state is unchanged.
8. Combine stable actions into one browser execution when possible: focus the composer, paste ordered references, verify attachments, enter the prompt, and submit once.
9. Take a page-asset inventory snapshot before submission. After generation, retrieve only the new asset IDs instead of rescanning or downloading every page image.
10. Use short adaptive polling rather than fixed long sleeps: check completion or new page assets every 5–10 seconds, with each wait below 30 seconds.

Do not submit twice because a browser call timed out. Inspect visible state or asset inventory first.

## Attach references and submit

Navigate to `https://chatgpt.com/`, confirm the visible session is signed in, and select the image-generation mode currently exposed by the site. Do not hardcode a model name.

When the file chooser is unreliable:

1. Focus the current composer.
2. Read one local image and write it to the browser clipboard with the correct MIME type.
3. Paste it using the platform shortcut.
4. Confirm the attachment appears.
5. Repeat in the job contract's reference order.
6. Enter the complete prompt after the references are attached.
7. Submit exactly once.

## Retrieve the candidate

Wait for the generation state to complete. Use the browser's documented page-asset or download capability to distinguish the new generated image from uploaded references. Save it as a new candidate file without overwriting references or accepted outputs.

If the result cannot be retrieved and inspected locally, do not claim that it passed.

## Apply the embedded QA contract

Require zero hard-gate failures and a total score at or above the job threshold. Use this default rubric unless the task supplies a better one:

| Category | Weight | Evaluate |
|---|---:|---|
| Prompt and constraint fidelity | 30 | Subject, action, setting, count, format, required and forbidden elements |
| Reference fidelity | 25 | Identity, product, layout, palette, or style assigned to each reference |
| Composition and usability | 15 | Framing, crop, hierarchy, negative space, intended use |
| Technical image quality | 15 | Anatomy, geometry, hands, edges, textures, lighting, artifacts, accidental text |
| Task-specific criteria | 15 | Brand, continuity, safety, factual, or domain requirements |

Record for every iteration:

```yaml
iteration: 1
candidate_path: /absolute/path/to/candidate.png
score: 78
hard_gate_failures:
  - "Required feature is missing"
deductions:
  - category: Reference fidelity
    points: -12
    evidence: "Visible mismatch against reference 02"
passed_attributes:
  - "Camera angle"
next_delta_prompt: "Preserve the camera angle. Correct only..."
```

Treat exact text errors, missing hard requirements, forbidden elements, and identity/product mismatches designated as exact as automatic failures.

## Iterate efficiently

When a candidate fails:

1. Identify the failed hard gates and largest deductions.
2. Lock every attribute that passed.
3. Write a short delta prompt changing only the failed attributes; never resend the entire long creative brief unless context was lost.
4. Continue in the same ChatGPT conversation and reuse its references.
5. Retrieve the next asset using the page-asset inventory delta.
6. Re-run the complete QA rubric because a correction can regress an earlier pass.

Never repeat an unchanged prompt or add unrelated creative direction during repair.

## Stop conditions

Stop when:

- the QA contract passes;
- the user stops or changes the task;
- an explicit iteration, time, or cost limit is reached;
- sign-in, CAPTCHA, rate limits, policy refusal, or browser unavailability blocks progress;
- three consecutive iterations fail for substantially the same reason;
- required references or acceptance criteria are missing and cannot be inferred safely.

Do not lower the acceptance threshold merely to finish. For subjective brand or artistic approval, mark the technical result as passed but still request user sign-off when the job contract requires it.

## Report the result

For a pass, return the accepted candidate path, final prompt, iteration count, QA score, and hard-gate evidence. For a stop without a pass, return the best candidate, remaining failures, attempted corrections, and exact blocker.
