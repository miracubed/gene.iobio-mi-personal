# **Phase 8 Option Ba: LAN Network Serving Setup**  
gene.iobio frontend v4.13.1 + gru-backend 2.0.0  
Network serving from HealthMed laptop (.240) to personal laptop (.52)  
Date completed: August 7, 2026  
Prerequisites: Phase 8 Option A (CSV/VCF export) completed or standalone  
Time estimate: 30–45 minutes setup + testing  

## Overview  
The Problem  
    • Frontend runs on .240 (HealthMed laptop, resource-constrained)  
    • Browser also runs on .240  
    • Browser rendering consumes .240 RAM  
    • Large variant lists (thousands of intronic/UTR variants) can crash .240 browser  
    • .240 has 2GB RAM allocation to WSL  
The Solution  
    • Frontend runs on .240 (resource-intensive server work)  
    • Browser runs on .52 (personal laptop, better resources)  
    • .240 freed from browser rendering overhead  
    • .52 browser has more RAM and processing power  
    • Large variant lists render reliably on .52  

## **Network Architecture**  
```  
.240 (HealthMed)                    .52 (Personal)
├─ WSL Ubuntu                       └─ Windows 10
│  ├─ gru-backend 2.0.0 (port 9001)    └─ Browser
│  └─ gene.iobio frontend (port 3000)
│     └─ Listens on 0.0.0.0:3000
└─ Windows Firewall
   ├─ Allow inbound 3000
   └─ Allow inbound 9001
```  

## **Prerequisites**  
Network Requirements  
    • ✅ Both laptops on same local network (192.168.1.x)  
    • ✅ Network connectivity confirmed  
    • ✅ IP addresses stable (static recommended)  
Software Requirements  
    • ✅ gene.iobio frontend v4.13.1 built on .240  
    • ✅ gru-backend 2.0.0 running on .240 (port 9001)  
    • ✅ Node.js and npm available in WSL on .240  
    • ✅ Browser available on .52  
System Requirements  
    • ✅ WSL Ubuntu running on .240  
    • ✅ PowerShell available on .240 Windows  
    • ✅ Administrator access on .240 Windows  

## **Step-by-Step Procedure**  
## **Step 1: Install `serve` globally (on .240 WSL)**  
`serve` is a lightweight static file server. Install it globally so it's available anywhere.  
bash  
```  
npm install -g serve
```  
Expected output:  
```  
added 85 packages, and audited 86 packages in Xs  
found 0 vulnerabilities  
```  
Verify installation:
bash  
```  
npm list -g serve
```  
Should show serve in the global packages list.  

## **Step 2: Verify and modify frontend configuration**  
> The frontend needs to know where to find the backend.  
> By default, it points to `localhost:9001` (only accessible on .240).  
> We need to change this to 192.168.1.240:9001 so .52 can reach it.  

Check current configuration:
bash  
```  
cat ~/gene.iobio/client/config.json  
```  
Expected output (before changes):  
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
Open the file in `nano` for editing:  
```  
nano ~/gene.iobio/client/config.json
```  
Locate and change these two lines:  
| Find | Change to |  
|---|---|  
| `"origin": "http://localhost:9001"` | `"origin": "http://192.168.1.240:9001"` |  
| `"default": "http://localhost:9001"` | `"default": "http://192.168.1.240:9001"` |  

How to edit in nano:  
    1. Use Ctrl+W to search for `localhost:9001`  
    2. Use arrow keys to position cursor  
    3. Delete old text and type new IP  
    4. Repeat for second occurrence  
    5. Save with Ctrl+O, press Enter  
    6. Exit with Ctrl+X  
Verify changes:  
bash  
```  
cat ~/gene.iobio/client/config.json
```  
## **Expected output (after changes)**:  
```  
{
  "backend": {
    "origin": "http://192.168.1.240:9001",
    "path_prefix": "/"
  },
  "backend_map": {
    "default": "http://192.168.1.240:9001"
  }
}
```  
## **Both `localhost` references should now show `192.168.1.240`**.

## **Step 3: Rebuild frontend**  
The configuration change must be compiled into the frontend build. Rebuild now:  
bash  
```  
cd ~/gene.iobio && npm run build
```  
## **Expected output (end of build, ~25 seconds)**:
```  
Hash: f2f281efd0020f65f64a
Version: webpack 3.12.0
Time: 25222ms
| Asset | Size | Chunks | Chunk Names |  
|---|---|---|---|  
| build.js | 30.5 MB | 0 | [emitted] [big] app |  
| [5] (webpack)/buildin/global.js | 509 bytes | {0} | [built] |  
| [43] ./client/js/appConfig.js | 1.05 kB | {0} | [built] |
```  
... (many more lines)  
> **Important: No errors should appear. If you see errors, the build failed—do not proceed.**  

