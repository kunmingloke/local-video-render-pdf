---
name: local-video-render-pdf
description: Generate a professional, detailed, figure-rich Chinese LaTeX course note and final PDF from a local video file in the current workspace or a user-provided filesystem path. Use when the user provides a local .mp4, .mkv, .webm, .mov, .avi, or similar video file and wants structured teaching notes, meeting notes, interview notes, lecture notes, or a rendered PDF based on the video's audio, subtitles, visible slides, diagrams, screens, formulas, code, and key frames.
---

# Local Video Render PDF

Use this skill to turn a local video file into a complete, compileable `.tex` note and a rendered PDF.

## Goal

Produce a professional Chinese teaching note from a local video file.

The output must:

- use the video's actual spoken and visual content, not transcript text alone
- support local paths and filenames with spaces, Chinese characters, and parentheses by quoting paths carefully
- use nearby subtitle or transcript timestamps to select useful figures
- include all necessary high-value key frames as figures, without adding redundant screenshots
- end with a final synthesis section that includes substantive closing discussion and distilled takeaways
- be structurally organized with `\section{...}` and `\subsection{...}`
- include a populated table of contents when `\tableofcontents` is present
- be a complete `.tex` document from `\documentclass` to `\end{document}`
- be compiled successfully to PDF as part of the final delivery, or use the fallback PDF pipeline when no local LaTeX engine is available

## Input Handling

1. Resolve the video path first.
   If the user gives a relative path, resolve it from the current working directory.
   If the user says "current folder" or gives only a filename, search the current working directory first.
   Do not search the entire home directory unless the user asks.

2. Accept common video containers:
   `.mp4`, `.mkv`, `.webm`, `.mov`, `.avi`, `.m4v`, `.ts`.

3. If multiple plausible local videos match the request, ask the user which one to process.
   If only one plausible video exists in the current working directory, use it.

4. Keep all artifacts local in a dedicated working directory:
   metadata, extracted cover frame, subtitle or transcript file, audio file, candidate frames, final assets, `.tex`, and `.pdf`.

## Source Acquisition

1. Inspect local video metadata with `ffprobe`.
   Record duration, resolution, frame rate, audio streams, subtitle streams, and container format.

2. Prefer existing subtitles before transcription.
   Check, in order:
   - user-provided subtitle file
   - same-directory subtitle files with matching basename, such as `.srt`, `.vtt`, `.ass`, `.ssa`
   - embedded subtitle streams in the video container
   - Whisper speech-to-text as fallback

3. If using embedded subtitles, extract them with `ffmpeg`.
   Preserve timestamps.

4. If using Whisper, extract audio first with `ffmpeg`, then transcribe to timestamped SRT.
   Prefer a language supplied by the user.
   If the language is unknown, let Whisper detect it or infer from the filename and early speech.

5. Acquire a cover image before writing the `.tex`.
   Use the user-supplied cover if provided.
   Otherwise, extract a clean representative frame from early in the video, avoiding black frames, countdown screens, loading screens, and empty desktop views.

## Long Video Strategy

For videos longer than 20 minutes, or transcripts with more than 300 subtitle entries:

- split the work into coherent segments by chapter markers, subtitle topic shifts, slide transitions, or fixed time windows
- keep a small overlap between neighboring segments when explanations cross boundaries
- summarize each segment by teaching goal, core claims, important formulas or code, required figures, and ambiguities
- integrate the final document into one coherent narrative rather than concatenating segment summaries

## Teaching Content Rules

Build the notes from all available local evidence:

- filename and metadata
- existing subtitles or Whisper transcript
- on-screen slides, diagrams, formulas, tables, plots, dashboards, terminal output, code, whiteboards, and screen shares
- speaker emphasis and examples
- short high-signal original dialogue snippets when the video is an interview, panel, meeting, or conversation

Skip content that does not contribute to the lesson:

- greetings
- repeated filler
- long silences
- routine logistics
- low-information closing pleasantries

Keep substantive closing discussion when it carries teaching value, such as limitations, tradeoffs, advice, open questions, or next steps.

## Writing Rules

1. Write the notes in Chinese unless the user explicitly requests another language.

2. Start from `assets/notes-template.tex`.
   Fill the metadata block and replace the body placeholder with the generated notes.

