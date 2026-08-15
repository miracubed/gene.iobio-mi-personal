# **Stack Build Steps — iobio gene.iobio-mi-personal Local Stack**  
*Reproducible Build Procedure — Phases 1–7 + Phase 8 Overview*  
*Last updated: H20260815*  

---  

## **Environment**  

| Item | Value |  
|---|---|  
| Host | LAPTOP-F (HealthMed Laptop, 192.168.1.240) |  
| OS | Windows 11 + WSL2 Ubuntu 26.04 LTS |  
| `WSL` location | `D:\WSL` |  
| `WSL` user | `mir` (sudo) - (Reader: this can be whatever you want, this is my login. Just be sure to change all references to your own login name.) |  
| Data drive | `D:` (SSK, 931GB NTFS single partition) |  
| RAM allocation | 2GB (`.wslconfig`) |  
| Swap | 14GiB total (4GB `.wslconfig` + 11GiB `/home/mir/swapfile`) |  
| Network share | `//192.168.1.52/E` → `/mnt/medlaptop/e` (persistent `fstab`) |  

> **System naming note:** This documentation refers to the two machines by short working  
> names in relevant contexts.  
> **`forge`** refers to LAPTOP-F (HealthMed Laptop, 192.168.1.240) — the active build and runtime environment.  
> **`vault`** refers to LAPTOP-1 (Personal Laptop, 192.168.1.52) — the secure storage and backup environment.  
> These terms are defined fully in the  
> **`Glossary`: `WSL`, `forge`, `vault`, `sudo` in `07_OVERVIEW.md`**.  

---

## **Phase 1 — WSL + Ubuntu Base**  
> **This phase installs Ubuntu 26.04 LTS directly to the `D:` drive using the `--location` flag.  
> This is not optional** — the `C:` drive on this build machine does not have sufficient space for WSL.  
> Installing to `C:` by default will cause space pressure that silently corrupts exports and causes cascading failures.  
> **The `--location` flag is required.**

**About usernames:** Throughout this guide, `mir` is the WSL username of the original builder. **Substitute your own chosen username wherever `mir` appears in paths and commands.** Your Windows login username and your WSL username are completely separate — your Windows profile folder will have your own Windows username, and the WSL username is set independently during Ubuntu first launch. Commands using ~ (tilde) always mean "your home directory" in WSL and work correctly for any username without modification. → **`Glossary`: ~ `(tilde)` — see `07_OVERVIEW.md`**  

**WSL backup location: Downloaded distro tar + `md5`/`MD5` backed up at \\192.168.1.52\E\MedLaptop Mirror\Ubuntu\DownloadedDistro\**  
| Step | Command / Action | Notes |  
|---|---|---|  
| Pre-flight checks	PowerShell (Admin) — verify features, space, clean state | Before any install attempt | — |   
| Remove `D:` `pagefile` | sysdm.cpl → Virtual Memory | Required before WSL install on `D:` |  
| Configure `.wslconfig` | `C:\Users\<yourname>\.wslconfig` | **Set BEFORE WSL install** |  
| WSL install to `D:` | `wsl --install -d Ubuntu --location D:\WSL` | **`--location`** required — keeps off `C:` |  
| Manual first launch | `wsl -d Ubuntu -u root` | Auto-launch after install will fail — this is expected |  
| Set default user | `/etc/wsl.conf` from inside WSL | Immediately after first launch |  
| Native swap | `fallocate -l 11G /home/mir/swapfile` + `/etc/fstab` | Use fallocate — **not dd** — on 4GB RAM |    
| `vm.overcommit_memory` | `/etc/sysctl.conf` | Persistent across reboots |  
| WSL export | PowerShell: `wsl --export Ubuntu ...` | Clean baseline — reboot first |  

> See **`Glossary`: `md5`/`MD5` at end of `07_OVERVIEW.md`**

**Why Export/Import Does Not Work Here — Read This First**  
> If you have used other WSL guides, you may have seen the `wsl --export` / `wsl --unregister` / `wsl --import`  
> approach for moving WSL to a different drive: **Do not use that approach for this build.**

On a machine where `C:` is nearly full, `wsl --export` will silently produce a truncated, corrupted tarball. The export command will appear to succeed — no error is shown — but the resulting tar file is incomplete. Every subsequent `wsl --import` attempt will fail with `WSL_E_IMPORT_FAILED` and a message about a truncated archive. This is not a recoverable situation without a fresh download.
**The `--location` flag installs Ubuntu directly to `D:` without any export or import step. It is the correct and only supported approach for this build.**  

### **Step 1 — Pre-flight Checks**  

**Where:** Windows PowerShell — run as Administrator → : PowerShell, Administrator (elevated) — see `07_OVERVIEW.md`**

Open the Start menu, type PowerShell, right-click Windows PowerShell, select Run as administrator. Click Yes on the UAC prompt.

### **1a. Verify `C:` and `D:` have the expected space:**  
Elevated PowerShell
```  
(Get-PSDrive C).Free / 1GB
```  
Elevated PowerShell
```  
(Get-PSDrive D).Free / 1GB
```  
Expected: `C:` shows some free space (at least 5GB for install overhead). `D:` shows the  
bulk of available space — in this build, **approximately 925GB**.  
If `C:` shows less than 5GB free: **Do not proceed**. Free space on `C:` before continuing.  
WSL installs some files to `C:` regardless of `--location` — approximately 500MB in 
***`C:\Program Files\WindowsApps`***.  If `C:` has less than 5GB, those writes will fail.  

### **1b. Confirm no leftover WSL distributions from prior attempts:**  
Elevated PowerShell
```  
wsl --list --verbose
```  
Expected: Windows Subsystem for Linux has no installed distributions.  
If any distributions are listed, unregister them before proceeding:  
Elevated PowerShell
```  
wsl --unregister Ubuntu
```  
Repeat for any other listed distributions.  

### **1c. Confirm D:\WSL does not exist or is empty:**  
Elevated PowerShell
```  
dir D:\WSL
```  
If the directory exists and contains an ext4.vhdx file from a prior attempt, clean it:  
In a PowerShell window:  
PowerShell  
```  
Remove-Item D:\WSL\* -Recurse -Force 2>$null
```  
PowerShell  
```  
dir D:\WSL
```  

Expected after cleanup: directory empty, or Cannot find path if it doesn't exist yet.  
Either is fine — the install creates it.  
### **1d. Confirm WSL version:**  
PowerShell  
```  
wsl --version
```  
Expected: WSL version 2.7.x or higher.  
If WSL is not installed at all, first run:  
PowerShell  
```  
wsl --update
```  

### **1e. Confirm WSL2 is the default:**  
PowerShell  
```  
wsl --set-default-version 2
```  

### **Step 2 — Remove the D: Pagefile**  

**Where:** Windows — `Run` dialog → **`Glossary`: `pagefile`, `sysdm.cpl` — see `07_OVERVIEW.md`**  

**Why:** A Windows `pagefile` on `D:` conflicts with WSL operations on `D:`.  
It must be removed before the WSL install.  
This change requires a reboot to fully apply — the reboot happens at the end of this step sequence.  

**Press `Windows + R`, type `sysdm.cpl`, press `Enter`.**  
**Navigate to: Advanced tab → Performance → Settings → Advanced tab → Virtual memory → Change**  
    • **Uncheck** `Automatically manage paging file size for all drives`  
    • **Click** `D:` in the drive list to select it  
    • **Select** `No paging file`  
    • **Click** `Set`  
    • **Leave `C:` `pagefile` as-is — do not change it**  
    • **Click** `OK`  
**A restart prompt will appear. `Dismiss` it for now — do not restart yet.** The reboot happens after **Step 3**.  

### **Step 3 — Create `.wslconfig` Before Install**  

**Where:** Windows PowerShell — run as Administrator

**Why this must be done before the install:** `.wslconfig` controls how much RAM WSL is allocated. On a 4GB RAM machine, an oversized configuration causes Hyper-V to enter a degraded state where it cannot create WSL partitions — producing `E_UNEXPECTED` errors on every install attempt. **The correct settings for this hardware must be in place before the first WSL launch.**
**No swapFile= line: Do not add a swapFile= line. The Linux-native swap file is created inside WSL in a later step. Adding swapFile= here on a 4GB machine creates a conflict.**

**Copy this entire block and run it in PowerShell (Admin)**
**Replacing `<yourwindowsusername>` with `your actual Windows login username — the name of your profile folder` under `C:\Users\`.**:  
Elevated PowerShell  
```  
@"
[wsl2]
memory=2GB
processors=1
swap=4GB
localhostForwarding=true
"@ | Set-Content C:\Users\<yourwindowsusername>\.wslconfig
```  
**Verify:** your file contents look like this:  
Elevated PowerShell  
```  
type C:\Users\<yourwindowsusername>\.wslconfig
```  
Expected output:  
```  
[wsl2]
memory=2GB
processors=1
swap=4GB
localhostForwarding=true
```  
**RAM note:** 2GB is the correct allocation for this hardware.  
It may seem low, but it is the tested and working value.  
**During large image pulls in Phase 4, you will temporarily increase this to 3GB. Always revert to 2GB after the pull completes.**  
see **`Glossary`: `pull` — `07_OVERVIEW.md`**.

### **Step 4 — Reboot Windows**  

**Where:** Windows  

**Full reboot now.** This applies the `D:` `pagefile` removal from **Step 2** and clears any residual WSL state from prior attempts.  
**Reboot. Do not skip this step.**  
After reboot, open PowerShell as Administrator and confirm the clean state:  
Elevated PowerShell  
```  
wsl --list --verbose
```  
Elevated PowerShell  
```  
(Get-PSDrive C).Free / 1GB
```  
Elevated PowerShell  
```  
(Get-PSDrive D).Free / 1GB
```  
Elevated PowerShell  
```  
dir D:\WSL
```   
Elevated PowerShell  
```  
wsl --list --verbose
```  
Elevated PowerShell  
```  
wsl --status
```  
Expected: Space numbers match what was seen in **`Step 1`**, an empty `D:\WSL` directory and no distributions listed.  

### **Step 5 — Install Ubuntu Directly to D:**  

**Where:** Windows PowerShell — run as Administrator  
Elevated PowerShell  
```  
wsl --install -d Ubuntu --location D:\WSL
```  
Expected output:  
```
`Downloading: Ubuntu`
`Installing: Ubuntu`
`Distribution successfully installed. It can be launched via 'wsl.exe -d Ubuntu'`
`Launching Ubuntu...`
`Catastrophic failure`
`Error code: Wsl/Service/E_UNEXPECTED`
```
**The "Catastrophic failure" after `--location` install is expected and normal.**  
The distribution installs successfully — Distribution successfully installed confirms this.  
The auto-launch that follows immediately after install fails because the system needs a  
manual first launch to complete provisioning.  
This is known behavior with `--location` installs. Do not be alarmed. Do not attempt to reinstall.  

Proceed to **Step 6**.  
Verify the distribution registered correctly despite the auto-launch failure:  
PowerShell  
```  
wsl --list --verbose
```    
Expected:  
```  
  NAME      STATE           VERSION
