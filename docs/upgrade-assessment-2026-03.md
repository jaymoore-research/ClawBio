# ClawBio Upgrade Assessment — March 2026

**Date**: 2026-03-03
**Scope**: Full codebase audit — skill maturity, test coverage, security, infrastructure, architecture

## 1. Skill Implementation Gap

Only 7 of 16 skills have Python implementations. The other 9 are SKILL.md-only.

| Status | Skills |
|--------|--------|
| **Fully implemented** (Python + tests) | pharmgx-reporter, equity-scorer, nutrigx_advisor, genome-compare |
| **Python, no tests** | bio-orchestrator, claw-metagenomics |
| **SKILL.md only** (no Python) | claw-ancestry-pca, claw-semantic-sim, drug-photo, labstep, lit-synthesizer, repro-enforcer, scrna-orchestrator, seq-wrangler, struct-predictor, vcf-annotator |

### Unbuilt skills from CONTRIBUTING.md "Skills We Need"

- GWAS Pipeline (PLINK/REGENIE)
- Pathway Enricher (GO/KEGG)
- Clinical Variant Reporter (ACMG)
- Phylogenetics Builder (IQ-TREE/RAxML)
- Proteomics Analyser (MaxQuant/DIA-NN)
- Spatial Transcriptomics (Visium/MERFISH)

The **paper-data-extractor** in `creature-requests/001` is fully spec'd but not yet built.

## 2. Test Coverage Gaps

- **bio-orchestrator**: No tests — this is the router, should be tested first
- **claw-metagenomics**: Has Python but zero tests
- **No integration tests**: Each skill tested in isolation; no end-to-end test
- **CI doesn't install biopython**: `requirements.txt` lists it but `ci.yml` doesn't install it

## 3. Open Security Findings

5 MEDIUM items from SECURITY-AUDIT.md remain unresolved:

| ID | Issue |
|----|-------|
| GC-001 | `--output` writes to arbitrary filesystem paths |
| INT-003 | Full filesystem paths leaked in Telegram error messages |
| TG-002 | Telegram filenames used unsanitized in temp paths |
| TG-003 | `_received_files` not scoped by `chat_id` |
| TG-004 | No file size or type validation before skill subprocess |

## 4. Infrastructure Gaps

- Dependencies not pinned in `requirements.txt` (uses `>=` not `==`)
- CI permissions not restricted — should add `permissions: contents: read`
- No linting in CI (no flake8, ruff, mypy)
- No `pyproject.toml` — test discovery relies on explicit paths
- Makefile missing `test-all` target

## 5. Bot (RoboTerri) Robustness

- No multi-turn conversation state beyond file receipt
- No rate limiting per user
- Hardcoded file type routing instead of using the orchestrator
- No timeout on skill subprocesses
- No graceful shutdown / signal handling

## 6. Cross-Skill Consistency

- Naming inconsistency: `nutrigx_advisor` (underscores) vs all others (hyphens)
- No shared utilities: each skill reimplements parsing, reporting, repro bundles
- Output format varies: some markdown, some JSON; no unified schema
- Logging inconsistent: some use `logging`, others `print()` to stderr

## 7. Architecture Improvements

- No caching layer for repeated analyses
- No config management (`.clawbio.yaml` or similar)
- No plugin auto-discovery (manual edits to CLAUDE.md + orchestrator.py for each new skill)
- No skill versioning (semver in SKILL.md frontmatter)
- No progress reporting for long-running skills

## 8. Priority Roadmap

### Immediate (security)
1. Fix TG-002/003/004 — filename sanitization, chat_id scoping, file size limits
2. Redact filesystem paths from Telegram error messages (INT-003)

### Short-term (reliability)
3. Add tests for bio-orchestrator and claw-metagenomics
4. Pin dependencies in `requirements.txt` and `ci.yml`
5. Add subprocess timeouts in RoboTerri and clawbio.py
6. Add integration test: orchestrator → skill → report

### Medium-term (features)
7. Implement vcf-annotator (most-requested SKILL.md-only skill)
8. Implement paper-data-extractor (fully spec'd in creature-requests)
9. Extract shared utilities (input parsing, report generation, repro bundle)
10. Add `pyproject.toml` with ruff/mypy config, add linting to CI

### Longer-term (architecture)
11. Plugin auto-discovery (scan `skills/*/SKILL.md` to build routing table)
12. Skill versioning (semver in SKILL.md frontmatter)
13. Caching layer for repeated analyses
14. Multi-turn conversation state in RoboTerri

---

*ClawBio is a research and educational tool. It is not a medical device and does not provide clinical diagnoses. Consult a healthcare professional before making any medical decisions.*