3. Reconstruct the teaching flow when needed.
   Do not dump subtitle content chronologically.
   Each major section should explain motivation, main idea, mechanism, example or evidence, and takeaway.

4. Use `importantbox` for core concepts, definitions, key mechanism summaries, theorem-like statements, critical algorithm steps, and compact restatements of dense material.

5. Use `knowledgebox` for helpful background, terminology comparisons, engineering context, and side knowledge.

6. Use `warningbox` for common misunderstandings, hidden assumptions, misleading heuristics, implementation mistakes, and places where the speaker contrasts wrong intuition with correct reasoning.

7. Use `dialoguebox` only for conversation-heavy videos when a brief original exchange is high-information, vivid, or especially useful.
   Keep it short and preserve timestamps.

8. Do not place images inside custom message boxes.

9. When a formula appears, explain it in plain Chinese, show it in display math, then explain every symbol.

10. When code or command examples appear, explain their role before the code and summarize behavior afterward.
    Prefer `verbatim` for short command snippets in Chinese XeLaTeX documents.

11. End every major section with `\subsection{本章小结}`.

12. End the document with a final top-level section such as `\section{总结与延伸}`.
    Include the speaker's substantive closing discussion, your structured distillation, and practical next steps.

## Figure Handling

Frame understanding must come from direct visual inspection.

- Use `ffmpeg` to extract dense candidate frames around subtitle-aligned intervals.
- Use contact sheets only for recall; final keep-or-reject decisions must be based on direct image inspection.
- Do not infer a frame's meaning only from nearby transcript text.
- Prefer the final fully populated state of slides, animations, whiteboards, dashboards, or progressive reveals.
- Reject frames where referenced visual content is incomplete or unreadable.
- Use crops when the relevant region is small, but keep enough surrounding context for understanding.

Whenever the `.tex` or PDF references a frame or crop derived from the video, record the source time interval in a footnote on the same page.

Use neutral timestamp-based names for raw candidate frames.
Rename a figure semantically only after visually confirming what it shows.

## Transcript Quality and Technical Term Correction

For technical videos, transcription quality directly affects the notes. Prefer a Whisper model that is strong enough for domain vocabulary:

- prefer `base` or `small` for technical lectures when runtime is acceptable
- use `tiny` only when speed or resource limits matter, and compensate with stronger visual and glossary checks
- build a short correction glossary from the video filename, slide text, and early transcript before writing the notes
- correct common domain terms in the working summary before final prose, such as protocol names, acronyms, register names, packet names, tool names, and vendor/product names
- for PCIe-like courses, normalize terms such as PCIe, PCI Express, PCI-X, MindShare, Arbor, TLP, DLLP, ACK/NAK, BAR, DMA, PIO, requester, completer, configuration space, memory space, and IO space

Do not blindly copy low-confidence transcript phrases into the final notes when slide text or technical context clearly identifies the intended term.

## Slide Recording Frame Strategy

When the local video is primarily a slide recording or screen-recorded lecture:

- extract candidate frames at regular intervals, normally every 10--20 seconds, plus subtitle-aligned topic boundaries
- create contact sheets for fast recall, but make final keep/reject decisions by inspecting the original full-resolution frame
- remove near-duplicate slides using visual comparison, perceptual hashing, or manual comparison when automated tools are unavailable
- prefer slides that introduce a new concept, table, diagram, timing waveform, architecture view, code block, terminal output, or final annotated state
- avoid repeating agenda slides unless the cursor, highlight, or annotation materially changes the teaching point
- keep a small set of high-value figures rather than many redundant screenshots

## Visualization

For concepts that remain hard to explain with screenshots and prose alone, add accurate visualizations.

Acceptable routes:

- create LaTeX-native visualizations with TikZ or PGFPlots
- generate figures ahead of time with Python, preferably vector PDF for plots and diagrams

When using TikZ, avoid generic style or key names that can conflict with TikZ or PGF internals. Do not define styles named `step`, `node`, `path`, `edge`, `matrix`, or other common TikZ keys. Prefer names such as `flowstep`, `videostep`, `pipelineNode`, or `conceptBox`.

Do not add decorative graphics that do not teach anything.

