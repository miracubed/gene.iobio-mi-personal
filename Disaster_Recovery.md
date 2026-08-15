# **Disaster_Recovery.md — Full Stack Restore Procedure**  

## **Overview**  

This document describes how to restore the complete `gene.iobio` stack  
as outlined in this repository from the final backup set in case of   
system failure, data loss, or the need to redeploy on a new machine.  

**This is your insurance policy.** Keep this document accessible  
alongside your backup files.  

---

## **When to Use This Document**

Use this procedure when:  

- The `forge` (LAPTOP-F, 192.168.1.240) WSL installation is corrupted  
  or lost  
- The `D:` drive fails and needs to be rebuilt  
- You are deploying the stack on a new machine  
- You need to recover from a catastrophic failure  
- You want to clone the entire working stack to a `second forge` for  
  testing or failover  

**Do not use this procedure for routine backups or version updates.**  
Use this only for disaster recovery or full-system restore.  

---

## **Backup Set Reference**  

The final production backup set (post Phase 8A + 8B) consists of  
three files:  

| File | Size | MD5 | Location |  
|---|---|---|---|  
| `ubuntu_full_stack_FINAL_20260808.tar` | `16.103 GB` | `A96A843F7501165F18895127C1C7B01F` | `vault` `E:` + `forge` `D:` |  
| `gru_data_2_0_0_FINAL_20260808.tar` | `116.841 GB` | `5A82DBDF9AA4BA75B0661777671188F6` | `vault` `E:` + `forge` `D:` |  
| `gene_iobio_frontend_v4_13_1_FINAL_20260808.tar` | `0.296 GB` | `0730638C348CD8616C2AFCC322F90F6F` | `vault` `E:` + `forge` `D:` |  

**Total:** `133.24 GB`  

These files are stored in:  
- **`forge`:** `D:\WSL_Backups\` and `D:\gene_iobio_dir_bkps\`  
- **`vault`:** `\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\` and `\\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\`  

---

## **Scenario 1: WSL Corruption — Restore WSL Only**  

Use this if the WSL installation is corrupted but the `D:` drive is  
intact.  

### Step 1: Verify Backup Integrity  

#### On forge (Windows), verify the WSL backup file exists:  
PowerShell  
```  
ls -lh D:\WSL_Backups\ubuntu_full_stack_FINAL_20260808.tar
```  
#### Verify MD5 (PowerShell):  
PowerShell  
```
$(Get-FileHash -Path D:\WSL_Backups\ubuntu_full_stack_FINAL_20260808.tar
  -Algorithm MD5).Hash
```
Expected: `A96A843F7501165F18895127C1C7B01F` 

### Step 2: Unregister Corrupted WSL Distribution  
#### List current WSL distributions:  
PowerShell  
```  
wsl --list -v
```  
#### Unregister the corrupted distribution (example: Ubuntu):  
PowerShell  
```  
wsl --unregister Ubuntu
```  

#### Verify it is gone:  
PowerShell  
```  
wsl --list -v
```  
Output should not show Ubuntu  

### Step 3: Import Backup Tar  
#### Import the backup tar as a new WSL distribution:  
PowerShell  
```
wsl --import Ubuntu D:\WSL D:\WSL_Backups\ubuntu_full_stack_FINAL_20260808.tar
```

Verify import succeeded:  
PowerShell  
```  
wsl --list -v
```  
Output should show Ubuntu with version 2  

### Step 4: Verify WSL Functionality  
#### Launch WSL:  
PowerShell  
```  
wsl
```  

#### Verify you are logged in as `mir`:  
bash  
```  
whoami
```  
Expected: mir  

#### Verify `gru_data` mount:  
bash  
```  
ls -lh /mnt/d/gru_data/
```  
Expected: directories `annotations/`, `vep-cache/`, `references/`, etc.  

#### Verify `vault` mount:  
bash  
```  
ls -lh /mnt/medlaptop/e/
```  
Expected: `MedLaptop Mirror/`, `zzzzLaptopSetup/`, etc.  

Exit WSL:  
bash  
```  
exit
```  

### Step 5: Restart Services  
#### Launch WSL:  
PowerShell  
```  
wsl
```  

#### Start `gru-backend`:  
bash  
```  
podman run --rm -it --name gru-backend-2.0.0
  -v /mnt/d/gru_data:/gru_data -p 9001:9001
docker.io/iobio/iobio-gru-backend:2.0.0
```  

#### In a new terminal (Terminal 2):  
PowerShell  
```  
wsl
```

#### Start frontend:  
PowerShell  
```  
cd ~/gene.iobio/client
```
PowerShell  
```  
~/start_network_frontend.sh
```  
#### Test in browser:  
`http://localhost:3000`


