# Complete GOTCHA Enumeration — gene.iobio Stack Build
*Phase-numbered format for cross-reference with 01_Stack_Build_Steps.md*
*Format: Phase.Item (e.g., 1.1, 2.3)*
*Last updated: H20260808*

---

## Phase 1 — WSL + Ubuntu Base

**1.1 WSL install location flag required**
`wsl --install --location D:\WSL` needed to keep off C: drive.
Without it, defaults to C: (space pressure).

**1.2 Default user must be set BEFORE first boot**
`/etc/wsl.conf` `[user] default=mir` required in advance or you
will be stuck as root on first launch.

**1.3 Export requires reboot-first sequence**
`wsl --shutdown` then `wsl --export` only works cleanly after
reboot. No WSL terminals open before export command.

**1.4 C: pagefile management**
Disable C: pagefile before WSL swap setup via `sysdm.cpl`,
re-enable after swap is persistent in `/etc/fstab`. Swap pressure
can corrupt WSL install.

**1.5 WSL distribution naming in export**
`wsl --export` requires distribution name as registered in WSL
(e.g., `Ubuntu`), not version number (e.g., `Ubuntu-26.04`).
Use `wsl --list -v` to confirm registered name before exporting.

---

## Phase 2 — Core Tools

**2.1 Miniforge3 installer must be run in batch mode**
`bash ~/Miniforge3.sh -b -p /mnt/d/miniforge3` — the `-b` flag
suppresses interactive prompts. Without it, the installer halts
waiting for keyboard input. The `-p` flag sets the install path
to D: drive, keeping it off the WSL root filesystem.

**2.2 conda init requires shell reload**
After `/mnt/d/miniforge3/bin/conda init bash`, the shell must be
reloaded via `source ~/.bashrc` before `conda` and `mamba` commands
are available. Opening a new terminal also works.

**2.3 mamba vs conda for bioconda packages**
Use `mamba install` (not `conda install`) for bioconda packages
like samtools. Mamba resolves dependencies significantly faster.
`conda` will work but is noticeably slower on low-RAM systems.

---

## Phase 3 — Podman Container Storage Fix

**3.1 Overlay storage driver incompatible with WSL ext4**
Podman fails silently or hangs with overlay driver on WSL2 ext4
filesystem. Fix: set `driver="vfs"` in `containers.conf`
`[storage]` section. VFS is slower but fully compatible.

**3.2 vm.overcommit_memory must be persistent**
Set `vm.overcommit_memory=1` in `/etc/sysctl.conf` — not just
at the command line — or it resets on reboot and Podman fails
again with memory allocation errors.

**3.3 Podman image_parallel_copies conflict**
Set `image_parallel_copies=1` in `containers.conf` `[engine]`
to avoid simultaneous pull conflicts on resource-constrained
systems.

**3.4 runRoot path matters for WSL**
Explicitly set
`runRoot="/home/mir/.local/share/containers/run"` to avoid
socket conflicts with rootless Podman on WSL2.

---

## Phase 4 — gru-backend Image Pull

**4.1 .wslconfig RAM must be temporarily increased for pulls**
Set `.wslconfig` to `memory=3GB` before pulling large images
(4+ GB). Revert to `memory=2GB` after pull completes. Pulling
on 2GB can stall or fail due to decompression memory pressure.

**4.2 Container name does not confirm image version**
A container named `gru-backend-2.0.0` does not guarantee the
2.0.0 image is running. Always verify with:
```bash
podman ps --format "{{.Names}} {{.Image}}"
```
The confirmed 2.0.0 image ID is `7e2c4a419267`. This was the
root cause of the Phase 7 API mismatch — the 1.17.0 image was
running under the 2.0.0 container name.

---

## Phase 5 — gru_data Directory Build

**5.1 Dead data sources — do not retry**
The following sources are confirmed dead. Do not attempt:
- `rsync://data.iobio.io:9009/gru/` — connection timed out
- `https://s3.amazonaws.com/iobio/assets/data-vol/v2.0/` — 403
  Forbidden
- `yiq/gene.iobio.local fetch-data.sh` — S3/rsync dead, script
  not skippable, entire approach obsolete

Use `https://files.iobio.io` via rclone HTTP backend only.
See GitHub issue #129.

