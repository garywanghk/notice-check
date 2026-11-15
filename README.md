# notice-check

Check an English notice against its Traditional Chinese translation with the
frozen **V11** checker — a learned head over LaBSE + mDeBERTa-XNLI + NLLB-200
cross-checks, plus deterministic bilingual fact rules.

Two files do the work:

| file | role |
|---|---|
| `index.html` | the interface; static, deployable to Vercel, talks to the checker over `127.0.0.1` |
| `v11.py` | the checker itself: all ten frozen files embedded, hash-verified, one command to run |

The models are 5.4 GB, so they run on your machine, never on the host. The page
is therefore a front end with an honest offline state rather than a fake demo.

## Use it

```bash
python v11.py --setup        # one time: pinned deps + model download
python v11.py --serve 8765   # start the checker
open index.html              # or use the deployed page; it finds 127.0.0.1:8765 automatically
```

Other entry points:

```bash
python v11.py --verify                                    # hash check only, no models
python v11.py --gui                                       # serve the built-in page and open it
python v11.py --english "…" --chinese "…" [--json]
python v11.py --inputs cases.csv --output results.json     # case_id,english,chinese
python v11.py --extract ./frozen                          # plain copy of the ten frozen files
```

## Deploy the page

Static, no build step, no serverless functions:

```bash
npx vercel --prod
```

or import the repo at <https://vercel.com/new>. `vercel.json` pins the project to
static output with `buildCommand: null` and `installCommand: null`, serves
`v11.py` as a download, and sets basic security headers.

An `https://` page can still reach `http://127.0.0.1:8765` because loopback is a
potentially-trustworthy origin; the bridge answers Chrome's Private Network
Access preflight with `Access-Control-Allow-Private-Network: true`.

## What the verdicts mean

| status | meaning |
|---|---|
| `✅ No discrepancy found` | the learned score is below `0.785` and no rule found a conflicting fact |
| `⚠️ Check` | either the score is `>= 0.785`, or a bilingual rule found a specific conflict (unit, number, date, weekday, obligation, field value) |

`decision_path` tells you which: `v11_learned_equivalence` vs
`v11_unsupported_detail`. The score is a decision score, not a calibrated
probability of error.

## Requirements

Python 3.11 (manifest pins 3.11.14; 3.14+ is rejected), ~5.4 GB disk for models,
~2.4 GB RAM while checking. `torch==2.15.0`, `transformers==5.22.1`,
`sentence-transformers==5.7.0`, `sentencepiece==0.2.3`,
`opencc-python-reimplemented==0.1.7`, `pandas`, `scipy`, `numpy` — `--setup`
installs exactly those.

## Provenance

The embedded files are byte-exact against `frozen_manifest.json` (10/10 SHA-256
verified on every run, and again by `run.py --require-frozen` in batch mode).
Model revisions are pinned in the same manifest.