* Ubuntu    Stopped         2
```  
Also verify the `VHDX` was created on `D:`  
```PowerShell  
dir D:\WSL
```  
Expected: `ext4.vhdx` present, size approximately 1–2GB.  

### **Step 6 — First Launch and User Setup**  

**Where:** Windows PowerShell — run as Administrator  
Launch Ubuntu manually as root to complete provisioning:  
Elevated PowerShell  
```  
wsl -d Ubuntu -u root
```  
Ubuntu will run through its initial provisioning sequence.  
It will prompt you to create a default Unix user account:  
**Create a default Unix user account:**  
Type `mir` (or your chosen username)  
bash  
```  
mir
```  
and press `Enter`.  

**New password:**  
Type a password, try to avoid spaces and special characters.    
bash  
```  
your password
```  
and press `Enter`. **The cursor will not move — this is normal for Linux password entry.**  

**Retype new password:**  
Re-type your password again  
```bash  
your password
```  
and press `Enter`.  

Ubuntu will finish provisioning and drop you to a prompt:  
**`mir@LAPTOP-...:~$`**  
**If the prompt shows your Windows System32 path (/mnt/c/Windows/System32) instead of ~, that is normal for this launch method.**  
Type  `cd ~`  
bash  
```     
cd ~
```  
to move to your **`/home` directory.**  

Verify the user was created correctly:  
bash  
```  
whoami
```  
Expected: mir (or whatever name you set as your login)  

List your /home (~) directory contents:  
bash  
```  
ls /home
```  
Expected: Contents of directory ~/mir listed  

### **Step 7 — Set Default User in `wsl.conf`**  

**Where:** WSL Ubuntu terminal — user: `mir` → **`Glossary`: `wsl.conf` — see `07_OVERVIEW.md`**.  
**Set `mir` (or whatever you set your login name to) as the permanent `default login user`.  
This ensures every time WSL launches, you are logged in as `mir` rather than `root`.**  
bash  
```  
echo -e "[user]\ndefault=mir" | sudo tee /etc/wsl.conf
```  
**When prompted for a password, enter the password you created in Step 6.**  
Verify:  
bash  
```  
cat /etc/wsl.conf
```  
Expected:  
```  
[user]
default=mir
```  

### **Step 8 — Create and Enable the Native Swap File**  

**Where:** WSL Ubuntu terminal — user: `mir` → **`Glossary`: `fallocate`, `fstab` — see `07_OVERVIEW.md`**  
Use `fallocate`, not `dd`: On a 4GB RAM machine, the `dd` command writes the swap file by zero-filling  
it in RAM before committing to disk. This exhausts available memory and crashes WSL silently —  
the terminal drops back to PowerShell with no error.  
`fallocate` reserves the disk space directly without the RAM-intensive zero-fill.  
**It is the correct tool for this hardware.**  
### **8a. Navigate to home directory:**  
bash  
```  
cd ~
```  

### **8b. Create the swap file:**  
bash  
```  
sudo fallocate -l 11G /home/mir/swapfile
```  
Type `your password`  
`fallocate` completes silently — **no output is success.** If it returns an error, check available space on `D:` with  
bash
```  
df -h /home
```  
### **8c. Verify the file was created:**  
bash  
```  
ls -lh /home/mir/swapfile
```  
Expected: file present, size 11G, owned by root.  

### **8d. Set correct permissions:**  
bash  
```  
sudo chmod 600 /home/mir/swapfile
```  
Type `your password`  

### **8e. Format as swap:**  
bash  
```  
sudo mkswap /home/mir/swapfile
```
Type `your password`  
Expected output includes something like:  
```  
Setting up swapspace version 1, size = 11 GiB (11811155968 bytes)
```  

### **8f. Activate immediately:**  
bash  
```  
sudo swapon /home/mir/swapfile
```  
Type `your password`  

### **8g. Make persistent across reboots:**  
bash  
```  
echo '/home/mir/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```  
Type `your password`  

Verify:  
bash  
```    
cat /etc/fstab
```  
Expected: **`/home/mir/swapfile none swap sw 0 0`** present at the bottom.  

### **8h. Verify total swap:**  
bash  
```  
free -h
```  
Expected: Swap: row shows approximately **`14Gi total (11G Linux swap + 4G from .wslconfig), 0B used`**.  

### **Step 9 — Set `vm.overcommit_memory` (Persistent)**  
**Where:** WSL Ubuntu terminal — user: `mir` → **`Glossary`: `sysctl.conf`, `nano`** — see **`07_OVERVIEW.md`**  

bash  
```  
sudo nano /etc/sysctl.conf
```  
Type `your password`  
Add the following line at the bottom. Use menu `Edit` `>` `Paste` from the terminal menu:  
bash  
```  
vm.overcommit_memory=1
```   
**Save: Ctrl+O, Enter, Ctrl+X**  
Apply immediately:  
bash  
```  
sudo sysctl -p
```  
Type `your password`  
Expected output:  
```  
vm.overcommit_memory = 1
```  
Verify:  
bash  
```  
cat /proc/sys/vm/overcommit_memory
```  
Expected: **`1`**  
**Why:** Without this setting, `Podman` can fail with memory allocation errors even when RAM is available.  
Setting it in `/etc/sysctl.conf` makes it persistent across WSL restarts.  
See **`Glossary`: `Podman` and see `Gotcha 3.2` in `04_GOTCHA_Enumeration.mc`**.  

### **Step 10 — Re-enable C: Pagefile**  
**Where:** Windows — `Run dialog`  
Press `Windows + R`, type `sysdm.cpl`, press `Enter`.  
Navigate to: `Advanced` tab → `Performance` → `Settings` → `Advanced` tab → `Virtual memory` → Change  
    • **Confirm** `D:` is still set to `No paging file`  
    • **Select** `C: drive`  
    • **Select** `System managed size`  
    • **Click** `Set`  
    • **Click** `OK`  
**Why re-enable `C:` `pagefile`:** The `C:` `pagefile` was left as-is during the `D:` removal in **Step 2**.  
Confirming it is set to `System managed size` here ensures Windows has a `pagefile` available for  
general system stability and Podman operations. **Required for Podman stability in Phase 3.**  

### **Step 11 — WSL Terminal Paste Method**  

**Where:** WSL Ubuntu terminal — reference note  

**Important for all future WSL steps: Standard paste shortcuts (`Ctrl+V`) do not work reliably in WSL terminal windows.  
The correct paste method is:**  
**menu `Edit` `>` `Paste`** from the `terminal menu bar`.  
> `Right-click paste` behavior varies by terminal version and **may not work on all systems**.  
> **Menu-based paste via `Edit` `>` `Paste`** is the reliable method confirmed for this build.  
> **All future steps that involve pasting into WSL assume this method.**  

### **Step 12 — Export Clean Baseline**  

**Where:** First, exit WSL. Then PowerShell (regular, not elevated).  
### **12a. Exit WSL:**  
bash  
```  
exit
```  

### **12b. Shut down WSL completely:**  
PowerShell  
```  
wsl --shutdown
```  

### **12c. Reboot Windows. Full reboot before export.**  

### **12d. After reboot — confirm distribution name:**  
PowerShell  
```  
wsl --list -v
```  
**The name in the NAME column must be Ubuntu exactly. Use this name in the export command. See **`Gotcha 1.5` in `04_GOTCHA_Enumeration.mc`.**  

### **12e. Create backup directory:**  
PowerShell  
```  
mkdir D:\WSL_Backups\DownloadedDistro
```  

#### **12f. Export:**  
PowerShell  
```  
wsl --export Ubuntu "D:\WSL_Backups\DownloadedDistro\ubuntu-clean.tar"
```  
No output during export is normal. Allow several minutes.  
> **NOTE: You will be repeating this same syntax but with different file names  
> during the entire build process after each addition of a software element,  
> so pay close attention to this process: (Reboot > PowerShell > wsl --export...)**  

### **12g. Verify MD5:**  
PowerShell  
```  
Get-FileHash "D:\WSL_Backups\DownloadedDistro\ubuntu-clean.tar" -Algorithm MD5
```  
**Note and record the hash. Just like the `wsl --export...`, you will do this again and again.**  
> Any md5 values referenced throughout this guide are from the maintainer's files  
> on the maintainer's system and will likely differ on yours.  
> Keep the naming convention when creating your own files,  
> Update the file's name date portion for your own exports.  
> Verify your hash matches itself across both copy locations — `forge` `D:` and `vault` `E:`.  
> A matching hash between locations confirms the file transferred intact.  

### **12h. Copy to `vault` and verify:**  
Copy your file to your `vault` location, and `paste` the below command into `PowerShell window`  
with your IP, your file path and name of your file if it is different, and hit the `Enter`.  
The larger the file, the longer it takes Windows to process the command.  
PowerShell  
```  
"D:\WSL_Backups\DownloadedDistro\ubuntu-clean.tar" "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\DownloadedDistro\"
```
PowerShell  
```  
Get-FileHash "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\DownloadedDistro\ubuntu-clean.tar" -Algorithm MD5
```  
**Hashes must match between the md5 you originally got on your `forge` `D:` drive after creating  
the .tar file and after you copy the .tar file to your `vault` location.**  

## **Phase 1 Completion**  
| Component | Status |  
|---|:---:|  
| `C:` and `D:` space verified | ✅ |  
| `D:` `pagefile` removed | ✅ |  
| `.wslconfig` configured `(2GB RAM, 1 CPU, 4GB swap)` | ✅ |  
| `Ubuntu 26.04 LTS` installed to `D:\WSL` via `--location` | ✅ |  
| User `mir` created during provisioning | ✅ |  
| Default user set in `/etc/wsl.conf` | ✅ |  
| `11GiB swapfile` created via `fallocate` + persistent via `/etc/fstab` | ✅ |  
| Total swap confirmed ~14Gi | ✅ |  
| `vm.overcommit_memory=1` persistent via `/etc/sysctl.conf` | ✅ |  
| `C:` `pagefile` confirmed System managed | ✅ |  
| WSL terminal paste method confirmed: `Edit` `>` `Paste` | ✅ |  
| Clean baseline export ubuntu-clean.tar created + MD5 verified `forge` + `vault` | ✅ |  


> A note on the path that did not work: This build went through multiple failed attempts  
> using the `wsl --export` / `wsl --unregister` / `wsl --import` approach before  
> the `--location` flag was discovered. The export/import path fails on this hardware  
> because `C:` space pressure silently truncates the tarball — `wsl --export` reports  
> success but produces an incomplete archive. Every `wsl --import` from that archive  
> then fails with `WSL_E_IMPORT_FAILED`.  
> If you encounter this error from a prior attempt, do not retry the import — the tarball is corrupted.  
> Start from **Step 1** of this phase with a fresh `wsl --install --location install`.  
> **The `--location` flag is the correct path for space-constrained systems.**  


**WSL export procedure:**  
Reboot first. Always. No WSL terminals open prior to  
export command. Use distribution name as registered in WSL (e.g.,  
`Ubuntu`), not version number.  
> Get an md5 hash of all exported .tar file backups and keep it in  
> a text .md5 file with hash and file size information for the .tar  
> file. If your procedures fail in any way, you can begin verifying  
> the tar with this value. Sometimes, just copying a file on your  
> network from directory to directory can be imperceptibly  
> interrupted, and the file integrity compromised and unusable. This  
> is especially painful with larger files, so plan accordingly to  
> protect your work and save yourself time and aggravation later.  
  
---

## **Phase 2 — Core Tools**  

> **Before starting Phase 2:** **Phase 1** must be complete. Ubuntu is running, user `mir` is logged in, `swap` and `.wslconfig` are configured.  

| Package | Install command | Version | Notes |  
| - | - | - | - |  
| Podman | `sudo apt install podman` | 5.7.0 | VFS driver required — configured in **Phase 3** |  
| podman-compose | `sudo apt install podman-compose` | 1.5.0 | — |  
| Perl | system | 5.40.1 | Pre-installed in Ubuntu 26.04 LTS |  
| cpanminus | `sudo apt install perl cpanminus git unzip -y` | — | Installed before `Phenolyzer` clone |  
| BioPerl | `sudo apt install bioperl` | 1.7.8 | `Phenolyzer` dep |  
| libgraph-perl | `sudo apt install libgraph-perl` | 0.9735 | `Phenolyzer` dep |  
| libjson-perl | `sudo apt install libjson-perl` | 4.10 | `Phenolyzer` dep |  
| `Phenolyzer` | `git clone https://github.com/WGLab/phenolyzer` | v0.1.9-17-gf939514 | Full clone, tag `v0.4.0`, head `f939514` |  
| rclone | `sudo apt install rclone` | 1.60.1+dfsg-4ubuntu3.2 | `gru_data` `sync` — used in **Phase 5** |  
| SAMtools | `sudo apt install samtools` | 1.22.1 | Upgraded to 1.24 post **Phase 6** via `mamba` |  
| aria2 | `sudo apt install aria2` | 1.37.0+debian-4 | Optional — personal preference |  
| Miniforge3 | wget installer | 26.3.2 | Installed to `/mnt/d/miniforge3` |  


