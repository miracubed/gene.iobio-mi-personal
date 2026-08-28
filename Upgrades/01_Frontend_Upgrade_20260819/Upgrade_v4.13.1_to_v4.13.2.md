# Upgrade_v4.13.1_to_v4.13.2.md  
# Upgrade: gene.iobio v4.13.1 → v4.13.2   
*Frontend-only upgrade — gru-backend and gru_data unchanged*  
*Released: 2026-08-18*  
*This Document: 2026-08-19*
  
---  
  
## Overview  
  
`gene.iobio v4.13.2` is a minor frontend release. The `gru-backend 2.0.0`  
and `gru_data 2.0.0` pairing introduced in `v4.13` is unchanged. No backend   
work is required for this upgrade.   
  
This document assumes your stack was built following the procedures in   
`01_Stack_Build_Steps.md` and is currently running the production-validated    
**`v4.13.1` + `gru-backend 2.0.0` pairing**.   
  
Throughout this document, `mir` is used as the example WSL username.   
Substitute your own username wherever `mir` appears in paths and commands.   
  
---  
## What Changed in 4.13.2   
  
### Upstream Changes (tonydisera / iobio)   
  
| PR | Change | Impact to local stack |  
|---|---|---|  
| `#1166` | Merged runtime config work into master | Already in your `4.13.1` base |  
| `#1167` | Nebula settings moved to runtime config | Already in your `4.13.1` base |   
| `#1169` | ClinVar VCF URL bug fix on genome build switch | Already in your `4.13.1` base |  
| `#1169` | Release notes for `4.13.1` | Documentation only |  
| `#1170` | Production config pointed to backend.iobio.io | Irrelevant — you override this |  
| `#1170` | Version bump to `4.13.2` | Apply manually — see below |  
  
**PR #1168 (automated security bot? — OMIM API key in config.json) was  not merged into master by tonydisera and is not part of the `4.13.2` release.**  
If your **`omim_api_key`** field in **`client/config.json`** is  already an empty string **`""`**, no action is required. 
See **`config.json`** section below.   
  
### Net Change for a Local Stack  
  
This is a version string and config pointer update. The substantive  
work (runtime config refactor, **`ClinVar`** bug fix) was already present in   
your **`4.13.1`** build.  