**5.2 rclone HTTP backend cannot reliably compare modification
times**
For small files where size may match across versions (e.g.,
`VERSION`), pre-delete the file before syncing to guarantee
rclone pulls the fresh copy. Do NOT pre-delete large files
(vep-cache/, references/) — let rclone skip those by size match.

**5.3 rclone HTTP backend may re-download unverifiable files**
Even if already present and correct, rclone may re-download if
it cannot confirm size/mtime via HTTP headers. Verify MD5
post-sync for any files that unexpectedly re-transferred.

**5.4 rclone sync always logs one 403 on lost+found/**
Server-side filesystem artifact, not accessible via browser
either. Not a transfer error. Expected behavior on every sync.
Safe to ignore.

**5.5 rclone character flag: :http: is colon-http-colon**
Not `://`. Easy to misread. Copy/paste strongly recommended
to avoid typos on this specific sequence.

**5.6 rclone sync is idempotent — safe to re-run**
Running rclone sync multiple times produces the same result as
running it once. Completed files are skipped. Partially
downloaded files restart from the beginning. Safe to re-run
after interruption without risk of duplication or corruption.

**5.7 2.0.0 update path vs. full dataset path — critical**
When upgrading gru_data 1.14.1 → 2.0.0, two rclone paths exist:
- UPDATE path `updates_1.14.0_to_2.0.0/` = delta only, ~77GiB,
  INCOMPLETE
- FULL path `gru_data/data/gru_data_2.0.0/` = complete dataset,
  ~128GiB

Always use the FULL path. Running full path after update path
downloads only missing files (~9.8GiB in this build). Always
check GitHub README for major version transitions — documentation
may lag on files.iobio.io directory structure.

**5.8 Fresh 2.0.0 build: skip 1.14.1 entirely**
If building fresh on gru_data 2.0.0 or later, rclone directly
from the full 2.0.0 data directory. Do not download 1.14.1 first.
Net savings: ~42GiB of downloads and significant time.

**5.9 VERSION file tracks gru_data version independently**
`VERSION` (e.g., `2.0.0`) is gru_data directory version. Docker
image tag (e.g., `2.0.0`) is a separate versioning track. Do not
conflate them — they happen to match in this build but are not
always synchronized.

**5.10 gru_data 2.0.0 directory fully restructured from 1.14.1**
Old top-level directories: `gnomad/`, `geneinfo/`, `hpo/`,
`gene2pheno/`, `genomebuild/`, `data/`, `references/`,
`md5_reference_cache/`
New: consolidated into `annotations/GRCh37/`,
`annotations/GRCh38/`, and `vep-cache/`.
36,073 deleted files + 6,217 deleted directories during
transition — expected and correct.

**5.11 2.0.0 FILES_TO_DELETE is space recovery only**
Old files do not break 2.0.0 function if present — simply
ignored by gru-backend 2.0.0. Safe to delay deletion until
2.0.0 is confirmed stable. Provides rollback safety net.

**5.12 Disk space: old and new files coexist during transition**
Verify D: has sufficient headroom before starting 2.0.0 upgrade.
Net disk space recovery (~75GB) happens only after
FILES_TO_DELETE applied.

**5.13 FILES_TO_DELETE sequencing for 2.0.0 upgrade**
Apply FILES_TO_DELETE AFTER rclone sync completes and new files
are verified present. NEVER delete before download. Sequence:
(1) rclone sync new files, (2) verify complete, (3) apply
FILES_TO_DELETE, (4) upgrade gru-backend image to 2.0.0,
(5) verify HTTP 200, (6) WSL export.

**5.14 GRCh38 FASTA uses UCSC chr1-style intentionally**
iobio ships `human_g1k_v38_decoy_phix.fasta` with `chr1`,
`chr2` contig naming (UCSC style) not Ensembl `1`, `2`. This
is intentional. VEP cache subdirectory numbers (`1`, `2`, `3`)
are filesystem sharding labels only, not contig name assertions.

**5.15 VEP cache tarballs extract with extra nesting**
Both GRCh37 and GRCh38 VEP cache tarballs extract with unwanted
`homo_sapiens_merged/` parent directory. Fix post-extraction:
`mv homo_sapiens_merged/109_* .` then
`rmdir homo_sapiens_merged/`. Future builds: use
`--strip-components=1` during extraction.

**5.16 Plugins directory must exist but may be empty**
VEP cache extraction creates `Plugins/` as an empty directory.
Do not delete it — VEP expects it present even if unused.

**5.17 md5_reference_cache pre-populated by iobio in 2.0.0**
`seq_cache_populate.pl` will show "Already exists" for all
entries. This is correct and expected. Files were pre-generated
by iobio. Running the script is verification only, not
population.

**5.18 seq_cache_populate.pl requires samtools in PATH**
Script fails silently if samtools not installed. Install
`sudo apt install samtools` before running
`seq_cache_populate.pl`.

**5.19 REF_CACHE environment variable uses %2s format**
Path is
`/mnt/d/gru_data/md5_reference_cache/%2s/%2s/%s` with `%2s`
as printf-style format specifiers for hex directory sharding.
Do not substitute manually.

**5.20 rclone bandwidth limiting during shared network use**
Use `--bwlimit 5M` flag to be polite to shared network
connections during on-call or active-use periods. Transfer
continues at reduced speed — no connection drop needed.

**5.21 rclone sync is resumable but partial files restart**
Completed files skip on resume. Partially downloaded files
restart from the beginning. Large files in progress should be
allowed to complete before stopping if possible.

**5.22 files.iobio.io is slow during peak hours**
New versions see heavy simultaneous user download activity.
ETAs are unreliable during US/Philippines business hours.
Normal behavior, not a failure. Known traffic distribution:
US 60.3%, Philippines 11.5%, Netherlands 10%, Algeria 7.9%,
Israel 5.3%.

---

## Phase 6 — Backend Verification

**6.1 Podman container does not stop with Ctrl+C**
Ctrl+C in foreground terminal prints `^C` characters but does
not stop the container. Use `podman stop <container_id>` or
`podman stop --all` from a second terminal.

**6.2 WARN "not a shared mount" message is harmless**
`WARN[0000] "/" is not a shared mount` appears on every
gru-backend startup. Expected, harmless, specific to rootless
Podman on WSL2. Volume mounts and port forwarding work correctly
despite the warning. Ignore always.

**6.3 Backend startup does not return to prompt**
`podman run --rm -it` runs in foreground. Blinking cursor =
server running and waiting for connections. Open a second
terminal to test with `curl localhost:9001`.

**6.4 Health endpoint format differs between image versions**
- 1.17.0: `<h1>I be healthful</h1>` (HTML string)
- 2.0.0: `{"service_description":"iobio gru backend
  server","version":"2.0.0","data_version":"2.0.0"}` (JSON)

Confirms correct image is running. Check format to verify
image version.

**6.5 Hung gru-backend — signs**
Audible disk thrashing, terminal slowness, blinking cursor on
commands, `podman stop` hangs >15s, browser XHR requests stuck
"pending", no graceful shutdown response.

**6.6 Hung gru-backend — resolution**
(1) Try `podman stop gru-backend-2.0.0`, wait 10–15s.
(2) If hangs: `podman kill gru-backend-2.0.0` (SIGKILL,
immediate).
(3) Verify stopped: `podman ps | grep gru` (no output = stopped).
(4) Restart: `podman run --rm -it ...` per Quick Reference.
Do not use `kill -9` on Podman processes directly. Do not
force-kill while terminal is showing output. Give system 5s
to settle after kill before restarting.

---

## Phase 7 — Frontend Build

**7.1 Node v13.14 is insufficient**
gene.iobio requires Node v16+ minimum. NVM install required for
v16+ version management. Use `nvm install 16` and
`nvm use 16` before building.

**7.2 Frontend requires a webpack build step**
Not static file serve. `npm run build` is required before
serving. Build process is non-trivial (~29 seconds on this
hardware) and must complete without errors before serving.

**7.3 Browser cache masks backend changes**
DevTools "Disable cache" checkbox essential during development.
Hard refresh (`Ctrl+Shift+R`) required after backend changes to
avoid stale cached responses.

**7.4 geneinfo.js native implementation in 2.0.0**
gru-backend 2.0.0 image ships with native geneinfo.js (package
v0.3.0) that handles `lookupEntries` correctly at lines 99-100
and 398. No patching required. Master branch patch from
H20260805 may be redundant but not harmful if already applied.

**7.5 gene.iobio v4.13.1 runtime-config architecture**
Deployment settings are loaded from `client/config.json` at
runtime instead of webpack `.env` values. This is a deployment
architecture change introduced in v4.13.1. Use `client/
config.json` for all local deployment configuration.

**7.6 Version compatibility: v4.13.1 + gru-backend 2.0.0**
Confirmed production pairing. Validated end-to-end with
production WGS data. No code changes required beyond the
Phase 8A gate fixes.

**7.7 Always inspect existing codebase before estimating scope**
Phase 8A demonstrated that a feature estimated as "substantial
work" (12–24 hours, 50–100 lines of code) was already fully
implemented — just gated incorrectly. Three one-line edits and
a 29-second rebuild was the actual fix. Check upstream master
branch and existing code before assuming a feature is missing.

---

## Phase 8A — CSV/VCF Export Feature Unlock

**8A.1 Export feature was already built — not missing**
CSV and VCF export were fully implemented in v4.13.1 but
inaccessible due to two broken conditional gates. The feature
did not need to be built from scratch. Always inspect the
existing codebase before assuming a feature is missing or
estimating implementation scope.

**8A.2 Export gate condition never satisfied**
Both `FlaggedVariantsCard.vue` (line 843) and
`Navigation.vue` (line 952) gated export on:
`cohortModel.flaggedVariants && cohortModel.flaggedVariants.length > 0`
Auto-populated variants during analysis land in a different data
structure — not `cohortModel.flaggedVariants`. This condition
was never true during normal use. Fix: change gate to
`cohortModel.isLoaded`.

**8A.3 CohortModel.js coverage refresh blocks export on large sets**
`CohortModel.js` line 2903: `reject(error)` in the coverage
refresh catch block caused export to fail when BAM depth
enrichment timed out on large variant sets. Fix:
`reject(error)` → `resolve()` allows export to proceed even
if coverage refresh fails. Large unfiltered variant sets
(60+ variants) may still be slow — this is expected BAM depth
overhead, not a bug. Filter before exporting for best results.

**8A.4 Always back up files before editing**
Back up all three files as `.bak` before editing:
`FlaggedVariantsCard.vue.bak`, `Navigation.vue.bak`,
`CohortModel.js.bak`. After rebuild, verify `.bak` files show
only the three intended edits and no leftover cruft from prior
sessions.

**8A.5 build.js ordering when applying multiple phases**
Both Phase 8A and Phase 8B require `npm run build`. Always apply
Phase 8A first, then Phase 8B. Never substitute a saved
`build.js` when applying both — the final build must incorporate
all changes from both phases in sequence.

---

## Phase 8B — LAN Network Serving + Static WSL IP

**8B.1 serve -H flag does not exist in this version**
The `-H` flag for host binding does not exist in this version
of serve. Not needed — serve binds to `0.0.0.0` by default.
Use `serve -s . -l 3000` only.

**8B.2 serve reports WSL internal IP — not the LAN IP**
serve output shows `172.18.x.x:3000` (WSL internal IP), not
`192.168.1.240:3000`. This is normal and expected. Windows
`netsh portproxy` handles routing from the Windows LAN IP to
the WSL internal IP. Do not be alarmed by the WSL IP in serve
output — it does not mean LAN access is broken.

**8B.3 Both ports must be forwarded — not just frontend**
Frontend (3000) AND backend (9001) each need their own
`netsh portproxy` rule. Missing the 9001 rule causes
`ERR_CONNECTION_TIMED_OUT` on all backend requests even when
the frontend loads fine. Both firewall rules and both portproxy
rules are required.

**8B.4 WSL internal IP can change on restart — lock it**
WSL2 runs on a Hyper-V virtual network (172.18.48.0/20). By
default, Hyper-V assigns the WSL guest IP dynamically — it can
change on restart, invalidating portproxy rules. Lock it
permanently via netplan inside WSL (see Phase 8B). This is not
optional for a stable LAN serving setup.

**8B.5 netplan file permissions must be exactly 600**
`/etc/netplan/01-netplan.yaml` must have permissions `600`
(readable only by root). Use `sudo chmod 600 /etc/netplan/
01-netplan.yaml`. Incorrect permissions prevent netplan from
applying the configuration and will produce a permissions
warning on apply.

**8B.6 netplan apply warns about systemd in WSL2 — harmless**
WSL2 does not run systemd as PID 1. `sudo netplan apply` will
produce a warning about this. Expected behavior — does not
affect static IP assignment. The netplan configuration is read
on WSL startup regardless of the systemd warning.

**8B.7 Use modern routes syntax — not deprecated gateway4**
In `/etc/netplan/01-netplan.yaml`, use:
```yaml
routes:
  - to: 0.0.0.0/0
    via: 172.18.48.1
```
Do not use `gateway4: 172.18.48.1` — this syntax is deprecated
in current netplan versions and will produce warnings or errors.

**8B.8 WSL host-side gateway is always 172.18.48.1**
The Hyper-V virtual network host-side address
(`vEthernet (WSL)`) is `172.18.48.1`. This is the correct
gateway for the netplan configuration. The router does not
see this network — static IP must be configured inside WSL
itself, not via router DHCP.

**8B.9 Verify static IP persists across full wsl --shutdown**
After applying netplan configuration, verify persistence with
a full WSL restart:
```bash
exit  # exit WSL
wsl --shutdown  # full shutdown in PowerShell
wsl  # relaunch
hostname -I  # must return 172.18.60.70
```
A simple terminal exit and re-open is not sufficient — must
use `wsl --shutdown` to test true persistence.

---

## Post-Build — Backup and Recovery

**B.1 WSL export captures environment only — not D: drive**
WSL tar exports contain the Ubuntu environment and container
images only — NOT `/mnt/d/gru_data/` (lives on D: outside WSL).
gru_data requires a separate backup (tar archive or rclone
re-sync).

**B.2 gru_data tar — use uncompressed**
Uncompressed tar (`tar -cf`) preferred for already-compressed
files (gnomAD, VEP cache, annotation files). Compression adds
CPU time with minimal size savings on pre-compressed content.

**B.3 MD5 verification at every copy step**
Always verify copied files with md5sum before and after network
transfer. Hash mismatch indicates incomplete or corrupted
transfer. Re-copy if mismatch detected.

**B.4 PowerShell Get-FileHash outputs uppercase MD5**
Linux `md5sum` outputs lowercase. Both are identical values —
comparison is case-insensitive. Document both formats for
cross-platform verification clarity.

**B.5 Sequential backup operations**
Do not run WSL export while rclone or tar operations are
running. Sequential operations prevent I/O contention and
reduce error risk.

**B.6 Job control only works in WSL bash terminal**
Not PowerShell. For long rclone/tar operations in WSL bash:
`Ctrl+Z` suspends, `bg` resumes in background, `fg` brings to
foreground, `jobs` lists. WSL terminal only.

**B.7 WSL export requires a final rebuild after each new phase**
If a new phase modifies the WSL environment (packages,
configs, frontend files), the WSL export must be re-run to
capture those changes. The `ubuntu_full_stack_FINAL_20260808.tar`
superseded the `ubuntu_full_stack_final_20260807.tar` because
Phase 8B (static WSL IP via netplan) modified the WSL
environment after the first export was taken.

---

## Post-Build — Known Limitations

**L.1 Modifier impact section browser rendering**
Browser rendering can crash on whole-genome variant lists
(thousands of intronic/UTR variants). Expected behavior on
large datasets, not a stack bug. Filter before expanding the
Modifier impact section on WGS data.

**L.2 CSV/VCF export on large unfiltered sets**
Exports of 60+ unfiltered variants may be slow due to BAM
depth coverage refresh overhead. This is expected behavior.
Filter to a manageable variant set before exporting for best
results.

**L.3 Vue development mode warning**
Vue development mode warning appears in browser console.
Expected — not a problem for local/LAN use. Does not affect
functionality.

**L.4 HTTPS warning on data URIs**
Browser compliance notice only. Export files download
correctly despite the warning.

**L.5 netplan apply systemd warning**
WSL2 does not run systemd. `sudo netplan apply` warns about
this on every run. Expected, non-fatal, does not affect static
IP assignment.

---

*Total: 62 GOTCHAs — phase-numbered to match
01_Stack_Build_Steps.md for reproducibility and
cross-reference during troubleshooting.*
---

---

≡≡≡≡≡ END: 04_GOTCHA_Enumeration.md ≡≡≡≡≡
