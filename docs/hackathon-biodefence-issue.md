# ClawBio for BIO x AI Hack 2026: AminoAnalytica Biodefence by Design Track

> **GitHub Issue**: Copy this content to create an issue at `jaymoore-research/ClawBio`
> **Labels**: `enhancement`, `hackathon`

---

## Context

The **BIO x AI Hack 2026** at Imperial College London (1-8 March 2026) includes the **AminoAnalytica Track: Biodefence by Design** — building concrete computational solutions to prepare for, detect, or respond to biological threats using AI-driven protein engineering.

ClawBio's agent-driven, skill-based architecture is a direct match for this track. The brief **explicitly names Claude Code as the recommended agentic setup** and describes an example workflow ("use the Amina CLI to run RFdiffusion... pipe through ProteinMPNN... validate with Boltz-2... score by predicted binding affinity") that maps 1:1 to ClawBio's orchestrator + skill pattern. AminoAnalytica even provides a Claude Code skills package (`npx skills add AminoAnalytica/amina-skills --all`) to teach agents how to chain Amina tools.

**Key quote from the brief**: *"This is the frontier of autonomous scientific research. Real research groups are beginning to deploy exactly this kind of agent-driven workflow. Building one during a hackathon, and applying it to a genuine biosecurity challenge, is a serious achievement. We will be paying close attention to teams that integrate agentic approaches into their submissions."*

## Hackathon Details

| Field | Detail |
|-------|--------|
| **Dates** | 1-8 March 2026 |
| **Venue** | Imperial College London, South Kensington |
| **Format** | Teams of up to 5 |
| **Prizes** | 1st: GBP 125 + Amina credits, 2nd: GBP 75 + credits, 3rd: GBP 50 + credits |
| **Workshop** | Thursday 5 March, 6:00 PM |
| **Submission deadline** | Saturday 8 March |
| **Demo** | 3-5 minute presentation for judging day |
| **Sponsor** | AminoAnalytica (YC S24, Imperial spinout, ARIA partner) |

## Judging Criteria (Weighted)

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **Impact** | **30%** | Does this address a real biosecurity need? Projects that demonstrate results against a specific, named biological target score higher than general-purpose tools without a worked example. |
| **Scientific Rigour** | **25%** | Is the computational approach sound? Are results validated appropriately? Are limitations acknowledged? |
| **Technical Execution** | **20%** | How well-built is the solution? Does it work? Is the code or pipeline reproducible? |
| **Creativity** | **15%** | Is the approach novel or surprising? Does it combine tools or ideas in interesting ways? |
| **Presentation** | **10%** | Is the submission clearly communicated? Could someone outside your team understand and build on your work? |

## Track Directions (Full Detail from Brief)

### Easy: Binder Design (Diagnostics & Therapeutics)

Both rapid diagnostics and therapeutic countermeasures start from the same problem: designing a protein that binds a pathogen target with high affinity and specificity.

- **Diagnostics**: Binder that recognises any accessible region of a pathogen surface protein for lateral flow tests or biosensors. Current lateral flow tests rely on antibodies that take 6-12 months to develop with only ~6% of candidates passing screening. Miniproteins and computationally designed binders offer a radically faster alternative.
- **Therapeutics**: Binder must target a functionally important region and disrupt the pathogen's mechanism — blocking receptor binding, preventing membrane fusion, or neutralising a toxin.

**Deliverable**: Pick a specific target (H5N1 HA, Nipah G, Ebola GP, SARS-CoV-2 spike, etc.), design a panel of candidate binders. Use RFdiffusion + ProteinMPNN to generate designs, validate with Boltz-2, score with **IPSAE** (average ipSAE across the interface). Classify candidates as diagnostic vs. therapeutic based on binding site. Deliver actual designed sequences with predicted binding affinities.

### Medium: Broad-Spectrum Countermeasure Design

A binder against one strain may be useless against the next. The most valuable countermeasures target conserved structural features the pathogen cannot easily mutate away from.

**Deliverable**: Pick a rare or emerging disease family (Nipah, Marburg, Lassa, novel influenza subtype). Use structural alignment and conservation analysis to identify shared epitopes. Design binders with RFdiffusion + ProteinMPNN targeting conserved regions. Validate cross-reactivity by predicting binding affinity against multiple named variant structures with Boltz-2. Present a single designed protein (or small panel) with predicted affinities for each variant.