The upgrade procedure is:     
1. **`Pull`** the **`4.13.2`** tag from upstream   
2. Verify/edit **`client/config.json`** (protect your local backend IP)   
3. Update two version strings
4. Update **`Node.js`** to **`v20`**   
5. Run **`npm install`** 
6. Re-apply **`Phase 8 Option B`** patch for backend server IP 
7. Re-verify and reapply **`Phase 8 Option A`** patches for **.csv**/**.vcf** variant file export enable (**line numbers may shift between builds**)   
8. Rebuild **`gene.iobio`** frontend   
9. Verify build
10. New **`FINAL backup tar files`** for **`WSL --export`** and **`~/gene.iobio/`** directory  
  
---  
## Pre-Upgrade Checklist  
  
Before touching any files, confirm your current stack state and protect   
your work.  
  
### Confirm Current Stack   
  
# Confirm frontend version   
bash  
```  
grep -n "version" ~/gene.iobio/package.json | head -3
```  
  
# Confirm correct gru-backend image is running   
bash   
```  
podman ps --format "{{.Names}} {{.Image}}"
```  
Expected outputs:  
   -   **`package.json`**: **`"version": "4.13.1"`**
   -   **`podman`**: **`container running iobio/iobio-gru-backend:2.0.0 image ID 7e2c4a419267`**  
    

If the wrong image is running, stop and correct it before proceeding.   
See **Phase 6 of `01_Stack_Build_Steps.md` and `GOTCHA 4.2`**.  

### Back Up Before Touching Anything  

Your current `FINAL` backup (**`gene_iobio_frontend_v4_13_1_FINAL_*.tar`**) contains your **`Phase 8  Option A`** and **`Phase 8 Option B`** work.  
Protect it before any edits.  

# Confirm your FINAL frontend backup exists before proceeding  
bash  
```  
ls -lh /mnt/d/gene_iobio_dirs/gene_iobio_frontend_dir/
```  
Verify your **`FINAL`** tar is present and MD5-verified before continuing.  
**Do not proceed if the backup is missing or unverified.**

---   
## Step 1 — Stash Local Changes and Pull 4.13.2  

Your working `Phase 8 Option A` and `Phase 8 Option B` modifications  will cause `git`  to refuse the checkout with an error about files being overwritten.   
This is expected and correct — `git` is protecting your work.   
The **`stash`** command parks your changes safely before the **`pull`**.  
   
They are not lost — you will reapply them manually in **Step 5**.   
bash   
```  
cd ~/gene.iobio
```  
bash   
```  
git stash push -m "Phase8A_8B_modifications_v4.13.1"
```  
Expected output:  
```  
Saved working directory and index state On v4.13.1-runtime-config: Phase8A_8B_modifications_v4.13.1
```  
Now checkout the 4.13.2 tag:  
bash   
```  
git checkout tags/gene_4.13.2
```  
Confirm the checkout succeeded:  
bash   
```  
grep -n "version" ~/gene.iobio/package.json | head -3
```  
Expected: **`3:  "version": "4.13.2",`**

**Do not proceed until this confirms `4.13.2`.**  

> Note: `git stash` is a temporary holding area only.  
> Your stashed changes are visible via the command `git stash list`  
> (Expected: **`stash@{0}: On v4.13.1-runtime-config: Phase8A_8B_modifications_v4.13.1`**) at any time.  
> They are not committed and not lost. `Step 5` walks you through reapplying all three **`Phase 8 Option A`** edits manually to the newly pulled **`4.13.2`** files.  

---
## Step 2 — Review/Edit client/config.json  

This file was touched by `PR #1167` and `PR #1170` upstream. You also modified it for **`Phase_8_Option_B.md`** (LAN serving and WSL IP persistence for local backend IP). 
The upstream `4.13.2` **`config.json`** points production to **`backend.iobio.io`** and may include Mosaic/Nebula/various backend entries irrelevant to a local stack.  
Replace the entire file contents with your minimal local configuration — your backend IP in both origin and default fields, all other fields set to empty strings or false as appropriate.
Verify with **`cat`** command before proceeding.

### What Your config.json Must Contain  
```  
{  
  "gene": {  
    "path_prefix": "/",  
    "site_name": "",  
    "default_mode": "advanced",  
    "show_intro": false,  
    "show_files_button": true,  
    "show_blogs_and_tutorials": true,  
    "phenolyzer_permitted": true,  
    "omim_api_key": "",  
    "intro_paragraph_1": "",  
    "intro_paragraph_2": "",  
    "intro_paragraph_3": "",  
    "intro_paragraph_4": ""  
  },  
  "backend": {  
    "origin": "http://<your-host-ip>:9001",  
    "path_prefix": "/"  
  },  
  "backend_map": {  
    "default": "http://<your-host-ip>:9001"  
  }  
}
```  
For the **`origin`** and **`default`** fields:  
Replace **`<your-host-ip>`** with your actual host machine IP address.  
bash  
```  
nano ~/gene.iobio/client/config.json
```  
If you copied the IP settings used in this repository's personal build process, this IP would be:   
**`192.168.1.240`**

Check it:
bash  
```  
cat  ~/gene.iobio/client/config.json
```  
Expected upon checking updated file:   
```  
{
  "gene": {
    "path_prefix": "/",
    "site_name": "",
    "default_mode": "advanced",
    "show_intro": false,
    "show_files_button": true,
    "show_blogs_and_tutorials": true,
    "phenolyzer_permitted": true,
    "omim_api_key": "",
    "intro_paragraph_1": "",
    "intro_paragraph_2": "",
    "intro_paragraph_3": "",
    "intro_paragraph_4": ""
  },
  "backend": {
    "origin": "http://192.168.1.240:9001",
    "path_prefix": "/"
  },
  "backend_map": {
    "default": "http://192.168.1.240:9001"
  }
}
```  
### OMIM API Key  

If **`omim_api_key`** is an empty string **`""`** in your current config, leave it as an empty string.   
Do not populate this field — API keys in client-side JSON are served to the browser  and are not secure.  
`PR #1168` flagged this pattern.   
The field is present for compatibility only.  

### After Editing  

Verify your backend IP is present in both **`origin`** and **`default`** fields and the **`omim_api_key`** has an empty string **`""`** before continuing.  
**Do not proceed to build with the upstream production IP in place.**  

---  
## Step 3 — Verify Version Strings  

The **`git pull`** from the **`4.13.2`** tag updates these automatically.  
This step is verification only — no editing required.  
bash  
```  
grep -n "version" ~/gene.iobio/package.json | head -3
```  
bash   
```  
grep -n "version" ~/gene.iobio/client/app/globals/GlobalApp.js
```  
Expected:  
   -   **`package.json`**:  **`"version": "4.13.2",`**   
   -   **`GlobalApp.js`**:  **`this.version = "4.13.2";`**  

If both show  **`4.13.2`**  — proceed to **Step 4**.  
If either still shows **`4.13.1`**  — something went wrong with the **`git checkout`**.  
Return to **Step 1** and confirm the tag checkout succeeded before continuing.  
   
---   
## Step 4 — Upgrade Node.js to v20  

**`gene.iobio` `v4.13.2`** upgraded to **`pure-js sass` (`1.89.2`)** to replace  **`node-gyp`** and **`node-sass`**.  
The **`pure-js sass`** package requires  **`Node.js` `v20.19.0`** or higher.   
Your current **`v16.20.2`** will produce  **`EBADENGINE`** warnings during **`npm install`** and may fail during build.  

**Upgrade to `Node.js` `v20` before proceeding**:  
bash  
```  
nvm install 20
```  
bash  
```  
nvm use 20
```  
bash  
```  
node --version
```
Expected:  **`v20.20.2`**  (confirm **`v20`** is active before continuing)
bash  
```  
npm --version
```
Expected:  **`10.8.2`**  
**Note these version changes in your own `03_Stack_Versioning_Reference.md`**.

---
## Step 5 — Re-Verify and Reapply Phase 8 Option A Patches for .csv/.vcf variant file export

### Phase 8 Option A Edits — Complete Mapping (4.13.1 → 4.13.2)

| File | `v4.13.1` Line | `v4.13.2` Line | Edit | Status |  
|---|---|---|---|---|  
| `FlaggedVariantsCard.vue` | `843` | `836` | Gate → `cohortModel.isLoaded` | ✅ Present |  
| `Navigation.vue` | `952` | `947` | Gate → `cohortModel.isLoaded` | ✅ Present |  
| `CohortModel.js` | `2903` | `2904` | `reject(error)` → `resolve()` | ⚠️ **Missing — needs reapply** |  

Some lines of code have changed/moved or have been overwritten with this update, so our edits to allow variant export of files of .csv and .vcf formats (premise of **`Phase_8_Option_A.md`**) needs to be addressed.  
Since lines of code can drift between versions: It is best to directly search for the actual code that we must change.  While I do reference the line numbers, to be safe, be sure to do your actual searches on word phrases that are given instead of going strictly by line numbers.

---
### Create backup of your CohortModel.js file just in case
bash  
```  
cp ~/gene.iobio/client/app/models/CohortModel.js ~/gene.iobio/client/app/models/CohortModel.js.bak
```  

### Reapply the CohortModel.js Edit  
Open **`CohortModel.js`** in nano for editing.  
bash  
```  
nano ~/gene.iobio/client/app/models/CohortModel.js
```  
Once nano opens, press  `Ctrl+W`  (search), then type:
```  
Cannot refresh flagged variant coverage
```  
That will jump you directly to the `coverage refresh` catch block.  
You'll see:
```
.catch(function(error) {
  console.log("Cannot refresh flagged variant coverage " + error)
  reject(error)
})
```

Look just below `"Cannot refresh flagged variant coverage "` and Change:  
```  
reject(error)
```  
To:
```  
resolve()
```  
Then  Save: **`Ctrl+O`, Enter, `Ctrl+X`  to exit**

Verify changes with:
bash  
```  
sed -n '2900,2910p' ~/gene.iobio/client/app/models/CohortModel.js
```  
Expected (starting on line 2900):
```  
        resolve();
      })
      .catch(function(error) {
        console.log("Cannot refresh flagged variant coverage " + error)
        resolve()
      })
    })
  }

  promiseExportFlaggedVariants(format = 'csv') {
    let self = this;
```  
Look on `line 2904` and see: **`resolve()`**  

Note:  The **`resolve()`** on `line 2904` does not require, nor does it have a semicolon — this is correct and the entry we are seeking to change.
  
The `resolve();` shown earlier in the block at `line 2900` **before** the `"Cannot refresh flagged variant coverage "` is a separate call entirely (**ignore it and do not alter the `resolve();` entry**).  

> See **`GOTCHA 8A.2`, `8A.3`, and `8A.4` in `04_GOTCHA_Enumeration.md`** for full root cause explanation of these patches.  

---  

## Step 6 — Rebuild Frontend  
bash  
```  
cd ~/gene.iobio
```  
bash  
```  
npm run build
```  
Expected:  
-   Build completes in approximately 24.6 seconds on constrained hardware    
-   No errors  
-   Output: client/dist/build.js approximately 30.5MB (your value may differ slightly)  
-   Warnings about bundle size or deprecated loaders are non-fatal  
    
If the build produces errors rather than warnings, do not proceed to serving.  
Review the error output against the **Phase 7** section of **`01_Stack_Build_Steps.md`**.   

---  

## Step 7 — Verify Build  

### Confirm Backend Still Responding   
bash  
```  
curl localhost:9001
```  
Expected: `JSON health response` confirming **`gru-backend 2.0.0`**.  
```  
{"service_description":"iobio gru backend server","version":"2.0.0","data_version": "2.0.0"}
```  
If you receive an HTML string instead of JSON, the wrong image version is running. (You should not see `<h1>I be healthful</h1>`)  
See **`GOTCHA 4.2` in `04_GOTCHA_Enumeration.md`**.  

---  

### Confirm Version String in Running UI  

Start the frontend 
bash  
```  
cd ~/gene.iobio/client
```  
bash  
```  
serve -s . -l 3000
```  
On the server system: Open Chromium-based web browser of choice (like Brave or Google Chrome).
Load the **`gene.iobio`** UI in browser by going to:
```  
http://localhost:3000
```  
Confirm the version displayed on the webpage reads **`4.13.2`**.   

---  

### Confirm Phase 8 Option B LAN Serving Operational  

If **`Phase_8_Option_B`** (LAN network serving) is configured, confirm access from a second machine on your LAN at:  
Load the **`gene.iobio`** UI in browser from a different computer on your LAN by going to:
```  
http://<your-host-ip>:3000.
```  
Like:  
```  
http://192.168.1.240:3000
```  

---  

### Confirm Phase 8 Option A File Export Feature  

Load a gene with variants. 
Confirm the **`.csv`**/**`.vcf`** export dialog appears and that both CSV and VCF export both produce valid files.  