## **Scenario 2: `D:` Drive Failure — Full Restore from `Vault`**  
Use this if the `D:` drive fails and needs to be completely rebuilt.
Prerequisites
    • New `D:` drive installed and formatted as NTFS
    • `D:` drive mounted and accessible as `D:` in Windows
    • Network access to `vault` (`E:` Seagate on `192.168.1.52`)
    • Adequate free space on `D:` (`~250GB minimum`)
### Step 1: Verify Vault Backup Integrity  
#### From `forge` (Windows), verify all three `vault` backup files:  
bash  
```  
ls -lh \\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu_full_stack_FINAL_20260808.tar
```  
bash  
```  
ls -lh \\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\gene_iobio_backend_dir\gru_data_2_0_0_FINAL_20260808.tar
```
bash  
```
ls -lh \\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\gene_iobio_frontend_dir\gene_iobio_frontend_v4_13_1_FINAL_20260808.tar
```  
Verify `MD5` of each (or at minimum the WSL tar):  
Compare against documented hashes in this document  

### Step 2: Copy Backup Files from `Vault` to `D:`  
#### Create backup directories on `D:`  
PowerShell  
```  
mkdir D:\WSL_Backups
```  
PowerShell  
```  
mkdir D:\gene_iobio_dir_bkps\gene_iobio_backend_dir -Force
```  
PowerShell  
```  
mkdir D:\gene_iobio_dir_bkps\gene_iobio_frontend_dir -Force
```  

#### Copy `WSL` backup (this will take ~30-60 minutes):  
PowerShell  
```
\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu_full_stack_FINAL_20260808.tar D:\WSL_Backups\ -Verbose
```  

#### Copy `gru_data` backup (this will take ~2-4 hours):  
PowerShell  
```  
\\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\gene_iobio_backend_dir\gru_data_2_0_0_FINAL_20260808.tar D:\gene_iobio_dir_bkps\gene_iobio_backend_dir\ -Verbose
```  
#### Copy `frontend` backup (this will take ~5-10 minutes):  
Powershell  
```  
\\192.168.1.52\E\MedLaptop Mirror\gene_iobio_dir_bkps\gene_iobio_frontend_dir\gene_iobio_frontend_v4_13_1_FINAL_20260808.tar D:\gene_iobio_dir_bkps\gene_iobio_frontend_dir\ -Verbose
```  
### Step 3: Verify MD5 of Copied Files  
Verify each copied file matches the original:  

#### WSL backup:  
PowerShell  
```  
$(Get-FileHash -Path D:\WSL_Backups\ubuntu_full_stack_FINAL_20260808.tar -Algorithm MD5).Hash
```
Expected: `A96A843F7501165F18895127C1C7B01F` (this is mine: yours may be different)  

#### `gru_data` backup:  
PowerShell  
```  
$(Get-FileHash -Path D:\gene_iobio_dir_bkps\gene_iobio_backend_dir\gru_data_2_0_0_FINAL_20260808.tar -Algorithm MD5).Hash
```
Expected: `5A82DBDF9AA4BA75B0661777671188F6` (this is mine: yours may be different)  

#### `frontend` backup:  
PowerShell  
```  
$(Get-FileHash -Path D:\gene_iobio_dir_bkps\gene_iobio_frontend_dir\gene_iobio_frontend_v4_13_1_FINAL_20260808.tar -Algorithm MD5).Hash
```  
Expected: `0730638C348CD8616C2AFCC322F90F6F` (this is mine: yours may be different)  

#### If any hash does not match, re-copy that file  

### Step 4: Extract WSL Backup  
#### Import the WSL tar:  
PowerShell  
```  
wsl --import Ubuntu D:\WSL D:\WSL_Backups\ubuntu_full_stack_FINAL_20260808.tar
```  
Verify:  
PowerShell  
```  
wsl --list -v
```  

### Step 5: Extract `gru_data` Backup  
#### Launch WSL:  
PowerShell  
```  
wsl
```  

#### Extract `gru_data` tar (this will take ~30-60 minutes):  
bash  
```  
cd /mnt/d
```  
bash  
```  
tar -xzf gene_iobio_dir_bkps/gene_iobio_backend_dir/gru_data_2_0_0_FINAL_20260808.tar
```  
Verify extraction:  
bash  
```  
ls -lh /mnt/d/gru_data/
```  
Expected: `annotations/`, `vep-cache/`, `references/`, etc.  

Exit WSL:  
bash  
```  
exit
```  

### Step 6: Extract Frontend Backup  
#### Launch WSL:  
PowerShell  
```  
wsl
```  
#### Extract frontend tar:  
bash  
```  
cd ~
```  
bash  
```  
tar -xzf /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_1_FINAL_20260808.tar
```  
Verify extraction:  
bash  
```  
ls -lh ~/gene.iobio/
```  
Expected: `client/`, `server/`, `package.json`, etc.  

