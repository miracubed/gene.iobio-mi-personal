# Stack Versioning Reference — gene.iobio Local Stack  
*Flat version reference — scannable lookup only*  
*Last updated: H20260815*  

**For build procedure:** See `01_Stack_Build_Steps.md`  
**For history:** See `02_Session_Handoff_Log.md`  

---  

## Host / Windows Layer  

| Component | Version | Notes |  
|---|---|---|  
| Windows 11 Home | 10.0.26200 Build 26200 | LAPTOP-F (HealthMed Laptop, 192.168.1.240) |  
| PowerShell | 7.6.4 | |  
| WSL | 2.7.11.0 | |  
| WSL Kernel | 6.18.33.2-2 | |  
| WSLg | 1.0.73.2 | |  
| MSRDC | 1.2.7214 | |  
| Direct3D | 1.611.1-81528511 | |  
| DXCore | 10.0.26100.1-240331-1435.ge-release | |  
| Windows (build) | 10.0.26200.8894 | |  

---

## WSL Configuration  

| Item | Value | Notes |  
|---|---|---|  
| Ubuntu | 26.04 LTS | GNU/Linux 6.18.33.2-microsoft-standard-WSL2 x86_64 |  
| WSL install location | `D:\WSL` | via `--location` flag |  
| WSL user | `mir` | sudo, default via `/etc/wsl.conf` |  
| RAM allocation | 2GB | `.wslconfig` — temporarily 3GB during image pulls, revert after |  
| CPU | 1 | `.wslconfig` |  
| Swap (`.wslconfig`) | 4GB | |  
| Swap (native) | 11GiB | `fallocate` at `/home/mir/swapfile`, persistent via `/etc/fstab` |  
| Total swap | ~14GiB | |  
| `localhostForwarding` | `true` | `.wslconfig` |  
| `vm.overcommit_memory` | `1` | persistent via `/etc/sysctl.conf` |  
| `C:` `pagefile` | enabled | re-enabled via `sysdm.cpl` |  
| `D:` `pagefile` | removed | |  

**Full `.wslconfig`:**  
```  
[wsl2]
memory=2GB
swap=4GB
processors=1
localhostForwarding=true
```  

---

## Windows Power Management  
| Item | Value |  
|---|---|
| `Power scheme` | `High Performance` |  
| `CONNECTIVITYINSTANDBY` | Locked via registry policy |  
| Registry key | `HKLM\SOFTWARE\Policies\Microsoft\Power\PowerSettings\f15576e8-98b7-4186-b944-eafa664402d9` |  
| `ACSettingIndex` | `1` |  
| `Wi-Fi adapter power management` | Disabled in Device Manager |  

`Modern Standby` workaround — applied H20260622. Required for WSL stability on laptop as well as dropped network shares happening while going into Modern Standby.

---


## Podman Configuration  
| Item | Value |  
|---|---|
| `Podman` | `5.7.0` |  
| `podman-compose` | `1.5.0` |  
| Storage driver | `vfs` (overlay incompatible with WSL ext4) |  
| `image_parallel_copies` | `1` |  
| `runRoot` | `/home/mir/.local/share/containers/run` |  

Full `~/.config/containers/containers.conf`:
```  
[engine]
image_parallel_copies=1
runRoot="/home/mir/.local/share/containers/run"

[storage]
driver="vfs"
```

---


## Core WSL Packages  
| Package | Version | Install method | Notes |  
|---|---|---|---|   
| `rclone` | `1.60.1+dfsg-4ubuntu3.2 `| `sudo apt install rclone` | `gru_data` `sync` |  
| `samtools` | `1.24` | `mamba install -c bioconda samtools=1.24` | Upgraded post-verification |  
| `aria2` | `1.37.0+debian-4` | `sudo apt install aria2` | |    
| `libhts3t64` | `1.22.1+ds2-1` | auto (`samtools` dep) | |  
| `libhtscodecs2` | `1.6.1-2build1` | auto (`samtools` dep) | |  	
| `libaria2-0` | `1.37.0+debian-4` | auto (`aria2` dep) | |  
| `libcares2` | `1.34.6-1` | auto (`aria2` dep) | |  
| `libncurses6` | `6.6+20251231-1` | auto (dep) | |  
| `Perl` | `5.40.1` | system | |  
| `cpanminus` | — | `sudo apt install` | `cpanminus` |  
| `BioPerl` | `1.7.8` | `sudo apt install bioperl` |  |    
| `libgraph-perl` |	`0.9735` | `sudo apt install libgraph-perl` | `Phenolyzer` dep |  
| `libjson-perl` | `4.10` | `sudo apt install libjson-perl` | `Phenolyzer` dep |  
| `Miniforge3 (conda)` | `26.3.2` | `wget` installer to `/mnt/d/miniforge3` | See **Phase 2** for full install steps |  