If the export dialog does not appear, your **`Phase 8 Option A`** patches did not survive the **`pull`**.   
Return to **Step 5** and reapply.

---  
## CLEANUP Tasks  

## Cleanup .bak file  

Once **Step 7** verifications are completed successfully and before proceeding to **Step 8**, remove the temporary **`CohortModel.js.bak`** backup file created earlier:    

bash  
```  
rm ~/CohortModel.js.bak
```  
Verify it is gone:  
bash   
```  
ls ~/CohortModel.js.bak
```  
Expected:`ls: cannot access /home/mir/CohortModel.js.bak: No such file or directory`  

Your home (~) directory should be clean before taking the `wsl --export` in **Step 9**.  

## Clean up git stash  

The `git stash` created in **Step 1** is no longer needed.  
Verify and drop it:   
bash  
```  
git stash list git stash drop stash@{0} 
```  
Check the `git stash` is gone:  
bash   
```  
git stash list
```  
Expected: no output on final **`git stash list`** — empty stash  

Both tasks completed successfully helps confirm no `cruft` remains before performing your **`FINAL`** backup tar file archiving.  

---  
  
## Step 8 — Back Up: Frontend Directory Tar   
  
This step captures the upgraded **`~/gene.iobio/`** frontend directory as a standalone tar.  
This is one of three tars in your **`D_Disaster_Recovery.md`** set.  
  
