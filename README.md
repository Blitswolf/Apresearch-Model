# apresearch

An offline **exploit-intelligence model** for device vulnerability research. Where an exact
`searchsploit` keyword grep returns nothing for an unusual device, apresearch reasons over the
*whole* local exploit database to generate its own research: it finds the **nearest analogues** by
similarity, predicts the most likely **vulnerability classes**, and deep-mines those neighbours'
exploit code to extract the concrete **endpoints / parameters / payloads** worth testing. Pure Python
standard library — no dependencies.

Built to run **continuously at low priority** so an otherwise-idle box (e.g. a capture appliance or a
spare Pi) puts its cores to productive research use.

## The model

- **Corpus** — the local exploit-db (`files_exploits.csv`), filtered to the relevant entry types.
- **Features** — word tokens **plus character 4-grams**. The n-grams give *fuzzy* matching: a device
  model string that never appears verbatim in any exploit title still matches similar hardware.
- **Vectors** — TF-IDF, cached to disk; a query is the device fingerprint terms; ranking is cosine
  similarity over the corpus.
- **Class prediction** — weighted vulnerability-class signatures (command-injection, auth-bypass,
  default-credentials, CSRF, XSS, path-traversal, buffer-overflow, info-disclosure, SSRF, …) scored
  over the nearest neighbours.
- **Deep pass** — reads the top-K neighbours' actual exploit source and extracts URL paths, parameter
  names and payload/technique markers into a prioritised, concrete **test plan**.

The output is a set of *hypotheses to test*, produced entirely offline — no target is contacted to
generate it.

## Usage

```bash
chmod +x apresearch.py            # Python 3, stdlib only

./apresearch.py build             # (re)build + cache the TF-IDF index over the exploit-db (CPU-heavy, once)
./apresearch.py predict netgear wireless router     # rank vuln classes for a fingerprint
./apresearch.py serve             # continuous low-priority loop over fingerprints dropped by a producer
```

Point it at your exploit-db location by editing the paths at the top of `apresearch.py`
(`EDB_DIR`, defaults to `/usr/share/exploitdb`).

### Modes
| mode | what it does |
|------|--------------|
| `build` | build + cache the index (run once; refreshes rarely) |
| `predict TERMS...` | one-shot: analogues + class predictions + mined test plan for the given terms |
| `serve` | continuous: re-run over the newest fingerprints; keeps idle cores working |

## As part of a pipeline

apresearch is the vulnerability-research counterpart to a capture/crack pipeline: a capture box
fingerprints a target (vendor, model, service banners, WPS/RSN data) and drops the fingerprint as
JSON; apresearch turns each fingerprint into a research plan. Because it is pure stdlib and
config-driven, it **relocates** to a larger machine unchanged and scales up (bigger corpus,
full-text index).

## Scope

Authorized security testing / research only. It produces *hypotheses* from public exploit data; it
does not attack anything. Use it only in support of testing systems you own or are permitted to test.
