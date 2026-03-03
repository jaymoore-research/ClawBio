# ClawBio should be able to one-shot bioinformatics hackathons

## Title

**Feature: One-shot hackathon mode — ClawBio auto-generates competition-ready skills from a challenge spec**

---

## Summary

ClawBio's SKILL.md pattern + Bio Orchestrator already enable rapid skill creation (template → methodology → demo data → script in hours). We should formalize this into a **hackathon mode** that takes a competition brief as input and produces a complete, submission-ready skill — code, plots, reproducibility bundle, and all.

The proof-of-concept target: **the UK AI Agent Hackathon BioDock Track** (Imperial College CBS, $1,250 prize pool), which asks participants to:

1. **Baseline ($1,000):** Parse kidney tissue GeoJSON segmentation data, compute glomerulus-to-nearest-vessel distances, output `glomeruli_distances.csv` + a distribution plot.
2. **Generalization Bounty ($250):** Build an AI copilot that converts *any* natural-language spatial-biology analysis request into a runnable BioDock script + visual workflow diagram.

ClawBio is uniquely positioned to do both tasks because we already have the architecture — skills are specifications, the agent implements them, and reproducibility is baked in.

---

## Motivation

### Why hackathons matter for ClawBio

- **Battle-tests the SKILL.md pattern** under time pressure and against novel problem domains
- **Proves composability** — can the orchestrator chain a new spatial-biology skill with existing ones?
- **Grows the skill library** — every hackathon produces a new, reusable skill
- **Demonstrates "specification-first" AI** — teams write methodology, the agent writes code

### Why BioDock specifically

- **Spatial biology is a gap** — we have genomics, metagenomics, pharmacogenomics, but no tissue-level spatial analysis
- **GeoJSON is tractable** — it's structured JSON with coordinates; no GPU/imaging pipeline needed for the baseline
- **The generalization bounty is literally what ClawBio does** — convert natural-language queries into runnable bioinformatics scripts

---

## Proposed Implementation

### 1. New skill: `skills/spatial-tissue/`

**SKILL.md methodology would cover:**

| Section | Content |
|---|---|
| **Why this exists** | Spatial biology (Visium, MERFISH, BioDock segmentation) produces coordinate-rich data that encodes tissue architecture. Distance metrics between cell types reveal functional relationships (e.g., glomerulus–vessel proximity in kidney correlates with filtration efficiency). |
| **Input detection** | `.geojson` files containing segmentation polygons with `classification` properties (e.g., `"glomerulus"`, `"vessel"`, `"tubule"`) |
| **Core algorithm** | 1. Parse GeoJSON features → Shapely geometries 2. Group by classification 3. For each glomerulus centroid, compute min distance to nearest vessel boundary (Shapely `nearest_points`) 4. Output distance matrix as CSV 5. Generate distribution plot (histogram + KDE + summary stats) |
| **Output format** | `glomeruli_distances.csv` (columns: `glomerulus_id`, `nearest_vessel_id`, `distance_um`, `glomerulus_centroid_x`, `glomerulus_centroid_y`) + `distance_distribution.png` |
| **Quality thresholds** | Warn if <10 glomeruli detected; warn if no vessels found; report median, IQR, outliers (>3 IQR) |
| **Dependencies** | `geopandas`, `shapely`, `matplotlib`, `scipy` |

**Python script: `spatial_tissue.py`**
```bash
python skills/spatial-tissue/spatial_tissue.py \
  --input kidney_segmentation.geojson \
  --source-class glomerulus \
  --target-class vessel \
  --output /tmp/spatial_demo
```

### 2. Generalization layer: `skills/biodock-copilot/`

This skill would implement the **Generalization Bounty** — an AI copilot that:

1. Accepts a natural-language analysis request (e.g., "measure average distance between tumor cells and immune cells in this lung tissue")
2. Parses the GeoJSON to discover available object classes
3. Generates a runnable BioDock-compatible Python script
4. Produces a **visual workflow diagram** (Mermaid → SVG/PNG) showing the analysis pipeline
5. Executes the script and returns results

