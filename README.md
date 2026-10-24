# mv-character-assets

An agent skill for the two questions that decide whether an MV's art gets finished:

1. **How many images do I actually need?** Answer, measured: about **one asset per 3-4 shots**. A 70-shot MV used 17 poses (+6 alternates), 14 backgrounds and 1 prop. "One image per shot" and "one pose per cut point" are both wrong, because the engine re-frames, re-grades, re-lights and silhouettes the same image many times.
2. **How do I keep the face from drifting across a batch?** One locked master reference, plus a verbatim character sentence naming the **side** of every asymmetric detail, plus two takes of any unusual angle.

## The traps it exists for

| Trap | What you see | Fix |
|---|---|---|
| Green chroma key on a teal/mint design | the whole outfit turns grey | pick the key from the character's palette — magenta `#FF00FF` when the design contains teal, aqua, mint or green |
| Native model alpha tops out at 252-254 | the background ghosts through the body | rescale the body's 99.5th percentile to 255, zero alpha <= 3, bleed colour outward |
| Extreme close-up requested as a cutout | a floating face crop | close-ups that fill the frame must be **opaque, full-bleed 16:9** |
| "Lying down, seen from above" | comes back standing | two takes, and describe camera + body orientation explicitly |
| Master not locked before a batch | five slightly different girls | lock `REF_master.png` first; every pose uses it as a reference image |

## Sizing facts worth knowing

- Backgrounds at **1672x941** survive cover-zoom to about **1.45x** at 1080p under line boil and grain. Deeper push-ins need a dedicated close-up, not a crop.
- Throughput: **31-115 s per image** (character median ~65 s), 5-8 in parallel, ~45 images in 10-15 minutes.
- Never generate readable text into backgrounds — the model invents fake kana. Typesetting is the text layer's job.

## Licence

MIT. See `LICENSE`.