### **Step 1 — Update Package Lists**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory → **`Glossary`: `apt`, `Phenolyzer`, `gru_data`, `SAMtools`, `aria2`, `Miniforge3` — see `07_OVERVIEW.md`**  

Always update the package list before installing anything new. This ensures `apt` knows about the latest available versions.  
bash  
```  
sudo apt update
```  
Type `your password`  
Expected: a list of package sources being refreshed, ending with a line like `X packages can be upgraded.` No errors.  


### **Step 2 — Install Podman and podman-compose**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
sudo apt install podman podman-compose -y
```  
Type `your password`  
This will install Podman and its dependencies. The output will be long — this is normal. When it returns to the prompt, verify:  
bash  
```  
podman --version
```  
Expected output:  
```  
podman version 5.7.0
```  
bash   
```  
podman-compose --version
```  
Expected output:  
```  
podman-compose version 1.5.0
```

> **Important:** `Podman` requires additional configuration before it will work correctly with `WSL2`.  
>  Do not attempt to pull or run any containers yet. That configuration is covered in **Phase 3**.  
>  If you try to run `Podman` now it may fail or behave unexpectedly.  


### **Step 3 — Install rclone, samtools, and aria2**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
sudo apt install rclone samtools aria2 -y
```  
Type `your password`  
Verify each:  
bash  
```  
rclone --version
```  
Expected: first line reads  
```  
rclone v1.60.1
```    
bash  
```  
samtools --version
```  
Expected: first line reads  
```  
samtools 1.22.1
```  
bash   
```  
aria2c --version
```  
Expected: first line reads
```  
aria2 version 1.37.0
```  

> **SAMtools note:** Version 1.22.1 is installed here as a working baseline.  
> It will be upgraded to 1.24 via `mamba` after the backend is verified in **Phase 6**.  
> Do not upgrade it now — `Miniforge3 (mamba)` is not installed yet.  


### **Step 4 — Install Perl Dependencies**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory → **`Glossary`: `cpanminus` — see `07_OVERVIEW.md`**  
bash  
```  
sudo apt install perl cpanminus git unzip -y
```  
Type `your password`  
> `git` and `unzip` are included in Ubuntu 26.04 LTS by default.  
>  This command ensures they are present and current regardless.  
>  If already installed, `apt` will report `"already the newest version"`  
>  and continue without issue.  

Then install the Phenolyzer Perl library dependencies:  
bash  
```  
sudo apt install bioperl libgraph-perl libjson-perl -y
```  
Type `your password`  
This installs `BioPerl` and two required `Perl` modules that `Phenolyzer` depends on. This step can take a minute or two — the output will be long.  
When it returns to the prompt, continue.  


### **Step 5 — Install Miniforge3**  

**Where:** WSL Ubuntu terminal — user: `mir`, home directory (`~`) → **`Glossary`: `conda`, `mamba`, `wget` — see `07_OVERVIEW.md`**  

`Miniforge3` provides `conda` and `mamba` — the package managers used to install and manage bioinformatics tools including the SAMtools 1.24 upgrade in **Phase 6**.  

### **5a. Navigate to home directory:**  
bash  
```  
cd ~
```  

### **5b. Download the installer:**  
bash  
```  
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh -O ~/Miniforge3.sh
```  

### **5c. Verify the download:**  
bash  
```  
ls -lh ~/Miniforge3.sh
```  
Expected: file present, size approximately 100–120MB.  

### **5d. Run the installer in batch mode, installing to D: drive:**  
bash  
```  
~/Miniforge3.sh -b -p /mnt/d/miniforge3
```  

> **Why `-b`?** Batch mode suppresses the interactive license and prompt screens. Without it,  
>  the installer pauses waiting for keyboard input. See **`Gotcha 2.1` in `04_GOTCHA_Enumeration.mc`**.  
> **Why `/mnt/d/miniforge3`?** Installs to the `D:` drive instead of the WSL root filesystem,  
>  keeping the WSL environment lean and the large `conda` package cache off the system drive.  

This will take a few minutes. Expected output ends with:  
```  
installation finished.
```  

### **5e. Initialize conda for the bash shell:**  
bash  
```  
/mnt/d/miniforge3/bin/conda init bash
```  

### **5f. Reload the shell configuration:**  
bash  
```  
source ~/.bashrc
```  

### **5g. Verify:**  
bash  
```  
conda --version
```  
Expected:  
```  
conda 26.3.2
```  


### **Step 6 — Install Phenolyzer**  
**Where:** WSL Ubuntu terminal — user: `mir`, home directory (`~`)  

### **6a. Navigate to home directory:**  
bash  
```  
cd ~
```  

### **6b. Clone the Phenolyzer repository:**  
bash  
```  
git clone https://github.com/WGLab/phenolyzer
```  

> **Note on clone speed:** This clone is 86.72MB across 6027 objects.  
> On this hardware it runs at approximately 826 KiB/s and will appear  
> to stall in the early percentage range for several minutes.  
> This is normal — the transfer is running. Do not interrupt it.  
> If you do accidentally interrupt, check the output before deleting  
> anything — the clone may have completed just as `Ctrl+C` was pressed.    
> See the full output and verify `6027/6027 objects` before deciding to
> attempt to re-clone.  

Expected final output:  
```  
Receiving objects: 100% (6027/6027), 86.72 MiB | 826.00 KiB/s, done.  
Resolving deltas: 100% (668/668), done.
```  

### **6c. Verify Phenolyzer runs:**  

```  
cd ~/phenolyzer  
perl disease_annotation.pl --help
```  
Expected: help text describing Phenolyzer usage options. No errors.   


### **Step 7 — Create Phenolyzer Results Directory and Run Script**  

**Where:** WSL Ubuntu terminal — user: `mir`  

### **7a. Create the results output directory on D::**  
bash  
```  
mkdir -p /mnt/d/data/output/phenolyzer_results
```  

### **7b. Create the run script:**  
bash  
```  
nano ~/phenolyzer_run.sh
```  
Right-click to paste (or use menu `Edit` `>` `Paste`) the following into `nano` exactly as shown:  
```  
if [ $# -ne 2 ]; then
    echo "Usage: $0 \"<phenotype term>\" <short_name>"
    exit 1
fi

PHENOTYPE="$1"
SHORTNAME="$2"
OUTDIR=/mnt/d/data/output/phenolyzer_results/${SHORTNAME}_out

perl ~/phenolyzer/disease_annotation.pl "$PHENOTYPE" -p -ph -logistic -out "$OUTDIR"

echo ""
echo "=== Seed genes ==="
cat "${OUTDIR}.seed_gene_list"
```  
Save: **Ctrl+O**, Enter, **Ctrl+X**  

### **7c. Make the script executable:**  
bash  
```  
chmod +x ~/phenolyzer_run.sh
```  

### **Step 8 — Verify Phenolyzer End-to-End**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
~/phenolyzer_run.sh "lysinuric protein intolerance" lpi
```  
This runs a full Phenolyzer analysis using a known disease term.  
It will produce several lines of progress output while it queries its databases.  
Expected final output:  
```  
=== Seed genes ===  
Rank    Gene    ID      Score  
1       SLC7A7  9056    1
```  

**This is the confirmed verification result.**:  
```  
SLC7A7 at Rank 1, Score 1
```  
...as the seed gene is the expected correct output. Any other result indicates a problem with the install.   
If you see errors before reaching the seed genes section,  
check that all three `Perl` libraries (`bioperl`, `libgraph-perl`, `libjson-perl`) installed without errors in **Step 4**.  


### **Step 9 — WSL Export: Phenolyzer Restore Point**  

**Where:** First, exit WSL. Then PowerShell (regular, not elevated).  

This export captures the full WSL environment including all Phase 2 tools and the verified Phenolyzer install.  

### **9a. Exit WSL:**  
bash  
```  
exit
```  

### **9b. Shut down WSL completely:**  
PowerShell  
```  
wsl --shutdown
```  

#### **9c. Reboot Windows. Full reboot before export.**  

### **9d. After reboot — export from PowerShell:**  
> Replace the date information `20260728` with your date information.  
> Do this any time this guide instructs you do do a tar export as backup.

PowerShell  
```  
wsl --export Ubuntu "D:\WSL_Backups\ubuntu-phenolyzer-20260728.tar"
```  

### **9e. Verify the export with MD5:**  
PowerShell  
```  
Get-FileHash "D:\WSL_Backups\ubuntu-phenolyzer-20260728.tar" -Algorithm MD5
```  

Note the hash. Record it for your future reference to verify your files if needed in the future.  
We never plan for disaster, but we try to plan to mitigate disasters when they occur.  

### **9f. Copy to `vault` and verify there too:**  
PowerShell  
```  
copy "D:\WSL_Backups\ubuntu-phenolyzer-20260728.tar" "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\"
```  

Then verify the copy:  
PowerShell  
```  
Get-FileHash "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu-phenolyzer-20260728.tar" -Algorithm MD5
```  

Hashes must match. If they don't, the copy was incomplete — repeat the copy step.  
> Save this md5 information for later reference as described above.  
> Your md5 hashes will not likely match those mentioned in this documentation, but    
> should match each other between your own `forge` and `vault` file copies.  

## **Phase 2 Completion**  

| Component | Version | Status |  
| - | - | - |  
| Podman | 5.7.0 | ✅ |  
| podman-compose | 1.5.0 | ✅ |  
| rclone | 1.60.1 | ✅ |  
| SAMtools | 1.22.1 (1.24 upgrade in Phase 6) | ✅ |  
| aria2 | 1.37.0 | ✅ |  
| Perl | 5.40.1 | ✅ system |  
| cpanminus | — | ✅ |  
| BioPerl | 1.7.8 | ✅ |  
| libgraph-perl | 0.9735 | ✅ |  
| libjson-perl | 4.10 | ✅ |  
| Phenolyzer | v0.1.9-17-gf939514 | ✅ verified — SLC7A7 Rank 1 |  
| Miniforge3 (conda/mamba) | 26.3.2 | ✅ |  
| Restore point | ubuntu-phenolyzer-20260728.tar | ✅ MD5 verified `forge` + `vault` |  

---

## **Phase 3 — Podman Container Storage Fix**  

> **Before starting Phase 3:** **Phase 2** must be complete.  
> Podman is installed but has not yet been configured.  
> Do not attempt to pull or run containers until this phase is complete.  

> **Why this phase exists:** Podman's default storage driver (`overlay`) is incompatible with the WSL2 ext4 filesystem.  
> Without this fix, Podman will fail silently or hang when pulling or running container images. This phase configures  
> Podman to use the `vfs` driver, which is fully compatible with WSL2.
> See **`Gotcha 3.1` in `04_GOTCHA_Enumeration.mc`**.  

| Item | Value |  
| - | - |  
| `~/.config/containers/containers.conf` `[storage]` | `driver="vfs"` |  
| `~/.config/containers/containers.conf` `[engine]` | `image_parallel_copies=1` |  
| `runRoot` | `/home/mir/.local/share/containers/run` |  
| `vm.overcommit_memory` | `1` (persistent via `/etc/sysctl.conf`) |  
| `C:` `pagefile` | Re-enabled via `sysdm.cpl` |  


### **Step 1 — Create the Containers Configuration Directory**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory → **`Glossary`: `mkdir` — see `07_OVERVIEW.md`**  

The directory for the Podman configuration file may not exist yet. Create it:  
bash  
```  
mkdir -p ~/.config/containers
```  

> The `-p` flag creates the full directory path including any parent directories that don't exist yet.  
> If the directory already exists, this command does nothing — it is safe to run regardless.  


### **Step 2 — Create and Configure containers.conf**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
nano ~/.config/containers/containers.conf
```  