Exit WSL:  
bash  
```  
exit
```  
### Step 7: Verify All Components  
#### Launch WSL:  
PowerShell  
```  
wsl
```  

#### Verify `gru_data`:  
bash  
```  
ls -lh /mnt/d/gru_data/ | head -20
```  
bash
```  
cat /mnt/d/gru_data/VERSION
```  
Expected: `2.0.0`

#### Verify `frontend`:  
bash  
```  
ls -lh ~/gene.iobio/client/dist/build.js
```  
Expected: ~30.5MB file  

#### Verify `backend` image is still present:  
bash  
```  
podman images | grep gru-backend
```  
Expected: `docker.io/iobio/iobio-gru-backend:2.0.0`  

Exit WSL:  
bash  
```  
exit
```  

### Step 8: Start Services  
#### Terminal 1: Start backend  
PowerShell  
```  
wsl
```  
bash  
```  
podman run --rm -it --name gru-backend-2.0.0
  -v /mnt/d/gru_data:/gru_data -p 9001:9001
  docker.io/iobio/iobio-gru-backend:2.0.0
```  
#### Terminal 2: Start frontend  
PowerShell  
```  
wsl
```  
bash  
```  
cd ~/gene.iobio/client
```  
bash  
```  
~/start_network_frontend.sh
```  
#### Test in browser:
`http://localhost:3000`

## **Scenario 3: New Machine Deployment**  
Use this if you are deploying the stack on a different forge machine.
Prerequisites
    • New machine with Windows 11 + WSL2 capability
    • `D:` drive with `at least 250GB free space`
    • Network access to `vault` (or physical access to backup media)
    • `WSL2` installed and updated
### Steps 1–8  
#### Follow **Scenario 2** (`D:` Drive Failure) exactly as written.  
The procedure is identical — you are copying from `vault` to a new `D:` drive and extracting.  
Additional: Reconfigure LAN Network Access/WSL IP Persistence (**Phase 8 Option B**)  
If you are deploying on a machine with a different IP address, you will need to update network configurations:  
#### Update firewall rules if needed (same as Phase 8B):  
PowerShell  
```  
New-NetFirewallRule -DisplayName "gene.iobio frontend" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow
```
```
New-NetFirewallRule -DisplayName "gru-backend" -Direction Inbound -Protocol TCP -LocalPort 9001 -Action Allow
```
#### Update `netsh portproxy` rules (see **Phase 8 Option B** for full procedure)  
#### Get new WSL IP:  
bash  
```
wsl hostname -I
```  
#### Add portproxy` rules with new IP:  
`netsh interface portproxy` add `v4tov4`  
bash   
```  
listenaddress=0.0.0.0 listenport=3000 connectaddress=<new-WSL-IP> connectport=3000
```  
bash
```  
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=9001 connectaddress=<new-WSL-IP> connectport=9001
```  

## **Scenario 4: Partial Restore — Only `gru_data` or `Frontend`**  
Use this if you only need to restore one component.  
Restore `gru_data` Only  
### If `gru_data` is corrupted but WSL and frontend are intact:  
bash  
```  
wsl
```  
### Remove corrupted gru_data:  
bash  
```  
rm -rf /mnt/d/gru_data
```  
### Extract from backup:  
bash  
```  
cd /mnt/d
tar -xzf gene_iobio_dir_bkps/gene_iobio_backend_dir/gru_data_2_0_0_FINAL_20260808.tar
```  
### Verify:  
bash  
```  
ls -lh /mnt/d/gru_data/
```  
bash
```  
cat /mnt/d/gru_data/VERSION
```  
### Restart backend:
### (kill current backend, restart per **Phase 8 Option B** `start_network_backend.sh` or `podman start gru-backend-2.0.0`)
Restore Frontend Only  
### If frontend is corrupted but `WSL` and `gru_data` are intact:  
PowerShell  
```  
wsl
```  
### Remove corrupted frontend:  
bash  
```  
rm -rf ~/gene.iobio
```  
### Extract from backup:  
bash  
```  
cd ~
```  
bash  
```  
tar -xzf /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_1_FINAL_20260808.tar
```
### Verify:  
bash  
```  
ls -lh ~/gene.iobio/client/dist/build.js
```  
### Restart frontend:  
### (kill current frontend, restart per **Phase 8 Option B**)  

# **Troubleshooting Restore**  

