# Audio Podcast Pipeline

A desktop application that turns a written podcast script into a fully produced MP3 episode — multi-voice, with overlaps, interruptions, configurable production styles and background music — using an LLM as an audio director and a TTS engine for the synthesis.

The application was built as part of an enterprise consulting toolkit; the source code is not public. This repository documents what the product does and the engineering decisions behind it.

---

## What it does

The user drops a script tagged with `HOST:` and `EXPERT:` lines (Word or plain text) and the application:

1. **Annotates** the script with an "Auto-Direct" pass: the LLM marks tone shifts, allowed interruptions, overlap moments, hesitations and emphasis, in line with the chosen production style (Classic, Formal or Debate).
2. **Renders** each turn segment-by-segment with the selected voices — one voice for the Host, one for the Expert, both customizable from a curated voice catalog (broadcaster-grade voices, Spanish/English/regional Latam variants).
3. **Mixes** the segments with the chosen background music preset, at the configured volume, with fades and crossfades.
4. **Exports** an MP3 episode ready to publish.

The user can edit segments before synthesis, swap voices per role, and re-render only the changed parts.

---

## Why it exists

Off-the-shelf TTS produces flat narration: each line is read in isolation, there are no overlaps, no interjections, no dramatic pauses, and the result sounds like a screen reader. For internal communications and client-facing podcasts, that is unusable.

The pipeline addresses three gaps:

- **Conversational realism.** Real podcasts have the Expert *talking over* the Host's "right, right" or *cutting in* mid-sentence. Auto-Direct decides where these moments fit, given the script and the selected style, and the renderer overlaps the audio accordingly.
- **Style as a first-class control.** The same script rendered as *Classic* (clean turn-taking), *Formal* (no overlaps, longer pauses) and *Debate* (frequent interjections, faster pace) produces three perceptibly different episodes. The director prompt is style-aware.
- **Production polish without a producer.** Background music, intro stingers, fades and gain staging are presets the user can pick from a dropdown; the pipeline applies them consistently across episodes.

---

## Functional capabilities

| Capability | What it covers |
|---|---|
| Segment editor | Edit, delete or insert turns before synthesis; nothing is hidden |
| Multi-voice selection | Pick a voice per role from a catalog of broadcaster, conversational and energetic voices in EN / ES / MX / CO / US accents |
| Auto-Direct | LLM annotates the script with tone, overlaps, interruptions and emphasis, scoped to the chosen production style |
| Production styles | Classic / Formal / Debate, each tuned with its own director prompt |
| Background music | Preset library (corporate, lounge, cinematic, soft piano, futuristic synth, epic drone, strings, tech minimal…) with adjustable per-episode volume |
| Segment-by-segment render | Re-render only edited segments, not the whole episode |
| Output | MP3 episode ready to publish |

---

## Engineering choices that mattered

- **TTS quality is bounded by the prompt the engine receives, not by the model.** Lifting the perceived quality required moving from "render this line" to "render this line *given this annotated direction*" — emphasis tags, interruption markers, overlap windows. Most of the engineering investment is in the director prompt and the segment-merge logic, not in the TTS call itself.
- **Overlap is a post-process, not a TTS feature.** TTS engines render each segment cleanly; overlaps are added at the mixing stage, with windows defined by the director's annotations. This keeps the renderer pluggable across providers.
- **Style is encoded in the director, not the voice.** Switching styles does not change the voice — it changes the director prompt and the mixer's pause/overlap policy. The same voice can deliver Formal and Debate at perceptibly different paces.
- **Voice catalog is curated, not exhaustive.** The application exposes a short, opinionated list of voices that work well for the use case (broadcaster cadence, clean diction, Spanish/English/Latam coverage) instead of dumping every available voice on the user.
- **Plug-and-play distribution.** The application is shipped as a one-click installer that provisions Python, the virtual environment, dependencies and launches the local server. The target user is not a developer.

---

## Companion utilities

Alongside the main application, the project ships ~60 standalone scripts used during development to evaluate voice catalogs, accents and rendering strategies — voice listing, regional sample generators (ES/MX/CO/US), conversational vs. broadcaster comparisons, pitch and pacing experiments. These are not user-facing but document the empirical work behind the curated voice catalog.

---

## Status

Active. v1.0 distributed as a self-contained installer. Auto-Direct, multi-voice, BGM presets and segment-level re-render are in production use.

---

## What this repository contains

This README. The code lives in a private corporate repository and is not redistributable.