## Fallback PDF Pipeline

The preferred path is still to generate a complete `.tex` document and compile it to PDF. However, some local environments do not have a usable LaTeX engine. If no `xelatex`, `pdflatex`, `lualatex`, or `tectonic` executable is available:

- still create a complete `.tex` source file when the filesystem permits it
- generate the deliverable PDF with a local fallback renderer such as ReportLab, HTML-to-PDF, or another available local PDF library
- clearly state in the final answer that the PDF was generated through the fallback renderer rather than LaTeX compilation
- do not pass raw LaTeX commands into fallback-rendered PDF text
- convert display math to readable Unicode or plain-text equivalents for the fallback PDF, while keeping proper LaTeX math in the `.tex` source
- convert inline math such as `$B_{\mathrm{peak}}$` to readable text such as `B_peak` in fallback output
- use static or manually generated table-of-contents entries if the fallback renderer cannot reliably resolve dynamic page numbers
- verify fallback PDFs by checking final file size, page count, extractable text, embedded images, and obvious rendering issues in pages containing formulas or figures

Example fallback formula rendering:

- LaTeX source: `B_{\mathrm{peak}} = f_{\mathrm{clk}} \times \frac{W}{8} \times N_{\mathrm{transfer}}`
- fallback PDF text: `B_peak = f_clk × (W / 8) × N_transfer`

If the workspace allows ordinary writes but blocks a specific PDF or source write because of sandboxing, file locks, or renderer behavior:

- try a new output job name first, such as `<basename>_fixed` or `<basename>_current`
- try generating the document into memory and then writing bytes to disk if the renderer supports it
- if the write still fails and the PDF is required for the user request, request escalated local write permission and record this in the working notes or final answer

## Compilation and QA

Compile the final `.tex` at least twice so the table of contents, references, links, and footnotes are resolved.

On Windows or MiKTeX, prefer a stable, lightweight LaTeX preamble:

- use `\usepackage{tcolorbox}` plus `\tcbuselibrary{breakable}` instead of `\usepackage[most]{tcolorbox}` unless the extra libraries are truly needed
- prefer `verbatim` or simple framed text for short command snippets when `listings` causes slow or unstable XeLaTeX compilation with `ctex`
- avoid heavy package combinations when a simpler LaTeX-native equivalent is enough

After compilation, do not treat the mere presence of a PDF as success. Verify:

- the compile command exits successfully on the final run
- the `.log` file does not contain fatal errors such as `! Package`, `! LaTeX`, `Fatal`, `Emergency stop`, or `No output PDF file written`
- if the document contains `\tableofcontents`, the generated `.toc` file exists and is non-empty
- if the `.toc` is empty, fix the earlier LaTeX error and recompile from a clean `.aux/.toc/.out` state
- the final PDF size changed after the latest successful compile and is not an old stale artifact
- pages containing formulas do not show raw LaTeX syntax unless the PDF was intentionally compiled by LaTeX and the syntax is part of a code example

If the output PDF cannot be overwritten because it is open in a viewer or locked by the operating system, compile with a new job name such as `<basename>_fixed` or `<basename>_v2`, then deliver the newly generated PDF and clearly name it as the current final version.

## Final Checklist

Before delivery, verify all of the following:

- the local video path was resolved correctly
- metadata, subtitle/transcript source, and cover-frame source are known
- no important teaching content has been dropped
- the text and figures are aligned
- inserted frames show the fullest relevant visual state rather than a transitional or incomplete state
- every inserted video frame has source time provenance
- the table of contents is populated when present, and the `.toc` file is not zero bytes after final compile, or a fallback static table of contents is present when using a non-LaTeX renderer
- the final `.log` has no fatal LaTeX/package errors when LaTeX compilation is used
- fallback-rendered PDFs do not expose raw LaTeX syntax in normal prose, formulas, captions, or headings

## Delivery

Deliver all of the following:

- the extracted or selected cover image referenced on the front page
- the subtitle or Whisper-generated SRT file used as the timing source, when available
- any extracted or generated figure assets referenced by the document
- the final `.tex` file and compiled or fallback-rendered `.pdf` file

## Asset

- `assets/notes-template.tex`: lightweight LaTeX template for local video notes