**Where:** WSL terminal, logged in as your WSL user (e.g., `mir`)  
**Not PowerShell. Not Windows Explorer. WSL bash terminal only.**  
  
If WSL is not already running, open your WSL terminal now:  
- Windows Start menu → type **`PowerShell`** → open it  
- Start WSL  
bash  
```  
wsl
```  
- You should see your WSL prompt: `(base) mir@LAPTOP-...:~$`  
- If you are not in your home directory, navigate there:  
bash  
```    
cd ~
```  

### Confirm the Build is the One You Want to Archive  
bash  
```  
grep -n "version" ~/gene.iobio/package.json | head -3
```  
Expected: **`"version": "4.13.2"`**

Do not archive until this confirms **`4.13.2`**.   
Archiving before the version bump is confirmed captures the wrong state.  

### Confirm (forge) Destination Directory Exists  
bash  
```  
ls /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/
```  
Expected: Your existing frontend tars are visible. If the path does not exist, create it before continuing:  
bash  
```  
mkdir -p /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/
```  
### Create the Frontend Tar  
bash  
```  
cd ~
```  
bash  
```    
tar -cf /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_2_FINAL_20260819.tar ~/gene.iobio/
```  
No output during tar creation is normal and expected. The command will return to prompt when complete. This will take a short time — the frontend directory is small (approximately 300MB in the `4.13.1` build; your `4.13.2` value will be similar).