This is essentially what the Bio Orchestrator already does for genomics — extended to spatial biology.

### 3. Hackathon mode for the orchestrator

Add a `--hackathon` flag to the Bio Orchestrator that:

```bash
python skills/bio-orchestrator/orchestrator.py \
  --hackathon "path/to/challenge_brief.md" \
  --data "path/to/competition_data/" \
  --output /tmp/hackathon_submission
```

**Workflow:**
1. Parse the challenge brief (extract tasks, judging criteria, output requirements)
2. Match tasks to existing skills or generate new SKILL.md from the template
3. Run the skill pipeline against provided data
4. Package the submission: code, outputs, reproducibility bundle, workflow diagram
5. Generate a self-assessment against the judging rubric

---

## How this maps to ClawBio's existing architecture

| ClawBio Component | Hackathon Role |
|---|---|
| `templates/SKILL-TEMPLATE.md` | Scaffolds the new spatial-tissue skill in minutes |
| `skills/bio-orchestrator/` | Routes "measure distances between X and Y" queries to spatial-tissue |
| Reproducibility bundle (`commands.sh`, `environment.yml`, checksums) | Satisfies hackathon "code quality" and "robustness" criteria |
| SKILL.md methodology encoding | Prevents hallucinated thresholds — every distance metric, warning, and plot parameter is documented |
| Demo data pattern | Package sample GeoJSON for testing before competition data is available |
| `CLAUDE.md` skill routing table | Add `spatial-tissue` and `biodock-copilot` entries for automatic routing |

---

## Acceptance Criteria

- [ ] `skills/spatial-tissue/SKILL.md` — complete methodology for GeoJSON spatial analysis
- [ ] `skills/spatial-tissue/spatial_tissue.py` — reads GeoJSON, computes pairwise distances, outputs CSV + plot
- [ ] Demo GeoJSON data with synthetic kidney tissue (glomeruli + vessels)
- [ ] `skills/biodock-copilot/SKILL.md` — natural-language → BioDock script generation methodology
- [ ] Workflow diagram generation (Mermaid/Graphviz → PNG)
- [ ] Bio Orchestrator updated with `--hackathon` flag for challenge-brief-driven automation
- [ ] `CLAUDE.md` routing table updated with new skill entries
- [ ] End-to-end test: feed the BioDock challenge brief → get submission-ready outputs

---

## Stretch Goals

- [ ] **Multi-track support** — hackathon mode handles multiple challenge tracks in one run
- [ ] **Judging rubric self-scoring** — auto-evaluate submission against stated criteria
- [ ] **Time-boxed mode** — orchestrator prioritizes tasks by points/difficulty ratio given a time budget
- [ ] **Post-hackathon skill promotion** — after the event, promote the hackathon skill to a permanent ClawBio skill with full tests and docs

---

## Reference: BioDock Track Details

**Event:** UK AI Agent Hackathon — BioDock Track
**Sponsor:** Biodock Inc. | Imperial College CBS
**Prize Pool:** $1,250

**Baseline Task ($1,000 in BioDock credits):**
Write a Python script that reads kidney tissue segmentation data (GeoJSON), computes distances from each glomerulus to its nearest vessel, and outputs `glomeruli_distances.csv` plus a distribution plot. Judged on correctness, robustness, and code quality.

**Generalization Bounty ($250 cash):**
Build an AI copilot that converts any natural-language analysis request into a runnable BioDock script — adapting across tissue types, object classes, and measurement tasks. Must also produce a visual workflow diagram.

**Resources:**
- Track video brief: https://www.loom.com/share/b62c9d6a72e046f3b2eee6135464bcbc
- BioDock Script docs: https://docs.biodock.ai/script
- Data: https://app.biodock.ai/ (registration required)
- Contact: nurlybek@biodock.ai

---

## Labels

`enhancement`, `hackathon`, `new-skill`, `spatial-biology`
