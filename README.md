# Inferring missingness and underrepresentation in black-box model knowledge 

Testing whether model transcription failure tracks how much comparable
material exists in the digital record.

The premise: you can't know what a model was trained on, but for some
material you can document what it almost certainly wasn't trained on —
a manuscript never digitized, never edited, or never transcribed online. If
error rate rises as documented coverage falls, coverage predicts failure.

Currently: call vision models on manuscript
page images, save the output, score it. 

## Status

- [x] Single model call on a page image
- [ ] Save structured output per call
- [ ] Multiple models, multiple runs
- [ ] CER / WER scoring
- [ ] Hand analysis of errors
- [ ] Coverage-gradient corpus

## Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Put your key in `.env` (gitignored):

```
ANTHROPIC_API_KEY=sk-ant-...
```

Loaded via `python-dotenv`, so it works regardless of how the script is
launched — no need to `source` anything.

## Running

```bash
python letsgo.py
```

## Data

Page images in `boh_images/`. Six pages of a Book of Hours containing
family birth records in a private 16th-century French hand — marriage in
October 1573, two children born 1575 and 1576, godparents named.

Worth noting: this is not the formulaic Latin liturgy that makes up most
of a Book of Hours. Private annotation by a non-professional hand is
idiosyncratic and thinly represented, which makes it a more useful test
case than the devotional text around it. The Regenstein Library's copy of
this text was owned by a family and passed down through generations, whose
owners recorded the births of children, marriages, and other family events
in it. ?Those annotations exist in a single copy, with no published edition
and no prior digitization? 

Images are resized to a fixed maximum dimension before encoding — the API
rejects base64 payloads over 10 MB, and a full-resolution page photo
exceeds that. Every page must use identical resize settings, or error
rates aren't comparable across pages. Record the setting used.

```bash
for f in boh_images/*.jpg; do
  sips -Z 2000 "$f" --out "boh_images_resized/$(basename "$f")"
done
```

Don't downscale aggressively. Abbreviation marks and ambiguous letterforms
are exactly what's being tested; resizing too small would result in measuring lost pixels
rather than model knowledge.

## Ground truth

The hard part. Editions get published; diplomatic transcriptions don't. An
edition is an editorial reconstruction — normalized spelling, expanded
abbreviations, emendations, modern punctuation — none of which a model can
be expected to reproduce. Scoring against one measures the gap between
manuscript and edition as much as model error.

Two workarounds while real transcriptions don't exist:

**Inter-model agreement.** Run several models on one page. Consensus is
probably correct; divergence marks the genuinely hard spots. Needs no
ground truth and identifies precisely where an expert's attention is worth
asking for.

**Draft-then-correct.** Produce a provisional transcription, flag uncertain
readings, ask the expert to adjudicate a short list rather than transcribe
from scratch. Any provisional ground truth must be labelled as such in the
file, and no reported number should derive from it.

## Prompt design

TODO 

## Metrics

**Primary:** character error rate against diplomatic transcription.

**Secondary:** word error rate; longest run of correct characters (separates
scattered failure from localized failure); chrF for text with no
standardized orthography.

Not BLEU or ROUGE — those exist because translation has many valid outputs.
Transcription has one correct reading, so CER is both more interpretable and
what the HTR field reports.

**Variance:** three runs per page per model at temperature 0. Output varies
between calls; a single-pass number isn't defensible.

**Normalization** is the biggest single lever on the numbers, bigger than
model choice. Decide what gets stripped — case, punctuation, editorial
brackets, u/v and i/j folding — name the preset, and report it with every
result. Note that folding u/v and i/j hides letterform confusion that may
itself be part of what's being measured.

## Planned

- A specialist HTR baseline (Transkribus). These are trained on
  historical hands and are what the field actually uses. Without one, the
  obvious objection is that general-purpose models were never meant for
  this. It's the control, not an unfair comparison.
- Line-level alignment. Page-level CER hides *where* failures happen.
- The coverage gradient itself — see `gap-eval-design.md`.
- Remediation arm: supply corrected examples or an abbreviation glossary
  from the same hand, re-run, measure the delta. Detection alone stops at
  auditing.

## Scope limit

This measures capability failure, not bias or harm. Nobody is harmed by a
misread 1576 birth record. Bias requires differential error across groups
plus a consequential decision, and this experiment has neither.

What it can establish is the mechanism — coverage predicts error — in a
domain where coverage can actually be documented. Whether that mechanism
generalizes to consequential domains is an argument, not a result here.