### Verify the Tar Exists and Get Its Size  
bash  
```  
ls -lh /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_2_FINAL_20260819.tar
```  
Confirm a non-zero file size is reported before continuing.  

### Generate MD5  
bash  
```  
md5sum /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_2_FINAL_20260819.tar
```  
Record this value. Your MD5 will differ from any reference value in this documentation — that is expected and correct. This is your personal reference value for this tar on your machine.  

### Copy to Secondary Storage (vault) and Verify  
copy from source path file name to destination path.  
cp <source path file name>  <destination path>  
bash  
```  
cp /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_2_FINAL_20260819.tar /mnt/medlaptop/e/MedLaptop\ Mirror/gene_iobio_dir_bkps/gene_iobio_frontend_dir/
```  
Verify the copy with MD5 — the value must match what you recorded above:  
bash  
```  
md5sum /mnt/medlaptop/e/MedLaptop\ Mirror/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_2_FINAL_20260819.tar
```  
Do not proceed to **Step 9** until the MD5 values match on both copies. A mismatch means the copy was incomplete or corrupted — delete and re-copy.  

---

## Step 9 — Back Up: WSL Full Stack Export  

This step exports the entire `WSL` environment — including the upgraded `frontend files`, `Node.js`, `nvm`, `Podman`, `Miniforge3`, `Phenolyzer`, and `all WSL-level configuration` — as a single tar.  
This completes your three-tar disaster recovery set.  
(If you do not remove the `CohortModel.js.bak` file and `git stash`, those too will remain in your export.)  

**After all `Step 8` MD5 verifications**:   

### Gracefully stop your frontend and backend servers:  
### In frontend WSL window:  
`Ctrl+C` to stop the server
Expect: 
```  
^C
INFO Gracefully shutting down. Please wait...
```  
 Log out of WSL type:  
bash  
```  
exit
```  
Expected: `logout`  
Close window:  
PowerShell  
```  
exit
```  
### In backend WSL window:  
To stop the backend server type:  
bash  
```  
podman stop gru-backend-2.0.0
```  
Expected response: **`gru-backend-2.0.0`**  
If no response after 15 seconds, type:  
bash  
```  
podman kill gru-backend-2.0.0
```  
To exit WSL  type:    
bash  
```  
exit
```  
Expected: `logout`  

