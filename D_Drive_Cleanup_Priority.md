# D_Drive_Cleanup_Prority.md
## Overview

Post-build drive cleanup and space reclamation.
The forge (LAPTOP-F, 192.168.1.240) `D:` drive (SSK, 931GB) fills
during the backup process. This document prioritizes which backup
files to remove from `D:` to reclaim space, in order of priority.
This is not really a phase, but necessary for housekeeping purposes
on resource-limited systems, therefore is labeled "D for disembodied
from the build project itself.

I hope you find the rationale helpful.

**Critical rule:** Do not delete a file from `D:` unless you have
verified that an identical copy exists on vault (`E:` Seagate) with
matching MD5 hash.

---

## Space Accounting

| Item | KB | GB | Location |
|---|---|---|---|
| WSL installation + tools | ~20,000 KB | ~0.019 GB | `D:\WSL\` |
| `gru_data 2.0.0` (live) | ~128,000,000 KB | ~122 GB | `D:\gru_data\` |
| Final backups (`WSL` + `frontend` + `backend`) | ~139,711,430 KB | ~133.2 GB | `D:\WSL_Backups\` + `D:\gene_iobio_dir_bkps\` |
| **Total in use** | **~267,711,430 KB** | **~255.2 GB** | — |
| **`D:` drive capacity** | 931,000,000 KB | 931 GB | — |
| **Remaining free** | ~663,288,570 KB | ~675.8 GB | — |

After final backups (H20260808), `D:` drive has adequate free space.
However, intermediate restore points from earlier phases consume
significant space and are now superseded by the final backup set.

---

## Backup Hierarchy

Backups fall into three categories:

### Category 1: FINAL (Keep indefinitely)

These are the production-ready backup set created post-Phase 8A + 8B.
Keep both copies (`forge` `D:` + `vault` `E:`) indefinitely.

| File | KB | GB | Forge | Vault | Status |
|---|---|---|---|---|---|
| `ubuntu_full_stack_FINAL_20260808.tar` | 16,885,440 KB | 16.103 GB | ✅ | ✅ | **KEEP** |
| `gru_data_2_0_0_FINAL_20260808.tar` | 122,516,280 KB | 116.841 GB | ✅ | ✅ | **KEEP** |
| `gene_iobio_frontend_v4_13_1_FINAL_20260808.tar` | 310,310 KB | 0.296 GB | ✅ | ✅ | **KEEP** |

**Total FINAL set:** 139,711,730 KB / 133.24 GB

---

### Category 2: INTERMEDIATE (Superseded, safe to delete from D: only)

These restore points from earlier phases are fully superseded by the
FINAL backup set. They exist only for historical reference and can
be deleted from D: if vault copies are verified.

| File | KB | GB | Forge | Vault | Superseded by | Delete from `D:` if |
|---|---|---|---|---|---|---|
| `ubuntu-clean.tar` | 1,233,570 KB | 1.176 GB | ❌ removed | ✅ | ubuntu-baseline | Vault copy verified |
| `ubuntu-baseline-20260728.tar` | 1,240,700 KB | 1.183 GB | ❌ removed | ✅ | ubuntu-podman | Vault copy verified |
| `ubuntu-podman-20260728.tar` | 1,766,840 KB | 1.685 GB | ✅ | ✅ | ubuntu-miniforge3 | Vault copy verified |
| `ubuntu-miniforge3-20260728.tar` | 1,869,550 KB | 1.783 GB | ✅ | ✅ | ubuntu-phenolyzer | Vault copy verified |
| `ubuntu-phenolyzer-20260728.tar` | 2,391,240 KB | 2.281 GB | ✅ | ✅ | ubuntu-updated | Vault copy verified |
| `ubuntu-updated-20260730.tar` | 2,390,016 KB | 2.279 GB | ✅ | ✅ | ubuntu-gru-pulled | Vault copy verified |
| `ubuntu-gru-pulled-20260730.tar` | 12,243,968 KB | 11.677 GB | ✅ | ✅ | ubuntu-gru-verified | Vault copy verified |
| `ubuntu-gru-verified-20260801.tar` | 11,957,100 KB | 11.403 GB | ✅ | ✅ | ubuntu_gru_data_1_14_1 | Vault copy verified |
| `ubuntu_gru_data_1_14_1_20260803.tar` | 12,028,060 KB | 11.471 GB | ❌ removed | ✅ | ubuntu_full_stack_final | Already removed |
| `Ubuntu_26_04_iobio_stack_production_20260806.tar` | 16,098,000 KB | 15.354 GB | ✅ | ✅ | ubuntu_full_stack_final | Vault copy verified |

**Total INTERMEDIATE set:** 63,219,044 KB / 60.29 GB

---

### Category 3: LIVE DATA (Never delete)

These are active directories required for the stack to function.

| Directory | KB | GB | Purpose | Delete? |  
|---|---|---|---|---|  
| `D:\WSL\` | ~20,000 KB | ~0.019 GB | Ubuntu 26.04 LTS filesystem | **NEVER** |  
| `D:\gru_data\` | ~128,000,000 KB | ~122 GB | Reference data (live) | **NEVER** |  
| `~/gene.iobio/` | 536 KB | 0.000000536 GB | gene.iobio frontend | **NEVER** | 

---

## Cleanup Priority (Lowest Risk to Highest)

Execute in this order. Stop when you have reclaimed enough space.

### Priority 1: Lowest Risk — Delete oldest intermediate backups first

These are the furthest from current production and least likely to
be needed.
Delete from SSK SSD `D:` (verify vault copy first):  
   1. `ubuntu-clean.tar` (1.176 GB) ❌ removed  
   2. `ubuntu-baseline-20260728.tar` (1.183 GB) ❌ removed   
   3. `ubuntu-podman-20260728.tar` (1.685 GB)  
   4. `ubuntu-miniforge3-20260728.tar` (1.783 GB)  

**Space reclaimed if deleting 4 vault files from `D:`** 6.03 GB

**Verification before delete:**
# On vault (192.168.1.52 `E:` Seagate), verify files exist:
bash  
```  
ls -lh "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu-clean.tar"
```
bash
```  
ls -lh "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu-baseline-20260728.tar"
```  
# etc.

# Verify MD5 matches:  
# Compare MD5 from 03_Stack_Versioning_Reference.md against vault file  
### Priority 2: Medium Risk — Delete mid-phase backups  
> These are from the middle of the build sequence and are fully superseded by later phases.  
#### Delete from `D:` (verify vault copy first):  
   5. ubuntu-phenolyzer-20260728.tar (2.281 GB)  
   6. ubuntu-updated-20260730.tar (2.279 GB)  
   7. ubuntu-gru-pulled-20260730.tar (11.677 GB)  
   8. ubuntu-gru-verified-20260801.tar (11.403 GB)  
   Space reclaimed: 27.64 GB (cumulative: 33.67 GB)  
    
### Priority 3: Medium-High Risk — Delete pre-final production backup
> This is the last backup before the FINAL set.
> It is fully superseded but is the most recent intermediate backup before **Phase 8 Option A** + **Phase 8 Option B**.
#### Delete from `D:` (verify vault copy first):
   9. Ubuntu_26_04_iobio_stack_production_20260806.tar (15.354 GB)  
   Space reclaimed: 15.35 GB (cumulative: 49.02 GB)  

### Priority 4: Already Done — `gru_data` intermediate backup
> This file was already removed from `D:` during H20260808 final backup operations due to space pressure. It still exists on vault.
Status: Already removed from `D:`
    gru_data_1_14_1_20260803.tar (11.471 GB) — exists on vault only

### Verification Checklist Before Deleting
For each file you plan to delete from `D:`, complete this checklist:
    • ☐ File exists on vault (`E:` Seagate) at the documented path
    • ☐ MD5 hash of vault copy matches the documented hash in 03_Stack_Versioning_Reference.md
    • ☐ You have tested vault file accessibility from forge (can you see it in Windows Explorer or via ls over the network share?)
    • ☐ You understand what phase this backup represents and confirm it is superseded by a later backup
    • ☐ You have space available on vault if you ever need to restore this file later

### Space Reclamation Summary
| Action | Space Reclaimed | Cumulative |  
| Priority 1 (oldest 4 backups) | 6.03 GB | 6.03 GB |  
| Priority 2 (mid-phase 4 backups) | 27.64 GB | 33.67 GB |  
| Priority 3 (pre-final backup) | 15.35 GB | 49.02 GB |  
| Total possible from intermediates | 49.02 GB | — |  

> Deleting all intermediate backups (Priorities 1–3) will reclaim approximately `49 GB` of space on `D:`,  
> leaving the `FINAL` backup set (`133.24 GB`) plus live data (`122 GB WSL` + `gru_data`).  

### Recommended Approach  
   1. Do not delete anything right now. `D:` has adequate free space (675.8 GB remaining).
   2. Monitor `D:` usage over the next few build sessions. **If you approach 85% full, start with Priority 1**.
   3. Keep the `FINAL` backup set on both `forge` and `vault` indefinitely. These are your production restore points.
   4. Delete intermediate backups from `D:` only — never from `vault`. `Vault` is your archive.  

### Disaster Scenario: Restoring from Intermediates  
If you ever need to restore from an intermediate backup (e.g., debugging a specific phase), you will need to:  
   1. Copy the intermediate tar from `vault` back to `forge` `D:`
   2. Extract and use it per the restore procedure in `Disaster_Recovery.md`
This is why `vault` copies are kept — they are your insurance policy.

### Future Backups
When you create new FINAL backup sets in future sessions:
   1. Update `03_Stack_Versioning_Reference.md` with new file names, sizes, and MD5 hashes
   2. Update this cleanup priority document with the new FINAL set
   3. The old FINAL set becomes an intermediate backup candidate for eventual deletion from `D:` (but keep on `vault`)
   4. Follow the same verification checklist before deleting anything

---

≡≡≡≡≡ END: D_Drive_Cleanup_Priority.md ≡≡≡≡≡  