Copy block below, in `nano' Right-click (if this works on your system) or  
menu `Edit` `>` `Paste` to paste the following into `nano` exactly as shown:  
```  
[engine]  
image_parallel_copies=1  
runRoot="/home/mir/.local/share/containers/run"  
  
[storage]  
driver="vfs"
```  
Save: **Ctrl+O**, Enter, **Ctrl+X**  

> **Why `image_parallel_copies=1`?** Prevents simultaneous pull conflicts on resource-constrained systems. See **`Gotcha 3.3` in `04_GOTCHA_Enumeration.mc`**.  

> **Why explicit `runRoot`?** Avoids socket conflicts with rootless Podman on WSL2. See **`Gotcha 3.4` in `04_GOTCHA_Enumeration.mc`**.  

> **Why `driver="vfs"`?** The overlay driver fails silently or hangs on WSL2 ext4. VFS is slower but fully compatible. See **`Gotcha 3.1` in `04_GOTCHA_Enumeration.mc`**.  

Verify the file looks exactly right:  
bash  
```  
cat ~/.config/containers/containers.conf
```  
Expected output:  
```  
[engine]  
image_parallel_copies=1  
runRoot="/home/mir/.local/share/containers/run"  
  
[storage]  
driver="vfs"
```  

### **Step 3 — Verify `vm.overcommit_memory` is Still Set**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

This was set in Phase 1 Step 7. Confirm it is still active:  
bash  
```  
cat /proc/sys/vm/overcommit_memory
```  

Expected output: **`1`**  

If the output is `0`, it did not persist. Re-apply:  
bash   
```  
sudo nano /etc/sysctl.conf
```  
Type `your password`  
Confirm the following line is present at the bottom. If not, add it:  
```  
vm.overcommit_memory=1
```  
Save: **Ctrl+O**, Enter, **Ctrl+X**  

Re-apply without rebooting:  
bash   
```  
sudo sysctl -p
```  
Type `your password`  
Expected output:  
```  
vm.overcommit_memory = 1
```  

### **Step 4 — Verify Podman Basic Function**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

Confirm `Podman` is operational with the new configuration:  
bash  
```  
podman info | grep -i driver
```  

Expected output includes:  
bash  
```  
graphDriverName: vfs
```  

This confirms Podman is reading the new configuration and using the correct storage driver.  

> **Expected warning:** `WARN[0000] "/" is not a shared mount` will appear.  
> This is harmless and expected on every Podman operation in WSL2.  
> See **`Gotcha 6.2` in `04_GOTCHA_Enumeration.mc`**.  


### **Step 5 — Temporarily Increase RAM for Image Pull**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

The `gru-backend image` (4.43GB) requires more RAM to decompress and load than the standard 2GB allocation.  
Before pulling in **Phase 4**, increase the allocation temporarily.  
bash  
```  
nano /mnt/c/users/mir/.wslconfig
```  

Change `memory=2GB` to `memory=3GB`.  

The file should look like this when you complete your edits:  
```  
[wsl2]  
memory=3GB  
processors=1  
swap=4GB  
localhostForwarding=true
```  
Save: **Ctrl+O**, Enter, **Ctrl+X**  

Check it:  
bash  
```  
cat /mnt/c/users/mir/.wslconfig
```  
The saved file contents should look like this:  
```  
[wsl2]  
memory=3GB  
processors=1  
swap=4GB  
localhostForwarding=true
```  

For the change to take effect, WSL must be fully restarted:  
bash  
```  
exit
```  

In PowerShell:  
PowerShell  
```  
wsl --shutdown
```  

Then relaunch WSL:  
PowerShell  
```  
wsl
```  

Verify the change took effect:  
bash  
```  
free -h
```  
Expected: **`Mem:` row shows approximately `3.0Gi` total**.  

> **Important:** This is a temporary setting for the image pull only.  
> After **Phase 4** is complete, revert `memory=3GB` back to `memory=2GB` using the same steps.  
> Running at 3GB for day-to-day operation is not necessary and leaves less headroom for Windows.   
> See **Phase 4** completion note.  


### **Step 6 — WSL Export: Podman Configured Restore Point**  

**Where:** First, exit WSL. Then PowerShell (regular, not elevated).  

> **Note on export timing:** This export is taken with RAM temporarily set to 3GB.  
> That is correct — the `.wslconfig` change is part of the configured state being captured.  
> The 3GB setting will be reverted in **Phase 4** after the pull completes, and a new export will be taken at that point.  

### **6a. Exit WSL:**  
bash  
```  
exit
```  

### **6b. Shut down WSL completely:**  
PowerShell  
```  
wsl --shutdown
```  

### **6c. Reboot Windows.** Full reboot before export.  

### **6d. Immediately After reboot — export from PowerShell:**  
PowerShell  
```  
wsl --export Ubuntu "D:\WSL_Backups\ubuntu-podman-20260728.tar"
```  

### **6e. Verify MD5:**  
PowerShell  
```  
Get-FileHash "D:\WSL_Backups\ubuntu-podman-20260728.tar" -Algorithm MD5
```  

Note the hash. Record it.  

> MD5 is from maintainer's system and will likely differ on yours.  
> Keep the naming convention, update the date portion for your own export.  
> Verify that your hash matches itself across both copy locations — `forge` `D:` and `vault` `E:`.  
> A matching hash between two locations confirms the file transferred intact.  
> A mismatch means the copy was incomplete or corrupted — re-copy and re-verify.  

### **6f. Copy to `vault` and verify:**  
PowerShell  
```  
copy "D:\WSL_Backups\ubuntu-podman-20260728.tar" "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\"
```  
PowerShell  
```  
Get-FileHash "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu-podman-20260728.tar" -Algorithm MD5
```  

Hashes must match. If they don't, re-copy and re-check md5 values.  

## **Phase 3 Completion**  

| Component | Status |  
| - | - |  
| `~/.config/containers/containers.conf` created | ✅ |  
| Storage driver set to `vfs` | ✅ |  
| `image_parallel_copies=1` set | ✅ |  
| `runRoot` explicitly configured | ✅ |  
| `vm.overcommit_memory=1` verified persistent | ✅ |  
| Podman `graphDriverName: vfs` confirmed | ✅ |  
| `.wslconfig` temporarily set to `3GB` for **Phase 4** pull | ✅ |  
| Restore point exported + MD5 verified `forge` + `vault` | ✅ |  

---

## **Phase 4 — gru-backend Image Pull**  

> **Before starting Phase 4:** Phase 3 must be complete.  
> Podman is configured with the VFS driver. `.wslconfig` is temporarily  
> set to `memory=3GB` from **Phase 3 Step 5**.  
> WSL has been restarted and RAM confirmed at approximately 3GB.  

> **What this phase does:** Pulls the gru-backend Docker image from Docker Hub into Podman local storage.  
> This is a 4.43GB download and will take significant time depending on your connection.  
> No containers are started in this phase — pull and verify only.  

| Item | Value |  
| - | - |  
| Image | `docker.io/iobio/iobio-gru-backend:2.0.0` |  
| Image ID | `7e2c4a419267` |  
| Size | 4.43GB |  
| VEP version | ensembl-vep 115 |  
| Pull command | `podman pull docker.io/iobio/iobio-gru-backend:2.0.0` |  

See **`Glossary`: `VEP` in `07_OVERVIEW.md`**  

### **Step 1 — Pull the gru-backend Image**

**Where:** WSL Ubuntu terminal — user: `mir`, any directory
bash  
```  
podman pull docker.io/iobio/iobio-gru-backend:2.0.0
```  

> **Expected warning:** `WARN\[0000\] "/" is not a shared mount` will appear at the start.  
> **This is harmless and expected on every Podman operation in WSL2.**

See **`Glossary`: `WSL2` in `07_OVERVIEW.md` and `Gotcha 6.2` in `04_GOTCHA_Enumeration.md`**.  

This pull is 4.43GB and will take considerable time. Progress is shown layer by layer.  
A slow or apparently stalled progress bar is normal — the layers are large and decompression  
is CPU and memory intensive on constrained hardware. Do not interrupt unless the terminal is  
completely unresponsive for an extended period.  

Expected final output:  
```  
Storing signatures  
7e2c4a419267...
```  

The image ID `7e2c4a419267` confirms you have pulled the correct 2.0.0 image.  
Note this ID — it matters. See **`Gotcha 4.2` in `04_GOTCHA_Enumeration.md`**.  


### **Step 2 — Verify the Pull**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

Confirm the image is present in local Podman storage:  
bash  
```  
podman images
```  

Expected output includes a row like:  
```  
REPOSITORY                          TAG     IMAGE ID      CREATED       SIZE  
docker.io/iobio/iobio-gru-backend   2.0.0   7e2c4a419267  ...           4.43 GB
```  

Confirm the image ID matches `7e2c4a419267`.  
If it does not match, do not proceed — you may have pulled a different version.  

> **Why the image ID matters:** A container can be named anything regardless of which  
> image it actually runs. Later, if a container named `gru-backend-2.0.0` is running but  
> the image ID is wrong, the entire stack will appear broken for reasons that are very  
> difficult to diagnose. The image ID is the only reliable confirmation you have the  
> correct image. This was the root cause of the Phase 7 API mismatch in the original build.  
> See **`Gotcha 4.2` in `04_GOTCHA_Enumeration.mc`**.  


### **Step 3 — Revert `.wslconfig` to 2GB**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

The pull is complete. Revert RAM allocation back to 2GB for normal operation:  
bash  
```  
nano /mnt/c/users/mir/.wslconfig
```  

Change `memory=3GB` back to `memory=2GB`. The file should look like:  
```   
[wsl2]  
memory=2GB  
processors=1  
swap=4GB  
localhostForwarding=true
```  
Save: **`Ctrl+O`**, Enter, **`Ctrl+X`**

Verify:  
bash  
```  
cat /mnt/c/users/mir/.wslconfig
```  
Confirm `memory=2GB` is present before continuing.  


### **Step 4 — WSL Export: gru-backend Pulled Restore Point**  

**Where:** First, exit WSL. Then PowerShell (regular, not elevated).  

This export captures the WSL environment with the `gru-backend 2.0.0` image pulled and `Podman` correctly configured.  
This is an important restore point — the image pull is the most time-consuming single step in the build.  
If anything goes wrong in later phases, restoring from this point saves re-downloading 4.43GB.  

### **4a. Exit WSL:**  
bash  
```  
exit
```  

### **4b. Shut down WSL completely:**  
PowerShell  
```  
wsl --shutdown
```  

### **4c. Reboot Windows.** Full reboot before export.  

#### **4d. After reboot — export from PowerShell:**  
PowerShell  
```  
wsl --export Ubuntu "D:\WSL_Backups\ubuntu-gru-pulled-20260730.tar"
```  
> This export will be significantly larger than previous exports — approximately  
> 11–12GB — because the 4.43GB container image is now stored inside the WSL environment.  
> Allow extra time.  

### **4e. Verify MD5:**
PowerShell  
```
Get-FileHash "D:\WSL_Backups\ubuntu-gru-pulled-20260730.tar" -Algorithm MD5
```

Note the hash. Record it.  

> MD5 is from maintainer's system and will likely differ on yours. Keep the naming convention,  
> update the date portion for your own export. Verify your hash matches itself across both copy  
> locations — `forge` `D:` and `vault` `E:`. A matching hash between locations confirms the file  
> transferred intact. A mismatch means the copy was incomplete or corrupted — re-copy and re-verify.  

### **4f. Copy to `vault` and verify:**  
PowerShell  
```  
copy "D:\WSL_Backups\ubuntu-gru-pulled-20260730.tar" "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\"
```  
PowerShell  
```  
Get-FileHash "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu-gru-pulled-20260730.tar" -Algorithm MD5
```  

Hashes must match. If they don't, re-copy.  


## **Phase 4 Completion**  

| Component | Status |  
| - | - |  
| gru-backend 2.0.0 image pulled | ✅ |  
| Image ID `7e2c4a419267` confirmed | ✅ |  
| Image size 4.43GB confirmed | ✅ |  
| `.wslconfig` reverted to `memory=2GB` | ✅ |  
| Restore point exported + MD5 verified `forge` + `vault` | ✅ |  


> **Note on the 1.17.0 image:** The original build pulled `iobio-gru-backend:1.17.0` first before 2.0.0 was available.  
> If you are following this guide fresh, pull 2.0.0 only. Do not pull 1.17.0.  
> If both images are present simultaneously and the wrong one is started under the 2.0.0 container name,  
> the stack will appear broken in ways that are very difficult to diagnose.  
> See **`Gotcha 4.2` in `04_GOTCHA_Enumeration.mc`** and the `H20260806` session entry in **`02_Session_Handoff_Log.md`**.    

---

## **Phase 5 — `gru_data` Directory Build**  

> **Before starting Phase 5: Phase 4** must be complete. `gru-backend 2.0.0` image is pulled and verified. `.wslconfig` is back to `memory=2GB`.  

> **What this phase does:** Downloads the `gru_data 2.0.0` reference dataset (~128GiB) from `files.iobio.io` using `rclone`.  
> This is the largest and most time-intensive phase of the entire build. The download is resumable — if it is interrupted,  
> `rclone` can be re-run and will skip completed files. Plan for this to run over multiple sessions if needed.  

> **Read this section fully before starting the download.** The gotchas here are important and some of them are not obvious until it is too late.  

| Item | Value |  
| - | - |  
| Dataset | `gru_data 2.0.0` |  
| Full size | ~128GiB |  
| Source | `https://files.iobio.io` via rclone HTTP backend |  
| Destination | `/mnt/d/gru_data/` |  
| `rclone` command type | `sync` (`idempotent` — safe to re-run) |  
| `VERSION` file | `2.0.0` |  

