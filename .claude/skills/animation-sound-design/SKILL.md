---
name: animation-sound-design
description: Plan and prototype sound for an animation or motion video — timed cue sheet (dialogue / foley / ambience / SFX / music), one generation prompt per sound, dialogue-safe mix and ducking plan, Web Audio / Tone.js previews for synthesizable effects, file naming and delivery specs, and licensing checks. Use when the user asks for صداگذاری، طراحی صدا، افکت صوتی، فولی، cue sheet, sound design, SFX or audio for an animation, video, reel or motion piece. Works alongside OpenMontage (the user's primary video skill) and a UI Sound Design skill when either is installed.
---

# Animation Sound Design

A planning and prototyping skill. It produces **cue sheets, prompts, mix plans and
synthesized previews**. It does **not** by itself record real foley, mix to broadcast
standard, or do professional editing — say so whenever the user's expectation
drifts that way, and point to the right tool or person (foley artist, sound
library, audio-generation model, DAW / NLE).

Reply in the user's language (usually Persian); keep technical terms (dB, LUFS,
ducking, foley) in English where that's clearer.

## 0. Hard rules

1. **Never modify the animation or any existing project file without explicit
   permission.** Write new files only (cue sheet, previews, prompts) and say where.
2. **Install nothing** (npm/pip/uv packages, apt tools, models, plugins) without
   asking first. Use only what is already available; check with `which` /
   `uv run python -c "import x"` before relying on it.
3. **Be honest about perception.** If you cannot actually see, hear or analyze the
   video, say so explicitly and ask for scene descriptions, a shot list, timestamps
   or reference frames. Never invent scene content or timings and present them as
   observed.
4. **Licensing:** every time you suggest a sound source, library, or generation
   model, state its commercial-use terms (or that they must be verified) — see §8.

## 1. Related skills — check first

- **OpenMontage** — the user's primary video/montage skill. If installed, read its
  SKILL.md and use its timeline, scene list and export conventions as the source of
  truth; this skill adds the audio layer on top of it, it does not replace it.
- **UI Sound Design** — if installed, read its SKILL.md and list what it can
  actually do (typically: short synthesized UI sounds — clicks, taps, whooshes,
  confirms, errors, notifications — via Web Audio / Tone.js). Reuse its patterns
  and code for the effects in §5. Use it only for sounds that suit synthesis.
- If either skill is **not found** (search `~/.claude/skills`, `.claude/skills`, and
  the skills list), tell the user plainly and continue with the built-in
  methods below — don't pretend to use it.

## 2. Inspect the project (step 1)

Look for readable media and timelines without changing anything:

```bash
find . -type f \( -iname '*.mp4' -o -iname '*.mov' -o -iname '*.webm' -o -iname '*.mkv' \
  -o -iname '*.gif' -o -iname '*.json' -o -iname '*.otio' -o -iname '*.edl' \
  -o -iname '*.xml' -o -iname '*.fcpxml' -o -iname '*.srt' -o -iname '*.vtt' \
  -o -iname '*.aep' -o -iname '*.prproj' -o -iname '*.blend' \) -not -path '*/node_modules/*'
```

