# Slice and Ship: what default part size should a multipart uploader use?

**Live demo:** https://shriyashekatkar25-byte.github.io/slice-and-ship/
**Gallery:** [APM Lab](https://shriyashekatkar25-byte.github.io/apm-lab/)

An APM Lab decision prototype by **Shriya Shekatkar** (UCLA, Statistics & Data Science minor) · UCLA Stats 102C Monte Carlo Methods.

## The launch that inspired it

On **October 23, 2026**, Databricks released **Files API 3.0 for Unity Catalog volumes** in Beta. It adds multipart uploads for files over 5 GB, plus version-conditional operations. The Files API docs say a single request can upload up to 5000 MiB, and larger files need a multipart upload. Reads, writes and deletes can also be made conditional on a file's `ETag` or `Last-Modified` value.

- Release note: [Databricks platform release notes, October 2026](https://docs.databricks.com/aws/en/release-notes/product/2026/october)
- Docs: [Files API for Unity Catalog volumes](https://docs.databricks.com/aws/en/volumes/files-api)

## The PM decision

**What default part size should the SDKs and CLI use for multipart uploads?**

Small parts mean thousands of requests and per-part overhead. Big parts mean each network error throws away more bytes and raises the chance the whole upload fails. The per-part failure rate is known only from a short Beta, so the prototype treats it as uncertain.

- **Success metric:** p90 time to upload an 80 GB file, taken across the posterior
- **Guardrails:** whole-upload failure ≤ 0.1% (p90) · bytes re-sent ≤ 3% · client memory (part × streams) ≤ 2 GiB · parts per file ≤ 10,000
- **Stakeholders:** ML and data engineers, the SDK/CLI team, storage and Files API engineering, Unity Catalog governance, support

With the default (simulated) inputs, the fastest part size that passes every guardrail is **256 MiB**. Sizes of 512 MiB and up break the memory guardrail with 8 streams, and very small parts lose time to request overhead. The memo also compares the Bayesian call to a plug-in analysis that uses only the raw Beta rate, and it flags when the two disagree.

## What the prototype shows about product judgment

| Databricks APM skill | Where it shows up |
|---|---|
| Customer problem framing | An 80 GB checkpoint upload dying at 97%, turned into one decision with one metric and four guardrails |
| Prototyping | Live posterior chart, an animated bookshelf where each spine is one part, a part-size sweep and KPI badges |
| SQL / Python | Illustrative SQL that pulls Beta evidence per client type, plus a NumPy posterior Monte Carlo |
| Data-driven decisions | Uncertainty about the failure rate flows into p90 time and failure risk instead of being hidden by a point estimate |
| Clear writing | A director memo and a one-sentence VP summary, both regenerated from the simulation |

## Methods

| Piece | Course method (Stats 102C) | How it's used |
|---|---|---|
| Failure rate q per 256 MiB | Ch2: Beta prior + Binomial data → Beta posterior (conjugate) | Prior from past launches, evidence from the Beta |
| Prior vs data | Ch2: posterior mean as a weighted average; flat prior gives MAP = MLE | Live weight bar in the math section |
| Uncertainty | Ch2: 95% quantile credible interval (vs HPD) | Yellow band on the posterior chart |
| Beta draws | Ch5: Box-Muller normals → gamma → beta | 4,000 posterior draws, seeded |
| Decision metrics | Ch1: Monte Carlo over posterior draws (p50/p90 from samples) | Upload time, failure risk and re-sent bytes for 9 part sizes |

## Design baseline

Four pins from Shriya's Pinterest home feed, each redrawn from scratch in CSS/SVG/canvas (no pin images are used, embedded or traced):

- **Hero** ← [Pick a colour, get a Bunny! (Café Bun Studio)](https://www.pinterest.com/pin/905082856390028117/): coral, apricot, teal, violet and bubblegum paint splats trailing into long brush strokes on a cream bokeh background, with a rainbow-gradient rounded title.
- **Cards & buttons** ← [Where flowers dance (Tinyghetto)](https://www.pinterest.com/pin/1109785533256726799/): a hand-poke flash sheet, with black ink ornament corners, clover glyphs and cards on graph paper.
- **KPI badges** ← [Job market so bad towel (alessandra)](https://www.pinterest.com/pin/640285272070958471/): lilac terry-cloth texture with red stitched slab-serif numbers and a dashed stitch border.
- **Simulation room** ← [Susan Stewart](https://www.pinterest.com/pin/184295809740008177/): a black shelf with sky-blue, ink-outlined book spines and halftone dots (bookshelf only, no character).

---

Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.