## **Step 4: Create Windows Firewall rules (on .240 Windows)**  
The frontend and backend need to accept inbound traffic from .52. Create firewall rules to allow this.  
Open PowerShell as Administrator:  
    • Right-click PowerShell icon → "Run as administrator"  
    • Or press Win+X, select "Windows PowerShell (Admin)"  
Create firewall rule for frontend (port 3000):  
Elevated PowerShell  
```  
New-NetFirewallRule -DisplayName "gene.iobio frontend" -Direction Inbound -LocalPort 3000 -Protocol TCP -Action Allow
```  
Create firewall rule for backend (port 9001):  
Elevated PowerShell  
```  
New-NetFirewallRule -DisplayName "gru-backend" -Direction Inbound -LocalPort 9001 -Protocol TCP -Action Allow
```  

Verify both rules were created:  
Elevated PowerShell  
```  
Get-NetFirewallRule -DisplayName "gene.iobio*" | Format-Table DisplayName, Enabled
```  
Elevated PowerShell  
```  
Get-NetFirewallRule -DisplayName "gru-backend" | Format-Table DisplayName, Enabled
```  
Expected output:  
```  
DisplayName         Enabled  
-----------         -------  
gene.iobio frontend    True  
gru-backend            True  
```  
**Both should show `True` for `Enabled`**.  

# **Phase 8 Option Bb: WSL IP Persistence**
## **Step 5: Configure WSL port forwarding (on .240 Windows)**  
WSL runs in a virtual network isolated from Windows. To make the frontend and backend accessible from .52, we need to forward traffic from Windows IP (192.168.1.240) to the WSL internal IP.  
Find your WSL internal IP (run in WSL first):  
# In WSL bash:  
bash  
```  
hostname -I
```  
Example output:  
```
172.18.60.70
```    
## Write down this IP. (It may differ from the example.)  
Back in PowerShell (as Administrator), add port forwarding rules:  

## Replace 172.18.60.70 with your actual WSL IP if different:  
Elevated PowerShell  
```  
netsh interface portproxy add v4tov4 listenport=3000 listenaddress=0.0.0.0 connectport=3000 connectaddress=172.18.60.70
```  
Elevated PowerShell  
```  
netsh interface portproxy add v4tov4 listenport=9001 listenaddress=0.0.0.0 connectport=9001 connectaddress=172.18.60.70
```  
## Verify port forwarding rules:  
PowerShell  
```  
netsh interface portproxy show all
```  
Expected output:  
```  
Listen on ipv4:             Connect to ipv4:  
Address         Port        Address         Port  
--------------- ----------  --------------- ----------  
0.0.0.0         3000        172.18.60.70    3000  
0.0.0.0         9001        172.18.60.70    9001  
```  
> **Both rules should be present**.  

## **Step 6: Create startup script (on .240 WSL, optional but recommended)**  
A frontend startup script saves time and reduces errors. It checks that the backend is running, then starts the frontend with the correct settings.
Create the script file:  
bash  
```  
nano ~/start_network_frontend.sh
```  
Copy and Paste this content into nano:  
```  
#!/bin/bash

echo "Starting gene.iobio frontend on network interface..."
cd ~/gene.iobio/client

# Verify backend is running
if ! curl -s http://localhost:9001 > /dev/null; then
  echo "ERROR: gru-backend not running on port 9001"
  exit 1
fi

echo "Backend confirmed running on localhost:9001"
echo "Starting frontend on 0.0.0.0:3000..."
echo ""
echo "Access from local browser: http://localhost:3000"
echo "Access from network (.52): http://192.168.1.240:3000"
echo ""

serve -s . -l 3000
```  
## **Save and exit**:
• Press Ctrl+O, then Enter to save  
• Press Ctrl+X to exit nano  