Shut down WSL:  
PowerShell  
```  
wsl --shutdown
```  
PowerShell  
```  
exit
```  
**REBOOT your computer!**  
Immediately upon returning from reboot and after logging onto your server computer:  
**Where:** **`PowerShell`**, run as `Administrator` on the Windows host.  
**Not WSL. Not a regular PowerShell window. Elevated PowerShell only.**  

> Open an **Elevated PowerShell** Window:  
> Windows Start menu → type **PowerShell** → right-click Windows PowerShell → **Run as administrator** → Yes  

### Step 9a — Confirm forge Export Destination Exists  
Elevated PowerShell  
```  
dir D:\WSL_Backups\
```  
If the directory does not exist, create it:  
Elevated PowerShell   
```  
New-Item -ItemType Directory -Path D:\WSL_Backups
```  

### Step 9b — Confirm forge D: Has Sufficient Space  

Your previous full stack **`wsl --export`** tar was approximately `16.9GB`.   
**Confirm D: has at least 20GB free before starting**:  
Elevated PowerShell  
```  
(Get-PSDrive D).Free / 1GB
```  
If space is insufficient, clear old tars or temporary files before proceeding.  
> Do not start the export without confirmed headroom — a partial export produces a silently corrupted tar.
> Consult `D_Drive_Cleanup_Priority.md` and 
> See **`Glossary: forge, vault` in `07_OVERVIEW.md`** and
**`GOTCHA 1.3`** in **`04_GOTCHA_Enumeration.md`**.  

### Step 9c — Export WSL to Tar  
Our recommended method and file naming convention:  
```  
wsl --export Ubuntu D:\WSL_Backups\ubuntu_full_stack_v<version>_FINAL_YYYYMMDD.tar
```  

Elevated PowerShell  
```  
wsl --export Ubuntu D:\WSL_Backups\ubuntu_full_stack_v4_13_2_FINAL_20260819.tar
```  
Note: `Ubuntu` is the WSL distribution name as registered — not the version number.  
If you are unsure of your registered distribution name:  
PowerShell  
```  
wsl --list --verbose
```  
Expect something like this:
```  
  NAME      STATE           VERSION
* Ubuntu    Stopped         2
```  
Use the name exactly as shown in the `NAME` column.  

The export will run silently and return to prompt when complete.  
This will take several minutes.  
**Do not close the `PowerShell` window or interrupt the process**.  

### Step 9d — Verify the Export  

In PowerShell:  
```  
(Get-Item D:\WSL_Backups\ubuntu_full_stack_v4_13_2_FINAL_20260819.tar).Length / 1GB
```  
Expected: approximately `16GB` or larger.  
A value significantly smaller than your previous WSL tar indicates a truncated export.  
**Do not use a truncated tar for disaster recovery — delete and re-export**.  

Generate MD5 in PowerShell:  
```  
Get-FileHash D:\WSL_Backups\ubuntu_full_stack_v4_13_2_FINAL_20260819.tar -Algorithm MD5
```  
PowerShell outputs MD5 in uppercase. Record this value.  
Your MD5 will differ from any reference value in this documentation — that is expected and correct.  

### Step 9e — Copy to vault Secondary Storage and Verify  

In `PowerShell`, copy to your secondary storage machine (`vault`). Your WSL mount path for secondary storage is /mnt/medlaptop/e/ — but for a file this large, copy from `PowerShell` directly to the network share to avoid double-writing through WSL:  
Copy-Item <source path file name> <destination path>
PowerShell  
```  
Copy-Item D:\WSL_Backups\ubuntu_full_stack_v4_13_2_FINAL_20260819.tar "\\<your-vault-ip>\E\MedLaptop Mirror\Ubuntu\WSL_Backups\"
```  
Replace `<your-vault-ip>` with your secondary machine's IP address.  

