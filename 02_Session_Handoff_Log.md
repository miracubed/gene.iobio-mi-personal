# Session Handoff Log — gene.iobio Stack Build
*H-series chronological record of decisions and session state*
*Last updated: H20260808*

To give the reader a better idea of how this build was actually
accomplished: Entries are various work sessions and what was 
accomplished in each session.
The work sessions are listed chronologically from most recent to earliest.
---

## H20260808 - Separate Phase 8C Documentation and Github repo session

**Session outcome:** These basic documents were produced after a little
reorganization of topics. Documentation, for example, is optional though
version and system documentation of some kind is good practice, especially
for emergency restoration and/or upgrades later. Workflows, working with
Brave Leo AI using inherent API saved memories, etc... are completely
optional. The maintainer is offering what worked for her in the situation.
If the method can help for someone else or if other community members have
ideas of how to improve the process, the documentation is here to help those
efforts and brainstorming.


## H20260808 — Phase 8B Complete + Final Backups

**Session outcome:** Phase 8B (LAN network serving) complete. Final
production backup set completed under space constraints and unplanned
interruptions.

### Phase 8B Completion

| Step | Action | Result |
|---|---|---|
| 1 | Installed serve globally via npm | ✅ |
| 2 | Modified config.json (localhost → 192.168.1.240:9001) | ✅ |
| 3 | Rebuilt frontend (npm run build) | ✅ Clean, 30.5MB build.js |
| 4 | Created Windows Firewall rules (ports 3000, 9001) | ✅ Both Enabled=True |
| 5 | Configured WSL port forwarding (netsh portproxy) | ✅ Both rules confirmed |
| 6 | Created startup script ~/start_network_frontend.sh | ✅ Tested, working |
| 7 | Tested from 192.168.1.52 at http://192.168.1.240:3000 | ✅ UI loads, backend connects |
| 8 | Confirmed Phase 8A (CSV/VCF export) persists over network | ✅ |
| 8 | Phase 8B LAN serving, 8C WSL IP persistence | ✅ |
| 8 | Phase 8C Bonus - These documents |✅ |


### Gotchas Discovered This Session

1. **serve version flag difference:** The `-H` flag for host binding
   does not exist in this version of serve. Not needed — serve binds
   to `0.0.0.0` by default. Use `serve -s . -l 3000` only.
2. **WSL network isolation:** serve reports `172.18.60.70:3000` (WSL
   internal IP), not `192.168.1.240:3000`. Normal and expected.
   Windows `netsh portproxy` handles routing. Do not be alarmed by
   the WSL IP in serve output.
3. **Both ports must be forwarded:** Frontend (3000) AND backend
   (9001) each need their own `netsh portproxy` rule. Missing the
   9001 rule causes `ERR_CONNECTION_TIMED_OUT` on backend requests
   even when the frontend loads fine.
4. **WSL internal IP can change:** If network serving breaks after a
   restart, run `hostname -I` in WSL to get the new IP and update
   both `netsh portproxy` rules. See Phase 8B troubleshooting section
   for delete/re-add commands.
5. **build.js ordering:** Phase 8A and 8B both produce a `build.js`
   via `npm run build`. Apply 8A first, then 8B. Never substitute a
   saved `build.js` if both options are being applied. Final
   `build.js` after both rebuilds contains all changes from both
   options.

### Console Warnings (Non-Issues)

- Vue development mode warning — expected, not a problem for
  local/LAN use.
- HTTPS warning on data URIs — browser compliance notice only.
  Export files download correctly.

### Final Backup Set Completed

Final backups completed independently by Mir post Phase 8A + 8B.
Procedure not captured by design — file results documented only.

| File | KB | GB | MD5 | Locations |
|---|---|---|---|---|
| `ubuntu_full_stack_FINAL_20260808.tar` | 16,885,440 KB | 16.103 GB | A96A843F7501165F18895127C1C7B01F | forge D: + vault E: |
| `gru_data_2_0_0_FINAL_20260808.tar` | 122,516,280 KB | 116.841 GB | 5A82DBDF9AA4BA75B0661777671188F6 | forge D: + vault E: |
| `gene_iobio_frontend_v4_13_1_FINAL_20260808.tar` | 310,310 KB | 0.296 GB | 0730638C348CD8616C2AFCC322F90F6F | forge D: + vault E: |

