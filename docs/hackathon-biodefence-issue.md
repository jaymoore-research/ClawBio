# ClawBio for BIO × AI Hack 2026: AminoAnalytica Biodefence by Design Track

> **GitHub Issue**: Copy this content to create an issue at `jaymoore-research/ClawBio`
> **Labels**: `enhancement`, `hackathon`

---

## Context

The [BIO × AI Hack 2026](https://dorahacks.io/hackathon/1985/detail) (UK AI Agent Hackathon EP4 x OpenClaw) includes the **AminoAnalytica Track: Biodefence by Design** — a track focused on building concrete computational solutions to prepare for, detect, or respond to biological threats using AI-driven protein engineering.

ClawBio's agent-driven, skill-based architecture is a natural fit: the hackathon **explicitly encourages Claude Code/OpenClaw agent-driven workflows**, and ClawBio already has the orchestrator + skill pattern that maps perfectly to multi-step protein design pipelines.

## Hackathon Track Requirements

**Goal**: Choose a specific biological target (e.g. H5N1 HA, Nipah G, Ebola GP, SARS-CoV-2 spike, toxins, prion-like proteins) and demonstrate real computational results — not just a general pipeline.

### Three Difficulty Directions

| Direction | Difficulty | Description |
|-----------|-----------|-------------|
| **Binder Design** | Easy | Use RFdiffusion + ProteinMPNN; validate with Boltz-2; score with ipSAE; deliver sequences + predicted affinities |
| **Broad-Spectrum** | Medium | Target conserved epitopes across variants; validate cross-variant binding |
| **Biosecurity Screening** | Hard | Build structural, motif, or anomaly detection systems; report detection and false positive rates |

### Tools & Resources Provided

- **Amina platform** (AminoAnalytica): 50+ protein engineering tools with remote GPU via CLI, including RFdiffusion, ProteinMPNN, Boltz, ESM, Protenix
- **LLM credits** provided to participants
- Agent-driven workflows (Claude Code / OpenClaw) **explicitly encouraged**

### Judging Criteria

Impact · Rigour · Execution · Creativity · Presentation

### Deliverables

- ≤ 4-page write-up
- Code (public GitHub repo with README, setup, architecture)
- Demo (5–10 min pitch/demo video)
- Designed outputs (binder sequences, structures, scores)
- Optional pitch deck

### Bounty Submission Requirements

- 5–10 min pitch/demo video covering: problem, solution, technical implementation, bounty integration, live demo
- Public GitHub repo with clear README (project overview, setup instructions, architecture, bounty-specific integration)
- Optional but preferred: pitch deck
- All links must be accessible before submitting

## Gap Analysis: ClawBio Today vs. What's Needed

### What ClawBio Already Has ✅

- **Bio Orchestrator** — multi-skill routing and chaining (ideal for multi-step design pipelines)
- **struct-predictor SKILL.md** — planned AlphaFold/Boltz prediction + RMSD comparison (specification exists, implementation pending)
- **Reproducibility framework** — `commands.sh`, `environment.yml`, checksums, audit logs (directly maps to "Rigour" criterion)
- **Report generation pattern** — all skills produce markdown reports + figures (maps to write-up deliverable)
- **Agent-driven CLI pattern** — `python skill.py --input X --output Y` (maps to demo deliverable)
- **Local-first privacy model** — no data upload without consent (good for biosecurity context)
- **17 existing skills** demonstrating the modular, composable architecture

### What's Missing ❌

- **No protein design pipeline** — no RFdiffusion, no ProteinMPNN, no de novo binder generation
- **No ipSAE/ipTM scoring** — no interface quality metrics for binder validation
- **No epitope analysis** — no conservation mapping, no surface accessibility
- **No biosecurity screening** — no threat motif detection, no anomaly detection
- **No Amina platform integration** — no bridge to remote GPU compute
- **struct-predictor not implemented** — only SKILL.md exists, no Python code
- **No sequence retrieval for threat targets** — no automated PDB/UniProt fetch for target proteins

## Proposed New Skills

### 1. `binder-designer` — Priority: Critical (enables Easy track)

**Purpose**: End-to-end de novo protein binder design against a specified target.

**Workflow**:

1. **Target acquisition**: Fetch target structure from PDB (e.g., Nipah G glycoprotein, `7TXZ`) or accept user PDB
2. **Hotspot selection**: Identify binding interface residues on the target
3. **Backbone generation**: Run RFdiffusion to generate binder backbones against target hotspots
4. **Sequence design**: Run ProteinMPNN to assign sequences to generated backbones
5. **Structure validation**: Predict binder-target complex with Boltz-2
6. **Scoring & filtering**: Compute ipSAE (threshold > 0.6), pLDDT, interface SASA, shape complementarity
7. **Report**: Ranked binder candidates with sequences, predicted structures, scores, and confidence plots

**Key metrics to implement**:

- **ipSAE** (interaction prediction Score from Aligned Errors) — replaces ipTM for binder triage; threshold > 0.6 for likely binders, > 0.8 for high confidence. Validated across 3,766 binders with 1.4x average precision improvement over ipAE ([Dunbrack 2025](https://www.biorxiv.org/content/10.1101/2025.02.10.637595v2.full), [Passaro et al. 2025](https://www.biorxiv.org/content/10.1101/2025.08.14.670059v1.full))
- **pLDDT** per-residue confidence
- **Interface buried surface area** (ΔiSASA)
- **Boltz-2 ipTM** for comparison

**Reference pipelines**:

- [ProteinDJ](https://www.biorxiv.org/content/10.1101/2025.09.24.678028v2.full) — modular RFdiffusion → MPNN → Boltz-2 pipeline on HPC with Nextflow
- [BinderFlow](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1013747) — automated binder design with ipSAE/CUTRE scoring

**Estimated effort**: 1–2 days (script + Amina CLI integration)

---

### 2. `epitope-mapper` — Priority: High (enables Medium track)

**Purpose**: Identify conserved epitopes across pathogen variants for broad-spectrum binder targeting.

**Workflow**:

1. **Variant collection**: Fetch all known variant sequences for target protein (e.g., SARS-CoV-2 spike across VOCs, or Henipavirus G across NiV/HeV/CedV)
2. **MSA construction**: Align variants with MAFFT or Clustal Omega
3. **Conservation scoring**: Per-residue Shannon entropy, ConSurf-style analysis
4. **Surface mapping**: Map conservation scores onto 3D structure, identify surface-exposed conserved patches
5. **Epitope ranking**: Score candidate epitopes by conservation × accessibility × druggability
6. **Cross-validation**: Feed top epitopes to binder-designer, validate binding across variant structures

**Enables**: "Broad-spectrum" direction — design binders against conserved regions that won't escape via mutation.

**Estimated effort**: 1 day

---

### 3. `threat-screener` — Priority: High (enables Hard track)

**Purpose**: Structural and sequence-based screening to detect biological threat agents.

**Workflow**:

1. **Structural similarity search**: DALI/Foldseek against curated threat protein database
2. **Motif detection**: Scan for known functional motifs (fusion peptides, receptor binding domains, pore-forming domains, toxin active sites)
3. **Anomaly detection**: Flag sequences with unusual codon usage, synthetic signatures, or chimeric domain architecture
4. **Reporting**: Detection confidence, false positive rate estimation, threat classification

**Key databases**: PDB, UniProt, NCBI Taxonomy, Select Agent list, Australia Group list

**Estimated effort**: 2–3 days (most complex skill)

---

### 4. `amina-bridge` — Priority: Critical (enables GPU compute)

**Purpose**: Integration layer between ClawBio and AminoAnalytica's Amina platform for remote GPU execution.

**Capabilities**:

- Submit jobs to Amina (RFdiffusion, ProteinMPNN, Boltz-2, ESMFold, etc.)
- Monitor job status and retrieve results
- Parse Amina outputs into ClawBio's standard formats
- Handle authentication and compute cost tracking

**Why separate skill**: Keeps GPU integration modular. Any skill can call `amina-bridge` for compute without reimplementing the API layer.

**Estimated effort**: 0.5–1 day (depends on Amina CLI documentation)

---

### 5. Implement `struct-predictor` — Priority: Critical (foundational dependency)

**Purpose**: SKILL.md already exists. Needs Python implementation for Boltz-2 prediction, PDB retrieval, RMSD/TM-score comparison, pLDDT/PAE visualization.

**This is a dependency for**: binder-designer (validation step), epitope-mapper (structure mapping), threat-screener (structural similarity).

**Estimated effort**: 1 day

---

### 6. Update `bio-orchestrator` — Priority: Medium (workflow automation)

**Purpose**: Add routing entries for new skills and support multi-step binder design workflows.

**New routes**:

- "design a binder" / "binder" / PDB input → `binder-designer`
- "conserved epitopes" / "broad spectrum" / "cross-variant" → `epitope-mapper`
- "screen" / "threat" / "biosecurity" / "detect" → `threat-screener`
- Chained workflow: epitope-mapper → binder-designer → struct-predictor (full broad-spectrum pipeline)

**Estimated effort**: 0.5 day

## Suggested Hackathon Strategy

### Recommended Target: Nipah Virus Glycoprotein G (NiV-G)

**Why Nipah**:

- BSL-4 pathogen with 40–75% case fatality rate — high **Impact** score
- Active community benchmark: [Adaptyv Bio Nipah Competition](https://www.adaptyvbio.com/blog/nipah-submissions/) provides validation context and comparables
- Well-characterized structure: PDB `7TXZ` (NiV-G bound to ephrin-B2 receptor)
- Limited therapeutic options — no approved antivirals or vaccines
- Clear binder design target: block ephrin-B2 receptor binding
- Cross-variant validation possible: NiV-Malaysia, NiV-Bangladesh, HeV, CedV

### Recommended Approach: Start Easy, Layer Up

| Day | Focus | Deliverable |
|-----|-------|-------------|
| 1–2 | Implement `amina-bridge` + `struct-predictor` + `binder-designer` core | Working pipeline |
| 2–3 | Run binder design against NiV-G, generate candidate sequences, score with ipSAE | Ranked binder candidates |
| 3–4 | Add `epitope-mapper` for conservation analysis across Henipavirus variants | Cross-variant validation |
| 4–5 | Write-up (≤ 4 pages), demo video (5–10 min), polish README and architecture docs | All deliverables |

This strategy delivers the **Easy** track (binder design with real sequences + scores) while demonstrating **Medium** track ambition (cross-variant conservation), maximizing score across all judging criteria.

### Demo Narrative

> "We asked ClawBio: design a binder against Nipah virus. The agent automatically fetched the target structure, ran RFdiffusion for backbone generation, designed sequences with ProteinMPNN, validated with Boltz-2, scored with ipSAE, and produced a ranked report — all orchestrated through natural language with full reproducibility."

## Alignment with Judging Criteria

| Criterion | How ClawBio Addresses It |
|-----------|-------------------------|
| **Impact** | Nipah = BSL-4, 40-75% CFR, no approved therapeutics. Designed binders could seed real countermeasure development. |
| **Rigour** | ipSAE scoring (validated across 3,766 binders in meta-analysis), reproducibility bundles (commands.sh + environment.yml + checksums), audit logs, full methodology traceability via SKILL.md |
| **Execution** | Agent-driven end-to-end pipeline via bio-orchestrator, real sequences + structures + scores as deliverables, Amina GPU integration |
| **Creativity** | Natural language → protein binder design via multi-skill orchestration; modular skill architecture reusable beyond this competition; ClawBio as the first bioinformatics agent framework for protein design |
| **Presentation** | ClawBio's built-in report generation produces publication-ready markdown + figures; demo shows conversational agent driving the entire pipeline |

## Deliverables Mapping

| Hackathon Deliverable | ClawBio Output |
|----------------------|----------------|
| ≤ 4-page write-up | Auto-generated `report.md` from binder-designer + epitope-mapper, edited for submission |
| Code | Public GitHub repo (ClawBio + new skills), clear README with architecture |
| Demo video (5–10 min) | Screen recording of agent-driven workflow: query → binder design → results |
| Designed outputs | Binder FASTA sequences, PDB structures, ipSAE score table, confidence plots |
| Pitch deck | Architecture diagram showing skill composition + results summary |

## Implementation Checklist

- [ ] Create `skills/amina-bridge/` — Amina platform CLI integration
- [ ] Implement `skills/struct-predictor/struct_predictor.py` — Boltz-2 prediction, PDB fetch, RMSD
- [ ] Create `skills/binder-designer/` — RFdiffusion → ProteinMPNN → Boltz-2 → ipSAE pipeline
- [ ] Create `skills/epitope-mapper/` — MSA conservation → surface mapping → epitope ranking
- [ ] Create `skills/threat-screener/` — structural/motif/anomaly detection (stretch goal)
- [ ] Update `skills/bio-orchestrator/orchestrator.py` — add new skill routes
- [ ] Update `CLAUDE.md` — add new skills to routing table
- [ ] Add demo data — NiV-G PDB (`7TXZ`), Henipavirus variant sequences, example outputs
- [ ] Write hackathon submission README (project overview, setup, architecture, bounty integration)
- [ ] Record 5–10 min demo video
- [ ] Write ≤ 4-page report
- [ ] Create pitch deck

## Key Technical References

- [RFdiffusion — Watson et al. 2023, Nature](https://www.nature.com/articles/s41586-023-06415-8) — De novo protein structure design via diffusion
- [ProteinMPNN — Dauparas et al. 2022](https://www.science.org/doi/10.1126/science.add2187) — Sequence design for protein backbones
- [ipSAE — Dunbrack 2025](https://www.biorxiv.org/content/10.1101/2025.02.10.637595v2.full) — Fixing AlphaFold's interface scoring
- [ipSAE meta-analysis — Passaro et al. 2025](https://www.biorxiv.org/content/10.1101/2025.08.14.670059v1.full) — 3,766 binder validation
- [ProteinDJ pipeline](https://www.biorxiv.org/content/10.1101/2025.09.24.678028v2.full) — Modular HPC binder design
- [BinderFlow pipeline](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1013747) — Automated design with ipSAE/CUTRE
- [Adaptyv Nipah Competition](https://www.adaptyvbio.com/blog/nipah-submissions/) — Community benchmark for NiV-G binders
- [AminoAnalytica Platform](https://www.aminoanalytica.com/) — 50+ protein engineering tools with remote GPU
- [Boltz-2](https://github.com/jwohlwend/boltz) — Fast structure prediction for binder validation
- [Levitate Bio — ipSAE explainer](https://levitate.bio/fixing-the-flaws-in-alphafolds-interface-scoring-meet-dunbracks-ipsae/) — Practical guide to ipSAE usage
