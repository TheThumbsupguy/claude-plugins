---
name: video-evidence
description: Records what a change does on screen and reads the recording back — capture from an Android device, a browser page or a scripted Playwright run, turn the video into numbered frame sheets, and find the frame where something changed. Use for a build-to-merge "check for real" step whose result unfolds over time (a transition, a flash, a camera move, game feel), when the owner hands over a screen recording to analyse, or says "record it", "take a video" or "show me the frames".
---

# Video evidence

A screenshot every few seconds misses a screen that lasts under two seconds. A video does not. But a video is the expensive read, so it comes second.

## Numbers before video

When the app exposes its state — a debug readout such as `window.__<name>`, a log line, a server row — drive the app and sample that. One call returns distance, speed, frames in the air, and there is no image to read. Record a video only when the question is visual: a flicker, a draw order, a transition, a layout that jumps.

**Hold keys, never tap.** Input that is polled once per frame can miss a press that goes down and up between two frames. Key down, wait, key up:

- Playwright: `page.keyboard.down(code)` … `page.keyboard.up(code)`.
- chrome-devtools-mcp: `press_key` only taps. The hold-and-sample script, and how to drive a page cheaply, are in `ttug-sdlc:drive-browser`.

## Capture

| Source | How | Watch for |
| :-- | :-- | :-- |
| Android device | `adb shell screenrecord --time-limit 60 /sdcard/<step>.mp4` in the background just before the action. Stop it with `adb shell killall -INT screenrecord`, wait two seconds, then `adb pull` | The file is only finished when the recorder exits, and `killall` returns before that. It stops every recorder on the device. Delete the phone's copy at teardown |
| Browser page, chrome-devtools-mcp | `screencast_start` with a `.webm` `filePath`, act, `screencast_stop` | Needs the server flag `--experimentalScreencast` and ffmpeg. **Always `.webm`.** `.mp4` — also the default when no path is given — uses an ffmpeg option that 6.1 lacks, and the server crashes with an empty file. A page that is not repainting also gives an empty file, with no error |
| Playwright script | `browser.newContext({ recordVideo: { dir, size } })`; the webm is written when the context closes | Headless software rendering can run at a low frame rate. Connect over CDP to a real browser when timing matters. Playwright's bundled ffmpeg only writes webm; it cannot read mp4 |
| The owner's own recording | Ask for the path and read it in place | Usually mp4, 30 fps |

## Read

**Did anything change?** Check before reading frames. Zero means the input never landed, and the sheets will show nothing:

```
ffmpeg -hide_banner -i <clip> -vf "select='gt(scene,0.008)',showinfo" -f null - 2>&1 | grep -c pts_time
```

`0.008` catches a change in part of the frame; drop it to `0.00002` for one small sprite moving on a still background. That low, a heavily compressed clip of a still image can count a few frames of encoder noise, so compare against a stretch you know was idle. Without the `-c`, the `pts_time` values mark where motion starts or changes, not every frame of a steady move.

**Frame sheets** — the default read. Numbered thumbnails, a few seconds per image:

```
ffmpeg -i <clip> -vf "fps=4,scale=300:-1,drawtext=text='%{n}':x=4:y=4:fontsize=22:fontcolor=yellow:box=1:boxcolor=black,tile=6x5" sheet%02d.png
```

Cell *n* shows the clip at about (*n* + 0.5) ÷ fps seconds. Tune three numbers: `fps` (4 for UI flows, 5 to 10 for fast motion), `scale` (150 for a phone in portrait, 300 to 600 for a desktop page) and `tile`. Crop to the area that matters (`crop=w:h:x:y` before `scale`) when the subject is small. Read the sheets and cite frame ranges.

**One exact frame**, by index, with the one before it. Frame *K* is time × the clip's frame rate, rounded:

```
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,r_frame_rate:format=duration -of csv=p=0 <clip>
ffmpeg -v error -i <clip> -vf "select='eq(n,K)'" -frames:v 1 -q:v 3 after.jpg
ffmpeg -v error -i <clip> -vf "select='eq(n,K-1)'" -frames:v 1 -q:v 3 before.jpg
```

Select by index, not by `-ss <time>`: a seek lands on the first frame at or after the time, so a mid-frame timestamp returns the next frame. To pull the full-size frame behind sheet cell *N*, reuse the sheet's own rate: `-vf "fps=4,select='eq(n,N)'"`.

**Where in the frame it changed.** Convert to RGB first; a difference taken in YUV comes out green all over:

```
ffmpeg -i before.jpg -i after.jpg -filter_complex "[0]format=gbrp[a];[1]format=gbrp[b];[a][b]blend=all_mode=difference,eq=brightness=0.3" -frames:v 1 diff.jpg
```

Unchanged areas are mid-grey and changed areas are brighter.

**A one-frame flicker** has a signature. With d1(f) = |f − (f−1)| and d2(f) = |f − (f−2)|: a spike in d1 at f and at f+1, with d2(f+1) small, because frame f+1 looks like f−1 again. A spike followed by stillness is a state change (a menu opening), not a flicker.

**An mp4 to share**, from a webm: `ffmpeg -i clip.webm -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" clip.mp4`.

No ffmpeg: install it from the system package manager. Do not download a static binary. Until then a browser can decode — a `file://` page with a `<video>` and a canvas, seeking frame by frame.

## Rules

- **The video stays out of git.** Commit frame sheets only, and only where the repo keeps evidence. The repo's settings name where videos and sheets go.
- **Check what the app is talking to before recording.** A recording of the wrong backend is itself the leak, and it shows whatever account is on screen.
- **Clean up what you started**, and only that: the recording on the device, the dev server, the browser you launched. Stop a server by its port (`fuser -k <port>/tcp`); `pkill -f <pattern>` can match the shell that runs it.
- **Report what the frames showed**, with frame numbers. Say when a result came from numbers and not from a video.

## Repo settings

Keys from build-to-merge's `settings.md`: `Readout:` (the debug readout), `Videos:` (where raw videos go), `Ports:`. Frame sheets go where the repo keeps evidence: the step 5 check or the record names it. Which source to capture from comes from the step 5 check that calls for the video.