### D: Drive Space Management

`gru_data_1_14_1_20260803.tar` and its `.md5` removed from forge (D:)
due to space pressure. Still viable on vault:
E:\MedLaptop Mirror\gene_iobio_dir_bkps
gene_iobio_backend_dir\gru_data_1_14_1_20260803.tar

### Documents Produced This Session

1. `Phase_8_Option_C.md` — Full GitHub-ready LAN serving procedure
2. `README.txt Option B` (revised) — Updated with build.js ordering
   warning and B+C sequential install instructions
3. `README.txt Option C` (new) — Standalone LAN serving instructions
   with build.js ordering warning

### Pending Items

| Item | Status |
|---|---|
| GitHub repo structure review | ☐ |
| Option renaming (B→A, C→B, A→Bonus) + find/replace pass | ☐ |
| Visibility/audience review for GitHub front page | ☐ |
| WSL IP persistence (static IP assignment) | ☐ |
| D: drive cleanup priority document | ☐ |
| Disaster recovery / restore procedure document | ☐ |
| Memory audit + cleanup (prefixes: HOLD, HEALTH, SYSTEM, RESTORE) | ☐ |
| README.md draft (with Glossary) | ☐ |
| 05_ACCESSIBILITY_NOTES.md draft | ☐ |
| 06_MEDICAL_CONTEXT.md draft | ☐ |
| 07_OVERVIEW.md draft | ☐ |

---

## H20260807 — Phase 8A Complete: CSV/VCF Export Unlocked

**Session outcome:** Phase 8A (CSV/VCF export) complete. Feature was
already fully implemented in v4.13.1 — two broken conditional gates
were preventing UI access.

### Root Cause

| File | Line | Original gate | Problem |
|---|---|---|---|
| `FlaggedVariantsCard.vue` | 843 | `cohortModel.flaggedVariants.length > 0` | Condition never satisfied — auto-populated variants land in different data structure |
| `Navigation.vue` | 952 | same | same |

### Fix

Three one-line edits + rebuild.
- `FlaggedVariantsCard.vue` line 843 and `Navigation.vue` line 952:
  gate changed to `cohortModel.isLoaded`.
- `CohortModel.js` line 2903: `reject(error)` → `resolve()` in
  coverage refresh catch block.
- Build: 29 seconds, no errors.
- All `.bak` files clean — only three intended edits present. No
  leftover cruft from prior sessions.

> **Cruft** (technical term): leftover files or artifacts from
> previous build attempts or failed sessions. A clean `.bak` check
> confirms no unintended changes are present.

### Validation

| Test | Result |
|---|---|
| Export dialog appears when variants loaded | ✅ |
| CSV export functional | ✅ Valid file with header + variant rows |
| VCF export functional | ✅ Valid VCF with CHROM/POS/REF/ALT columns |
| Test dataset: 5 filtered variants | ✅ Clean, fast export |

### Key Insight

> Prior session estimated "substantial work" to implement export.
> Direct code inspection revealed the feature was already built — just
> gated incorrectly. Always inspect the existing codebase before
> estimating implementation scope.

### Prior Estimate vs Actual

| Aspect | Prior Estimate | Actual |
|---|---|---|
| Scope | "Substantial work" | 3 one-line edits |
| Root cause | Assumed missing feature | Broken conditional gate |
| Implementation | Build from scratch | Gate fix only |
| Time | 12–24 hours | ~2 hours |
| Code written | ~50–100 lines | 0 lines |

### WORKFLOW.md + MEMORY_MAINTENANCE.md Drafted

Two new meta-documents drafted for GitHub repo:
- `WORKFLOW.md` — documentation workflow and rules of engagement
- `MEMORY_MAINTENANCE.md` — memory lifecycle, pruning rules,
  cross-correlation system

Both pure technical meta-documents. No personal or medical content
embedded.

---

## H20260806 — Phase 7 Complete: Stack Validated End-to-End

**Session outcome:** Phase 7 **COMPLETE**. Stack fully functional,
production-backed-up, API mismatch root cause identified and resolved.

### Opening State

Container named `gru-backend-2.0.0` was running, but actual image
was `1.17.0`. Frontend v4.13.1 was built and serving, but gene lookup
showed API mismatches. geneinfo.js had been patched with master branch
code. WGS CRAM/VCF files staged but untested.