If `ffprobe` / `ffmpeg` already exist (don't install them), you can read facts:

- Duration, fps, resolution, existing audio streams:
  `ffprobe -v error -show_entries format=duration:stream=codec_type,r_frame_rate,sample_rate -of json FILE`
- Scene-change candidates:
  `ffmpeg -i FILE -vf "select='gt(scene,0.3)',showinfo" -f null - 2>&1 | grep pts_time`
- Reference frames for viewing (to scratchpad, not the project):
  `ffmpeg -i FILE -vf fps=1 SCRATCH/frame_%04d.png` — then view a few with Read.

Report what was **measured** (duration, fps, cuts) separately from what is
**inferred** (what's happening on screen). Scene-detection timestamps are
candidates; ask the user to confirm key sync points (impacts, footsteps, lip
sync). Lottie/JSON or timeline files may give exact keyframe times — prefer them.

If the media is on an external link (WeTransfer, Drive…) and the network blocks
it, say so and ask the user to add the file to the repo / session, or describe it.

**Frame ↔ time:** `t = frame / fps`. Express cue times as `MM:SS.mmm` and also the
frame number when fps is known, so the editor can snap exactly.

## 3. Cue sheet (step 2 — before generating any sound)

Produce a table, sorted by start time. One row per sound event; long beds
(ambience, music) get one row each with their full span.

| # | ID | Start | End | Layer | Description | Rel. level | Sync note |
|---|----|-------|-----|-------|-------------|-----------|-----------|
| 1 | AMB_01_forest_bed | 00:00.000 | 00:12.400 | Ambience | Soft forest air, distant birds | −24 dB (bed) | Fades in over first 1 s; dips at cut 00:06.2 |
| 2 | FOL_02_steps_grass | 00:01.250 (f30) | 00:03.900 | Foley | Light footsteps on grass, ~2 steps/s | −14 dB | Each step on heel-contact frame |
| 3 | DLG_03_hero_line1 | 00:04.000 | 00:06.100 | Dialogue | "…" | 0 dB ref (−18 to −14 LUFS-S) | Lip sync; duck AMB/MUS |
| 4 | SFX_04_door_slam | 00:08.333 (f200) | 00:09.100 | SFX | Heavy wooden slam + short room tail | −6 dB peak | Transient exactly on contact frame; 1–2 frame pre-roll whoosh optional |

Layers: **Dialogue (DLG)**, **Foley (FOL)**, **Ambience (AMB)**, **SFX**, **Music (MUS)**.
Relative level is a starting point relative to dialogue, not a final mix value.
Mark rows `ASSUMED` when timings come from the user's description rather than
measurement.

Also list: total duration, fps, sample rate target, and open questions.

## 4. One prompt per sound (step 3)

For every non-dialogue cue, write an independent prompt usable in a sound
library search or an audio-generation model. Each prompt specifies:

- **Source / material** — what makes the sound (wood, gravel, cloth, metal, synth)
- **Texture** — crisp, gritty, airy, wet, hollow, granular
- **Distance & space** — close-mic / medium / distant; dry, small room, hall, outdoor
- **Movement** — static, left→right pan, approaching, receding, Doppler
- **Duration** — exact seconds (match the cue span; add tail if needed)
- **Mood / feel** — playful, tense, magical, cartoonish vs realistic
- **Evolution over time** — attack, swell, decay, pitch rise/fall, layered stages

Template:

```
[ID] — <duration> s, <mono|stereo>
<Source/material>, <texture>, <distance/space>, <movement>.
Envelope: <attack> → <body> → <decay/tail>; <pitch/intensity changes over time>.
Mood: <feel>. Style: <realistic | stylized cartoon | hybrid>.
Avoid: music, voices, background noise, <anything else>.
Library search keywords: <3–6 keywords>
```

Dialogue gets a **separate script table** (character, line, start/end, delivery
notes) — never mix dialogue text into SFX prompts.

## 5. Dialogue vs effects, mix & ducking (step 4)

- Keep dialogue on its own bus/stem; never bake it into SFX or music files.
- Suggested stems: `DLG`, `FOL`, `AMB`, `SFX`, `MUS` → mix → master.
- **Ducking:** sidechain MUS and AMB from DLG; typical −6 to −10 dB (MUS), −3 to
  −6 dB (AMB), attack ~50–150 ms, release ~300–600 ms. List each duck window
  in the cue sheet (start/end matching dialogue lines).
- **Frequency room for speech:** cut 1.5–4 kHz on music/ambience under dialogue
  (2–4 dB), high-pass non-bass layers (80–150 Hz) to reduce mud.
- Avoid loud SFX transients on top of important words; shift by a few frames or
  lower them, and note the conflict in the sheet.
- Starting loudness targets (verify against the platform): web/social ≈ −14 LUFS
  integrated, −1 dBTP; broadcast EBU R128 −23 LUFS. Dialogue normally reads
  around −18 to −14 LUFS short-term on web deliverables.
- These are guide values; the final mix needs real listening in a DAW/NLE.

## 6. Synthesized previews (step 5)

**Good for synthesis:** UI blips, pops, whooshes, risers, impacts/booms, sparkles,
magic shimmers, cartoon boings, beeps, simple tonal stingers, wind/noise beds.
**Poor for synthesis (say so and suggest library/generation instead):** realistic
footsteps, cloth, paper, water, glass, animals, crowds, specific real places,
vehicles, and voices.

For suitable cues, write a **standalone HTML preview** to the scratchpad (or a new
`sound-previews/` folder only if the user agrees). Prefer plain Web Audio API (no
dependency). Tone.js only if it's already in the project or the user approves the
CDN load. Each preview: one button per cue labeled with its ID, plays on click
(browsers block autoplay), and an optional "play timeline" button that schedules
cues at their cue-sheet times.

Minimal pattern:

```html
<!doctype html><meta charset="utf-8"><title>Sound previews</title>
<button data-cue="SFX_04_whoosh">SFX_04_whoosh</button>
<button id="timeline">Play timeline</button>
<script>
const ctx = new (window.AudioContext || window.webkitAudioContext)();
function noise(sec) {
  const b = ctx.createBuffer(1, ctx.sampleRate * sec, ctx.sampleRate);
  const d = b.getChannelData(0); for (let i = 0; i < d.length; i++) d[i] = Math.random() * 2 - 1;
  const s = ctx.createBufferSource(); s.buffer = b; return s;
}
const cues = {
  SFX_04_whoosh(t = ctx.currentTime, dur = 0.6) {
    const n = noise(dur), f = ctx.createBiquadFilter(), g = ctx.createGain(), p = ctx.createStereoPanner();
    f.type = 'bandpass'; f.Q.value = 1.2;
    f.frequency.setValueAtTime(300, t); f.frequency.exponentialRampToValueAtTime(3000, t + dur * 0.6);
    g.gain.setValueAtTime(0.0001, t); g.gain.exponentialRampToValueAtTime(0.6, t + dur * 0.4);
    g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
    p.pan.setValueAtTime(-0.8, t); p.pan.linearRampToValueAtTime(0.8, t + dur);
    n.connect(f).connect(g).connect(p).connect(ctx.destination); n.start(t); n.stop(t + dur);
  },
};
const timeline = [["SFX_04_whoosh", 1.25]]; // [id, seconds from cue sheet]
document.querySelectorAll('[data-cue]').forEach(b => b.onclick = () => { ctx.resume(); cues[b.dataset.cue](); });
document.getElementById('timeline').onclick = () => { ctx.resume(); const t0 = ctx.currentTime + 0.1;
  timeline.forEach(([id, at]) => cues[id](t0 + at)); };
</script>
```

To deliver an audible file, render the same graph with `OfflineAudioContext`
and export WAV (encode PCM in JS, then download) — or, if Playwright/Chromium is
available, render headlessly and save the WAV to the scratchpad. Always tell the
user these are **prototypes/placeholders**, not final production sound.

## 7. File output, naming & specs (step 6)

Naming: `<project>_<cueNo>_<LAYER>_<short-desc>_v<NN>.wav`
e.g. `eden_004_SFX_door-slam_v01.wav`, `eden_000_AMB_forest-bed_v02.wav`.
Zero-padded cue number matches the cue sheet so files sort in timeline order.

Specs for video sync:
- **WAV (PCM)**, **48 kHz**, **24-bit** (16-bit acceptable for previews); MP3/OGG
  only for quick listening, never as edit masters.
- Mono for point sources (foley, most SFX); stereo for ambience, music, moving
  sounds.
- Length = cue span + natural tail; for frame-accurate placement, either start
  each file exactly at the sync transient (note any pre-roll in ms) or deliver
  full-length stems that start at 00:00:00.000 of the video.
- No normalization to 0 dBFS on individual files; leave ≥ 1 dB headroom.
- Ship the cue sheet alongside (Markdown or CSV) with the same IDs.

## 8. Sources, generation tools & licensing

When suggesting where to get a sound, include licensing every time:

- **Freesound** — per-file license (CC0, CC-BY, CC-BY-NC). CC-BY needs credit;
  **NC forbids commercial use**. Check each file.
- **Pixabay / Mixkit / Zapsplat / Sonniss GDC bundles / BBC Sound Effects** — each
  has its own terms (e.g. BBC archive is largely personal/educational/research
  only unless licensed; Zapsplat free tier requires attribution). Read the
  current license page before commercial use.
- **Paid libraries** (Epidemic Sound, Artlist, Soundsnap, Boom Library, Splice):
  commercial use usually tied to an active subscription/license — check whether
  it survives cancellation and covers the target platform/client.
- **AI audio / music generators** (e.g. ElevenLabs Sound Effects, Stable Audio,
  Suno, Udio, AudioCraft/MusicGen): terms differ by plan; free tiers often
  disallow commercial use, and some open-weight models carry non-commercial
  licenses (e.g. CC-BY-NC weights). Verify the current terms of service and plan
  — don't state them as settled from memory.
- **Synthesized previews from this skill** — original code-generated audio, no
  third-party samples; the user owns the output unless they add samples.

Always end a recommendation with a short "verify licence before commercial use"
reminder listing which cues depend on third-party material.

## 9. Workflow summary

1. Check related skills (§1) and inspect the project (§2); state what you could
   and could not analyze.
2. Ask for missing info (scene descriptions, frames, dialogue script, target
   platform, style: realistic vs cartoon).
3. Deliver the cue sheet (§3) → get user confirmation.
4. Prompts per cue (§4) + dialogue script table + mix/ducking plan (§5).
5. Synth previews for suitable cues (§6); library/generation suggestions with
   licensing for the rest (§8).
6. File naming/specs (§7). Leave project files untouched; list every file you
   created.
