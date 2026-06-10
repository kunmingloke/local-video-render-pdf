# Local Video Render PDF Skill

`local-video-render-pdf` is a personal Codex skill for turning a local video file into structured Chinese teaching notes and a compiled PDF.

It is designed for videos already present in the current workspace or another local filesystem path, such as:

- lectures
- course recordings
- technical talks
- screen recordings
- interview recordings
- meeting recordings with substantive teaching content

## Credit

This skill is adapted from the workflow ideas in [`wdkns/wdkns-skills`](https://github.com/wdkns/wdkns-skills), especially the YouTube/Bilibili video-to-PDF note generation approach.

Credit goes to the original author(s) of `wdkns/wdkns-skills` for the original platform-video rendering concept and teaching-note workflow.

This repository is my personal modified version. I changed the workflow to fit my own usage:

- local video files instead of YouTube/Bilibili URLs
- local subtitle detection
- embedded subtitle extraction
- Whisper fallback for videos without subtitles
- local cover-frame extraction
- Windows/MiKTeX-oriented LaTeX stability checks
- fallback PDF guidance when no local LaTeX engine is available

## Repository Layout

```text
skills/
  local-video-render-pdf/
    SKILL.md
    agents/
      openai.yaml
    assets/
      notes-template.tex
NOTICE.md
README.md
```

## What The Skill Does

Given a local video file, the skill guides Codex to:

1. Resolve the video path from the current working directory or a user-provided path.
2. Inspect video metadata with `ffprobe`.
3. Prefer existing subtitles before transcription.
4. Extract embedded subtitles when available.
5. Use Whisper speech-to-text when no subtitles are available.
6. Extract a representative cover frame.
7. Extract and inspect key video frames.
8. Write a structured Chinese LaTeX note.
9. Compile the note to PDF.
10. Validate the PDF, table of contents, logs, figures, and generated assets before delivery.

## Example Prompts

```text
$local-video-render-pdf
请把当前工作文件夹里的 lecture.mp4 生成中文讲义 PDF。
```

```text
$local-video-render-pdf
视频：meeting-recording.mkv
字幕：meeting-recording.srt
请整理成中文结构化讲义，并生成最终 PDF。
```

```text
$local-video-render-pdf
请处理这个本地技术课程视频：
C:\Users\me\Videos\pcie-course.mp4
没有字幕的话请用 Whisper 转写。
```

## Supported Inputs

The skill is intended for common local video containers:

- `.mp4`
- `.mkv`
- `.webm`
- `.mov`
- `.avi`
- `.m4v`
- `.ts`

It also looks for subtitle files such as:

- `.srt`
- `.vtt`
- `.ass`
- `.ssa`

## Recommended Dependencies

The skill itself is an instruction set, but the workflow works best when these tools are available locally:

- `ffmpeg`
- `ffprobe`
- `whisper` or `openai-whisper`
- `xelatex` with Chinese LaTeX support, such as MiKTeX + `ctex`
- optional image tooling for contact sheets or crops

If LaTeX is not available, the skill includes fallback PDF guidance, but the preferred output remains a real LaTeX source file plus a compiled PDF.

## Installation

Install with the skills CLI from this repository:

```powershell
npx skills add kunmingloke/local-video-render-pdf --skill local-video-render-pdf -a codex -g -y
```

Or manually copy:

```text
skills/local-video-render-pdf
```

into:

```text
%USERPROFILE%\.codex\skills\local-video-render-pdf
```

Restart Codex after installing or updating the skill.

## Notes On Output Quality

The skill is intentionally strict about validation because video-to-PDF workflows can fail quietly.

It asks Codex to verify:

- the local video path was resolved correctly
- subtitles or transcript source are known
- extracted figures match the surrounding explanation
- every inserted video frame has timestamp provenance
- LaTeX compiles successfully
- the table of contents is populated
- `.toc` is not zero bytes
- the final `.log` does not contain fatal LaTeX errors
- fallback-rendered PDFs do not leak raw LaTeX syntax

## Why This Version Exists

The original `wdkns/wdkns-skills` workflow targets platform videos. I often need the same kind of structured PDF note generation for local recordings, where there is no platform metadata, no official thumbnail, and sometimes no subtitle track.

This version fills that gap by making local files the primary input.

## License

No license is declared for this personal modified version yet.

If you plan to publish, reuse, or redistribute this repository publicly, review the license status of the upstream materials you adapted from and choose an appropriate license for your own contributions.