## **Make the script executable**:  
bash  
```  
chmod +x ~/start_network_frontend.sh
```  
## Test the script:  
bash  
```  
~/start_network_frontend.sh
```  
## Expected output:  
```  
Starting gene.iobio frontend on network interface...
Backend confirmed running on localhost:9001
Starting frontend on 0.0.0.0:3000...

Access from local browser: http://localhost:3000
Access from network (.52): http://192.168.1.240:3000

To stop frontend: Type CTRL+C or close this window.

   ┌──────────────────────────────────────────┐
   │                                          │
   │   Serving!                               │
   │                                          │
   │   - Local:    http://localhost:3000      │
   │   - Network:  http://172.18.60.70:3000   │
   │                                          │
   │   Copied local address to clipboard!     │
   │                                          │
   └──────────────────────────────────────────┘
```  

## **Step 7: Start frontend server**  
Using the startup script (recommended):  
bash  
```  
~/start_network_frontend.sh
```  
Or, direct command (if you didn't create the script):  
bash  
```  
cd ~/gene.iobio/client && serve -s . -l 3000
```  
## Expected output:  
```
   ┌──────────────────────────────────────────┐
   │                                          │
   │   Serving!                               │
   │                                          │
   │   - Local:    http://localhost:3000      │
   │   - Network:  http://172.18.60.70:3000   │
   │                                          │
   │   Copied local address to clipboard!     │
   │                                          │
   └──────────────────────────────────────────┘
```
Leave this running. The server is now active. Proceed to **Step 8**.  

## **Step 8: Test from .52 browser**  
On personal laptop (.52), open a web browser and navigate to:  
Copy  
```  
http://192.168.1.240:3000
```  
## Expected:  
   • gene.iobio UI loads  
   • Gene name search field visible  
   • No timeout or connection errors  
If the page loads: Proceed to **Step 9**.  
If it doesn't load: See Troubleshooting section below.  

## **Step 9: Functional test with real data**  
On .52 browser at `http://192.168.1.240:3000` :  
    1. Load test WGS data: Use the "Load" button to load a `VCF` or `CRAM` file  
    2. Perform gene lookup: Search for a gene (e.g., **`SLC7A7`** or **`RAI1`**)  
    3. Load variant list: Click **`Analyze all`** or select variants  
    4. Expand variant sections: Especially large sections (modifier/UTR variants)  
    5. Test export (if **Phase 8 Option A** has been completed): Click `CSV` or `VCF` export button  
## **Expected results**:  
   • ✅ Page loads without errors  
   • ✅ Gene search returns results  
   • ✅ Variant lists render smoothly on .52  
   • ✅ No browser crashes  
   • ✅ Export files download correctly (if Phase 8B completed)  
   • ✅ All functionality works over network  

# **Troubleshooting**  
# **Symptom: Browser on .52 shows "Cannot reach server" or blank page**  
**Check 1**: Is frontend running on .240?  
# On .240 WSL:  
bash  
```  
ps aux | grep serve
```  
Should show a line with serve -s .  
# **Fix**: If not running, start with:     
bash  
```  
~/start_network_frontend.sh
```  

# **Symptom: Frontend loads but backend requests timeout (`ERR_CONNECTION_TIMED_OUT`)**  
**Check 1**: Is backend running on .240?  
# On .240 WSL:  
bash  
```  
curl -s http://localhost:9001
```  
Should return JSON response (not empty).  
# **Fix**: If backend isn't running, start it:    
bash  
```  
podman start gru-backend-2.0.0
sleep 5
curl -s http://localhost:9001
```  
**Check 2**: Are firewall rules enabled on .240?  
# On .240 Windows PowerShell (as Administrator):  
Elevated PowerShell  
```  
Get-NetFirewallRule -DisplayName "gru-backend" | Format-Table DisplayName, Enabled
```  
Should show `Enabled = True`  
# **Fix**: If not enabled:  
Elevated PowerShell  
```  
Get-NetFirewallRule -DisplayName "gru-backend" | Set-NetFirewallRule -Enabled $true
```  
**Check 3**: Are port forwarding rules in place?  
# On .240 Windows PowerShell (as Administrator):  
Elevated PowerShell  
```  
netsh interface portproxy show all
```  
**Should show both port 3000 and 9001 forwarding rules.**  
# **Fix**: If missing, re-run **Step 5** commands with correct WSL IP.  

# **Symptom**: Frontend loads but shows errors in browser console  
**Check 1**: Clear browser cache on .52  
Press `Ctrl+Shift+R` (hard refresh, clears cache)   
**Check 2**: Verify `config.json` has correct IP  
# On .240 WSL:  
bash  
```  
cat ~/gene.iobio/client/config.json | grep 192.168.1.240
```  
Should show two lines with `192.168.1.240:9001`  
# **Fix**: If not present, re-run **Step 2** and **Step 3** (modify config and rebuild)  

# **Symptom**: Port 3000 or 9001 already in use  
Find which process is using the port:  
# On .240 WSL:  
bash  
```  
netstat -tlnp | grep -E "3000|9001"
```  
# Or on newer systems:  
bash  
```  
ss -tlnp | grep -E "3000|9001"
```  
# **Fix**: Stop the conflicting process, or use a different port (edit startup script or serve command).  

# **Symptom**: WSL internal IP changed (port forwarding broke)  
WSL internal IP can change after restart. If .52 suddenly can't reach .240, the IP may have changed.  
Find new WSL IP:  
# On .240 WSL:  
bash  
```  
hostname -I
```  
## **Update port forwarding rules**:  
# On .240 Windows PowerShell (as Administrator):  
# First, **remove old rules**:  
Elevated PowerShell  
```  
netsh interface portproxy delete v4tov4 listenport=3000 listenaddress=0.0.0.0
```  
Elevated PowerShell  
```  
netsh interface portproxy delete v4tov4 listenport=9001 listenaddress=0.0.0.0
```  
# Then add new rules with the new IP:
(Replace <NEW_IP> with the IP from the above `hostname -I` command result.)  
Elevated PowerShell  
```  
netsh interface portproxy add v4tov4 listenport=3000 listenaddress=0.0.0.0 connectport=3000 connectaddress=<NEW_IP>
```  
Elevated PowerShell  
```  
netsh interface portproxy add v4tov4 listenport=9001 listenaddress=0.0.0.0 connectport=9001 connectaddress=<NEW_IP>
```  
# Verify:  
PowerShell  
```  
netsh interface portproxy show all
```  

## **Success Criteria — Phase 8 Option A/B Complete**  
   ✅ Frontend configuration updated (localhost → 192.168.1.240:9001)  
   ✅ Frontend rebuilt with new configuration  
   ✅ Windows Firewall rules created (ports 3000, 9001)  
   ✅ WSL port forwarding configured (netsh portproxy)  
   ✅ Startup script created for convenience  
   ✅ Frontend accessible from .52 at http://192.168.1.240:3000  
   ✅ Backend connectivity confirmed from .52  
   ✅ Gene lookup works over network  
   ✅ Large variant lists render without crashing on .52  
   ✅ CSV/VCF export functional (if Phase 8A completed)  
   ✅ Procedure documented for future use  

# **Integration with Phase 8 Options A & B**  
  • **Option A** (`CSV`/`VCF` File Export): `VCF`/`CSV` export works on network-served frontend  
  • **Option B** (Network Serving): (This document)  
  • **Option C** (Documentation): Document network serving setup and any new gotchas  

If both Option A and Option B are applied, the frontend will support both `CSV`/`VCF` file export AND network serving. No conflicts.  

# **Key Gotchas (remember to check out `04_GOTCHA_Enumeration`)**
   1. WSL internal IP can change: If port forwarding breaks after restart, re-run Step 5 with the new IP.  
   2. Firewall rules must be enabled: Check with Get-NetFirewallRule if connection fails.  
   3. Config must be rebuilt: Changing config.json alone isn't enough—run npm run build.  
   4. Backend must be running: Frontend will load but backend requests will timeout if backend isn't started.  
   5. Both ports needed: Frontend (3000) AND backend (9001) must be forwarded and firewalled.  

# Notes for Future Runs  
Always start backend first (see `E_Extra_Startup_Scripts.md` to write your own backend startup script, but then...
Starting the frontend for future sessions is simple:  
# On .240 WSL:  
bash  
```  
~/start_network_frontend.sh
```  
Then access from .52 browser at `http://192.168.1.240:3000`  
No reconfiguration needed unless WSL IP changes.  

Recap of final backup copies of your entire final stack build setup now that all changes have been applied:  
1. Ubuntu `wsl --export`: If you completed `Phase_8_Option_B.md`, you need a new export to capture your changes in WSL.  
2. `/mnt/d/gru_data` .tar:  If you completed **Phase 5**, you have a FINAL gru_data backup and do not need another.  
3. `~/gene.iobio/` .tar: If you completed `Phase_8_Option_A.md`, you need a new export to capture your changes made to `gene.iobio` files.

You may need to use your FINAL restoration set for: `Disaster_Recovery.md`.

---

≡≡≡≡≡ END: Phase_8_Option_B.md ≡≡≡≡≡  
