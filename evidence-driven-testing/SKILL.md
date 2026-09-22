---
name: evidence-driven-testing
description: >
  Records visual proof while testing UI behavior — the agent tests the app
  hands-on via computer use while a screen recording with structured
  test/assertion annotations captures the session — then posts the video and a
  results summary to the PR and tracker issue. Use whenever a change needs
  verifiable evidence that it works, instead of prose claims — including
  headless environments (scripted screenshots and probes) and non-UI changes
  (measured numbers, output pairs).
compatibility: Screen-recording path requires a GUI environment the agent can drive — built-in computer use, or the cua-driver CLI (trycua/cua) when the harness has no computer-use tools — plus an authenticated browser session for the app under test. The bundled recorder (scripts/evidence.py) runs on Linux (X11 via x11grab, Wayland via wf-recorder), macOS (avfoundation, needs Screen Recording permission) and Windows (gdigrab) and needs Python 3 plus ffmpeg + ffprobe built with libx264 and the ass filter. The headless path requires only a running app and a scriptable browser (e.g. Playwright via npx). Posting evidence requires gh (GitHub CLI) or equivalent.
metadata:
  version: "1.2"
---

# Evidence-Driven Testing

Record annotated proof of behavior, attach to PR and tracker issue.

Drive the app live via computer use (or `cua-driver` when no computer-use tools exist). Every action in the video is the test; the recording has no value unless it shows that interactive session.

## Inputs

- **Test targets** (required): Behaviors/flows to verify as testable statements.
- **PR / issue** (optional): Where to post evidence. If omitted, deliver to requester only.

## The recorder

`EVIDENCE` = path to `scripts/evidence.py` in this skill's folder (e.g. `~/.claude/skills/evidence-driven-testing/scripts/evidence.py`).

- **Check first**: `python3 $EVIDENCE doctor` — verifies ffmpeg, ffprobe, libx264, ass filter, and screen-capture source. Exits non-zero only when toolchain is missing.
- **Platforms** (`--source auto` picks first available):

  | OS | Source | Needs |
  |---|---|---|
  | Linux X11/XWayland | `x11` (x11grab) | `DISPLAY` set |
  | Linux Wayland | `wayland` (wf-recorder) | `WAYLAND_DISPLAY`, `wf-recorder`, wlroots compositor (Sway, Hyprland, river, Wayfire, labwc, dwl, niri). GNOME/KDE not supported. |
  | macOS | `avfoundation` | Screen Recording permission; `doctor` lists screen indexes for `--screen-index` |
  | Windows | `gdigrab` | any standard ffmpeg build |

  Capture is full screen by default; `--geometry WxH` and `--offset X,Y` crop a region (x11, wayland, gdigrab).

- **Crash-safe**: raw recording is MPEG-TS (`raw.ts`), so killed/crashed recorder still yields usable file. `stop` remuxes to `evidence.mp4`.
- **Stopping**: Linux signals via pidfd. macOS/Windows: `start` launches supervisor that owns ffmpeg child; `stop` asks supervisor via `stop.request`, which escalates (interrupt → terminate → kill), writes `recorder-exit.json`. If supervisor dies while recorder runs, `stop` refuses — stop by hand.
- **Fallback recorders** when `doctor` reports no capture source: `cua-driver recording start <dir>`/`stop`, or OS recorder (macOS: `screencapture -v out.mov`). No annotation overlay on these paths — keep protocol as `assertions.md` files.
- **Never** use `--source test` as UI evidence. It's a synthetic pattern generator for smoke-testing the toolchain.

## Instructions

### 1. Prepare the screen

- Maximize browser/app; close popups and extra panels.
- Navigate to starting state BEFORE recording, unless setup itself is under test.
- Note exact revision: `git rev-parse HEAD` + `git branch --show-current` (or deployment URL).

### 2. Start recording

```bash
python3 $EVIDENCE start \
  --output .artifacts/<task-name> \
  --title "<what is being verified>" \
  --commit "$(git rev-parse HEAD)" --branch "$(git branch --show-current)" \
  --environment "<OS / browser / display / deployment>"
```

Output: JSON with `session` path and `source`. Keep `SESSION=...` for later commands.

Add a `setup` annotation:

```bash
python3 $EVIDENCE annotate "$SESSION" --type setup \
  --message "Logged in, navigating to connectors page"
```

### 3. Test via computer use, annotating as you go

Work at watchable pace — let UI settle after each action. At each test's start:

```bash
python3 $EVIDENCE annotate "$SESSION" --type test_start \
  --message "It should execute the tool directly when permission is 'always'"
```

After each check, add assertion:

```bash
python3 $EVIDENCE annotate "$SESSION" --type assertion --result passed \
  --message "Tool ran without a permission prompt"
```