---

## Application Layer — Current Production Stack  
| Component | Version | Location | Image ID | Notes |  
|---|---|---|---|---|
| `iobio-gru-backend` | `2.0.0` | Podman container gru-backend-2.0.0 | 7e2c4a419267 | 4.43GB, HTTP 200 verified H20260806 |  
| `ensembl-vep` | (in `2.0.0`) | 115 | in container | — |  
| `gru_data` | `2.0.0` | `/mnt/d/gru_data/` | — | Complete, verified |  
| `gnomAD` | `4.1` | in `gru_data 2.0.0` | — |  
| `vcfanno` | updated | in `gru_data 2.0.0` | — | Annotations bug fix per `CHANGELOG` |  
| `Phenolyzer` | `v0.1.9-17-gf939514` | `~/phenolyzer` | — | tag v0.4.0, head f939514 |    

---

## Frontend Layer  
| Component | Version | Location | Notes |  
|---|---|---|---|
| `gene.iobio` | `v4.13.1-runtime-config` | `~/gene.iobio/` | `commit cb28d29a` |  
| `NVM` | `0.39.0` | `~/.nvm/` |  |   
| `Node.js` | `v16.20.2` | via `nvm` |  |  
| `npm` | `8.19.4` | via `nvm` |  |  
| `webpack` | `3.12.0` | `npm` dep | build output `client/dist/build.js` `30.5MB` |  |  
| `serve` | `installed globally` | via `npm` `npm install -g serve` |  |  

---

## Phase 8 Option A Modifications (`CSV`/`VCF` Export)  
| File | Line | Change |  
|---|---|---|  
| `client/app/components/viz/FlaggedVariantsCard.vue` | `843` | gate → `cohortModel.isLoaded` |  
| `client/app/components/viz/Navigation.vue` | `952` | gate → `cohortModel.isLoaded` |  
| `client/app/models/CohortModel.js` | `2903` | `reject(error)` → `resolve()` |  

All originals backed up as *`.bak`. Build: 29 seconds, clean.  

---

## Phase 8 Option B Configuration (LAN Network Serving/WSL IP Persistence)  
| Item | Value |  
|---|---|  
| `config.json` `backend.origin` | `http://192.168.1.240:9001` |  
| `config.json` `backend_map.default` | `http://192.168.1.240:9001` |  
| `Firewall rule` — `frontend` | `port 3000, Inbound, Allow, Enabled=True` |  
| `Firewall rule` — `backend` | `port 9001, Inbound, Allow, Enabled=True` |  
| `netsh portproxy` — `frontend` | `0.0.0.0:3000 → 172.18.60.70:3000` |  
| `netsh portproxy` — `backend` | `0.0.0.0:9001 → 172.18.60.70:9001` |  
| Startup script | `~/start_network_frontend.sh` |  
| LAN access `URL` | `http://192.168.1.240:3000` |  

WSL IP note: 172.18.60.70 is the WSL internal IP at time of configuration.  
This IP can change after restart.  
If LAN serving breaks, run hostname -I in WSL and update both portproxy rules.  
See **`Phase_8_Option_B.md`* Troubleshooting section in `01_Stack_Build_Steps.md`.  

---

## gru_data 2.0.0 Contents  
| Item | Path | Status |  
|---|---|---|  
| `VEP cache` `GRCh37` | `vep-cache/homo_sapiens_merged/115_GRCh37/` | ✅ |  
| `VEP cache` `GRCh38` | `vep-cache/homo_sapiens_merged/115_GRCh38/` | ✅ |  
| `VEP Plugins` dir | `vep-cache/Plugins/` | ✅ |  
| `Annotations` `GRCh37` | `annotations/GRCh37/` | ✅ |  
| `Annotations` `GRCh38` | `annotations/GRCh38/` | ✅ |  
| `References` `GRCh37` | `references/GRCh37/` | ✅ |  
| `References` `GRCh38` | `references/GRCh38/` | ✅ |  
| `md5_reference_cache` | `md5_reference_cache/` | ✅ pre-populated |  
| `geneinfo` | `geneinfo/` | ✅ |  
| `hpo` | `hpo/` | ✅ |  
| `gene2pheno` | `gene2pheno/` | ✅ |  
| `genomebuild` | `genomebuild/` | ✅ |  
| `data` | `data/` | ✅ |  
| `VERSION` | `2.0.0` | ✅ |  
| `CHANGELOG.md` | — |  | ✅ (only goes to `1.15.0`) |  
| `FILES_TO_DELETE` | — |  | ✅ applied |  