### Hard: Biosecurity Screening & Safeguards

AI-redesigned sequences can retain dangerous function while evading sequence-level DNA synthesis screening (demonstrated in Science, October 2025). The field needs computational filters that catch dangerous designs before they reach a laboratory.

Three specific approaches called out:
1. **Structural homology screening**: Predict fold of designed protein, compare against databases of known toxins, virulence factors, immune evasion proteins — catching threats that sequence-level tools miss
2. **Motif-level analysis**: Flag conserved dangerous features — receptor-binding domains, membrane-disrupting regions, protease cleavage sites, aggregation-prone sequences (prion-like), especially when designs show high structural homology to disease-causing pathogens despite low sequence identity
3. **Behavioural anomaly detection**: Classify usage patterns — distinguishing diagnostic binder design from systematic immune evasion optimisation

**Deliverable**: Pick a specific threat class (redesigned toxins, immune evasion proteins, prion-like sequences). Assemble a test set of known dangerous sequences + benign controls. Run through your screening tool. Present detection rates, false positive rates, and specific examples of what your approach catches that sequence-level screening misses.

### Novel Approaches (also welcome)

Enzyme engineering for decontamination, phage-based countermeasures, population-level modelling, synthetic biology circuit design for biosensors.

## Amina Platform Details (from Brief)

### Tool Categories

| Category | Tools |
|----------|-------|
| **Structure prediction** | Boltz-2, ESMFold, OpenFold3, Chai-1 |
| **Protein design** | RFdiffusion, ProteinMPNN, LigandMPNN |
| **Molecular docking** | DiffDock, AutoDock Vina |
| **Molecular dynamics** | OpenMM |
| **Property prediction** | Solubility, toxicity, aggregation, stability |
| **+ more** | 50+ total tools |

### CLI Setup

```bash
# Install
pip install amina-cli

# Authenticate (API key from https://app.aminoanalytica.com/settings/api)
amina auth set-key "ami_your_api_key"

# Explore tools
amina tools
amina tools --category design
amina tools --search dock

# Run a tool
amina run esmfold --sequence "MKFLIL..." -o ./results/

# Per-tool help
amina run <tool> --help

# Background jobs
amina run <tool> --background
amina jobs status <job_id>
amina jobs download <job_id> -o ./results/
```

### Agent Integration (Claude Code — recommended by brief)

```bash
# Install Amina skills for Claude Code
npx skills add AminoAnalytica/amina-skills --all

# Then inside Claude Code (or Cursor, Windsurf, any compatible agent):
/amina-init
```

The skills teach the agent how to discover tools, format commands, chain outputs, and handle errors. Then you work entirely in natural language:

> *"Use the Amina CLI to run RFdiffusion against PDB 7TYK chain A, generate 50 backbones, pipe them through ProteinMPNN for sequence design, then validate the top 10 with Boltz-2. Score by predicted binding affinity and present results."*
>
> — Example from the hackathon brief

### OpenClaw Integration (alternative/complement)

OpenClaw runs as a persistent background agent reachable via WhatsApp/Telegram/Slack. Useful for fire-and-forget long-running tasks. Can orchestrate Amina CLI the same way Claude Code does. **Combine both**: Claude Code for interactive sessions, OpenClaw for autonomous monitoring.

### Other Resources

- Public databases: PDB, UniProt, GISAID, NCBI GenBank
- Open-source tools: AlphaFold, ESM-2, ColabFold, PyMOL
- Z.ai providing LLM credits to all participants
- Any programming language, framework, or library

## Gap Analysis: ClawBio Today vs. What's Needed

### What ClawBio Already Has

