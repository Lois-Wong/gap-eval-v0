# Inferring missingness and underrepresentation in black-box model knowledge 

Testing whether model transcription failure tracks how much comparable
material exists in the digital record.

## Hypothesis
Underrepresentation or gaps in training data causes model error (and bias -> harm but this connection is blocked for later). 

## The premise
Yyou can't know what a model was trained on, but for some
material, you can posit it as something a model almost certainly wasn't trained on —
a manuscript never digitized, never edited, transcribed, or put online. If
error rate rises as documented coverage falls, coverage can be predictive of failure.

## This project
Ask models to transcribe manuscripts, save the output, and score it. 

## Status

- [x] Single model call on a page image
- [ ] Save structured output per call
- [ ] Multiple models, multiple runs
- [ ] CER / WER scoring
- [ ] Hand analysis of errors
- [ ] Coverage-gradient corpus



## Data

Page images in `boh_images/`. Six pages of a Book of Hours containing
family birth records in a private 16th-century French hand — marriage in
October 1573, two children born 1575 and 1576, godparents named.

This is not the formulaic Latin liturgy that makes up most
of a Book of Hours. Private annotation by a non-professional hand is
idiosyncratic and sparsely represented, which makes it a more useful test
case than the devotional text around it. The University of Chicago Regenstein Library's copy of
this text was owned by a family and passed down through generations, whose
owners recorded the births of children, marriages, and other family events
in it. ??Confirm: Those annotations exist in a single copy, with no published edition
and no prior digitization??

Images are resized to a fixed maximum dimension before encoding — the API
rejects base64 payloads over 10 MB, and a full-resolution page photo
exceeds that. Every page must use identical resize settings, or error
rates aren't comparable across pages. Record the setting used.

## Ground truth

Editions get published; diplomatic transcriptions don't. An
edition is an editorial reconstruction — normalized spelling, expanded
abbreviations, emendations, modern punctuation — none of which a model can
be expected to reproduce. Scoring against one measures the gap between
manuscript and edition as much as model error. (check this too)

Two workarounds while real transcriptions don't exist:

**Inter-model agreement.** Run several models on one page. Consensus is
probably correct; divergence marks the genuinely hard spots. Needs no
ground truth and identifies precisely where an expert's attention is worth
asking for. Inspired by inter-annotator agreement metrics. 

**Draft-then-correct.** Produce a provisional transcription, flag uncertain
readings, ask a subject matter expert to adjudicate a short list rather than transcribe
from scratch (prioritize this farther down the line). 

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

## Scope limit

Establish that/how coverage predicts error, in a
domain where coverage can actually be documented. 

Remediation: TBD, but supply corrected examples, RLHF, etc., 