See **`Glossary`: `rclone`, `sync`, `idempotent` in `07.OVERVIEW.md`**  

### **Step 1 — Check Available Space on D:**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

Before starting a ~128GiB download, confirm `D:` has enough headroom:  
bash  
```  
df -h /mnt/d
```  

Expected: `Avail` column shows at least 130GB free. More is better — the download and a working  
WSL environment need to coexist on `D:` throughout this phase and all remaining phases.  

> **Space planning note:** `gru_data 2.0.0` at full size is ~128GiB.  
> The WSL environment plus container image already occupies space on `D:`.  
> If space is tight, see `D_Drive_Cleanup_Priority.md` before proceeding.  


### **Step 2 — Create the gru_data Directory**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
mkdir -p /mnt/d/gru_data
```  

Verify it exists:  
bash  
```  
ls -la /mnt/d/ | grep gru_data
```  
Expected: `gru_data` directory listed.  


### **Step 3 — Pre-delete the `VERSION` File**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

> **Why:** The `rclone` HTTP backend cannot reliably compare modification times for small  
> files where size may match across versions.  
> The `VERSION` file is small and its size may be identical between old and new versions.  
> Pre-deleting it guarantees rclone pulls a fresh copy rather than potentially skipping it.  
> See **`Gotcha 5.2` in `04_GOTCHA_Enumeration.mc`**.  

> Do NOT pre-delete large files (vep-cache, references). Only delete `VERSION`.  

If a `VERSION` file already exists from a previous attempt:  
bash  
```  
rm -f /mnt/d/gru_data/VERSION
```  

If this is a fresh build, this command does nothing — it is safe to run regardless.  


### **Step 4 — Run the rclone Sync**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

> > >**Before typing this command, read all the notes below it.**< < <

Copy and paste into WSL terminal window.  
bash  
```  
rclone sync --progress
  --http-url https://files.iobio.io
  :http:gru_data/data/gru_data_2.0.0/
  /mnt/d/gru_data/
```  

> **Critical: use the FULL dataset path.** The path above (`gru_data/data/gru_data_2.0.0/`) is the  
> complete 2.0.0 dataset.  
> There is also an update path (`updates_1.14.0_to_2.0.0/`) which is a delta only and is incomplete —  
> it is missing approximately 9.8GiB of files. Always use the full path.  
> See **`Gotcha 5.7` in `04_GOTCHA_Enumeration.mc`**.  

> **`:http:` syntax:** The `:http:` portion is literally colon-http-colon. It is not `://`.  
> Copy and paste this command rather than typing it to avoid misreading this sequence.  
> See **`Gotcha 5.5` in `04_GOTCHA_Enumeration.mc`**.   

> **rclone sync is idempotent:** Running this command multiple times produces the same result as running it  
> once.  
> Completed files are skipped. If the download is interrupted for any reason — network drop, system sleep,  
> WSL crash — simply re-run the exact same command. It will resume from where it left off, skipping  
> already-completed files. Partially downloaded files restart from the beginning.  
> See **`Gotcha 5.6` in `04_GOTCHA_Enumeration.mc`**.  

> **Expected `403` on `lost+found/`**: rclone will log one `403 error` on `lost+found/` during every `sync`.    
> This is a server-side filesystem artifact.  
> It is not a transfer error and does not affect the download.  
> See **`Gotcha 5.4` in `04_GOTCHA_Enumeration.mc`**.  

> **Speed and ETAs:** Transfer speed and estimated completion times are unreliable, especially during US and  
> Philippines business hours when `files.iobio.io` experiences heavy traffic. A slow or stalled-looking progress  
>  bar is normal. Do not assume failure based on ETA alone.  
> See **`Gotcha 5.22` in `04_GOTCHA_Enumeration.mc`**.  

> **Shared network:** If this machine is on a shared network connection during active use, add `--bwlimit 5M` to  
> the command to limit bandwidth consumption. The download will be slower but will not impact other network users.  
> See **`Gotcha 5.20` in `04_GOTCHA_Enumeration.mc`**.  

> **`.wslconfig` RAM during long downloads:** The download itself does not require 3GB RAM — rclone runs fine at 2GB.  
> Do not change `.wslconfig` for this phase.  


### **Step 5 — Monitor the Download**  

**Where:** WSL Ubuntu terminal — user: `mir`  

The `--progress` flag shows a live transfer display. You will see:  
- Individual file names as they transfer  
- Transfer speed (will vary widely)  
- Total transferred vs total size  
- One `403 error` on `lost+found/` — expected, ignore it  

The download is expected to take many hours.  
It is safe to leave it running unattended.  
> If your terminal session closes or WSL restarts,  
> re-run the exact command from **Step 4**.  

> **Modern Standby / sleep note:** Windows Modern Standby can suspend WSL during unattended operation,  
> interrupting the download. If you have not already applied the Modern Standby workaround documented  
> in `03_Stack_Versioning_Reference.md`, consider doing so before starting a long unattended download.  
> The workaround locks the power scheme to `High Performance` and disables `Wi-Fi adapter power management`.  


### **Step 6 — Verify the Download is Complete**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

When `rclone` finishes, verify the `VERSION` file first:  
bash  
```  
cat /mnt/d/gru_data/VERSION
```  

Expected output:  
```  
2.0.0
```  

> If the `VERSION` file is missing or shows a different version, the sync did not complete correctly: Re-run **Step 4**.  

Then verify the top-level directory structure:  
bash  
```  
ls -la /mnt/d/gru_data/
```  

Expected top-level contents:  
```  
annotations/  
vep-cache/  
references/  
md5_reference_cache/  
data/  
geneinfo/  
hpo/  
gene2pheno/  
genomebuild/  
gnomad_header.txt  
CHANGELOG.md  
VERSION
```  

> **2.0.0 directory structure note:** gru_data 2.0.0 is significantly restructured from earlier versions.  
> Annotation data that previously lived at the top level is now consolidated under `annotations/GRCh37/` and  
> `annotations/GRCh38/`. If you see top-level directories like `gnomad/` alongside `annotations/`, you may  
> have a mixed state from a previous version. Unless you are updating/upgrading instead of doing the  
> FULL 2.0.0 installation: You should not see a `FILES_TO_DELETE` direectory.  
> See **`Glossary`: `gnomAD` in `07_OVERVIEW.md` and `Gotcha 5.10` in `04_GOTCHA_Enumeration.md`**.  

Verify the key subdirectories:  
bash  
```  
ls /mnt/d/gru_data/annotations/
```  
Expected: `GRCh37/` and `GRCh38/`  

bash  
```  
ls /mnt/d/gru_data/vep-cache/homo_sapiens_merged/
```  
Expected: `115_GRCh37/` and `115_GRCh38/`  

bash  
```  
ls /mnt/d/gru_data/references/
```  
Expected: `GRCh37/` and `GRCh38/`  


### **Step 7 — Apply `FILES_TO_DELETE`**  
(Unnecessary if you are using the FULL `gru backend 2.0.0` pull path and have no previous gru installations)  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

> **Sequencing is critical:** Only apply `FILES_TO_DELETE` after the sync is complete and the directory structure is verified.  
> Never delete before the download finishes.  
> See **`Gotcha 5.13` in `04_GOTCHA_Enumeration.mc`**.  

The `FILES_TO_DELETE` file lists old files from previous gru_data versions that are no longer needed.  
These files do not break 2.0.0 function if present — they are simply ignored by `gru-backend 2.0.0`.  
However removing them reclaims significant disk space.  

View what will be deleted first:  
bash  
```  
cat /mnt/d/gru_data/FILES_TO_DELETE
```  

Review the list. When satisfied, apply it:  
bash  
```  
cd /mnt/d/gru_data
while IFS= read -r file; do
  [ -n "$file" ] && rm -f "$file"
done < FILES_TO_DELETE
```  

> This reads each filename from FILES_TO_DELETE and removes it.  
> Files that are already absent are skipped silently.  
> Safe to run even if some listed files are not present.  

Verify space was reclaimed:  
bash  
```  
df -h /mnt/d
```  

The `Avail` (available) column should show more free space than before deletion.  


### **Step 8 — Verify md5_reference_cache**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

> The `md5_reference_cache` directory is pre-populated by iobio in the 2.0.0 dataset.  
> Running `seq_cache_populate.pl` against it is verification only — not population.  
> All entries will report "Already exists." This is correct and expected.  
> See **`Gotcha 5.17` in `04_GOTCHA_Enumeration.mc`**.  

**Prerequisite:** `seq_cache_populate.pl` requires SAMtools in PATH. Verify SAMtools is available:  
bash  
```  
samtools --version
```  
Expected: first line reads  
```  
samtools 1.22.1
```  
(the version installed in **Phase 2** — this is sufficient for this verification step).  

Run verification for `GRCh37`:    
bash  
```  
seq_cache_populate.pl
  -root /mnt/d/gru_data/md5_reference_cache
  /mnt/d/gru_data/references/GRCh37/human_g1k_v37_decoy_phix.fasta
```  
Expected: all entries report **`Already exists`**  

Run verification for `GRCh38`:  
bash  
```  
seq_cache_populate.pl
  -root /mnt/d/gru_data/md5_reference_cache
  /mnt/d/gru_data/references/GRCh38/human_g1k_v38_decoy_phix.fasta
```  
Expected: all entries report `Already exists`  

> **GRCh38 FASTA naming note:** The `GRCh38` reference ships with `chr1`, `chr2` style contig naming (UCSC style)  
> rather than Ensembl `1`, `2` style. This is intentional. The VEP cache subdirectory numbers are filesystem  
> sharding labels only, not contig name assertions.  
> See **`Gotcha 5.14` in `04_GOTCHA_Enumeration.mc`**.  


