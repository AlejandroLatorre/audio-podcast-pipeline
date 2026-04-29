# Audio Podcast Pipeline

A desktop application that takes a written podcast script and produces a finished MP3 episode: two voices, overlaps where they belong, background music, the lot. It uses a language model as an audio director and a TTS provider for the actual synthesis.

The application is part of a private corporate toolkit and the source code is not published. This README describes what the tool does and the choices that shaped it.

## What you give it, what it gives back

Input is a script tagged with `HOST:` and `EXPERT:` lines, in Word or plain text. Output is an MP3 episode. Between those two ends, the application does four things.

First, it reads the script and adds direction. A pass called Auto-Direct goes through the dialogue and marks tone shifts, places where the Expert is allowed to interrupt, places where the Host can react with a short overlap, hesitations, and emphasis. The annotations are scoped to the production style the user picked: Classic for clean turn-taking, Formal for longer pauses and no overlaps at all, Debate for frequent interjections and a faster pace. The director prompt is style-aware; switching styles produces three perceptibly different episodes from the same script.

Second, it renders each turn through TTS. The user picks one voice for the Host and one for the Expert from a curated catalogue, and each segment is synthesised on its own. Segment-level rendering matters because the user wants to be able to re-roll a single line without rebuilding the whole episode.

Third, it mixes. The cleanly rendered segments go into a mixer that applies the chosen background music preset, sets the volume, adds fades and crossfades, and crucially adds the overlaps that the director marked. TTS engines render every line in isolation; overlap is something the pipeline imposes on top, not something it asks the engine for.

Fourth, it exports. One MP3 file, ready to publish.

## Why the architecture looks like this

The honest reason the application exists is that off-the-shelf TTS sounds like a screen reader. Each line is read on its own, there is no overlap, no interjection, no pacing, and the result is unusable for anything client-facing. Closing that gap turned out to be less about picking a better voice and more about everything around the voice.

The big lever is the director, not the renderer. Lifting perceived quality came from changing the request to the engine from "render this line" to "render this line, given this annotated direction." Most of the engineering investment lives in the director prompt and the segment-merge logic, not in the TTS call itself.

Overlaps are a post-process. We tried prompting the TTS engine to overlap turns; it cannot. We tried generating both turns separately and crashing them together with a fixed offset; it sounded mechanical. The current pipeline lets the director annotate where overlap windows should live (length, where in the previous turn they begin, what kind of reaction they are), and the mixer assembles them after rendering. This also keeps the TTS layer pluggable across providers.

Style sits in the director, not in the voice. The same voice can deliver Formal and Debate at perceptibly different paces because what changes between them is the director prompt and the mixer's pause-and-overlap policy. Voice selection stays orthogonal to style, which keeps the user's two decisions independent.

The voice catalogue is short on purpose. The application exposes maybe a dozen voices that work well for the use case (broadcaster cadence, clean diction, decent coverage of EN, ES, MX, CO, US accents) instead of dumping every voice the provider supports on the user. That decision came out of the companion utilities described below.

## Companion scripts

Alongside the application sits a folder of around sixty standalone scripts written during development. They list voices, generate sample readings of fixed text in different accents, compare conversational voices against broadcaster voices, run pitch and pacing experiments, and produce reference renders of the same paragraph with a dozen different voices side by side. None of them are user-facing. Their job was to inform the curated catalogue and the default style settings; they are kept around as documentation of the empirical work.

## Distribution

The application ships as a one-click installer. Double-click sets up Python, creates a virtual environment, installs the dependencies, and launches the local server. The user this is built for is a consultant putting together an internal podcast, not a developer.

## Status

Version one is in production. Auto-Direct, multi-voice rendering, the BGM preset library, and segment-level re-render are all shipped.

## About this repository

You will only find this README here. The code is not public.