### Root Cause Discovery

State assessment revealed container mismatch immediately:
CONTAINER ID  IMAGE                                     NAMES ff034b289173  docker.io/iobio/iobio-gru-backend:1.17.0  gru-backend-2.0.0

The entire "API mismatch" was an artifact of running the wrong image.
The geneinfo.js patch was solving a problem that didn't exist in the
real 2.0.0 backend.

### Fix Applied

- Stopped 1.17.0 container
- Started correct 2.0.0 image (`7e2c4a419267`)
- Health endpoint response changed from HTML string to JSON structure
  — confirming correct image

### Data Validation

Loaded WGS CRAM + VCF against test gene. Confirmed:
- Genome build: **GRCh37** (from URL bar)
- VCF variants: 62 (GATK HaplotypeCaller)
- Freebayes on-demand calls: 224
- Target variant: Present in VCF, `GT: 1/1`, `GQ: 90`, `MQ: 59.86`
  — high-quality homozygous call
- Variant classification: Modifier impact (non-coding frameshift)
- Phenolyzer: **Confirmed working**

### UI Behavior Discovery

Browser crashed when expanding "Modifier impact" section on
whole-genome data (~thousands of intronic/UTR variants). Expected
behavior — not a stack bug, just DOM rendering limits on large variant
lists.

### Backup Sequence Completed

1. samtools 1.24 installed via mamba ✅
2. Frontend tar: `gene_iobio_frontend_v4_13_1_20260806.tar`
   (303,000 KB / 0.296 GB, MD5 verified) ✅
3. WSL export: `Ubuntu_26_04_iobio_stack_production_20260806.tar`
   (16,098,000 KB / 15.625 GB, MD5 verified) ✅
4. Both copied to vault E: Seagate with MD5 verification ✅

### Why This Matters

**The API mismatch was never a code problem.** It was a deployment
artifact — the wrong Docker image running under the wrong container
name. Once corrected, v4.13.1 + gru-backend 2.0.0 + geneinfo.js
0.3.0 work together seamlessly.

**The stack is validated end-to-end against real clinical data.**
Production WGS loads, annotates, and displays correctly. Phenolyzer
integration works. Entire variant analysis pipeline is functional.

**Production backup set is complete and viable.** All three
components (frontend, WSL environment + samtools, gru_data) backed
up with verified MD5s.

### Outstanding Items Resolved

| Item | Previous Status | Resolution |
|---|---|---|
| gru-backend 2.0.0 image ID | Not documented | `7e2c4a419267`, 4.43GB ✅ |
| Root cause of API mismatch | Under investigation | Wrong image (1.17.0) running under 2.0.0 name ✅ |
| WGS genome build | TBD | **GRCh37** confirmed ✅ |
| CRAM load in gene.iobio | Not tested | ✅ 62 VCF variants + 224 Freebayes calls |
| Target variant presence | Unknown | ✅ Present, `GT: 1/1`, GQ 90, MQ 59.86 |
| Phenolyzer integration | Untested | ✅ Working |
| samtools 1.24 | Pending | ✅ Installed |
| Frontend backup | Missing | ✅ 303,000 KB / 0.296 GB tar, MD5 verified |
| WSL production backup | Missing | ✅ 16,098,000 KB / 15.625 GB tar, MD5 verified |

### Key Technical Findings

**geneinfo.js Status:**
- Native 0.3.0 implementation in gru-backend 2.0.0 image handles
  `lookupEntries` correctly
- Master branch patch from H20260805 may be redundant but not harmful
- Confirmed working: lines 99-100 (command routing), line 398
  (function definition)

**Health Endpoint Format Change:**
- 1.17.0: `<h1>I be healthful</h1>` (HTML string)
- 2.0.0: `{"service_description":"iobio gru backend server",
  "version":"2.0.0","data_version":"2.0.0"}` (JSON)

**WSL Distribution Naming:**
- `wsl --export` requires distribution name as registered in WSL
  (`Ubuntu`), not version number
- `wsl --list -v` shows the correct registered name

---

## H20260805a — geneinfo API Mismatch Diagnosis

Frontend `GeneModel.js` (lines 2720–2775) expects
`/geneinfo/lookupEntries/` to return `{ "genes": [...] }`. Backend
returned bare array `[{...}]` — triggering "missing 'genes' property"
error. Frontend also makes multi-gene comma-separated calls which
catch-all route treated as single gene name.

