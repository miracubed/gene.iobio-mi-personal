# 07_OVERVIEW.md — Project Overview

## What Is This?

This repository contains a complete, reproducible procedure guide 
for building a fully local installation of the
[gene.iobio](https://github.com/iobio/gene.iobio) genomic variant
analysis platform on a Windows 11 Home laptop using WSL2 (Windows
Subsystem for Linux, version 2).

When the stack is running, you get a full-featured genomic variant
browser — accessible from a web browser on your local machine or
local network — that can:

- Load whole-genome or whole-exome sequencing data in CRAM/VCF
  format
- Annotate variants against gnomAD 4.1, ClinVar, and VEP 115
- Call variants on-demand using Freebayes
- Filter, review, and export variants in CSV or VCF format
- Run phenotype-driven gene prioritization via Phenolyzer
- Operate entirely offline — no data ever leaves your machine

> See **`Glossary`: `Freebayes`** (below)
---

## Who Is This For?

This build was created by and for someone analyzing their own
whole-genome sequencing data in the context of a rare disease
diagnosis, under significant hardware and resource constraints.

It may be useful to you if you are:

- A patient or caregiver who has obtained WGS/WES data and wants
  to analyze it privately, without uploading to external platforms
- Anyone with a poor internet connection still needing backend
  services in areas with poor connectivity.
- A researcher or clinician who needs a self-contained local
  gene.iobio instance for sensitive data
- A developer who wants to run gene.iobio locally for testing or
  development purposes
- Anyone who has struggled to get the iobio stack running locally
  and wants a step-by-step procedure that is known to work

This is not an official iobio project. It is an independent build
procedure documented by a community user.

---

## What Hardware Was This Built On?

| Item | Value |
|---|---|
| Machine | LAPTOP-F (HealthMed Laptop) |
| OS | Windows 11 Home |
| RAM | Limited — 2GB allocated to WSL2 |
| Storage | 931GB NTFS drive (D:) |
| Network | Home LAN, two machines |

This is a modest consumer laptop — not a server, not a workstation.
If this build works here, it will work on comparable or better
hardware.

---

## What Does the Stack Consist Of?

The stack has three main components:

### 1. gru-backend (the data engine)
A containerized backend service (Docker/Podman image) that handles
all data queries — variant annotation, VEP processing, gene lookups,
and gnomAD queries. Runs in a Podman container on WSL2.

- Image: `docker.io/iobio/iobio-gru-backend:2.0.0`
- Serves on port 9001
- Requires the gru_data directory (~128GiB) mounted at `/gru_data`

### 2. gru_data (the reference dataset)
A large directory of reference data (~128GiB) that the backend
requires to function. Includes VEP cache, gnomAD annotations,
reference FASTA files, gene information databases, HPO data, and
more.

- Version: 2.0.0
- Synced from `https://files.iobio.io` via rclone
- Lives at `/mnt/d/gru_data/` on the WSL filesystem

### 3. gene.iobio frontend (the user interface)
A Vue.js web application that provides the browser-based variant
analysis interface. Built locally from source and served via a
static file server.

- Version: v4.13.1-runtime-config (commit cb28d29a)
- Built with webpack, served on port 3000
- Modified with Phase 8A (CSV/VCF export) and Phase 8B
  (LAN network serving) patches

---

## Two-Machine Architecture

This build uses two machines on a home LAN:

| Working name | Role | IP |
|---|---|---|
| forge | Active build and runtime environment | 192.168.1.240 |
| vault | Secure storage and backup environment | 192.168.1.52 |

**forge** (LAPTOP-F, HealthMed Laptop) is where the stack runs.
All commands in `01_Stack_Build_Steps.md` are executed on forge
unless explicitly noted otherwise.

**vault** (LAPTOP-1, Personal Laptop) is where backups and
reference data copies are stored. It does not run any part of
the stack.

These are working shorthand names used in the documentation to
keep things readable. Full machine names and IPs are always
provided alongside them.

> **Why these names?** "Forge" implies building and running —
> this is where things are made. "Vault" implies safe storage —
> this is where things are kept. They are documentation
> conventions only, not hostnames or network identifiers.

---

## Build Phases at a Glance

| Phase | What happens |
|---|---|
| Phase 1 | WSL2 + Ubuntu 26.04 LTS installed to D: drive |
| Phase 2 | Core tools installed (Podman, Miniforge3, Perl, Phenolyzer, rclone, samtools) |
| Phase 3 | Podman container storage configured for WSL2 compatibility |
| Phase 4 | gru-backend 2.0.0 image pulled from Docker Hub |
| Phase 5 | gru_data 2.0.0 (~128GiB) synced from files.iobio.io |
| Phase 6 | Backend verified (HTTP 200, health endpoint confirmed) |
| Phase 7 | Frontend built from source, validated end-to-end with real WGS data |
| Phase 8A | CSV/VCF export unlocked (three one-line code fixes) |
| Phase 8B | LAN network serving configured (netsh portproxy + firewall rules) |

---

## What Has Been Validated

This stack has been validated end-to-end against real production
whole-genome sequencing data (GRCh37):

- Gene lookup and variant card rendering ✅
- CRAM + VCF loading (62 VCF variants + 224 Freebayes on-demand
  calls) ✅
- Variant annotation (VEP 115, gnomAD 4.1, ClinVar) ✅
- CSV and VCF export with full annotations ✅
- Phenolyzer phenotype-driven gene prioritization ✅
- LAN access from a second machine on the same network ✅

---

## Known Limitations

- **Large variant sets:** The "Modifier impact" section in the UI
  can cause browser crashes when displaying thousands of
  intronic/UTR variants from whole-genome data. This is a browser
  DOM rendering limit, not a stack bug. Filter before expanding.
- **CSV/VCF export on large sets:** Exporting 60+ unfiltered
  variants may be slow due to BAM depth coverage refresh overhead.
  Filter to a manageable set before exporting.
- **WSL IP persistence:** The WSL2 internal IP address can change
  after a system restart, breaking LAN serving. Run `hostname -I`
  in WSL and update the netsh portproxy rules if this happens.
  See Phase 8B troubleshooting in `01_Stack_Build_Steps.md`.

---

## Where to Start

If you want to build this stack yourself, start here:

1. Read `01_Stack_Build_Steps.md` — the full reproducible build
   procedure from Phase 1 through Phase 8B.
2. Read `04_GOTCHA_Enumeration.md` — indexed list of things that
   can go wrong and how to fix them.
3. Check `03_Stack_Versioning_Reference.md` if you need a quick
   version lookup without reading the full procedure.

If you want to understand how this documentation system works:

- `08_WORKFLOW.md` — how sessions are run and documents are
  maintained
- `09_MEMORY_MAINTENANCE.md` — how the AI memory system is
  managed across sessions

---

## Glossary

## Glossary pf Terms used throughout the **`gene.iobio-mi-personal`** Github repository.
## Terms in Order of Appearance.  

| Term | First Appears | Definition |  
|---|:---:|---|  
| `Freebayes` | **`07_OVERVIEW.md`** | A Bayesian, haplotype-based genetic variant detector designed to identify small polymorphisms in high-throughput sequencing data. It detects SNPs, indels, MNPs, and complex events by analyzing the literal sequences of reads rather than their precise alignment, which helps resolve ambiguities in repetitive regions. It accepts BAM/CRAM alignments and a reference genome as input, outputting variants in VCF format, and is particularly noted for its ability to handle pooled samples and non-diploid genomes. |  
| `WSL` | **`Throughout`** | `Windows Subsystem for Linux` essentially synonymous to `WSL2`. Lets you run a Linux environment (Ubuntu) directly inside Windows without a separate machine or virtual machine. |  
| `forge` | **`Throughout`** | Working name for LAPTOP-F (HealthMed Laptop, 192.168.1.240) — the active build and runtime environment where the stack runs. |  
| `vault` | **`Throughout`** | Working name for LAPTOP-1 (Personal Laptop, 192.168.1.52) — the secure storage and backup environment. |  
| `sudo` | **`Throughout`** | Linux command prefix meaning "run as administrator." Required for commands that modify system files or settings. |  
| `md5`/`MD5` | **`Throughout`** | A checksum algorithm that produces a fixed-length fingerprint of a file. Used here to verify that backup files copied correctly — if the MD5 of the source and destination match, the file is. `md5` is one of many `hash` `checksum` types used to verify file integrity. |  
| `~ (tilde)` | **`Throughout`** | The `~ (tilde)` character is representative of the user `~/home` directory. |  
| `PowerShell` | **`Phase 1 Step 1`** | Windows command-line shell. Used for WSL management, firewall rules, and port forwarding. Different from the WSL/Ubuntu terminal — runs Windows commands, not Linux commands. |  
| `Administrator (elevated)` | **`Phase 1 Step 1`** | Running a program with full system permissions. Required for certain Windows operations. Right-click the program and select `Run as administrator`. |  
| `wsl.conf` | **`Phase 1 Step 2`** | Configuration file inside the WSL install that sets startup behavior — including which user to log in as by default. Must be set before first boot. |  
| `pagefile` | **`Phase 1 Step 4`** | Windows virtual memory file. Supplements physical RAM by using disk space. Required for `Podman` stability on memory-constrained systems. |  
| `sysdm.cpl` | **`Phase 1 Step 4`** | Windows System Properties control panel. Accessed via Run dialog (Windows+R). Used here to manage virtual memory settings. |  
| `pull` | **`Phase 1 Step 4`** | Pull = Download: It describes the direction of data flow: from remote → to local. The local machine initiates the request and "pulls" data from the server or remote storage. |  
| `fallocate` | **`Phase 1 Step 5`** | Linux command that pre-allocates disk space for a file. Used here to create the swap file. |  
| `fstab` | **`Phase 1 Step 5`** | Linux file (/etc/fstab) that defines filesystems and swap space to mount automatically at boot. |  
| `.wslconfig` | **`Phase 1 Step 6`** | Windows-side configuration file for `WSL2`. Controls RAM allocation, CPU count, swap, and networking behavior. Lives in the Windows user profile folder. |  
| `sysctl.conf` | **`Phase 1 Step 7`** | Linux file (`/etc/sysctl.conf`  ) for persistent kernel parameter settings. Used here to set vm.overcommit_memory=1 across reboots. |  
| `nano` | **`Phase 1 Step 9`, `Throughout`** | Simple text editor that runs in the Linux terminal. Used throughout this build for creating and editing configuration files. To open a file: nano filename. To save: Ctrl+O, Enter. To exit: Ctrl+X. To open at a specific line: `nano +linenumber filename`. To paste: right-click or `Edit` > `Paste` in the terminal window. |  
| `Podman` | **`Phase 1 Step 9`** | A tool for running containers — self-contained software packages — without needing a background service running as root. Similar to Docker but rootless-friendly. A **container** (sometimes called a Docker image) packages an application and all its dependencies into one portable unit. |  
| `apt` | **`Phase 2 Step 1`** | Ubuntu's package manager. Used to install, update, and remove software. sudo apt install is the standard way to install packages in this build. |  
| `Phenolyzer` | **`Phase 2`** | A tool that ranks candidate genes based on phenotype descriptions — you describe symptoms, it scores genes by relevance. Used here for rare disease gene prioritization. |  
| `gru_data` | **`Phase 2`** | The reference data directory required by the `gru-backend`. Contains `VEP cache`, `gnomAD annotations`, `reference FASTA` files, and supporting databases. Named by the `iobio` project. |  
| `SAMtools` | **`Phase 2 Step 1`** | It specifically handles the SAM (Sequence Alignment/Map), BAM (Binary Alignment/Map), and CRAM file formats. Key functions include sorting and indexing large alignment files, converting between formats, filtering data, and performing variant calling (identifying SNPs and indels). |  
| `aria2` | **`Phase 2 Step 1`** | A lightweight, multi-protocol command-line download utility. It maximizes speed by splitting files into segments to download from multiple sources simultaneously (HTTP, BitTorrent, Metalink). Known for its minimal memory usage and ability to resume interrupted downloads. |   
| `Miniforge3` | **`Phase 2 Step 1`** |  installer for the Conda package and environment manager. Miniforge3 comes pre-configured to use only the open-source conda-forge channel by default, avoiding proprietary repositories. It typically includes both the standard conda solver and the faster mamba solver, making it a lightweight and fully open-source choice environments in Python, R, and others. |   
| `cpanminus` | **`Phase 2 Step 4`** | Perl package manager. Used to install `Perl` (programming language) modules not available via `apt`. Required before `Phenolyzer` dependencies. |  
| `conda` / `mamba` | **`Phase 2 Step 5`** | Package managers for scientific software. Provided by `Miniforge3`. `mamba` is faster than `conda` for resolving dependencies — use `mamba` for `bioconda` packages like `SAMtools`. |  
| `wget` | **`Phase 2 Step 5`** | Linux command-line download tool. Used here to download the `Miniforge3` installer. |  
| `mkdir` | **`Phase 3 Step 1`** | Linux command to create directories. The `-p` flag creates the full path including any missing parent directories. |  
| `VEP` | **`Phase 4`** | `Variant Effect Predictor`. A tool developed by Ensembl that annotates genetic variants — it looks up what each variant does, where it falls in the genome, and what databases say about it. |   
| `WSL2`  | **`Phase 4 Step 1`** | `Windows Subsystem for Linux version 2` essentially synonymous with `WSL`. A compatibility layer built into Windows 11 that lets you run a full Linux environment without a separate machine or virtual machine.|  
| `rclone` | **`Phase 5`** | Command-line tool for syncing files to and from cloud and HTTP storage. Used here to download `gru_data` from `files.iobio.io`. |  
| `sync` (`rclone`) | **`Phase 5 Step 4`** | `rclone` operation that makes the destination match the source — downloading new or changed files and skipping already-present ones. |  
| `idempotent` | **`Phase 5 Step 4`** | A process that produces the same result whether run once or many times. `rclone` `sync` is `idempotent` — safe to re-run after interruption without risk of duplication or corruption. |  
| `gnomAD` | **`Phase 5 Step 6`** | Genome Aggregation Database. A large public database of human genetic variation used to determine how common or rare a given variant is in the general population. |   
| `NVM` | **`Phase 7 Step 1`** | `Node Version Manager`. Allows installing and switching between `Node.js` versions. Required because the system `Node.js` version in Ubuntu is insufficient for `gene.iobio`. |  
| `Node.js` / `npm` | **`Phase 7 Step 1`** | `Node.js` is a JavaScript runtime. `npm` is its package manager. Both are required to install dependencies and build the `gene.iobio frontend`. |  
| `git checkout` | **`Phase 7 Step 4`** | `Git` command to switch to a specific branch, tag, or commit. Used here to select the exact verified version of `gene.iobio`. |  
| `commit` | **`Phase 7 Step 4`** | A saved snapshot of a codebase at a specific point in time. Identified by a hash (e.g., cb28d29a). Checking out a specific commit ensures you are building exactly the verified version. | 
| `config.json` | **`Phase 7 Step 6`** | Runtime configuration file for `gene.iobio` (`client/config.json`). Tells the frontend where to find the backend. Edited directly — no rebuild needed for config changes alone, but a rebuild is required for the changes to take effect in the served app. |  
| `serve` / `npx` | **`Phase 7 Step 8`** | `serve` is a lightweight static file server used to serve the built gene.iobio frontend. npx is an npm tool runner — used as a temporary substitute if serve is not yet globally installed. |  
| `CRAM` | **`Phase 7 Step 10`** | A compressed format for storing aligned sequencing reads — the raw output of whole-genome sequencing after alignment to a reference genome. Smaller than BAM format.|  
| `VCF` | **`Phase 7 Step 10`** | `Variant Call Format`. A standard file format for storing genetic variant calls — the list of positions in the genome where an individual's DNA differs from the reference.|  
| 'data URI` | **Phase 7 Step 10`** | A Data URI (Uniform Resource Identifier) is a scheme that allows small files, such as images, text, or stylesheets, to be embedded directly inline within a document rather than linked as external resources. |  
| `cruft` | **`Phase 8 Option A`** | Leftover files or artifacts from previous build attempts or failed sessions. When we check for cruft, we are verifying that no unintended files remain from earlier work that could interfere with the current build. | 
| `LAN` | **`Phase 8 Option B`** | `Local Area Network`. The private network connecting devices in your home or office. LAN serving means the stack is accessible to other devices on the same network without being exposed to the internet. |  
| `Windows Firewall` | **`Phase 8 Option B Step 3`** | Windows security feature that controls which network connections are allowed. Inbound rules must be added for `ports 3000` and `9001` to allow LAN devices to reach the frontend and backend. |  
| `netsh portproxy` | **`Phase 8 Option B Step 4`** | Windows built-in tool that forwards network traffic from one address and port to another. Used here to route LAN requests arriving at the Windows IP down into the WSL virtual network where the services are running. |  
| `Hyper-V virtual network` | **`Phase 8 Option B Step 4`** | Internal Windows virtual network that WSL2 runs on. Uses the 172.18.x.x address range. Not visible to the router — static IP must be configured inside WSL itself. |  
| `netplan` | **`Phase 8 Option B Step 6`** | Network configuration tool used in Ubuntu. Used here to lock the WSL internal IP statically so port forwarding rules remain stable across restarts. |  
| `handoff` | **`02_Session_Handoff_Log.md`** | The process of saving complete session state — what was done, what was decided, what is pending — so work can resume in a new session without losing context. |  
| `ROE` | **`09_MEMORY_MAINTENANCE.md`** | Rules of Engagement. The agreed-upon working rules that govern how documentation sessions are conducted. Defined in `08_WORKFLOW.md`. |   

---

## Glossary pf Terms used throughout the **`gene.iobio-mi-personal`** Github repository.
## Terms in Alphabetical Order:

| Term | First Appears | Definition |
|---|:---:|---|
| `~ (tilde)` | **`Throughout`** | The `~ (tilde)` character is representative of the user `~/home` directory. |
| `.wslconfig` | **`Phase 1 Step 6`** | Windows-side configuration file for `WSL2`. Controls RAM allocation, CPU count, swap, and networking behavior. Lives in the Windows user profile folder. |
| `Administrator (elevated)` | **`Phase 1 Step 1`** | Running a program with full system permissions. Required for certain Windows operations. Right-click the program and select `Run as administrator`. |
| `apt` | **`Phase 2 Step 1`** | Ubuntu's package manager. Used to install, update, and remove software. `sudo apt install` is the standard way to install packages in this build. |
| `aria2` | **`Phase 2 Step 1`** | A lightweight, multi-protocol command-line download utility. It maximizes speed by splitting files into segments to download from multiple sources simultaneously (HTTP, BitTorrent, Metalink). Known for its minimal memory usage and ability to resume interrupted downloads. |
| `commit` | **`Phase 7 Step 4`** | A saved snapshot of a codebase at a specific point in time. Identified by a hash (e.g., `cb28d29a`). Checking out a specific commit ensures you are building exactly the verified version. |
| `conda` / `mamba` | **`Phase 2 Step 5`** | Package managers for scientific software. Provided by `Miniforge3`. `mamba` is faster than `conda` for resolving dependencies — use `mamba` for `bioconda` packages like `SAMtools`. |
| `config.json` | **`Phase 7 Step 6`** | Runtime configuration file for `gene.iobio` (`client/config.json`). Tells the frontend where to find the backend. Edited directly — no rebuild needed for config changes alone, but a rebuild is required for the changes to take effect in the served app. |
| `cpanminus` | **`Phase 2 Step 4`** | Perl package manager. Used to install `Perl` (programming language) modules not available via `apt`. Required before `Phenolyzer` dependencies. |
| `CRAM` | **`Phase 7 Step 10`** | A compressed format for storing aligned sequencing reads — the raw output of whole-genome sequencing after alignment to a reference genome. Smaller than BAM format. |
| `cruft` | **`Phase 8 Option A`** | Leftover files or artifacts from previous build attempts or failed sessions. When we check for cruft, we are verifying that no unintended files remain from earlier work that could interfere with the current build. |
| `data URI` | **`Phase 7 Step 10`** | A Data URI (Uniform Resource Identifier) is a scheme that allows small files, such as images, text, or stylesheets, to be embedded directly inline within a document rather than linked as external resources. |
| `fallocate` | **`Phase 1 Step 5`** | Linux command that pre-allocates disk space for a file. Used here to create the swap file. |
| `forge` | **`Throughout`** | Working name for LAPTOP-F (HealthMed Laptop, 192.168.1.240) — the active build and runtime environment where the stack runs. |
| `Freebayes` | **`07_OVERVIEW.md`** | A Bayesian, haplotype-based genetic variant detector designed to identify small polymorphisms in high-throughput sequencing data. It detects SNPs, indels, MNPs, and complex events by analyzing the literal sequences of reads rather than their precise alignment, which helps resolve ambiguities in repetitive regions. It accepts BAM/CRAM alignments and a reference genome as input, outputting variants in VCF format, and is particularly noted for its ability to handle pooled samples and non-diploid genomes. |  
| `fstab` | **`Phase 1 Step 5`** | Linux file (`/etc/fstab`) that defines filesystems and swap space to mount automatically at boot. |
| `git checkout` | **`Phase 7 Step 4`** | `Git` command to switch to a specific branch, tag, or commit. Used here to select the exact verified version of `gene.iobio`. |
| `gnomAD` | **`Phase 5 Step 6`** | Genome Aggregation Database. A large public database of human genetic variation used to determine how common or rare a given variant is in the general population. |
| `gru_data` | **`Phase 2`** | The reference data directory required by the `gru-backend`. Contains `VEP cache`, `gnomAD annotations`, `reference FASTA` files, and supporting databases. Named by the `iobio` project. |
| `handoff` | **`02_Session_Handoff_Log.md`** | The process of saving complete session state — what was done, what was decided, what is pending — so work can resume in a new session without losing context. |
| `Hyper-V virtual network` | **`Phase 8 Option B Step 4`** | Internal Windows virtual network that WSL2 runs on. Uses the `172.18.x.x` address range. Not visible to the router — static IP must be configured inside WSL itself. |
| `idempotent` | **`Phase 5 Step 4`** | A process that produces the same result whether run once or many times. `rclone` `sync` is `idempotent` — safe to re-run after interruption without risk of duplication or corruption. |
| `LAN` | **`Phase 8 Option B`** | `Local Area Network`. The private network connecting devices in your home or office. LAN serving means the stack is accessible to other devices on the same network without being exposed to the internet. |
| `md5` / `MD5` | **`Throughout`** | A checksum algorithm that produces a fixed-length fingerprint of a file. Used here to verify that backup files copied correctly — if the MD5 of the source and destination match, the file is intact. `md5` is one of many `hash` `checksum` types used to verify file integrity. |
| `mkdir` | **`Phase 3 Step 1`** | Linux command to create directories. The `-p` flag creates the full path including any missing parent directories. |
| `Miniforge3` | **`Phase 2 Step 1`** | Installer for the Conda package and environment manager. Miniforge3 comes pre-configured to use only the open-source conda-forge channel by default, avoiding proprietary repositories. It typically includes both the standard conda solver and the faster mamba solver, making it a lightweight and fully open-source choice for managing environments in Python, R, and others. |
| `nano` | **`Phase 1 Step 9`, `Throughout`** | Simple text editor that runs in the Linux terminal. Used throughout this build for creating and editing configuration files. To open a file: `nano filename`. To save: Ctrl+O, Enter. To exit: Ctrl+X. To open at a specific line: `nano +linenumber filename`. To paste: `Edit` > `Paste` in the terminal window. |
| `netplan` | **`Phase 8 Option B Step 6`** | Network configuration tool used in Ubuntu. Used here to lock the WSL internal IP statically so port forwarding rules remain stable across restarts. |
| `netsh portproxy` | **`Phase 8 Option B Step 4`** | Windows built-in tool that forwards network traffic from one address and port to another. Used here to route LAN requests arriving at the Windows IP down into the WSL virtual network where the services are running. |
| `Node.js` / `npm` | **`Phase 7 Step 1`** | `Node.js` is a JavaScript runtime. `npm` is its package manager. Both are required to install dependencies and build the `gene.iobio` frontend. |
| `NVM` | **`Phase 7 Step 1`** | `Node Version Manager`. Allows installing and switching between `Node.js` versions. Required because the system `Node.js` version in Ubuntu is insufficient for `gene.iobio`. |
| `pagefile` | **`Phase 1 Step 4`** | Windows virtual memory file. Supplements physical RAM by using disk space. Required for `Podman` stability on memory-constrained systems. |
| `Phenolyzer` | **`Phase 2`** | A tool that ranks candidate genes based on phenotype descriptions — you describe symptoms, it scores genes by relevance. Used here for rare disease gene prioritization. |
| `Podman` | **`Phase 1 Step 9`** | A tool for running containers — self-contained software packages — without needing a background service running as root. Similar to Docker but rootless-friendly. A **container** (sometimes called a Docker image) packages an application and all its dependencies into one portable unit. |
| `PowerShell` | **`Phase 1 Step 1`** | Windows command-line shell. Used for WSL management, firewall rules, and port forwarding. Different from the WSL/Ubuntu terminal — runs Windows commands, not Linux commands. |
| `pull` | **`Phase 1 Step 4`** | Pull = Download: It describes the direction of data flow: from remote → to local. The local machine initiates the request and "pulls" data from the server or remote storage. |
| `rclone` | **`Phase 5`** | Command-line tool for syncing files to and from cloud and HTTP storage. Used here to download `gru_data` from `files.iobio.io`. |
| `ROE` | **`09_MEMORY_MAINTENANCE.md`** | Rules of Engagement. The agreed-upon working rules that govern how documentation sessions are conducted. Defined in `08_WORKFLOW.md`. |
| `SAMtools` | **`Phase 2 Step 1`** | Handles the SAM (Sequence Alignment/Map), BAM (Binary Alignment/Map), and CRAM file formats. Key functions include sorting and indexing large alignment files, converting between formats, filtering data, and performing variant calling (identifying SNPs and indels). |
| `serve` / `npx` | **`Phase 7 Step 8`** | `serve` is a lightweight static file server used to serve the built `gene.iobio` frontend. `npx` is an npm tool runner — used as a temporary substitute if `serve` is not yet globally installed. |
| `sudo` | **`Throughout`** | Linux command prefix meaning "run as administrator." Required for commands that modify system files or settings. |
| `sync` (`rclone`) | **`Phase 5 Step 4`** | `rclone` operation that makes the destination match the source — downloading new or changed files and skipping already-present ones. |
| `sysdm.cpl` | **`Phase 1 Step 4`** | Windows System Properties control panel. Accessed via Run dialog (Windows+R). Used here to manage virtual memory settings. |
| `sysctl.conf` | **`Phase 1 Step 7`** | Linux file (`/etc/sysctl.conf`) for persistent kernel parameter settings. Used here to set `vm.overcommit_memory=1` across reboots. |
| `vault` | **`Throughout`** | Working name for LAPTOP-1 (Personal Laptop, 192.168.1.52) — the secure storage and backup environment. |
| `VCF` | **`Phase 7 Step 10`** | `Variant Call Format`. A standard file format for storing genetic variant calls — the list of positions in the genome where an individual's DNA differs from the reference. |
| `VEP` | **`Phase 4`** | `Variant Effect Predictor`. A tool developed by Ensembl that annotates genetic variants — it looks up what each variant does, where it falls in the genome, and what databases say about it. |
| `vault` | **`Throughout`** | Working name for LAPTOP-1 (Personal Laptop, 192.168.1.52) — the secure storage and backup environment. |
| `vault` | **`Throughout`** | Working name for LAPTOP-1 (Personal Laptop, 192.168.1.52) — the secure storage and backup environment. |
| `wget` | **`Phase 2 Step 5`** | Linux command-line download tool. Used here to download the `Miniforge3` installer. |
| `Windows Firewall` | **`Phase 8 Option B Step 3`** | Windows security feature that controls which network connections are allowed. Inbound rules must be added for `ports 3000` and `9001` to allow LAN devices to reach the frontend and backend. |
| `WSL` | **`Throughout`** | `Windows Subsystem for Linux` (`WSL2`). Lets you run a Linux environment (Ubuntu) directly inside Windows without a separate machine or virtual machine. |
| `WSL2` | **`Phase 4 Step 1`** | `Windows Subsystem for Linux version 2`. A compatibility layer built into Windows 11 that lets you run a full Linux environment without a separate machine or virtual machine. |
| `wsl.conf` | **`Phase 1 Step 2`** | Configuration file inside the WSL install that sets startup behavior — including which user to log in as by default. Must be set before first boot. |

---

≡≡≡≡≡ END: 07_OVERVIEW.md ≡≡≡≡≡  