## **Tar Extraction Hangs or Is Very Slow**  
**Symptom**: tar -xzf command appears frozen or takes hours.  
**Cause**: Large tar files on network shares or slow `D:` drives can take a long time. This is normal.  
**Solution**:  
    • Let it run. Do not interrupt.  
    • Monitor disk activity (`Windows Task Manager` → `Performance` → `Disk`).  
    • If disk activity stops for >5 minutes, the process may have hung.  
    • In that case, kill the tar process and re-extract.  

## **MD5 Mismatch After Copy**  
**Symptom**: Copied file MD5 does not match original.  
**Cause**: File corruption during copy, network interruption, or incomplete copy.  
**Solution**:  
    • Delete the corrupted copy from `D:`  
    • Re-copy the file from `vault`  
    • Verify `MD5` again  
    • If `MD5` still mismatches, the `vault` copy may be corrupted — check `vault` copy `MD5` against the documented hash in this document  

## **WSL Import Fails with Error**  
**Symptom**: `wsl --import` returns an error (`E_UNEXPECTED`, `E_ABORT`, etc.)  
**Cause**: Multiple possible causes — corrupted tar, insufficient disk space, WSL version mismatch, etc.  
**Solution**:  
    • Verify the tar file `MD5` matches the documented hash  
    • Verify `D:` drive has `at least 50GB free`  
    • Verify WSL is up to date: `wsl --update`  
    • Try unregistering any existing Ubuntu distribution and re-importing  
    • If still failing, check `Windows Event Viewer` for `WSL` errors  

## **Backend Container Does Not Start**  
**Symptom**: `podman run` hangs or fails with an error.  
**Cause**: Image not present, gru_data not mounted correctly, port already in use, etc.  
**Solution**:  
    • Verify image exists: `podman images | grep gru-backend`  
    • If missing, pull it: `podman pull docker.io/iobio/iobio-gru-backend:2.0.0`  
    • Verify `gru_data` extracted: `ls -lh /mnt/d/gru_data/VERSION`  
    • Verify `port 9001` is not in use: `netstat -an | grep 9001`  
    • Try starting with verbose output: `podman run --rm -it ... 2>&1 | tee startup.log`  

# **Restoration Time Estimates**  
## Operation	Time  
## Copy WSL backup from vault (16.1 GB)	30–60 minutes  
## Copy gru_data backup from vault (116.8 GB)	2–4 hours  
## Copy frontend backup from vault (0.3 GB)	5–10 minutes  
## Extract WSL tar	10–20 minutes  
## Extract gru_data tar	30–60 minutes  
## Extract frontend tar	2–5 minutes  
## **Total full restore	4–6 hours**  

These times assume:  
    • Network speed: 1 Gbps LAN  
    • `D:` drive: modern SATA or NVMe SSD  
    • No other heavy disk I/O  
Network speed and disk speed are the main variables. Adjust expectations accordingly.  

# **Post-Restore Validation**  
After restoring, validate the stack is working:  

## Terminal 1: Start backend  
PowerShell  
```  
wsl
```  
bash
```  
podman run --rm -it --name gru-backend-2.0.0
  -v /mnt/d/gru_data:/gru_data -p 9001:9001
  docker.io/iobio/iobio-gru-backend:2.0.0
```  

## Terminal 2: Check backend health  
PowerShell  
```  
wsl
```  
bash  
```  
curl -s localhost:9001 | jq .
```
Expected: JSON with `service_description`, `version`, `data_version`  

## Terminal 3: Start frontend  
PowerShell  
```  
wsl
```  
bash  
```  
cd ~/gene.iobio/client
```  
bash
```  
~/start_network_frontend.sh
```  

## Terminal 4: Test in browser  
## Open `http://localhost:3000`  
## Verify UI loads  
## Try a gene lookup (e.g., `RAI1`)    
## Verify variant card renders  
If all tests pass, the restore is complete and successful.  

# **Backup Maintenance**  
After a successful restore:  
    1. Create a new FINAL backup set if any changes were made during recovery  
    2. Update `03_Stack_Versioning_Reference.md` with new backup file names, sizes, and `MD5` hashes  
    3. Update `D_Drive_Cleanup_Priority.md` if necessary  
    4. Verify both `forge` and `vault` copies of all backups before considering recovery complete  

# **Emergency Contact**  
If you encounter an issue during restore that this document does not cover:  
    1. Check `04_GOTCHA_Enumeration.md` for phase-specific gotchas  
    2. Review `01_Stack_Build_Steps.md` for the relevant phase  
    3. Check `iobio` project issues:  
        ◦ `gene.iobio` issues  
        ◦ `iobio-gru-backend` issues  
This is a community-maintained build. If you discover a new issue or edge case, consider documenting it and contributing back to the repository.  

---

≡≡≡≡≡ END: Disaster_Recovery.md ≡≡≡≡≡  
