# New Skill: `gwas-risk` — Polygenic Risk Scores from Genome Data

> **Labels**: `enhancement`, `new-skill`
> **Priority**: High — completes the personal genomics trifecta (pharmgx + nutrition + disease risk)

---

## Summary

Add a `gwas-risk` skill that computes **polygenic risk scores (PRS)** from individual genome files (23andMe, AncestryDNA, VCF) using published GWAS summary statistics. This lets users answer: *"Based on my genome, what are my genetic risk factors for common diseases?"*

GWAS integration is already listed as a wanted skill in [CONTRIBUTING.md](../CONTRIBUTING.md) ("GWAS Pipeline: PLINK/REGENIE automation"). This issue scopes a user-facing implementation focused on **risk estimation from existing GWAS catalogs**, not running new association studies.

---

## Motivation

ClawBio already tells users about pharmacogenomics (pharmgx-reporter) and nutrition (nutrigx_advisor) from their genome files. The most-requested missing piece is **disease risk** — the classic "what am I at higher/lower genetic risk for?" question. This requires:

1. A curated set of GWAS summary statistics (effect sizes + risk alleles per SNP per trait)
2. A scoring engine that reads the user's genotypes and computes weighted risk scores
3. Ancestry-aware percentile calibration (risk scores are only meaningful relative to a population)
4. Clear communication that PRS are probabilistic, not diagnostic

This skill would make ClawBio a complete personal genomics platform: **pharmacogenomics + nutrition + ancestry + disease risk** from one genome file.

---

## Proposed Skill: `skills/gwas-risk/`

### Core Capabilities