#### **Step 9 — WSL Export: gru_data Verified Restore Point**  

**Where:** First, exit WSL. Then PowerShell (regular, not elevated).  

> **Important:** WSL exports capture the Ubuntu environment only — NOT `/mnt/d/gru_data/`.  
> The `gru_data` directory lives on `D:` outside WSL and is not included in the WSL tar.  
> `gru_data` has its own separate backup.  
> See **`Gotcha B.1` in `04_GOTCHA_Enumeration.mc`**.  

This WSL export captures the environment state after `gru_data` is verified —   
a clean restore point before the backend is started for the first time.  

### **9a. Exit WSL:**  
bash  
```  
exit
```  

### **9b. Shut down WSL completely:**  
PowerShell  
```  
wsl --shutdown
```  

#### **9c. Reboot Windows.** Full reboot before export.  

### **9d. After reboot — immediately export WSL from PowerShell:**  
PowerShell  
```  
wsl --export Ubuntu "D:\WSL_Backups\ubuntu-gru-verified-20260801.tar"
```  

### **9e. Verify MD5:**  
PowerShell  
```  
Get-FileHash "D:\WSL_Backups\ubuntu-gru-verified-20260801.tar" -Algorithm MD5
```  

Note and record the hash.  

> MD5s referenced anywhere in these documents are from maintainer's system and will likely differ on yours.  
> Keep the naming convention, update the date portions for your own exports.  
> Verify your hash matches itself across both of your copy locations (`forge`/`vault`).  

### **9f. Copy to `vault` and verify:**  
PowerShell  
```  
copy "D:\WSL_Backups\ubuntu-gru-verified-20260801.tar" "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\"
```  
PowerShell  
```  
Get-FileHash "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\ubuntu-gru-verified-20260801.tar" -Algorithm MD5
```  

Hashes must match.  

#### **9g. Create the gru_data archive separately:**  

> This is the separate gru_data backup — this one DOES capture the 128GiB dataset.  
> It will be large (~116GB uncompressed) and will take significant time.  
> Use uncompressed tar — the dataset files are already compressed internally and  
> recompressing adds CPU time with minimal size savings.  
> See **`Gotcha B.2` in `04_GOTCHA_Enumeration.mc`**.

bash  
```  
cd /mnt/d
```  
bash   
```  
tar -cf /mnt/d/gene_iobio_dir_bkps/gene_iobio_backend_dir/gru_data_2_0_0_FINAL_20260805.tar gru_data/
```  

> Create the destination directory first if it does not exist:  

bash  
```  
mkdir -p /mnt/d/gene_iobio_dir_bkps/gene_iobio_backend_dir
```  

Verify MD5 after completion:  
PowerShell  
```  
md5sum /mnt/d/gene_iobio_dir_bkps/gene_iobio_backend_dir/gru_data_2_0_0_FINAL_20260805.tar
```  

Note and record the hash. Copy to `vault` and verify hash matches.  


> Note on `gru_data` backup sufficiency: Once the gru_data 2.0.0 sync is verified complete and
> your tar archive is created and MD5-verified at the end of Phase 5, that archive is your
> permanent FINAL backup for the backend data directory.
> `Phase_8_Option_A.md` and `Phase_8_Option_B.md` make no changes whatsoever to `/mnt/d/gru_data/`
> — not a single file of `gru_data` is touched by either phase.
> There is no need to re-archive `gru_data` after completing `Phase 8`.
> Your `Phase 5` tar is sufficient as the FINAL backend backup regardless of what phase you are on.


## **Phase 5 Completion**  