Verify the copy in PowerShell:  
PowerShell  
```  
Get-FileHash "\\<your-vault-ip>\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu_full_stack_v4_13_2_FINAL_20260819.tar" -Algorithm MD5
```  
The MD5 value must match what you recorded in `Step 9d`.  
A mismatch means the network copy was incomplete — delete and re-copy.

---  

## Disaster Recovery Set — Post Upgrade State  

| Tar | Version | Date | forge D: | vault E: | KB | MD5 |   
|---|:---:|:---:|:---:|:---:|:---:|:---:|    
| `ubuntu_full_stack_v4_13_2_FINAL_20260819.tar` | WSL + frontend `v4.13.2` | 20260819 | ✅ | ✅ | `17078760` | `9301C452A1C6DBE7535915AD84B98FF8` |    
| `gru_data_2_0_0_FINAL_20260808.tar` | `gru_data 2.0.0` | 20260808 | ✅ | ✅ | `122516280` | `5A82DBDF9AA4BA75B0661777671188F6` |    
| `gene_iobio_frontend_v4_13_2_FINAL_20260819.tar` |   frontend `v4.13.2` | 20260819 | ✅ | ✅ | `310720` | `C47464D3B71CFC85E81F9B5C7A43FDC5` |    

The **`gru_data`** tar date intentionally remains **`20260808`**.
It was not modified during this upgrade, therefore no need for a new `gru_data` export at this time. Do not rename it.

**All three tars present and MD5-verified on both forge and vault
= upgrade complete and disaster recovery set current.**
Remember: My MD5 hashes will not likely match YOUR MD5 hashes as they are coming from different systems. Values provided are representative only.

---  

## Upgrade Summary  

| Step | Action | gru-backend | gru_data |  
|---|---|---|---|  
| **`Pull 4.13.2`** | **`git fetch`** + **`checkout`** | No change | No change |  
| **`config.json`** | **`Protect local backend IP`** | — | — |  
| **`Version strings`** | **`GlobalApp.js`** + **`package.json`** | — | — |  
| **`Node.js`** `v20`** | **`nvm install 20`** + **`nvm use 20`** | — | — |  
| **`npm install`** | Dependency refresh | — | — |  
| **`Phase 8A patches`** | Verify 2 + reapply 1 **edit** | — | — |  
| Rebuild frontend | **`npm run build`** | — | — |  
| Verify | UI **`version`**, backend **`health`**, **`export`**, **`LAN`** | — | — |  
| **Back up frontend** | New **`~/gene.iobio`** tar + **MD5** to **`forge`**| — | — |  
| **Back up frontend** | New **`~/gene.iobio`** **tar** + **MD5** to **`vault`**| — | — |  
| **Back up WSL** | New **`wsl --export`** + **MD5** to **`forge`** | — | — |  
| **Back up WSL** | New **`wsl --export`** + **MD5** to **`vault`** | — | — |  


**`gru-backend-2.0.0` and `gru_data 2.0.0` are unchanged.  
No backend work is required for this upgrade.**

---

## Reference  

| Item | Value |  
|---|---|  
| Upstream release tag | **`gene_4.13.2`** |  
| Release commit | **`1854f59`** |  
| Released | **`2026-08-18`** |  
| gru-backend pairing | **`2.0.0`** (**image `7e2c4a419267`**) |  
| gru_data pairing | **`2.0.0`** |  
| Node.js (confirmed) | **`v20.20.2`** |  
| npm (confirmed) | **`10.8.2`** |  
| Phase 8A patches | Required — verified after **`pull`** |  
| Phase 8B config | Protect **`config.json`** backend IP |  
| PR #1168 status | Not merged — no action required |  

---

*For build procedure from scratch: **`01_Stack_Build_Steps.md`***
*For version history: **`03_Stack_Versioning_Reference.md`***
*For gotcha reference: **`04_GOTCHA_Enumeration.md`***

---

≡≡≡≡≡ END: Upgrades/01_Frontend_Upgrade_20260819/Upgrade_v4.13.1_to_v4.13.2.md ≡≡≡≡≡