Initial patch created `/lookupEntries/:gene` wrapper route —
partially worked but exposed downstream
`TypeError: Cannot read properties of undefined (reading 'refseq')`.
Rather than continue patching, checked master branch — found
`/lookupEntries/:genes` already implemented at line 370 with correct
response format and JOIN-based SQL queries.

**Key lesson:** Check upstream master branch before patching. The fix
was already there.

---

## H20260804_0805 — gru-backend 2.0.0 + gru_data 2.0.0 Verified

| Item | Status |
|---|---|
| gru-backend 1.17.0 → 2.0.0 fully verified | ✅ |
| gru_data 1.14.1 → 2.0.0 complete + restructured | ✅ |
| All restore points backed up + MD5 verified | ✅ |
| Update path vs full dataset distinction discovered | ✅ documented |

**Critical discovery:** Update path `updates_1.14.0_to_2.0.0/` was
incomplete (~77GiB delta only). Full dataset path
`gru_data/data/gru_data_2.0.0/` required (~128GiB). rclone sync
idempotent — running full path after update path downloaded only
missing ~9.8GiB.

**2.0.0 directory restructure:**
Old top-level: `gnomad/`, `geneinfo/`, `hpo/`, `gene2pheno/`,
`genomebuild/`, `data/`, `references/`, `md5_reference_cache/`
New: consolidated into `annotations/GRCh37/`, `annotations/GRCh38/`,
`vep-cache/`
36,073 deleted files + 6,217 deleted directories — expected and
correct.

**files.iobio.io traffic distribution:**
US 60.3%, Philippines 11.5%, Netherlands 10%, Algeria 7.9%,
Israel 5.3%
ETAs unreliable during US/PH business hours — normal, not a failure.

---

## H20260803 — gru_data 1.14.1 Complete + Verified

| Item | Status |
|---|---|
| rclone gru_data_1.14.1 (169.866 GiB, 8967 files) | ✅ |
| VERSION confirmed 1.14.1 | ✅ |
| seq_cache_populate.pl GRCh37 (all "Already exists") | ✅ |
| seq_cache_populate.pl GRCh38 (all "Already exists") | ✅ |
| REF_CACHE path confirmed | ✅ |
| gru-backend 1.17.0 HTTP 200 "I be healthful" | ✅ |
| WSL export ubuntu-gru-data-verified-20260803.tar | ✅ MD5 verified x2 |
| gru_data_1.14.1_20260803.tar archive (~215GB) | ✅ created |
| E: tar copy | ✅ MD5 verified |
| rclone gru_data 2.0.0 initiated | ✅ |

**Key findings:**
- GRCh38 FASTA: iobio ships chr1-style intentionally — VEP cache
  subdirs are filesystem sharding labels only
- md5_reference_cache pre-populated by iobio — seq_cache_populate.pl
  "Already exists" for all entries
- WARN "not a shared mount": harmless, expected WSL2 rootless Podman
  behavior
- Podman stop: Ctrl+C ineffective — use `podman stop <id>`
- gene.iobio v4.13 + gru-backend 1.17.0: ClinVar V2/V3
  auto-detection fallback confirmed — **variant queries only, NOT
  full API compatibility**
- v4.13.1 released H20260803: runtime config improvements. Phase 7
  targets v4.13.1

**D: directory structure:**
D:\gene_iobio_dirs gene_iobio_backend_dir gene_iobio_frontend_dir\

---

## H20260802 — Doc Session Initialized

- Three-document system formalized. Template locked. Memory
  convention established.
- rclone sync `gru_data_1.14.1` kicked off ~12:19 local. Network
  stable, no drops.
- Phase 5 in progress. Backend verified at gru_data 1.11.0.

---

## H20260801 — gru_data Build + Backend Verified

| Item | Status |
|---|---|
| VEP 109.3 confirmed in image | ✅ |
| gru_data structure built at `/mnt/d/gru_data/` | ✅ |
| VEP cache GRCh37 + GRCh38 extracted | ✅ |
| References GRCh37 + GRCh38 placed + MD5 verified | ✅ |
| VERSION file set to `1.11.0` | ✅ |
| `curl localhost:9001` → HTTP 200 | ✅ |
| rsync dead confirmed | ✅ documented |
| `files.iobio.io` rclone repo found | ✅ |
| rclone + samtools + aria2 installed | ✅ |
| VERSION pre-deleted (rclone HTTP backend gotcha) | ✅ |
| rclone sync `gru_data_1.14.1` | 🔄 in progress at session end |