| Component | Status |  
| - | - |  
| `gru_data 2.0.0` `sync`ed via `rclone` | ✅ |  
| `VERSION` confirmed `2.0.0` | ✅ |  
| Directory structure verified | ✅ |  
| `annotations/GRCh37/` + `GRCh38/` present | ✅ |  
| `vep-cache/homo_sapiens_merged/115_GRCh37/` + `115_GRCh38/` present | ✅ |  
| `references/GRCh37/` + `GRCh38/` present | ✅ |  
| `FILES_TO_DELETE` applied (if upgrading/updating from previous `gru backend` installation | ✅ |  
| `md5_reference_cache/` verified (`GRCh37` + `GRCh38`) | ✅ |  
| WSL restore point exported + MD5 verified `forge` + `vault` | ✅ |  
| `gru_data` archive created + MD5 verified `forge` + `vault` | ✅ |  

---

## **Phase 6 — Backend Verification**  

> **Before starting Phase 6:** **Phase 5** must be complete.  
> `gru_data 2.0.0` is `sync`ed, verified, and the directory structure is confirmed.  
> WSL is running, `.wslconfig` is at `memory=2GB`.  

> **What this phase does:** Starts the `gru-backend` container for the first time against the real `gru_data 2.0.0` dataset  
> and confirms it returns a healthy response.  
> This is the first time the full backend stack runs end-to-end.  
> The frontend is not yet built — backend verification happens first, in isolation.  

| Item | Value |  
| - | - |  
| Container name | `gru-backend-2.0.0` |  
| Image | `docker.io/iobio/iobio-gru-backend:2.0.0` |  
| Image ID | `7e2c4a419267` |  
| Data mount | `/mnt/d/gru_data` → `/gru_data` |  
| Port | `9001` |  
| Health endpoint | `http://localhost:9001` |  
| Expected response | `{"service_description":"iobio gru backend server","version":"2.0.0","data_version":"2.0.0"}` |  



### **Step 1 — Verify the Correct Image is Present**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

Before starting anything, confirm the correct image is loaded in `Podman`:  
bash  
```  
podman images
```  

Confirm the output includes:  
```  
docker.io/iobio/iobio-gru-backend   2.0.0   7e2c4a419267  ...   4.43 GB
```  

The **image ID `7e2c4a419267` must match exactly**.  
If it does not, stop here and return to **Phase 4**.   

> **Why this check matters:** A container can be named anything regardless of which image it actually runs.  
> If the wrong image runs under the `gru-backend-2.0.0` container name, the entire stack will appear broken  
> in ways that are very difficult to diagnose. **The image ID is the only reliable confirmation.**  
> This was the root cause of the API mismatch in the original build — the 1.17.0 image was running under the  
> 2.0.0 container name for an entire session before it was caught.  
> See **`Gotcha 4.2` in `04_GOTCHA_Enumeration.mc` and `H20260806` in `02_Session_Handoff_Log.md`**.  


### **Step 2 — Open Two WSL Terminals**  

**Where:** Windows  

This phase requires two WSL terminals open simultaneously:  

- **Terminal 1** runs the backend container in the foreground  

- **Terminal 2** sends the health check curl command  

Open your first WSL terminal.  
Then open a second WSL terminal — right-click the Ubuntu icon in the taskbar or open a new tab if your terminal supports it. 


### **Step 3 — Start the Backend (Terminal 1)**  

**Where:** Terminal 1 — WSL Ubuntu, user: `mir`, any directory  
bash  
```   
podman run --rm -it
  --name gru-backend-2.0.0
  -v /mnt/d/gru_data:/gru_data
  -p 9001:9001
  docker.io/iobio/iobio-gru-backend:2.0.0
```  

> **Expected warning:** `WARN\[0000\] "/" is not a shared mount` will appear immediately.  
> This is harmless and expected on every `Podman` operation in `WSL2`.  
> See **`Gotcha 6.2` in `04_GOTCHA_Enumeration.mc`**.  

After the warning, the container will start and the backend will initialize.  
This takes a moment. When initialization is complete the terminal will show a  
blinking cursor with no prompt returned — **this is correct**.  
The backend is running and waiting for connections.  
Terminal 1 is now occupied for as long as the backend is running.  

> **Do not close Terminal 1.** Closing it stops the container.  
> Do not press Ctrl+C unless you intend to stop the backend.  

> **The `--rm` flag:** This flag removes the container automatically when it stops.  
> This is intentional for this first verification run — it keeps things clean.  
> Day-to-day backend startup uses a different method without `--rm`.  
> See the **`Quick Reference`** section and **`E_Extra_Startup_Scripts.md`**.  


### **Step 4 — Verify the Health Endpoint (Terminal 2)**  

**Where:** Terminal 2 — WSL Ubuntu, user: `mir`, any directory  
bash  
```  
curl localhost:9001
```  
Expected response is JSON message:  
```  
{"service_description":"iobio gru backend server","version":"2.0.0","data_version":"2.0.0"}
```  

> **This JSON response confirms:**  

> - The correct 2.0.0 image is running  

> - The backend is healthy and accepting connections  

> - `gru_data 2.0.0` is mounted and recognized  

> **If you receive an HTML string** `<h1>I be healthful</h1>` instead of JSON,  
> the wrong image is running.  
> Stop the container, verify the image ID in **Step 1**, and restart.  
> See **`Gotcha 6.4` in `04_GOTCHA_Enumeration.mc`**.  

> **If curl returns nothing or connection refused:** The backend is still initializing.  
> Wait 10–15 seconds and try again.  
> If it still fails after 30 seconds, the container may have failed to start —  
> check Terminal 1 for error output.  


### **Step 5 — Stop the Backend**  

**Where:** Terminal 1 — WSL Ubuntu  

The verification is complete. Stop the backend cleanly:  

Press **`Ctrl+C`** in Terminal 1.  

> **Note:** Ctrl+C may print `^C` characters without immediately stopping the container.  
> If the container does not stop within 15 seconds:  

Switch to Terminal 2 and run:  
bash  
```  
podman stop gru-backend-2.0.0
```  

If that hangs beyond 15 seconds:  
bash  
```  
podman kill gru-backend-2.0.0
```  

Verify it has stopped:  
bash  
```  
podman ps | grep gru
```  
Expected: No output. No output means no gru container is running.    

> **After a kill:** Give the system 5 seconds to settle before doing anything else.  
> Do not restart immediately after a force kill.  
> See **'Gotcha 6.5` and `6.6` in `04_GOTCHA_Enumeration.mc`**.


### **Step 6 — Upgrade SAMtools to 1.24**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory → **`Glossary`: `mamba`, `bioconda` — see `07_OVERVIEW.md`**  

> **What this is:** SAMtools 1.22.1 was installed in **Phase 2** as the **`Ubuntu 26.04 LTS`** default.  
> This upgrade to 1.24 is a personal preference — it is not required for gene.iobio stack functionality.  
> It is performed here, after backend verification and before the frontend build, as a clean logical step.  
> See the SAMtools note in **Phase 2**.

bash  
```  
mamba install -c bioconda samtools=1.24
```  

When prompted to confirm the install, type `y` and press Enter.  

Verify:  
bash  
```  
samtools --version
```  
Expected: first line reads **`samtools 1.24`**  


### **Step 7 — WSL Export: Production Stack Restore Point**  

**Where:** First, exit all WSL terminals. Then PowerShell (regular, not elevated).  

This export captures the WSL environment with:  

- `Podman` configured and verified  

- `gru-backend 2.0.0` image present and confirmed healthy  

- `samtools 1.24` installed  

- All **Phase 2** tools in place  

This is the last restore point before the frontend build begins.  

### **7a. Exit all WSL terminals:**  
bash  
```  
exit
```  

Close both Terminal 1 and Terminal 2.  

### **7b. Shut down WSL completely:**  
bash  
```  
wsl --shutdown
```  

#### **7c. Reboot Windows.** Full reboot before export.  

### **7d. After reboot — export from PowerShell:**  
If you are not continuing to **Phase 8**, you can put 'FINAL' in your file name. 
PowerShell  
```  
wsl --export Ubuntu "D:\WSL_Backups\Ubuntu_26_04_iobio_stack_production_20260806.tar"
```  

> This export will be approximately 15–16GB. Allow extra time.  

### **7e. Verify MD5:**  Once .tar is created.  
PowerShell  
```  
Get-FileHash "D:\WSL_Backups\Ubuntu_26_04_iobio_stack_production_20260806.tar" -Algorithm MD5
```  

Note and record the hash.  

> MD5 is from maintainer's system and will likely differ on yours. Keep the naming convention,  
> update the date portion for your own export. Verify your hash matches itself across both copy  
> locations — `forge` `D:` and `vault` `E:`.  
> A matching hash between locations confirms the file transferred intact.  
> A mismatch means the copy was incomplete or corrupted — re-copy and re-verify when hashes do not match.  

### **7f. Copy to `vault` and verify:**  
PowerShell  
```  
copy "D:\WSL_Backups\Ubuntu_26_04_iobio_stack_production_20260806.tar" "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\"
```  
PowerShell  
```  
Get-FileHash "\\192.168.1.52\E\MedLaptop Mirror\Ubuntu\WSL_Backups\Ubuntu_26_04_iobio_stack_production_20260806.tar" -Algorithm MD5
```  

Hashes must match.  
If you are not continuing to `Phase_8_Option_A.md` and/or `Phase_8_Option_B.md` the `Ubuntu_26_04_iobio_stack_production_20260806.tar`  
is your WSL FINAL backup. If you are continuing, changes will be made and a new set of backups will be needed.  
> See `Phase 5` Note for explanation that `Phase 8 Option A` or `B` do not touch `gru_data`, but do alter WSL and gene.iobio frontend data.


## **Phase 6 Completion**  

| Component | Version | Status |  
| - | - | - |  
| gru-backend 2.0.0 | image ID `7e2c4a419267` | ✅ confirmed |  
| Health endpoint | JSON response | ✅ HTTP 200 |  
| `service_description` | `iobio gru backend server` | ✅ |  
| `version` | `2.0.0` | ✅ |  
| `data_version` | `2.0.0` | ✅ |  
| `SAMtools` | 1.24 | ✅ upgraded |  
| WSL restore point | exported + MD5 verified `forge` + `vault` | ✅ |  


> **The stack backend is now fully verified.** `gru-backend 2.0.0` with `gru_data 2.0.0` is confirmed healthy.  
> **Phase 7** builds the frontend against this verified backend.  


One flag before you review:  

---

## **Phase 7 — Frontend Build**  

> **Before starting Phase 7:** **Phase 6** must be complete. `gru-backend 2.0.0` is verified healthy  
> at `localhost:9001`. `gru_data 2.0.0` is complete and verified.  
> `SAMtools 1.24` is installed.  

> **What this phase does:**  
> Installs `Node.js` via `NVM`,  
> clones (copies) the `gene.iobio frontend` repository,  
> checks out the correct version,  
> configures it for local use,  
> builds it,  
> and serves it.  
> At the end of this phase the full stack is functional end-to-end.  

| Item | Status |  
| - | - |  
| `gru-backend 2.0.0` running HTTP 200 | ✅ prerequisite |  
| `gru_data 2.0.0` complete + verified | ✅ prerequisite |  
| `SAMtools` 1.24 upgrade installed | ✅ stack preference, not required for `gene.iobio` build |  


| Component | Version | Notes |  
| - | - | - |  
| `NVM` | 0.39.0 | Node version manager |  
| `Node.js` | v16.20.2 | via `NVM` — `v16+` required |  
| `npm` | 8.19.4 | via `NVM` |  
| `webpack` | 3.12.0 | `npm` dependency — auto-installed |  
| `serve` | v14.2.6 | static file server |  
| `gene.iobio` | v4.13.1-runtime-config | `commit cb28d29a` |  


### **Step 1 — Install `NVM`**  

**Where:** WSL Ubuntu terminal — user: `mir`, home directory (`~`) → **`Glossary`: `NVM`, `Node.js`, `npm` — see `07_OVERVIEW.md`**  

> `NVM` (Node Version Manager) allows installing and switching between `Node.js` versions.  
> `gene.iobio` requires `Node v16 or higher`.  
> The system `Node version` in `Ubuntu 26.04 LTS` is **insufficient** — **`NVM` is required to get the correct version.**  

### **1a. Navigate to home directory:**  
bash  
```  
cd ~
```  

### **1b. Download and run the NVM install script:**  
bash  
```  
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```  

### **1c. Reload shell configuration:**  
bash  
```  
source ~/.bashrc
```  

### Activate NVM  
bash  
```  
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```  

### **1d. Verify `NVM` installed correctly:**  
bash  
```  
nvm --version
```  

Expected output:  
```  
0.39.0
```  


### **Step 2 — Install `Node.js v16`**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
nvm install 16
```  

This downloads and installs `Node.js v16` and its matching `npm` version.  
Expected output ends with something like:  
```  
Now using node v16.20.2 (npm v8.19.4)
```  

Set `v16` as the default:  
bash  
```  
nvm use 16
```  

Verify both:  
bash  
```  
node --version
```  
Expected: `v16.20.2`  
bash  
```  
npm --version
```  
Expected: `8.19.4`  

> **Why `v16` specifically?** `Node v13.14` (which may already be present on the system) is insufficient for `gene.iobio`.  
> `v16+` is required. `NVM` makes it easy to install the correct version without affecting anything else on the system.  
> See **`Gotcha 7.1` in `04_GOTCHA_Enumeration.mc`**.  


### **Step 3 — Clone `gene.iobio`**    

**Where:** WSL Ubuntu terminal — user: `mir`, home directory (`~`)  

### **3a. Navigate to home directory:**  
bash  
```  
cd ~
```  

### **3b. Clone the repository:**  
bash  
```  
git clone https://github.com/iobio/gene.iobio
```  

This clones the `gene.iobio frontend` Github repository to `~/gene.iobio/`.  
The download is moderate in size and should complete without issues.  

### **3c. Enter the repository directory:**  
bash  
```  
cd ~/gene.iobio
```  

### **Step 4 — Checkout the Correct Version**  

**Where:** WSL Ubuntu terminal — user: `mir`, directory `~/gene.iobio` → **`Glossary`: `git checkout`, `commit` — see `07_OVERVIEW.md`**  

The production-verified version for this stack is `v4.13.1-runtime-config` at commit `cb28d29a`.  
Checking out this specific version ensures you are building against the exact code that has been  
validated with `gru-backend 2.0.0`.  
bash  
```  
git checkout v4.13.1-runtime-config
```  

Verify the correct commit:  
bash  
```  
git log --oneline -1
```  

Expected: output begins with **`cb28d29a`**  

> **Why this version?** `v4.13.1-runtime-config` introduced the runtime configuration architecture —  
> deployment settings are loaded from `client/config.json` at runtime instead of being baked into the  
> `webpack` build via `.env` values.  
> This is what makes local deployment configuration practical.  
> See **`Gotcha 7.5` in `04_GOTCHA_Enumeration.mc`**.  

> **Note on active maintenance:** The iobio team released `v4.13.1` on 20260730 —  
> during the original build the maintainer had in progress.  
> The frontend project is actively maintained by the iobio team.  
> If you are building at a later date and a newer version is available,  
> be aware that this guide is validated against `v4.13.1-runtime-config` specifically.  
> Newer versions may work but have not been tested against this procedure.  


### **Step 5 — Install `npm` Dependencies**  

**Where:** WSL Ubuntu terminal — user: `mir`, directory `~/gene.iobio`  
bash  
```  
npm install
```  

This installs all frontend dependencies including `webpack`.  
The output will be long. On this hardware it may take several minutes.  

Expected final output includes:  
```  
added 994 packages
```  

> **npm warnings:** You may see deprecation warnings and audit notices during install.  
> These are expected and do not affect functionality.  
> The build will work correctly regardless of these warnings.  


### **Step 6 — Configure `client/config.json`**  

**Where:** WSL Ubuntu terminal — user: `mir`, directory `~/gene.iobio` → **`Glossary`: `config.json`, `runtime config` — see `07_OVERVIEW.md`**  

> **This is the most important configuration step in Phase 7.** The `client/config.json` file  
> tells the frontend where to find the backend.  
> Getting this wrong means the frontend will build successfully but  
> fail to connect to the backend at runtime.  

Open the config file with `nano`:  
bash  
```   
nano ~/gene.iobio/client/config.json
```  

Replace the entire contents with the following. Right-click to paste (or whatever works on your system):  
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
    "origin": "http://localhost:9001",
    "path_prefix": "/"
  },
  "backend_map": {
    "default": "http://localhost:9001"
  }
}
```  
Save: **Ctrl+O**, Enter, **Ctrl+X**

Verify the file:  
bash  
```  
cat ~/gene.iobio/client/config.json
```  

Confirm `localhost:9001` appears in both `backend.origin` and `backend_map.default`.  

> **localhost vs. LAN IP:** This config is for localhost access only —  
> the frontend and backend on the same machine.  
> If you intend to serve `gene.iobio` over your local network so other devices can access it,  
> the `localhost:9001` references will be changed to your machine's LAN IP in **Phase 8 Option B**.  
> Do not make that change now — complete **Phase 7** with localhost first and  
> confirm everything works before proceeding to **Phase 8**.  

---

### **Step 7 — Build the Frontend**  

**Where:** WSL Ubuntu terminal — user: `mir`, directory `~/gene.iobio`  
bash  
```  
cd ~/gene.iobio
```  
bash  
```  
npm run build
```  

This runs `webpack` and compiles/builds the frontend into a single deployable file (**`build.js`**).    
On this hardware it takes approximately 29 seconds.  

Expected final output looks something like this:  
```  
Hash: 63096a5e6b975d9b6d4e
Version: webpack 3.12.0
Time: 24064ms
   Asset     Size  Chunks                    Chunk Names
build.js  30.5 MB       0  [emitted]  [big]  app
```  

> **The build is successful when:** `webpack` completes with no errors and  
> `client/dist/build.js` is present at approximately 30.5MB.  
> Warnings are acceptable — errors are not.  
> If the build exits with an error, do not proceed to serving.  

Verify the build output file exists:  
bash  
```  
ls -lh ~/gene.iobio/client/dist/build.js
```  
Expected: file present, size approximately 30.5MB.  


### **Step 8 — Install `serve`**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory → **`Glossary`: `serve`, `npx` — see `07_OVERVIEW.md`**  

`serve` is a lightweight static file server used to serve the built frontend.  
bash  
```  
npm install -g serve
```  

Verify:  
bash  
```  
serve --version
```  
Expected: `14.2.6`  

> **Global install note:** The `-g` flag installs `serve` globally so it is available  
> from any directory without `npx`.  
> Once installed globally, use `serve` directly.  
> If for any reason global install is not yet complete at this point,  
> `npx serve` can be used as a temporary substitute —  
> but complete the global install before Phase 8B.  


### **Step 9 — Start the Backend and Serve the Frontend**  

**Where:** Two WSL Ubuntu terminals — user: `mir`  

This requires the same two-terminal approach used in **Phase 6**.  

**Terminal 1 — Start the backend:**  

> For this initial full-stack test, start the backend using `podman run`  
> since the container was already created in **Phase 6**.  
> The `--rm` flag used in **Phase 6** removed the container after stopping —  
> so we need to recreate the container this first time.  

bash  
```  
podman run -d
  --name gru-backend-2.0.0
  -v /mnt/d/gru_data:/gru_data
  -p 9001:9001
  docker.io/iobio/iobio-gru-backend:2.0.0
```  

> **Note the difference from **Phase 6:** `-d` replaces `--rm -it`.  
> The `-d` flag runs the container detached in the background, returning you to the prompt immediately.  
> This is the normal day-to-day startup method.  
> The `--rm -it` in **Phase 6** was for first-time verification only.  
> Unless you remove the container again, you should not need to create the container again.  
> Subsequent starts of the backend server can be via the start_network_backend.sh script or by typing:  

bash  
``` 
podman start gru-backend-2.0.0
```  
Verify the backend is running:  
bash  
```  
curl -s localhost:9001
```  
Expected: JSON health response confirming `version: 2.0.0`  

**Terminal 2 — Serve the frontend:**  
bash  
```  
cd ~/gene.iobio/client
```  
bash  
```  
serve -s . -l 3000
```  

> **`serve` will report a `172.18.x.x` address** in its output — this is the WSL internal IP, not your Windows IP.  
> This is normal and expected.  
> Access the frontend using `http://localhost:3000` from the same machine, not the WSL IP.  
> See **`Gotcha 8B.2` in `04_GOTCHA_Enumeration.mc`**.  


### **Step 10 — Verify the Full Stack**  

**Where:** Windows browser on the same machine  

Open a browser and navigate to:  
bash  
```  
http://localhost:3000
```  

The `gene.iobio` webpage interface should load.   

**Gene lookup test:**  

In the search box, type a gene name such as `RAI1` and confirm:  
- The gene card (information about the gene you are searching for) loads  
- Transcript information appears  
- No backend connection errors in the browser console  

> **Why `RAI1`?** RAI1 is one of the genes included in gene.iobio's built-in demo dataset.  
> It is loaded automatically when you click the* **`RUN WITH DEMO DATA`** *button —  
> the most visible button on the interface when you first open it.  
> Using `RAI1` *as the verification gene keeps your manual test consistent with what the demo button does.  
> If `RAI1` loads correctly, the core stack is working.  

**Full verification with `VCF`/`CRAM`:**  

If you have `VCF` and `CRAM` files available:  

1. Use the **Load** button in the frontend webpage   
2. Load your `VCF` and `CRAM` files  
3. Confirm variants load and annotate correctly  
4. Confirm `Phenolyzer` is accessible via the `http://localhost:3000` `gene.iobio` webpage.  

> **Browser console warnings (non-issues):**  
> - Vue development mode warning — expected, not a problem for local use  
> - HTTPS warning on data `URI`s — browser compliance notice only  
> - (many browsers now flag unsigned/uncertified localhost addresses as unsafe).  
> Neither affects functionality. See **`Glossary`: `CRAM`, `VCF`, `URI` in `07_OVERVIEW.md`** and  
> See **`Gotcha L.3 and L.4` in `04_GOTCHA_Enumeration.mc`**.  

#### **Step 11 — Frontend Backup**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

### Create a tar archive of the frontend directory:  
> This is not as resource intensive as the 'wsl --export` tar creation,  
> so there should be no need to reboot before creating this tar file.

bash  
```  
mkdir -p /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir
```  
If you are not continuing to **`Phase 8`**, you can add the word `FINAL` to the file name.
bash  
```  
tar -cf /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_1_20260806.tar ~/gene.iobio/
```  

Verify MD5:  
```  
md5sum /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_1_20260806.tar
```  

Note and record the hash.  

> MD5 is from maintainer's system and will likely differ on yours.  
> Keep the file naming conventions, update the date portions for your own export.  
> Verify that your hash matches itself across both copy locations.

Copy to `vault`:  
bash  
```  
cp /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_1_20260806.tar "/mnt/medlaptop/e/MedLaptop\ Mirror/gene_iobio_dir_bkps/gene_iobio_frontend_dir/"
```  

Verify `vault` MD5:  
bash  
```  
md5sum "/mnt/medlaptop/e/MedLaptop\ Mirror/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_v4_13_1_20260806.tar"
```  

Hashes must match.  

If you are not continuing to `Phase_8_Option_A.md` and/or `Phase_8_Option_B.md` this is  
then your `gene.iobio frontend` FINAL backup.  
If you continue to the `Phase_8_Option_A.md` and/or `Phase_8_Option_B.md` phases, you will  
want to repeat this process to capture all your fixes and name your file to include the word `FINAL`.

## **Phase 7 Completion**  

| Component | Version | Status |  
| - | - | - |  
| `NVM` | 0.39.0 | ✅ |  
| `Node.js` | v16.20.2 | ✅ |  
| `npm` | 8.19.4 | ✅ |  
| `gene.iobio` | v4.13.1-runtime-config commit cb28d29a | ✅ |  
| `npm` dependencies | 994 packages | ✅ |  
| `client/config.json` | localhost:9001 | ✅ |  
| `webpack` build | 30.5MB build.js | ✅ clean |  
| `serve` | v14.2.6 | ✅ globally installed |  
| Backend health | HTTP 200 JSON | ✅ |  
| Frontend loads | http://localhost:3000 | ✅ |  
| Gene lookup | RAI1 card renders | ✅ |  
| `VCF`/`CRAM` load | variant annotation pipeline | ✅ functional |  
| `Phenolyzer` | hook confirmed | ✅ working |  
| Frontend backup | tar + MD5 verified `forge` + `vault` | ✅ |  


> **The stack is now fully functional.**  
> `gene.iobio v4.13.1` frontend with `gru-backend 2.0.0` and `gru\=_data 2.0.0` is validated end-to-end.  
> **Phase 8 Options A and B extend functionality — they are not required for a working local stack.  

### Now is a great time to do full production stack backups of:  
   > WSL  
   > ~/gene.iobio  
   > gru_data  

---


## **Next Steps**

> On to **Phase 8**!  
> Phase 8 consists of two distinct extensions of functionality to the `gene.iobio` stack as built in this guide:  
> Phase 8 Option A and Phase 8 Option B... and Option C (which is completely optional and involves document creation).  
> If you choose both Options A and B:  
> I would suggest doing them in order for your final backups to be done most efficiently to include both fixes.  
> Each time `build.js` is created during the `npm run build` process:  
> A new updated `build.js` is created, therefore an alteration in your stack.  
> Do one or the other `Phase 8 Option A` and/or `B` and if doing both, do them in order.  
> The stack is functional as it is after **Phase 7**, but  
> if you want to save out variant files in `.csv` or `.vcf` format and/or  
> have the LAN serving capability with WSL IP persistence, `Phase 8 Option A` and `Option B` is for you.  
> Your choice as to what you choose.  
> I would just strongly suggest additional FINAL exports/tars of your  
> WSL, ~/gene.iobio and gru_data again so you have full finals with all fixes  
> in case disaster strikes (`Disaster_Recocery.md`)

## **Next Steps Documents**  
| Document | Purpose |  
| - | - |  
| **`Phase_8_Option_A.md`** | Build documentation to make three one-line fixes to enable .csv and .vcf file export save dialogue in `gene.iobio` browser webpage UI |  
| **`Phase_8_Option_B.md`** | Build documentation to set up LAN webpage serving with WSL IP persistence. |  


Totally optional (though encouraged, at least for your own documentation of your own stack) Document creation activity:  
| Document | Purpose |  
| - | - |  
| **`Phase_8_Option_C.md`** | Document creation process in case you are documenting your own or wish to create your own documentation to share. |  


The following companion documents are part of this documentation set. Refer to them as needed  
This `01_Stack_Build_Steps.md` and these three are the first four 'core' documents for build process:  
| Document | Purpose |  
| - | - |  
| `02_Session_Handoff_Log.md` | Chronological record of all build sessions — the full story of how this stack was built, including dead ends, decisions, and discoveries. Useful context if something breaks and you need to understand why a decision was made. |  
| `03_Stack_Versioning_Reference.md` | Flat version and configuration reference — scannable lookup for any component version, file location, MD5 hash, or configuration value. |  
| `04_GOTCHA_Enumeration.md` | Phase-numbered list of 62 gotchas cross-referenced to this document. First stop for troubleshooting. |  

You should have started with:  
| Document | Purpose |  
| - | - |  
| `README.md` | Introduction |  
| `07_OVERVIEW.md` | Overview, and system naming reference. If a term in this document is unfamiliar, check the **`Glossary`** there first. |  

These are mostly for after you have completed your stack, unless you are running low on `forge` space to store your backups during the build:  
| Document | Purpose |  
| - | - |  
| `D_Drive_Cleanup_Priority.md` | Which backups to remove from `forge`/`vault` first to reclaim space, in order of priority. |  
| `Disaster_Recovery.md` | Full restore procedure from the final backup set. |  
| `E_Extra_Startup_Scripts.md` | Backend and frontend startup scripts for you to create. |  

Totally optional (though encouraged to document the stack you are building for yourself):  
| Document | Purpose |  
| - | - |  
| `WORKFLOW.md` | How to work with a Brave Leo (or maybe any) AI assistant across sessions to achieve build goals. |  
| `MEMORY_MAINTENANCE.md` | Memory lifecycle, tagging conventions, and session handoff rules. See `MEMORY_MAINTENANCE_EXAMPLES.md` |  
| `MEMORY_MAINTENANCE_EXAMPLES.md` | Real Memory lifecycle entries, tagging conventions, and session handoff rules. Companion to `MEMORY_MAINTENANCE.md` |  

About why the maintainer of this repository set out to build her own analytics stack:  
| Document | Purpose |  
| - | - |  
| `05_ACCESSIBILITY_NOTES.md` | Notes about the maintainer's accessibility obstacles and some workarounds to manage them. |  
| `06_MEDICAL_CONTEXT.md` | Maintainer's medical context that might shed light on how/why this project was undertaken. |  

---

## **Quick Reference**  

> **Day-to-day startup.** Both backend and frontend must be running for `gene.iobio` to function.  
> Always start the backend first and verify it before starting the frontend.  

## **Start Backend**  

**Where:** WSL Ubuntu terminal — user: `mir`  
bash  
```  
podman start gru-backend-2.0.0
```  

Wait a moment, then verify:  
bash  
```  
curl -s localhost:9001
```  
Expected: JSON health response confirming **`version: 2.0.0`**  

> **If `podman start` fails with "no such container":**  
> The container does not exist — it was either never created or was removed with `--rm`.  
> Recreate it with the full `podman run -d` command from **Phase 7 Step 9**.  

> **If the backend becomes unresponsive:**  
bash  
```  
podman stop gru-backend-2.0.0
```  

If it hangs beyond 15 seconds:  
bash  
```  
podman kill gru-backend-2.0.0
```  

Verify stopped:  
bash  
```  
podman ps | grep gru
```  

> No output = stopped. Wait 5 seconds before restarting.  


## **Start Frontend — Localhost Only**  

**Where:** WSL Ubuntu terminal — user: `mir`  
bash  
```  
cd ~/gene.iobio/client
```  
bash  
```  
serve -s . -l 3000
```  

Access: `http://localhost:3000` using web browser.  


## **Start Frontend — LAN Access (Phase 8 Option Ba)**  

**Where:** WSL Ubuntu terminal — user: `mir`  
bash  
```  
~/start_network_frontend.sh
```  

Access:  
```  
http://localhost:3000        # same machine  
http://192.168.1.240:3000    # any LAN device
```  

> **`serve` will report a `172.18.x.x` address** in its output.  
> This is the WSL internal IP — normal and expected.  
> Use the addresses above, not the WSL IP shown by serve.  


## **Stop Backend**  

**Where:** WSL Ubuntu terminal — user: `mir`  
bash  
```  
podman stop gru-backend-2.0.0
```  


## **Load Data**  

Use the **Load** button in the `gene.iobio` web browser webpage UI (user interface) to load `VCF` and `CRAM`/`BAM` files.  

Use the **`RUN WITH DEMO DATA`** button to verify the stack is working with the built-in demo dataset.  
**`RAI1`** is one of the demo genes — if it loads and the gene card populates, the stack is functioning correctly.  


## **Verify Running Image**  

Always verify the correct image is running if anything seems off:  
bash  
```  
podman ps --format "{{.Names}} {{.Image}}"
```  
Expected:  
```  
gru-backend-2.0.0   docker.io/iobio/iobio-gru-backend:2.0.0
```  

Image ID confirmation:  
bash  
```  
podman images | grep gru
```  

**The 2.0.0 image ID must be `7e2c4a419267`**  

---  

≡≡≡≡≡ END: 01_Stack_Build\Steps.md ≡≡≡≡≡
