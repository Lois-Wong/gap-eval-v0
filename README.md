# Inferring missingness and underrepresentation in black-box model knowledge (POC)

Identifying gaps in data/model knowledge via prompting / other black box probing techniques and developing targeted remediation

## Contents

- [Hypothesis](#hypothesis)
- [Why this design](#why-this-design)
- [Structure](#structure)
- [Data](#data)
- [Ground truth](#ground-truth)
- [Prompt design](#prompt-design)
- [Metrics](#metrics)
- [Confounding Variables](#confounding-variables-to-control)
- [Scope](#scope)
- [Part 2: remediation](#part-2-remediation)
- [This project](#this-project)

## Hypothesis
Underrepresentation in training data causes model error. Error rate can be predicted by how much comparable material exists in the digital record.

Underlying idea: Gaps in training data causes model error. Some of that
error is bias, and bias produces harm, but that link is deferred. See [Scope](#scope).

## Why this design 

The standing objection to all black-box work on training data is that you can't know what a model saw or was trained on. For some material, though, you can make a
strong documented argument about what it almost certainly *didn't* see: a
manuscript never digitized, never transcribed online, and held in a single
library.

That inverts the problem. Instead of guessing at presence, document absence. Our institutional position - UChicago Library special collections - makes it possible with access to holdings, provenance, and digitization records. 

## Structure

Three tiers, ranked by likely representation in training data.

**Tier 1 — almost certainly seen.** Canonical texts with published critical
editions, digitized decades ago, transcriptions online. Expect low error.
The ceiling.

**Tier 2 — plausibly seen.** Material in digitized collections with some
scholarly presence: catalogued, perhaps partially transcribed, findable but
not canonical.

**Tier 3 — almost certainly not seen.** Unpublished, undigitized special
collections holdings. No edition, no online transcription, no prior
digitization. Expect high error. The floor.

**Prediction:** error rate increases monotonically across tiers. If it does,
coverage predicts failure. If it doesn't, the thesis is wrong in an
interesting way, and that's also worth knowing.


## Data

Note: Images are not distributed in this repo.

A Book of Hours from the University of Chicago's special collections containing
family records in a private 16th-century French hand — a marriage in
October 1573, two children born 1575 and 1576, etc..

This is not the formulaic Latin liturgy that makes up most
of a Book of Hours. Private annotation by a non-professional hand is
idiosyncratic and sparsely represented, which makes it a more useful test
case than the devotional text around it. UChicago's copy was owned by a family and passed down
through generations, whose owners recorded births, marriages, and other
family events in it.


> **TODO — confirm:** that these annotations exist in a single copy, with no
> published edition and no prior digitization.

Images are resized to a fixed maximum dimension before encoding. The API
rejects base64 payloads over 10 MB, and a full-resolution page photo
exceeds that. Every page uses identical resize settings, or error
rates aren't comparable across pages. 

> **TODO:** record the resize setting used.

## Ground truth

Editions get published; diplomatic transcriptions don't. An
edition is an editorial reconstruction — normalized spelling, expanded
abbreviations, emendations, modern punctuation. Scoring against one measures the gap between
manuscript and edition as much as model error. 

> **TODO — verify** this characterization with Emily before relying on it.

Workaround:

Inter-model agreement: Run several models on the same page. Consensus is probably right; divergence points are where the difficulty is. This needs no ground truth and identifies exactly the spots worth an expert's attention. Borrowed from inter-annotator agreement metrics. 


## Prompt design

> **TODO.** One shared prompt across all models — tuning per model breaks
> the comparison. Must explicitly forbid markdown, editorial annotation,
> abbreviation expansion, and commentary; left unconstrained, models return
> formatted editions with deletions marked. Store the exact prompt text
> with every output, since the prompt will change.

## Metrics

Primary: character error rate (CER) against diplomatic transcription. (Not against a published edition)

Secondary: word error rate; longest run of correct characters (separates scattered failure from localized failure); chrF as a more forgiving character n-gram measure for text with no standardized orthography.

Variance: three runs per page per model at temperature 0. Report standard deviation. Output varies between calls, so a single-pass number
isn't defensible. 

Normalization is the single biggest lever on the numbers, larger than
model choice. Decide what gets stripped — case, punctuation, editorial
brackets, u/v and i/j folding — name the preset, and report it with every
result. Folding u/v and i/j hides letterform confusion that may itself be
part of what's being measured.

Not BLEU or ROUGE — those exist because translation has many valid outputs.
Transcription has one correct reading, so CER is both more interpretable and
what the HTR field reports.



> **TODO:** hand-read the first 50 errors and build a taxonomy with Emily —
> abbreviation, letterform confusion, orthographic variant, hallucinated
> plausible word, omitted line. Clustering is the finding; scatter is the
> null.

## Confounding variables to control

Script and hand. Undigitized material may also be in harder hands. Hold script type and period roughly constant across tiers, or the experiment measures legibility rather than coverage. If you can't hold it constant, have a palaeographer rate difficulty independently and report it.

Physical condition. Damage, faded ink, and show-through are legibility problems, not coverage problems. Exclude badly degraded items or rate them.

Image quality: Same capture conditions, resolution, and compression for every page. Record the settings. 

Language and period. Differences here change the task, not just the coverage. Keep them as consistent as the material allows.


## Scope

TODO 

## Part 2 - Remediation 

Remediation: TBD, but supply corrected examples, few-shot prompting, fine-tuning, etc., 

Example: take a page with high error. Supply the model with corrected examples from the same hand, or a glossary of that scribe's abbreviations, or comparable transcribed material. Re-run and measure the delta.

If targeted supplementation closes a measurable part of the gap, the loop is complete — gap detected, gap patched, improvement quantified. That is a generalizable methodology.

Establish the relationship bewteen error and bias (which is harmful). 

## This project
letsgo.py - Ask models to transcribe manuscripts, save the output, and score it. 

### Status

- [x] Single model call on a page image
- [ ] Save structured output per call
- [ ] Multiple models, multiple runs
- [ ] CER / WER scoring
- [ ] Hand analysis of errors
- [ ] Coverage-gradient corpus