# Vaideo – Product Requirements Document (PRD)

| Field   | Value                          |
|---------|--------------------------------|
| Status  | Draft v0.1                     |
| Date    | 2026-10-28                     |
| Product | Vaideo                         |
| Form    | Python CLI tool and library    |

## 1. Overview

Vaideo is a CLI tool and Python library that lets content creators produce complete videos with artificial intelligence. Starting from a topic or an idea, it generates a script, illustrates it with AI images, assembles the images into a video, and optionally adds audio and subtitles synchronized with that audio.

## 2. Problem Statement

Content creators spend hours switching between tools to write scripts, produce visuals, record or generate narration, edit the video and create subtitles. This workflow is slow, fragmented, and hard to automate or repeat.

## 3. Goals and Non-Goals

### Goals
- Provide an end-to-end pipeline: idea → script → images → video → audio → subtitles.
- Allow every stage to be run independently or chained together.
- Be scriptable (CLI) and embeddable (Python API).
- Keep AI providers swappable behind clear interfaces.

### Non-Goals (MVP)
- No graphical user interface or web application.
- No manual timeline-based video editor.
- No hosting, publishing, or social media uploading.
- No model training or fine-tuning.

## 4. Target Users

**Primary persona: content creator** (YouTube, TikTok, Instagram, educational channels) who:
- Produces videos frequently and wants to reduce production time.
- Is comfortable running a command-line tool or a simple Python script.
- Wants control over the output (style, language, duration) without advanced editing skills.

## 5. User Stories

1. As a creator, I want to generate a script from a topic so that I do not start from a blank page.
2. As a creator, I want to generate images for each scene of my script so that I get consistent visuals.
3. As a creator, I want to generate new images from an existing image so that I can reuse or vary my own visual assets.
4. As a creator, I want to turn a sequence of images into a video so that I get a publishable clip.
5. As a creator, I want to add an audio track (narration or music) to the video so that it is complete.
6. As a creator, I want subtitles synchronized with the audio so that my video is accessible and engaging.
7. As a creator, I want to run the full pipeline with a single command so that I can automate production.

## 6. Functional Requirements (MVP)

### FR-1 Script generation
- Input: topic/prompt, optional parameters (language, tone, target duration, number of scenes).
- Output: structured script (JSON/Markdown) split into scenes, each with narration text and a visual description.
- The user can edit the script file before continuing the pipeline.

### FR-2 Text-to-image generation
- Generate one or more images per scene from the scene's visual description.
- Configurable style, resolution, and aspect ratio (e.g., 16:9, 9:16, 1:1).
- Images are saved with deterministic names linked to scene order.

### FR-3 Image-to-image generation
- Input: a source image and a prompt.
- Output: a new image derived from the source (variation, style transfer, edit).
- Can be used standalone or as part of the scene image stage.

### FR-4 Video creation from images
- Assemble the ordered images into a video.
- Configurable duration per image, transitions, frame rate, resolution and output format (MP4 by default).
- Duration per scene can be derived from the narration audio when available.

### FR-5 Audio
- Add an audio track to the video: user-provided file (narration/music) or generated narration from the script (text-to-speech).
- Adjust volume and trim/loop audio to match the video length.

### FR-6 Synchronized subtitles
- Produce subtitles synchronized with the audio (speech-to-text with word/segment timestamps, or alignment with the known script).
- Export as `.srt` and optionally burn them into the video.
- Configurable font, size, color, position.

### FR-7 Pipeline orchestration
- A single command runs all stages and stores intermediate artifacts in a project directory.
- Stages can be re-run individually without regenerating previous results (caching).
- Project configuration through a file (YAML/TOML) and CLI overrides.

## 7. CLI Surface (draft)

```
vaideo script    --topic "..." --language es --scenes 8 -o script.json
vaideo images    --script script.json --style "cinematic" -o images/
vaideo img2img   --image in.png --prompt "..." -o out.png
vaideo video     --images images/ --seconds-per-image 4 -o video.mp4
vaideo audio     --video video.mp4 --audio narration.mp3 -o video_audio.mp4
vaideo subtitles --video video_audio.mp4 --burn -o final.mp4
vaideo run       --config vaideo.yaml
```

The same capabilities are exposed through a Python API (`vaideo.ScriptGenerator`, `vaideo.ImageGenerator`, `vaideo.VideoBuilder`, `vaideo.AudioMixer`, `vaideo.SubtitleGenerator`, `vaideo.Pipeline`).

## 8. Non-Functional Requirements

- **Extensibility:** AI providers (LLM, image, speech) implemented behind abstract interfaces so they can be swapped.
- **Configuration & secrets:** API keys read from environment variables or a config file, never stored in artifacts.
- **Reproducibility:** support seeds and persist the parameters used for every generated artifact.
- **Performance:** a 60-second video must be producible in a reasonable time dominated by provider latency; local processing (assembly, subtitles) should stay under 2 minutes on a standard laptop.
- **Reliability:** retries with backoff on provider failures; clear error messages; partial results preserved.
- **Observability:** structured logging with levels; a verbose flag for debugging.
- **Portability:** Linux, macOS, and Windows; Python 3.11+.
- **Quality:** unit tests for every stage, integration tests using mocked providers.
- **Cost awareness:** show an estimate of API usage before running paid stages (post-MVP nice-to-have).

## 9. Technical Considerations

- Language: Python, object-oriented design (one class per pipeline stage, shared `Stage` interface).
- CLI framework: Typer or Click.
- Video/audio processing: FFmpeg (via `ffmpeg-python` or MoviePy).
- Subtitles: Whisper (or equivalent) for timestamps; SRT/ASS generation.
- Provider candidates (to be decided): OpenAI / Anthropic / local LLMs for scripts; Stable Diffusion / DALL·E / Flux for images; ElevenLabs / OpenAI TTS / Piper for narration.
- Dependencies: FFmpeg must be installed on the host.

## 10. Success Metrics

- Time from idea to finished video under 10 minutes for a 1-minute video.
- At least 90% of pipeline runs complete without manual intervention.
- Subtitle timing error below 200 ms on average.
- Adoption: number of videos generated per active user per week.

## 11. Milestones

| Milestone | Scope |
|-----------|-------|
| M1 | Project scaffolding, config, logging, provider interfaces, CLI skeleton |
| M2 | Script generation (FR-1) |
| M3 | Text-to-image and image-to-image (FR-2, FR-3) |
| M4 | Video assembly (FR-4) |
| M5 | Audio and subtitles (FR-5, FR-6) |
| M6 | Pipeline orchestration, caching, docs, v0.1 release (FR-7) |

## 12. Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| Visual inconsistency between generated images | Style presets, shared seeds, reference images via image-to-image |
| Provider cost and rate limits | Caching, retries, usage estimates, pluggable providers |
| Copyright / content policy concerns | Document usage terms of each provider; allow user-supplied assets |
| Subtitle drift versus audio | Use word-level timestamps and validate with tests |
| FFmpeg setup friction | Clear installation docs and an environment check command (`vaideo doctor`) |

## 13. Open Questions

1. Which AI providers will be supported first (hosted APIs versus local models)?
2. Will narration be generated by the tool (TTS) or always supplied by the user?
3. Which languages must be supported for scripts and subtitles at launch?
4. Is a plugin system for third-party providers required in v1?
5. What licensing and distribution model applies (PyPI package, open source per `LICENSE`)?

## 14. Out of Scope for Future Phases

- Web/desktop UI and a visual storyboard editor.
- AI-generated video clips (text-to-video) beyond image sequences.
- Direct publishing to YouTube/TikTok.
- Multi-user collaboration and cloud storage.
