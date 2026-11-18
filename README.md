# Who Is Talking

A wearable that hears only its wearer, so a voice assistant can listen all day without recording anyone else.

> **🚧 Project in progress. Updates coming soon.**
> No results yet. The numbers below are targets, fixed before any data is collected.

## The problem

Voice wearables keep the microphone on all day. A normal voice detector fires on any speech: a TV, a friend 1 m away, a café. So the device records people who never agreed to it, and the assistant fills up with other people's words.

Who Is Talking is a gate that fires only when **the wearer** talks. It is small enough to run on the device all day.

## How it works

```
accelerometer, 1.6 kHz ──► skull-vibration energy, 100–300 Hz ──┐
(glasses temple)                                                 ├──► tiny CNN, INT8, on the board ──► per frame: {t, p_wearer, p_speech}
microphone, 16 kHz ──────► log-mel spectrum (loudness, echo) ────┘                                              │
                                                                                     wearer talking? keep  ◄────┤
                                                                                     anyone else? drop in memory ◄┘
```

1. **Accelerometer: does the skull shake?** Your own voice vibrates your skull. Another person's voice reaches you only through the air, and air barely moves a solid head. Voice pitch is about 85–255 Hz. At 1,600 readings a second the sensor sees everything up to 800 Hz, so it catches the pitch and its first harmonics. Head movement (nodding, turning) stays under about 20 Hz, well below this band.
2. **Microphone: how close is the mouth?** The wearer's mouth is a few centimetres from the mic. Their speech is louder and has less room echo than a person 1 m or more away.
3. **Fusion.** Each sensor covers the other's failures. The accelerometer misses whispers and is fooled by chewing and walking. The microphone is fooled by a loud bystander.
4. **Output contract.** Every frame gets `{t, p_wearer, p_speech}`. This contract is frozen so later stages can build on it.

## Privacy rule: the wearer's speech only

- Audio outside the wearer's own speech is discarded in memory. It is never written to disk and never transcribed.
- There is no full-conversation mode.
- Why: some places, for example Washington State, require every party's consent to record a private conversation. A wearer-only gate keeps bystanders out of the recording by design.
- The cost: the assistant hears only one side. If a friend says "send it Friday" and the wearer says "sure", the commitment is lost unless the wearer restates it ("OK, I'll send it Friday").

## Targets (pre-registered)

These are goals, not results. They will be scored once, on data the model never saw, after the evaluation protocol is locked with a git tag.

| Metric | Target | Test set |
|---|---|---|
| Wearer recall: share of the wearer's speech that is kept | ≥ 98% | held-out room |
| Background accept: share of other people's utterances that are kept | ≤ 1% | held-out room |

Also reported: onset delay (ms), false wakes per hour, µs per frame, RAM and flash (KB), current draw (mA).

## Evaluation plan

- **Leave one wearer out:** train on all wearers but one, test on that one.
- **Held-out room:** one room is never used for training.
- **Distance shift:** train with bystanders at 1–2 m, test at 0.5 m, the hardest case.
- **Conditions recorded:** wearer talking; bystander at 0.5, 1 and 2 m; TV or podcast; wearer silent while chewing or walking.
- **Baselines:** energy gate, WebRTC VAD, Silero VAD, accelerometer-energy threshold.
- **Public check set:** Vibravox (body-conducted speech dataset on Hugging Face).

## Hardware

- Arduino Nano 33 BLE Sense Rev2: PDM microphone, BMI270 accelerometer (up to 1.6 kHz).
- Main placement: glasses temple. Second placement: chest pendant. Not the wrist, which is too far from the skull.
- Version 1 is tethered to a computer over USB.

## Roadmap

**Phase 1: own-voice detector**
- [ ] Go/no-go: does the 100–300 Hz accelerometer band light up only for the wearer's voice?
- [ ] Logger firmware: accelerometer at 1.6 kHz and audio at 16 kHz, synced over USB
- [ ] Recording sessions with 4–5 consenting participants, with labels
- [ ] Baselines and model training
- [ ] INT8 model running on the board, with timing and current measured
- [ ] Locked held-out evaluation and results

**Phase 2: assistant context**
- [ ] Keep only the wearer's segments and transcribe them locally (whisper.cpp)
- [ ] Extract commitments, facts, decisions and open questions into a personal assistant

**Phase 3: personal knowledge graph**
- [ ] Turn each day's statements into linked notes: people, topics, open loops

## Data and consent

Every recording is made with participants who agreed to be recorded. Bystander speech in the test set comes from consenting friends or public audiobook playback (LibriSpeech).

## Status

Started November 2026. This repo fills in as each step lands: firmware, data protocol, then results.