| Capability | Hackathon Relevance |
|-----------|-------------------|
| **Bio Orchestrator** — multi-skill routing + chaining | Directly maps to agent-driven multi-step design pipelines (the demo narrative) |
| **struct-predictor SKILL.md** — AlphaFold/Boltz spec | Foundation for binder validation; needs implementation |
| **Reproducibility framework** — commands.sh, environment.yml, checksums, audit logs | Maps directly to Scientific Rigour (25%) and Technical Execution (20%) |
| **Report generation** — all skills produce markdown + figures | Maps to write-up deliverable and Presentation (10%) |
| **Agent-driven CLI pattern** — `python skill.py --input X --output Y` | Natural fit for Amina CLI wrapping |
| **OpenClaw skill format** — YAML frontmatter + markdown instructions | Aligns with `npx skills add AminoAnalytica/amina-skills` pattern |
| **17 existing skills** — proven modular architecture | Demonstrates Creativity (15%) — reusable framework, not a one-off |

### What's Missing

| Gap | Impact on Hackathon |
|-----|-------------------|
| **No protein design pipeline** (RFdiffusion, ProteinMPNN, binder generation) | Blocks Easy/Medium tracks entirely |
| **No IPSAE scoring** (average ipSAE across interface) | The brief's specified metric; required for binder ranking |
| **No Amina CLI integration** | The platform powering all GPU compute |
| **No epitope conservation analysis** | Blocks Medium track |
| **No biosecurity screening** (structural homology, motif, anomaly detection) | Blocks Hard track |
| **struct-predictor not implemented** | Boltz-2 validation is a dependency for every track |
| **No PDB/UniProt fetch** for threat targets | First step in any binder design workflow |

## Proposed New Skills

### 1. `amina-bridge` — Priority: CRITICAL (prerequisite for everything)

**Purpose**: Wrapper around the `amina-cli` that integrates Amina into ClawBio's skill framework.

**Why this first**: Every other skill depends on Amina for GPU compute. Without this bridge, nothing runs.

**Implementation**:

```python
# Core functions needed:
amina_run(tool, **kwargs)         # Execute: amina run <tool> <args> -o <output>
amina_run_background(tool, **kw)  # Execute with --background, return job_id
amina_job_status(job_id)          # Poll: amina jobs status <job_id>
amina_job_download(job_id, dest)  # Fetch: amina jobs download <job_id> -o <dest>
amina_tools_list(category=None)   # Discover: amina tools [--category <cat>]
```

**Also**: Install the official Amina skills package:
```bash
npx skills add AminoAnalytica/amina-skills --all
```

**Estimated effort**: 0.5 day

---

### 2. `struct-predictor` implementation — Priority: CRITICAL (foundational dependency)

**Purpose**: SKILL.md already exists. Implement Python for:
- PDB structure retrieval (fetch target by PDB ID)
- Boltz-2 prediction via `amina run boltz2`
- ESMFold fast prediction via `amina run esmfold`
- RMSD / TM-score comparison between structures
- pLDDT and PAE confidence visualisation

**Dependency for**: binder-designer (validation), epitope-mapper (structure mapping), threat-screener (fold comparison).

**Estimated effort**: 1 day

---

### 3. `binder-designer` — Priority: CRITICAL (the core hackathon deliverable)

**Purpose**: End-to-end de novo protein binder design against a specified pathogen target.