1. **PRS computation** — Calculate polygenic risk scores for common diseases/traits from individual genotype data using published GWAS summary statistics
2. **Multi-trait panel** — Score across a curated set of well-validated traits (e.g., coronary artery disease, type 2 diabetes, breast cancer, Alzheimer's, BMI, LDL cholesterol)
3. **Ancestry-aware calibration** — Adjust PRS percentiles by genetic ancestry (scores derived from EUR GWAS have reduced predictive power in non-EUR populations — this must be flagged)
4. **Risk categorisation** — Classify each trait as elevated / average / reduced risk with percentile rankings
5. **Variant-level detail** — For each trait, show the top contributing SNPs (highest effect sizes) with rsIDs, risk alleles, and individual contributions to the total score
6. **GWAS Catalog integration** — Pull or reference published summary statistics from GWAS Catalog, PGS Catalog, or bundled curated files

### Input Formats

| Format | Description |
|--------|-------------|
| 23andMe raw data (`.txt`) | Tab-separated: rsid, chromosome, position, genotype |
| AncestryDNA raw data (`.txt`) | Tab-separated with header: rsid, chromosome, position, allele1, allele2 |
| VCF (`.vcf`, `.vcf.gz`) | Single-sample or multi-sample VCF with GT field |
| Genotype CSV/TSV | Columns: rsid, genotype (or allele1 + allele2) |

### Workflow

```
User genotype file
    │
    ▼
┌──────────────────────┐
│ 1. Parse genotypes    │  Extract rsID → genotype map
│    (reuse existing    │  Support 23andMe, AncestryDNA, VCF
│     parse_input.py)   │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 2. Load GWAS weights │  Curated summary stats per trait:
│    (bundled or PGS   │  rsID, effect_allele, beta/OR, p-value,
│     Catalog fetch)   │  population, PMID
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 3. Compute PRS       │  For each trait:
│                      │    score = Σ (dosage_i × beta_i)
│                      │  where dosage = 0/1/2 copies of effect allele
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 4. Ancestry context  │  Estimate ancestry (reuse genome-compare
│                      │  or claw-ancestry-pca)
│                      │  Flag EUR-derived scores for non-EUR users
│                      │  Calibrate percentiles per population
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 5. Risk report       │  Per-trait: score, percentile, risk category
│                      │  Top contributing variants per trait
│                      │  Ancestry transferability warnings
│                      │  Risk summary figure
│                      │  Reproducibility bundle
└──────────────────────┘
```

### Suggested Trait Panel (MVP)

Focus on traits with **well-validated, large-effect PRS** and published PGS Catalog scores:

| Trait | PGS Catalog ID | Key SNPs | GWAS Source | Clinical Relevance |
|-------|---------------|----------|-------------|-------------------|
| Coronary artery disease | PGS000018 | ~200+ | CARDIoGRAMplusC4D | Leading cause of death globally |
| Type 2 diabetes | PGS000014 | ~150+ | DIAGRAM | Preventable with lifestyle changes |
| Breast cancer | PGS000004 | ~300+ | BCAC | Screening interval decisions |
| Prostate cancer | PGS000662 | ~250+ | PRACTICAL | PSA screening stratification |
| Alzheimer's disease | PGS000334 | ~80+ | IGAP/Bellenguez | Care planning, risk awareness |
| BMI / obesity | PGS000027 | ~500+ | GIANT | Metabolic risk context |
| LDL cholesterol | PGS000063 | ~150+ | GLGC | Cardiovascular risk factor |
| Atrial fibrillation | PGS000016 | ~100+ | AFGen | Stroke risk stratification |
| Celiac disease | — | ~40 | Trynka et al. | Dietary relevance (chains to nutrigx) |
| Age-related macular degeneration | — | ~30 | IAMDGC | Preventive screening |

**MVP approach**: Start with a smaller curated panel of ~50-100 high-effect SNPs per trait (like pharmgx-reporter's curated SNP approach) rather than full genome-wide scores. This ensures the skill works with DTC data that only covers ~600K-700K SNPs.

### Output Structure

```
output_dir/
├── report.md                      # Full risk report with figures
├── figures/
│   ├── risk_summary.png           # Horizontal bar chart: all traits with percentile + risk category
│   ├── trait_detail_cad.png       # Per-trait: top SNP contributions
│   └── ancestry_context.png       # PCA plot showing ancestry calibration
├── tables/
│   ├── risk_scores.csv            # trait, raw_score, percentile, risk_category, ancestry_group
│   ├── variant_contributions.csv  # trait, rsid, effect_allele, user_genotype, dosage, beta, contribution
│   └── ancestry_warnings.csv      # trait, transferability_rating, source_population, user_population
└── reproducibility/
    ├── commands.sh
    ├── environment.yml
    └── checksums.sha256
```

### CLI Interface

```bash
# Basic usage
python skills/gwas-risk/gwas_risk.py \
  --input <genotype_file> --output <report_dir>

# With specific traits
python skills/gwas-risk/gwas_risk.py \
  --input <genotype_file> --traits cad,t2d,breast_cancer --output <report_dir>

# With ancestry override (skip auto-detection)
python skills/gwas-risk/gwas_risk.py \
  --input <genotype_file> --ancestry EUR --output <report_dir>

# Demo mode
python skills/gwas-risk/gwas_risk.py \
  --input skills/gwas-risk/demo_patient.txt --output /tmp/gwas_demo
```

### Demo Data

- `skills/gwas-risk/demo_patient.txt` — Synthetic patient with ~200 GWAS-relevant SNPs across the trait panel (mixed risk profile: elevated CAD, average T2D, reduced Alzheimer's)
- `skills/gwas-risk/data/gwas_weights.json` — Curated effect sizes from PGS Catalog / published GWAS

---

## CLAUDE.md Routing Entry

```
| GWAS, polygenic risk, PRS, genetic risk, disease risk, "what am I at risk for", risk score, common diseases | `skills/gwas-risk/` | Run `gwas_risk.py` |
```

## Bio-Orchestrator Integration

**Keyword triggers**: `gwas`, `polygenic`, `risk score`, `prs`, `disease risk`, `genetic risk`, `risk factor`

**Chaining opportunities**:
- **After claw-ancestry-pca** → ancestry informs PRS calibration population
- **After pharmgx-reporter** → combine drug response + disease risk for complete profile
- **After nutrigx_advisor** → nutrition recommendations contextualised by disease risk (e.g., elevated CAD risk → emphasise omega-3 / Mediterranean diet findings from nutrigx)
- **Before lit-synthesizer** → for elevated-risk traits, auto-search recent literature for prevention strategies

---

## Safety Considerations

These are critical for a disease risk tool and must be embedded in the SKILL.md methodology:

1. **Not diagnostic** — PRS provide probabilistic risk estimates, not diagnoses. The standard ClawBio disclaimer must be prominent, with additional trait-specific language: *"A high polygenic risk score does not mean you will develop this condition. A low score does not mean you are protected."*
2. **Ancestry transferability** — Most GWAS are conducted in European-ancestry cohorts. PRS accuracy drops significantly for non-European populations. The skill MUST:
   - Auto-detect ancestry and flag reduced transferability
   - Show transferability ratings per trait (EUR, EAS, AFR, SAS, AMR)
   - Never present percentiles calibrated on a mismatched population without warning
3. **Actionability gating** — Only include traits where genetic risk is scientifically actionable (screening, lifestyle, monitoring). Avoid traits that could cause harm without clinical context (e.g., psychiatric conditions, intelligence proxies).
4. **No raw risk numbers without context** — Always show odds ratios / percentiles relative to population average, never absolute lifetime risk (which requires age, sex, family history, clinical data we don't have).
5. **Genetic data stays local** — All computation is local. GWAS weights are bundled, not fetched from external APIs at runtime (optional PGS Catalog fetch can be added later with user consent).

---

## Dependencies

| Package | Purpose | Required? |
|---------|---------|-----------|
| `pandas` | Genotype + weight table manipulation | Yes |
| `numpy` | PRS computation, statistics | Yes |
| `matplotlib` | Risk summary plots, contribution charts | Yes |
| `scikit-learn` | PCA for ancestry estimation (or reuse existing) | Optional |
| `requests` | PGS Catalog API fetch (future enhancement) | Optional |

No external binaries required for MVP. Full PLINK/REGENIE integration (for users running cohort-level GWAS) can be a Phase 2 enhancement.

---

## Implementation Phases

### Phase 1 — MVP (curated panel, local weights)
- [ ] Create `skills/gwas-risk/SKILL.md` with full methodology
- [ ] Implement `gwas_risk.py` with curated SNP weights for 5-10 traits
- [ ] Input parsing (reuse `parse_input.py` patterns from pharmgx/nutrigx)
- [ ] PRS computation engine: `score = Σ(dosage × beta)`
- [ ] Basic ancestry detection or accept `--ancestry` flag
- [ ] Risk summary figure (horizontal bar chart with percentiles)
- [ ] Variant contribution table (top SNPs per trait)
- [ ] Ancestry transferability warnings
- [ ] Demo patient data
- [ ] Add CLAUDE.md routing + orchestrator keywords
- [ ] Tests

### Phase 2 — PGS Catalog Integration
- [ ] Fetch published PGS from [PGS Catalog API](https://www.pgscatalog.org/rest/)
- [ ] Support full genome-wide PRS (thousands of SNPs per trait)
- [ ] Population-specific percentile calibration from PGS Catalog reference distributions
- [ ] Expand trait panel beyond MVP

### Phase 3 — Cohort-Level GWAS
- [ ] PLINK2 integration for running association tests on multi-sample VCFs
- [ ] REGENIE support for biobank-scale data
- [ ] Manhattan plot + QQ plot generation
- [ ] Genomic inflation factor (lambda) QC
- [ ] LD clumping + conditional analysis

---

## Example Queries That Would Trigger This Skill

```
"What diseases am I at genetic risk for?"
"Calculate my polygenic risk scores from this 23andMe file"
"Run GWAS risk analysis on my genome"
"What does my DNA say about my heart disease risk?"
"Show me my genetic risk factors"
"Compute PRS for type 2 diabetes and coronary artery disease"
"Am I at elevated genetic risk for Alzheimer's?"
```

---

## References

- [PGS Catalog](https://www.pgscatalog.org/) — Standardised polygenic score repository
- [GWAS Catalog](https://www.ebi.ac.uk/gwas/) — NHGRI-EBI catalog of published GWAS
- [Khera et al. 2018, Nature Genetics](https://doi.org/10.1038/s41588-018-0183-z) — Genome-wide polygenic scores for common diseases
- [Mars et al. 2020, Nature Medicine](https://doi.org/10.1038/s41591-020-0800-0) — PRS and clinical risk prediction
- [Martin et al. 2019, Nature Genetics](https://doi.org/10.1038/s41588-019-0379-x) — Ancestry diversity in PRS research (the transferability problem)
- [Wand et al. 2021, Nature](https://doi.org/10.1038/s41586-021-03243-6) — Improving PRS transferability across populations
- [pgsc_calc](https://github.com/PGScatalog/pgsc_calc) — PGS Catalog's Nextflow PRS calculator (reference implementation)