---

## Network / Share Architecture  
| Share | Path | WSL mount | Notes |  
|---|---|---|---|  
| `vault` `E:` | `\\192.168.1.52\E` | `/mnt/medlaptop/e` | `Seagate Expansion 4TB`, `persistent fstab` |  
| `HealthMed` share | `\\192.168.1.240\HealthMed` | — | `mir + smbshare Full Control`, `Everyone revoked` |  
| `WSL Backups` | `\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\` | `/mnt/medlaptop/e/MedLaptop\ Mirror/Ubuntu/WSL_Backups` |  |  
| `Gene iobio` dir backups | `\\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\` | `/mnt/medlaptop/e/gene_iobio_dir_bkps/` |  |  
| Software archive | `\\192.168.1.52\E\zzzzLaptopSetup\` | `/mnt/medlaptop/e/zzzzLaptopSetup/` |  |  
| `Downloaded distro` | `\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\DownloadedDistro\` | — |  | `Ubuntu tar` + `MD5` |  

`E:` drive letter reserved on `forge` (`192.168.1.240`) to avoid confusion.  
`vault` (`192.168.1.52`) share architecture intentionally minimal — Windows 10 aging.  

---

## WSL Backup Inventory  
| File | State | KB | GB | MD5 |  
|---|---|---|---|---|  
| `ubuntu-clean.tar` | Clean `Ubuntu 26.04 LTS` distro | `1,233,570 KB` | `1.176 GB` | `36835DADFBE100762426102B4A1A9305` |  
| `ubuntu-baseline-20260728.tar` | + `base config` | `1,240,700 KB` | `1.183 GB` | `E63ED5C2F840FC801BDCE181617EEA50` |  
| `ubuntu-podman-20260728.tar` | + `Podman 5.7.0` | `1,766,840 KB` | `1.685 GB` | `673908031F4DD9367B54FE91C406AA3D` |  
| `ubuntu-miniforge3-20260728.tar` | + `Miniforge3` | `1,869,550 KB` | `1.783 GB` | `3C5CACF49979E1E3A9F91AB5AAB1E6C2` |  
| `ubuntu-phenolyzer-20260728.tar` | + `Phenolyzer` verified | `2,391,240 KB` | `2.281 GB` | `E17FFEEBFD4CB8F94E0A6E8F8ECC1F9E` |  
| `ubuntu-updated-20260730.tar` | + `apt update/upgrade` | `2,390,016 KB` | `2.279 GB` | `F749816671DFCC585E07DDA867A3A149` |  
| `ubuntu-gru-pulled-20260730.tar` | + `gru-backend:1.17.0` `pulled` | `12,243,968 KB` | `11.677 GB` | `37798E6FA96DCD1F99E2468F708A6D2E` |  
| `ubuntu-gru-verified-20260801.tar` | + `gru-backend 1.17.0` `HTTP 200`, `gru_data 1.11.0` | `11,957,100 KB` | `11.403 GB` | `467B88D70157A3FAD27A261E1BEBB96D` |  
| `ubuntu_gru_data_1_14_1_20260803.tar` | + `gru_data 1.14.1 complete`, `VERSION 1.14.1` | `12,028,060 KB` | `11.471 GB` | `FDFF3020F02C7FF540A110B3B285E68C` |  
| `Ubuntu_26_04_iobio_stack_production_20260806.tar` | + `gru-backend 2.0.0`, `samtools 1.24`, `frontend v4.13.1` | `16,098,000 KB` | `15.354 GB` | `B477831278EDE7877ACDE5C59FBF22A5` |  
| `ubuntu_full_stack_FINAL_20260808.tar` | `FINAL` — `post Phase 8A + 8B` | `16,885,440 KB` | `16.103 GB` | `A96A843F7501165F18895127C1C7B01F` |  

## WSL backup locations:  
   • `forge`: `D:\WSL_Backups\`  
   • `vault`: `\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\`  
   • WSL path: `/mnt/medlaptop/e/MedLaptop\ Mirror/Ubuntu/WSL_Backups/`  

---

## Gene iobio Directory Backup Inventory  
| File | State | `KB` | `GB` | `MD5` | `forge` `D:` | `vault` `E:` |  
|---|---|---|---|---|---|---|   
| `gru_data_1_14_1_20260803.tar` | `gru_data 1.14.1`, `169.866 GiB`, `8967 files` | `215,589,410 KB` | `205.648 GB` | `E89770EB54C8856802FEB18BF0D8FB0A` | ❌ removed (space) | ✅ present |  
| `gru_data_2_0_0_FINAL_20260808.tar` | `FINAL` `gru_data 2.0.0 full dataset` | `122,516,280 KB` | `116.841 GB` | `5A82DBDF9AA4BA75B0661777671188F6` | ✅ | ✅ |    
| `gene_iobio_frontend_v4_13_1_20260806.tar` | `frontend v4.13.1-runtime-config`, `pre-Phase 8` | `303,000 KB` | `0.289 GB` | `C281CC8F6FA0E7D3FF69E64C8F20DC3D` | ✅ | ✅ |  
| `gene_iobio_frontend_v4_13_1_FINAL_20260808.tar` | `FINAL` `frontend post Phase 8A + 8B` | `310,310 KB` | `0.296 GB` | `0730638C348CD8616C2AFCC322F90F6F` | ✅ | ✅ |  

My backup frequency during different stages, file names/dates/md5s will differ from yours  
as I was learning during the process and your system is different from my system.  

Note: gru_data_1_14_1_20260803.tar removed from `forge` `D:` due to space pressure during final backup operations H20260808.  
Viable copy confirmed on `vault` `E:` Seagate. Do not re-download unless vault copy is also lost.  
Gene iobio directory backup locations:  
   • Backend — `forge`: `D:\gene_iobio_dir_bkps\gene_iobio_backend_dir\`  
   • Backend — `vault`: `\\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\gene_iobio_backend_dir\`  
   • Frontend — `forge`: `D:\gene_iobio_dir_bkps\gene_iobio_frontend_dir\`  
   • Frontend — `vault`: `\\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\gene_iobio_frontend_dir\`  

---

## Production Stack Summary  
| Component | Version | Status |  
|---|---|---|  
| `gru-backend` | `2.0.0` | ✅ HTTP 200 verified |   
| `gru_data` | `2.0.0` | ✅ Complete |  
| `VEP` | `115` | ✅ |  
| `gnomAD` | `4.1` | ✅ |  
| `gene.iobio frontend` | `v4.13.1-runtime-config` | ✅ |  
| `samtools` | `1.24` | ✅ |  
| `Node.js` | v16.20.2 | ✅ |  
| `Ubuntu WSL` | 26.04 LTS | ✅ | 
| `CSV`/`VCF` export (**`Phase 8A`**) | unlocked | ✅ |  
| `LAN network serving/WSL IP Persistence` (**`Phase 8B`**) | operational | ✅ |  

Validated: End-to-end with production WGS data (`GRCh37`).  
Variant annotation, gene lookup, phenotype association, `Phenolyzer` integration, `CSV`/`VCF` files export, and LAN serving all functional.  

---

##  Known Limitations  
   • Modifier impact section: Browser rendering can crash on whole-genome variant lists. Expected behavior on large datasets.  
   • `CSV`/`VCF` export — large sets: Exports of 60+ unfiltered variants may be slow due to BAM depth coverage refresh overhead. Filter first for smaller data set for best results.  
   • `WSL IP persistence`: WSL internal IP can change after restart. If LAN serving breaks, check `hostname -I` and update `netsh portproxy` rules.  
   • Console warnings (non-issues): `Vue` development mode warning and HTTPS warning on data `URI`s are expected — do not affect functionality.  

## External Links  
   • [gene.iobio Closed Issue #129](https://github.com/iobio/iobio-gru-backend/issues/129)  
   • [files.iobio.io](https://files.iobio.io)  
   • [Gene.Iobio Vue](https://github.com/iobio/gene.iobio)  
   • [Gene.Iobio gru-backend](https://github.com/iobio/iobio-gru-backend)  
   
---

≡≡≡≡≡ END: 03_Stack_Versioning_Reference.md ≡≡≡≡≡