**Workflow** (maps directly to brief's example):

```
Target PDB → Hotspot Selection → RFdiffusion (N backbones)
           → ProteinMPNN (sequence design per backbone)
           → Boltz-2 (complex validation, top K)
           → IPSAE Scoring → Ranked Report
```

**Step-by-step**:

1. **Target acquisition**: Fetch from PDB or accept user file. Extract target chain.
2. **Hotspot selection**: Identify binding interface residues (receptor binding site for therapeutics, any accessible surface for diagnostics)
3. **Backbone generation**: `amina run rfdiffusion --target <pdb> --chain A --hotspots <res> --num_designs 50 -o ./backbones/`
4. **Sequence design**: `amina run proteinmpnn --input ./backbones/ --num_seqs 4 -o ./sequences/`
5. **Structure validation**: `amina run boltz2 --input ./sequences/top_designs.fasta --target <pdb> -o ./validation/`
6. **Scoring**: Compute **IPSAE** (average ipSAE across interface), pLDDT, ΔiSASA, shape complementarity
7. **Classification**: Label each candidate as diagnostic (surface epitope binder) or therapeutic (functional site blocker)
8. **Report**: Ranked table of candidates with sequences, IPSAE scores, pLDDT, binding site classification, confidence plots

**Key metric — IPSAE**:
- ipSAE = interaction prediction Score from Aligned Errors (Dunbrack 2025)
- IPSAE (hackathon metric) = average ipSAE across interface residues
- Threshold: > 0.6 likely binder, > 0.8 high confidence
- Validated: 1.4x average precision improvement over ipAE across 3,766 experimentally tested binders
- Boltz-2 ipSAE is faster than AF2 (8.6s vs 16.4s per prediction)

**Estimated effort**: 1-2 days

---

### 4. `epitope-mapper` — Priority: HIGH (enables Medium track)

**Purpose**: Identify conserved epitopes across pathogen variants for broad-spectrum targeting.

**Workflow**:

1. **Variant collection**: Fetch variant sequences from UniProt/GenBank for target protein family
2. **MSA construction**: Align with MAFFT (or via Amina if available)
3. **Conservation scoring**: Per-residue Shannon entropy, identify positions with < 5% variability
4. **Structural mapping**: Map conservation onto 3D structure, compute surface accessibility (SASA)
5. **Epitope ranking**: Score patches by: conservation x surface_accessibility x patch_size
6. **Binder handoff**: Feed top conserved epitopes as hotspots to `binder-designer`
7. **Cross-validation**: Run designed binders against each variant structure via Boltz-2, report IPSAE per variant

**Estimated effort**: 1 day

---

### 5. `threat-screener` — Priority: CRITICAL (enables Hard track — the differentiator)

**Purpose**: Structural and motif-based screening to detect biological threats that evade sequence-level DNA synthesis checks. This is the most impactful track per the brief — the October 2025 Science paper showed AI-redesigned toxins can retain function while evading all current screening software. The brief explicitly states this creates "an urgent need for people who understand the technology deeply enough to build countermeasures."

**Three detection modes** (from brief):

1. **Structural homology screening**: Fold query protein (ESMFold/Boltz-2 via Amina), compare against curated threat protein database using Foldseek/DALI. Flag if structural TM-score > 0.5 to known toxin/virulence factor despite < 30% sequence identity. This is the core insight: dangerous redesigns preserve structure while changing sequence.

2. **Motif detection**: Scan for conserved dangerous functional motifs:
   - Receptor-binding domains (RBD patterns from known pathogens)
   - Membrane-disrupting regions (amphipathic helices, pore-forming motifs)
   - Protease cleavage sites (furin sites, TMPRSS2 sites, etc.)
   - Aggregation-prone sequences (prion-like, amyloid cross-beta)
   - Immune evasion domains (glycan shields, complement evasion motifs)

3. **Anomaly scoring**: Flag sequences with:
   - Unusual codon usage (synthetic signatures vs natural evolution)
   - Chimeric domain architecture (domains from different pathogen families fused together)
   - High structural similarity to threat agents but low sequence identity (the key evasion signal)
   - Optimisation trajectories suggesting iterative immune evasion (design patterns, not research patterns)

**Threat database construction**:
- **Positive set**: Known toxins (ricin, abrin, botulinum), virulence factors (Ebola GP, anthrax PA/LF), immune evasion proteins, fusion peptides, pore-forming toxins — sourced from UniProt/PDB with "toxin" and "virulence" annotations
- **Negative set**: Benign human proteins, common enzymes, structural proteins, designed therapeutic binders
- **Grey zone**: Dual-use proteins (e.g., ACE2 binders that could be diagnostic or evasive)

**Test framework**: Run both sets through the screener. Report:
- True positive rate (TPR) / sensitivity — what % of threats detected
- False positive rate (FPR) — what % of benign proteins flagged
- Precision, recall, F1 score
- Specific examples of catches that BLAST/sequence-level screening would miss
- Analysis of failure modes (what slips through, and why)

**Integration with binder-designer**: The threat-screener can also be run as a safety check on our own binder designs — demonstrating responsible use of protein design tools. This is a strong narrative for Impact (30%).

**Estimated effort**: 2-3 days

---

### 6. Update `bio-orchestrator` + `CLAUDE.md` — Priority: MEDIUM

**New routing entries**:

| Intent | Skill |
|--------|-------|
| Binder design, protein design, RFdiffusion, ProteinMPNN, "design against pathogen" | `binder-designer` |
| Conserved epitopes, broad-spectrum, cross-variant, conservation analysis | `epitope-mapper` |
| Biosecurity screening, threat detection, structural screening, anomaly detection | `threat-screener` |
| Amina, run tool, GPU compute, protein prediction | `amina-bridge` |

**Chained workflow**: `epitope-mapper` -> `binder-designer` -> `struct-predictor` (full broad-spectrum pipeline in one command)

**Estimated effort**: 0.5 day

## Recommended Hackathon Strategy

### Approach: All Three Tracks Against One Target

The brief rewards depth over breadth: *"A well-executed project against a single pathogen will always be valued more highly than an ambitious framework with no worked application."* We tackle **all three difficulty tracks** (Easy + Medium + Hard) against a single pathogen to show the full pipeline — from designing countermeasures to screening for the very threats those countermeasures defend against.

This is the narrative that wins: **"We can both build the sword and build the shield."**

### Target: Nipah Virus Glycoprotein G (NiV-G)

**Why Nipah** (maximises Impact at 30% weight):
- BSL-4 pathogen, 40-75% case fatality rate — one of the deadliest known human pathogens
- Explicitly named in the brief as a suggested target
- Active community benchmark: Adaptyv Bio Nipah Competition provides comparables
- Well-characterised structure: PDB `7TXZ` (NiV-G bound to ephrin-B2)
- No approved antivirals or vaccines
- Clear therapeutic target: block ephrin-B2 receptor binding to prevent cell entry
- Cross-variant validation: NiV-Malaysia, NiV-Bangladesh, HeV, CedV (4+ variants for Medium track)
- The same NiV-G protein can be used as the threat target for the Hard track screening demo

**Alternative targets** (if another team picks Nipah):
- H5N1 haemagglutinin — most timely given current avian flu crisis ("completely out of control" per U. Glasgow)
- Ebola GP — high-impact, well-characterised, multiple variant structures available

### How All Three Tracks Reinforce Each Other

```
                          ┌─────────────────────────────┐
                          │     NIPAH VIRUS (NiV-G)     │
                          └─────────┬───────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
    ┌─────────────────┐  ┌──────────────────┐  ┌──────────────────┐
    │   EASY TRACK    │  │  MEDIUM TRACK    │  │   HARD TRACK     │
    │ Binder Design   │  │  Broad-Spectrum  │  │ Threat Screening │
    │                 │  │                  │  │                  │
    │ Design binders  │  │ Find conserved   │  │ Build structural │
    │ against NiV-G   │  │ epitopes across  │  │ screening that   │
    │ ephrin-B2 site  │  │ NiV/HeV/CedV     │  │ catches redesign │
    │                 │  │ Design cross-    │  │ ed NiV-G even    │
    │ RFdiffusion →   │  │ reactive binders │  │ with <30% seq    │
    │ ProteinMPNN →   │  │                  │  │ identity         │
    │ Boltz-2 → IPSAE │  │ Validates Easy   │  │                  │
    │                 │  │ track binders    │  │ Also screens our │
    │ → Diagnostic &  │  │ across variants  │  │ own designs as a │
    │   Therapeutic   │  │                  │  │ safety check     │
    │   candidates    │  │                  │  │                  │
    └────────┬────────┘  └────────┬─────────┘  └────────┬─────────┘
             │                    │                     │
             └────────────────────┼─────────────────────┘
                                  ▼
                    ┌──────────────────────────┐
                    │    UNIFIED SUBMISSION    │
                    │ "Both sword and shield"  │
                    │ against Nipah virus      │
                    └──────────────────────────┘
```

- **Easy feeds Medium**: binder candidates designed in Easy are validated across variant structures in Medium
- **Easy feeds Hard**: our own binder designs become test cases for the threat screener (responsible use demo)
- **Hard validates Easy**: screening tool verifies our therapeutic binders don't trigger threat flags (benign confirmation)
- **Hard stands alone**: screening tool independently detects redesigned NiV-G variants that evade BLAST

### Execution Plan (All Three Tracks)

| Day | Work | Tracks | Judging Criteria |
|-----|------|--------|-----------------|
| **Day 1** (Mon) | Set up Amina CLI + API key. Install `amina-skills`. Implement `amina-bridge`. Start `struct-predictor` implementation. | Infrastructure | Technical Execution (20%) |
| **Day 2** (Tue) | Finish `struct-predictor`. Implement `binder-designer` core. Fetch NiV-G (PDB 7TXZ). First binder design run. | **Easy** | Technical Execution (20%) |
| **Day 3** (Wed) | Full binder pipeline: RFdiffusion (50 backbones) -> ProteinMPNN -> Boltz-2 -> IPSAE scoring. Ranked candidate table. Start `epitope-mapper`. | **Easy + Medium** | Impact (30%), Rigour (25%) |
| **Day 4** (Thu) | **Workshop day**. Conservation analysis across NiV/HeV/CedV. Cross-variant IPSAE validation. Start `threat-screener`: build threat database, implement structural homology screening + motif detection. | **Medium + Hard** | Rigour (25%), Creativity (15%) |
| **Day 5** (Fri) | Finish `threat-screener`: assemble test sets (known threats + benign controls), run screening benchmark, compute TPR/FPR/F1. Run our own binder designs through screener as safety check. Anomaly scoring module. | **Hard** | Impact (30%), Creativity (15%) |
| **Day 6** (Sat) | Write 4-page report (all 3 tracks). Record 3-5 min demo. Polish README + architecture diagram. Update orchestrator + CLAUDE.md. Submit on DoraHacks. | All | Presentation (10%) |

### Demo Narrative (3-5 min)

1. **Problem** (30s): "Nipah virus kills 40-75% of those infected with no approved therapeutics. Worse: AI tools that can design countermeasures can also redesign threats that evade current screening. We need both offence and defence."
2. **Solution** (30s): "ClawBio is an agent-driven bioinformatics framework. We built skills for all three tracks — binder design, broad-spectrum validation, and biosecurity screening — all powered by AminoAnalytica's Amina platform."
3. **Live demo** (2 min): Natural language query -> agent designs binders against NiV-G -> validates across Henipavirus variants -> screens designs for safety
4. **Threat screening demo** (30s): Show redesigned NiV-G sequence that BLAST misses but our structural screener catches. Show TPR/FPR stats.
5. **Results** (30s): Top binder candidates with IPSAE scores, cross-variant binding table, screening benchmark results, diagnostic vs therapeutic classification

## Alignment with Judging Criteria

| Criterion | Weight | How ClawBio Maximises Score |
|-----------|--------|---------------------------|
| **Impact** | **30%** | All three tracks against Nipah (BSL-4, 40-75% CFR, no therapeutics). Binders + cross-variant validation + biosecurity screening = complete biodefence pipeline. The threat-screener directly addresses the Science 2025 paper showing current screening is broken. |
| **Scientific Rigour** | **25%** | IPSAE scoring (validated across 3,766 binders). Cross-variant validation with named structures. Threat screening with quantified TPR/FPR/F1 on real test sets. Reproducibility bundles. Limitations section in write-up. |
| **Technical Execution** | **20%** | 5 new skills working together. Amina CLI integration via official skills package. Modular, reproducible code. Bio-orchestrator chains all skills automatically. Every step is logged and reproducible. |
| **Creativity** | **15%** | All three tracks unified against one target. "Sword and shield" narrative. Agent-driven protein design. Threat-screener that checks our own designs (responsible AI). First bioinformatics agent framework with built-in biosecurity screening. |
| **Presentation** | **10%** | Auto-generated reports. 3-5 min demo showing all three tracks in one workflow. Architecture diagrams. Clear README. The narrative arc (design -> validate -> screen) is inherently compelling. |

## Submission Deliverables Mapping

| Hackathon Requirement | ClawBio Output |
|----------------------|----------------|
| Write-up (max 4 pages) | Page 1: Problem + approach. Page 2: Binder design results (Easy). Page 3: Cross-variant validation (Medium) + screening results (Hard). Page 4: Architecture, limitations, next steps. |
| Code (GitHub repo) | ClawBio repo with 5 new skills, clear README (overview, setup, architecture, Amina integration) |
| Demo (3-5 min) | Screen recording: NL query -> binder design -> cross-variant validation -> threat screening -> full report |
| Designed sequences/structures/datasets | Binder FASTA sequences, PDB structures, IPSAE score CSV, cross-variant binding table, screening benchmark (TPR/FPR/F1), threat database, confidence plots |

## Implementation Checklist

### Critical Path — All Three Tracks
- [ ] Set up Amina CLI + authenticate (`pip install amina-cli && amina auth set-key`)
- [ ] Install Amina skills package (`npx skills add AminoAnalytica/amina-skills --all`)
- [ ] Create `skills/amina-bridge/` — Amina CLI wrapper for ClawBio
- [ ] Implement `skills/struct-predictor/struct_predictor.py` — PDB fetch, Boltz-2, ESMFold, RMSD
- [ ] Create `skills/binder-designer/` — RFdiffusion -> ProteinMPNN -> Boltz-2 -> IPSAE (Easy track)
- [ ] Run binder design against NiV-G (PDB 7TXZ) — produce ranked candidates with real IPSAE scores
- [ ] Create `skills/epitope-mapper/` — conservation analysis across NiV/HeV/CedV (Medium track)
- [ ] Cross-variant IPSAE validation of binder candidates across Henipavirus variants
- [ ] Classify binder candidates as diagnostic vs therapeutic
- [ ] Create `skills/threat-screener/` — structural homology + motif detection + anomaly scoring (Hard track)
- [ ] Build threat protein database (toxins, virulence factors) + benign control set
- [ ] Run screening benchmark — report TPR, FPR, precision, recall, F1
- [ ] Screen our own binder designs as a safety/responsibility check
- [ ] Show specific example of threat that evades BLAST but is caught by structural screening
- [ ] Update `skills/bio-orchestrator/orchestrator.py` with new routes + chained workflows
- [ ] Update `CLAUDE.md` routing table with all new skills
- [ ] Write 4-page report covering all three tracks
- [ ] Record 3-5 min demo
- [ ] Submit on DoraHacks by Saturday 8 March

### Enhancements (strengthens submission further)
- [ ] Add demo data to repo (NiV-G PDB, variant sequences, threat DB, example outputs)
- [ ] Property prediction (solubility, toxicity, aggregation, stability) via Amina for top binder candidates
- [ ] Architecture diagram for pitch deck
- [ ] Behavioural anomaly detection module (design pattern analysis — the most creative Hard track direction)
- [ ] Dual-use analysis: classify grey-zone proteins (e.g., ACE2 binders) with confidence intervals

## Key Technical References

- [RFdiffusion — Watson et al. 2023, Nature](https://www.nature.com/articles/s41586-023-06415-8) — De novo protein structure design via diffusion
- [ProteinMPNN — Dauparas et al. 2022](https://www.science.org/doi/10.1126/science.add2187) — Sequence design for protein backbones
- [Boltz-2](https://github.com/jwohlwend/boltz) — Fast structure prediction and complex validation
- [ipSAE — Dunbrack 2025](https://www.biorxiv.org/content/10.1101/2025.02.10.637595v2.full) — Fixing AlphaFold's interface scoring
- [ipSAE meta-analysis — Passaro et al. 2025](https://www.biorxiv.org/content/10.1101/2025.08.14.670059v1.full) — 3,766 binder validation
- [ProteinDJ pipeline](https://www.biorxiv.org/content/10.1101/2025.09.24.678028v2.full) — Modular HPC binder design (Nextflow)
- [BinderFlow pipeline](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1013747) — Automated design with ipSAE/CUTRE scoring
- [Adaptyv Nipah Competition](https://www.adaptyvbio.com/blog/nipah-submissions/) — Community benchmark for NiV-G binders
- [AminoAnalytica Platform](https://www.aminoanalytica.com/) — Amina: 50+ protein tools with remote GPU
- [Levitate Bio — ipSAE explainer](https://levitate.bio/fixing-the-flaws-in-alphafolds-interface-scoring-meet-dunbracks-ipsae/) — Practical guide
- [Science 2025 — toxin redesign evading screening](https://www.science.org/journal/science) — Motivation for Hard track
- [SKYCovione](https://www.ipd.uw.edu/) — First computationally designed protein medicine (UW IPD)
- [NIST/Microsoft preprint](https://www.nist.gov/) — AI protein design capability assessment
