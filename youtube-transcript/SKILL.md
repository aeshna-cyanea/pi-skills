---
name: youtube-transcript
# kept OUT of the model-facing catalog 2026-11-13 by pan at autumn's call (Key 371 closed): 0 opens recorded in
# 135 session trees, and its one-line usage is folded into the hidden body of ~/.pi/agent/skills/web-browsing/
# SKILL.md, which costs 0 B. The script stays where it is and is tested working:
# ~/.agents/skills/pi-skills/youtube-transcript/transcript.js <video-id-or-url>   (deps installed, EBw7gsDPAYQ)
disable-model-invocation: true
description: Fetch transcripts from YouTube videos for summarization and analysis.
---

# YouTube Transcript

Fetch transcripts from YouTube videos.

## Setup

```bash
cd {baseDir}
npm install
```

## Usage

```bash
{baseDir}/transcript.js <video-id-or-url>
```

Accepts video ID or full URL:
- `EBw7gsDPAYQ`
- `https://www.youtube.com/watch?v=EBw7gsDPAYQ`
- `https://youtu.be/EBw7gsDPAYQ`

## Output

Timestamped transcript entries:

```
[0:00] All right. So, I got this UniFi Theta
[0:15] I took the camera out, painted it
[1:23] And here's the final result
```

## Notes

- Requires the video to have captions/transcripts available
- Works with auto-generated and manual transcripts