Assertion rules:
- One assertion per meaningful state change — don't annotate per UI label.
- "Precondition: ..." for starting state.
- Under 80 characters, high-signal (recorder rejects longer messages).
- If test can't run, mark `untested` with reason — never skip silently.
- Timestamp records when you asserted, not whether it was true — check screen before choosing `passed`.

### 4. Stop and review

```bash
python3 $EVIDENCE stop "$SESSION"
```

Stops capture (gracefully), burns annotations into `evidence.mp4`, probes result, writes `report.md` + `manifest.json`. Prints `"verified": true` on success.

- If rendering fails: session marked `finalization_failed` — fix cause, run `stop` again.
- If recorder process can't be signalled: session marked `recorder_lost` — run `stop` again to finalize whatever was captured (only after confirming recorder is gone).
- Confirm recording captured key moments: extract frames at assertion timestamps (`ffmpeg -ss <t> -i evidence.mp4 -frames:v 1 frame.png`).
- Fill in Caveats section of `report.md`; never leave placeholder.

### 5. Post the evidence

- `report.md` is the report: what was tested, environment + commit, pass/fail, caveats. Extend rather than rewrite.
- Post video + summary as PR comment. `gh pr comment` can't attach video — upload via PR comment box in browser, or upload to host and link (e.g. `before-and-after` adapters). Confirm video plays before claiming posted.
- Attach same video to tracker issue with one-line result.
- Send report + recording to requester.

## Guardrails

- Video must show live test session. Never present scripted playback, stitched clips, or synthetic footage. If no computer-use tools but GUI exists, drive via `cua-driver`; no GUI → headless path.
- Never record half-covered/tiled window — maximize first.
- Never record screen with secrets, tokens, customer data, or payment details — mark `untested` and say why.
- When verifying a fix, show old failure alongside new success.
- Always state exact commit/branch/deployment tested.

## No computer-use tools? Drive with cua-driver (GUI available)

Use [cua-driver](https://github.com/trycua/cua) (macOS/Windows/Linux) as actuator. Still live testing — only input mechanism differs.

- Verify setup with `cua-driver doctor`. If a `cua-driver` skill is installed, follow its protocol.
- Loop per interaction: `launch_app` → `get_window_state` (accessibility tree + screenshot) → act via `element_token` (`click`, `type_text`, `press_key`) → `verify_state` for expected postcondition.
- Wherever `doctor` reports `capture_ready: yes`, use bundled recorder for video/annotations; cua-driver only supplies input.
- Otherwise, `cua-driver recording start <dir>`/`stop` is the recorder (daemon must be running: `cua-driver serve`). On Windows/Linux shells out to ffmpeg — missing ffmpeg yields only per-turn trajectory folders (no video). Verify `recording.mp4` exists before citing; if absent, present per-turn before/after screenshots.
- No annotation overlay → keep protocol as `assertions.md` files (same as headless path).

## Headless path (no GUI available)

Same assertion discipline; swap recorder for scripted capture:

- Save to `.artifacts/<task-name>/` (gitignore — evidence gets uploaded, never committed). Keep capture script beside captures for reproducibility.
- **Screenshots**: `before-and-after` CLI (`@vercel/before-and-after`) captures URLs/elements for PR embeds. Containers/VMs: set `AGENT_BROWSER_ARGS="--no-sandbox"`.
- **Video / multi-step flows**: one-off Playwright script:

  ```bash
  npx --yes --package=playwright node record.mjs
  ```

  Minimal `record.mjs`:

  ```js
  import { chromium } from "playwright";
  const browser = await chromium.launch();
  const context = await browser.newContext({
    recordVideo: { dir: ".artifacts/<task-name>/" },
  });
  const page = await context.newPage();
  await page.goto("http://localhost:3000/path-under-test");
  // ...drive the flow...
  await context.close(); // finalizes the .webm
  await browser.close();
  ```

- **Annotation protocol as files**: number captures in test order — `01-precondition-signed-in.png`, `02-it-saves-on-blur-passed.png` — with `assertions.md` listing each `test_start`/`assertion` with result.

## Non-UI changes still need evidence

- **API / performance**: scripted probe with measured numbers → `probe-output.txt`.
- **Rendering / canvas / shader**: rendered frames + pixel assertions (diff values) as PNGs.
- **Agent behavior**: relevant transcript excerpt.
- **Bug fixes**: reproduce and capture failure **before** writing the fix — that's the "before" half.

## Capture hygiene

- Confirm the server runs *your* code (right port, right process): `lsof -i :<port>` or `ss -ltnp "sport = :<port>"` → `ps -p <pid> -o args=`.
- Evidence complements repo checks; never replaces them.
- Hand before/after media pairs to a before/after tool for PR embed (e.g. `before-and-after before.png after.png --markdown`).
