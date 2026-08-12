# AI Video Generation — veo3 (kie.ai)

An [n8n](https://n8n.io) workflow that takes a story title from a form, writes a two-part cinematic short story with GPT, converts each part into a detailed ~8-second video prompt, and renders both video clips using kie.ai's `veo3_fast` model — automatically saving the results to Google Drive.

## What it does

1. **On form submission** — a form collects a story `title`.
2. **story genrator** (GPT-5-mini) — writes a two-part connected short story from the title, structured into `part1` / `part2`.
3. **prompt genrator** (GPT-5-mini) — converts each story part into a cinematic video-generation prompt (`prompt1` / `prompt2`), including camera movement, lighting, mood, and environment detail, each covering ~8 seconds of video.
4. **post request1 / post request2** — POST each prompt to kie.ai's `veo/generate` endpoint using the `veo3_fast` model (16:9, text-to-video).
5. **Wait + Poll loop** — `Wait` nodes pause, then `get video` / `get video2` poll `veo/record-info` for job status; an `If` node checks `successFlag` and loops back to waiting until the render finishes.
6. **Convert + Upload** — once each video finishes rendering, it's downloaded as binary data and uploaded to a Google Drive folder (`part1`, `part2`).

## Architecture

```
Form (title) → Story generator (GPT) → Prompt generator (GPT) ─┬─ kie.ai veo/generate (part 1) → poll → download → Google Drive
                                                        └─ kie.ai veo/generate (part 2) → poll → download → Google Drive
```

## Tech / Services used

| Component | Purpose |
|---|---|
| n8n | Workflow orchestration + form trigger |
| OpenAI GPT-5-mini | Story writing + cinematic prompt generation |
| kie.ai (`veo3_fast` model) | AI text-to-video generation |
| Google Drive | Storage for generated video clips |

## Setup

1. Import `video-generation-veo3-kie-api.json` into n8n.
2. Configure credentials:
   - **OpenAI** — API key.
   - **Google Drive OAuth2** — connect your Google account and update the target `folderId` in the two `Upload file` nodes.
3. Replace the placeholder `Bearer YOUR_KIEAI_API_KEY_HERE` in all four kie.ai `HTTP Request` nodes with your own [kie.ai](https://kie.ai) API key.
4. Activate the workflow and submit the form to generate a story + two video clips.

> **Security note:** The original export contained a live kie.ai API key hardcoded in multiple HTTP Request nodes. It has been redacted to a placeholder in this published copy — use your own key.