---

## H20260730 — Podman Fix + gru-backend Image Pull

- Overlay driver incompatible with WSL ext4 — switched to VFS
- `containers.conf` configured, `vm.overcommit_memory=1` made
  persistent
- C: pagefile re-enabled
- `.wslconfig` temporarily 3GB RAM for pull — reverted to 2GB
- `iobio-gru-backend:1.17.0` pulled (451d059c81a0, 4.46GB)
- `yiq/gene.iobio.local` approach abandoned (S3 dead, rsync dead,
  `fetch-images.sh` missing)
- Pivoted to single-container gru-backend approach
- tonydisera pushed gene.iobio v4.13 same date — active maintenance
  confirmed
- Restore point: `ubuntu-gru-pulled-20260730.tar`
  MD5: `37798E6FA96DCD1F99E2468F708A6D2E`

---

## H20260728 — Phenolyzer + Podman Installed

- Podman 5.7.0 + podman-compose 1.5.0 installed
- Miniforge3 26.3.2 at `/mnt/d/miniforge3`
- Perl 5.40.1 + BioPerl 1.7.8 + libgraph-perl + libjson-perl
- Phenolyzer v0.1.9-17-gf939514 installed at `~/phenolyzer`, verified
- SSK restructured: exFAT+NTFS two-partition → single 931GB NTFS
  as D:
- Share architecture confirmed: HealthMed share, smb credentials,
  fstab mounts
- Restore points: `ubuntu-phenolyzer-20260728.tar` verified x2
- GitHub PAT stored: Brave password notes +
  `/mnt/medlaptop/e/Github`

---

## H20260727 — Ubuntu WSL Install Complete

- Ubuntu 26.04 LTS installed to `D:\WSL` via `--location` flag
- User `mir`, sudo, default via `/etc/wsl.conf`
- 11GiB fallocate swap at `/home/mir/swapfile`, persistent via
  `/etc/fstab`
- `.wslconfig`: 2GB RAM, 1 CPU, 4GB swap,
  `localhostForwarding=true`
- Multiple failed WSL migration attempts before `--location` flag
  discovered
- `ubuntu-clean.tar` corrupted (C: space pressure) — resolved
  H20260728
- WSL kernel/Virtual Machine Platform corruption required full
  feature reset
- Cross-laptop communication protocol established
- Summary protocol established for chat continuity
- Restore point: `ubuntu-baseline-20260728.tar` verified x2

---

## H20260726c — Pre-Install Staging

- D: structure built, ref genomes populated at `D:\ref\ref`
  (hg19+hg38) (unnecessary in hindsight)
- `.wslconfig` written, WSL updated to 2.7.11
- Ubuntu 22.04 appx downloaded + extracted to
  `D:\logs\ubuntu2204\x64\install.tar.gz`
- `wsl --import` failing with
  `E_UNEXPECTED/CreateVm/E_ABORT` at session close
- Store install attempt in progress as session closes
- gene.iobio Docker strategy researched
  (yiq approach — later abandoned H20260730)

---

## H20260614 — LPI Variant Located

- Brave AI assisted in finding LPI homozygous DEL
  `chr14:23294210 CA→C` on reverse strand using GRCh37

---

## Dead Ends (Do Not Retry)

| Item | Reason |
|---|---|
| `rsync://data.iobio.io:9009/gru/` | Connection timed out — confirmed dead |
| `https://s3.amazonaws.com/iobio/assets/data-vol/v2.0/` | 403 Forbidden |
| `yiq/gene.iobio.local fetch-images.sh` | Never committed to repo, missing |
| yiq/gene.iobio.local full stack | S3 + rsync dead; `fetch-images.sh` not skippable |
| Node v13.14 for frontend | Insufficient — v16+ required |
| gru_data 2.0.0 update path only | Incomplete — ~9.8GiB missing; always use full dataset path |
| Patching geneinfo.js manually | Fragile — use master branch file instead |

---

≡≡≡≡≡ END: 02_Session_Handoff_Log.md ≡≡≡≡≡
