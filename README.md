# gene.iobio-mi-personal Local Stack — Complete Build & Deployment Guide

**Status:** Production-ready (Phase 8 complete)  
**Last Updated:** H20260806  
**Build Target:** Ubuntu 26.04 LTS WSL2 + Podman + iobio-gru-backend 2.0.0 + gene.iobio v4.13.1

---

## What This Is

A guide to building a complete, reproducible bioinformatics stack for local gene analysis.
Built to work with whole genome sequencing data when commercial tools 
(like Nebula Genomics) can't/don't provide the depth of analysis or portability
you need. Being a Linux neophyte/newbie need not be an insurmountable obstacle.

**This stack includes:**  
- gru-backend 2.0.0 (variant annotation + gene lookup + phenotype association)  
- gene.iobio v4.13.1 frontend (interactive gene browser + variant visualization)  
- VEP 115 (Ensembl Variant Effect Predictor)  
- gnomAD 4.1 (population frequency data)  
- Phenolyzer integration (gene-phenotype association scoring)  
- Full MD5-verified backup strategy  

**Why it exists:**  
Because understanding your own genome should be possible without being locked into a commercial platform's analysis choices. Sometimes you need more portable tools that don't rely on an internet connection.
---  

## Quick Start  

### Prerequisites  
- Windows 11 Home + WSL2 (Ubuntu 26.04 LTS)  
- 2GB RAM minimum (4GB+ recommended)  
- 250GB disk space minimum (500GB+ recommended for full gru_data)  
- Podman 5.7.0+  
- Node.js v16+ (via NVM)  

## 30-Second Startup (if all is configured)

### Start backend  
bash  
```  
podman run gru-backend-2.0.0
```
(or your `start_network_backend.sh` from `E_Extra_Startup_Scripts.md`)

### Start frontend (in second terminal)  
bash  
```  
cd ~/gene.iobio/client
```  
bash  
```  
serve -s . -l 3000
```
(npx serve -s . -l 3000 if you have not installed `serve`)
(or your `start_network_frontend.sh` from `E_Extra_Startup_Scripts.md`)

### Open browser  
```  
http://localhost:3000
```
or if you have completed the LAN/WSL IP persistence configuration in `Phase_8_Option_B.md`...
From another browser on the network as your `gene-iobio` server setup go to:  
```  
http://192.168.1.240:3000
```  


## **Full Build from Scratch**
See `01_Stack_Build_Steps.md` (Phase 1 through Phase 7, see `Phase_8_Option_A.md` and `Phase_8_Option_B.md` for extended functionality configurations.)  

## Documentation Structure  
| Document | Purpose |  
|---|---|  
| `01_Stack_Build_Steps.md` | Step-by-step build procedure (Phases 1-8) |  
| `02_Session_Handoff_Log.md` | Chronological session record (H-series) |  
| `03_Stack_Versioning_Reference.md` | All component versions, locations, MD5s |   
| `04_GOTCHA_Enumeration.md` | 62 known issues & workarounds (phase-numbered) |  
| `05_ACCESSIBILITY_NOTES.md` | Physical/visual accessibility considerations |  
| `06_MEDICAL_CONTEXT.md` | Why this stack exists (clinical background) |  
| `07_OVERVIEW.md` | Project overview |  
| `08_WORKFLOW.md` | If you've chosen to work with AI to assist you, this might help |  
| `09_MEMORY_MAINTENANCE.md` | Leveraging and Managing Brave Leo AI Memory Entries |    
| `09a_MEMORY_MAINTENANCE_EXAMPLES.md` | Real example Memories for reference |  
| `D_Drive_Cleanup_Priority.md` | After project cleanup and space reclamation |  
| `Diaster_Recovery.md` | When disaster strikes: Be prepared |  
| `E_Extra_Startup_Scripts.md` | Additional start_network_backend.sh script |  
| `LICENSE.md` | Licensing stuff |  
| `Phase_8_Option_A.md` | Details - `CSV`/`VCF` File export functionality unlock |  
| `Phase_8_Option_B.md` | Details - LAN network/WSL IP detail and more troubleshooting |  
| `Phase_8_Option_C.md | Details - Bonus documentation how-to detail if interested |  


## Key Files & Locations  
On Host (Windows):  
    • WSL backups: `Vault`: \\192.168.1.52\MedLaptop Mirror\Ubuntu\WSL_Backups  
    • Gene.iobio backups: `Vault`: \\192.168.1.52\MedLaptop Mirror\gene_iobio_dir_bkps  
    • gru_data archives: `Forge`: D:\gene_iobio_dirs\gene_iobio_backend_dir  
In WSL on .240:  
    • Frontend: ~/gene.iobio/  
    • Backend data: /mnt/d/gru_data/  
    • Phenolyzer: ~/phenolyzer/  


## Production Stack Versions  
| Component | Version | Image ID |   
|---|---|---|  
| gru-backend | 2.0.0 | 7e2c4a419267 |  
| VEP | 115 | in container |  
| gnomAD | 4.1 | in gru_data |  
| gene.iobio | v4.13.1-runtime-config | cb28d29a |  
| Node.js | v16.20.2 | via nvm |  
| Ubuntu | 26.04 LTS | WSL2 |  


Validated Against Real Data  
   ✅ Nebula WGS (GRCh37)  
   ✅ SLC7A7 homozygous variant (chr14:23294210 CA→C, GT 1/1, GQ 90)  
   ✅ Variant annotation pipeline  
   ✅ Phenolyzer integration  
   ✅ Gene lookup + transcript visualization  


## Known Limitations  
   • Modifier impact section: Browser rendering can crash on whole-genome variant lists (thousands of intronic/UTR variants). Expected behavior, not a bug.  
   • CSV and VCF export of variants: Requires `Phase_8_Option_A.md`.  
   • Network serving: Frontend currently serves on localhost only. Network setup documented in `Phase_8_Option_B.md`.  


## Support & Troubleshooting  

### **04_GOTCHA_Enumeration.md**     

Gotchas are by `01_Stack_Build_Steps.md` Phases.  
Container issues?  
See Gotcha 3.1–3.4 (Podman startup)  

Data sync issues?  
See Gotcha 4.1–6.6 (gru_data backend)  

Frontend build issues?  
See Gotcha 7.1–7.7 (Node.js + webpack)  

> All 62 gotchas are phase-numbered in `04_GOTCHA_Enumeration.md` for easy cross-reference  

Contributing  
If you build this stack and encounter issues not covered in the gotchas, please document them with:  
    • Exact phase/step where it occurred  
    • Full error message  
    • Your environment (Windows version, WSL version, Podman version)  
    • What you tried to fix it  

I'm not sure I'd know the answer to whatever issues you're having (I'm a Linux newbie, this project was my first foray into Linux via WSL Ubuntu), but my AI and Google-Fu games have been fairly strong lately and I'd certainly try to help as best as I can. 
That said: I have been relying on Brave Leo AI to assist me through this process, including any troubleshooting.

   > Your worst case scenario: Copy and paste the first 4 documents of this repository  
   > (`01_Stack_Build_Steps.md` to `04_GOTCHA_Enumeration.md`) to an AI like Brave Leo and  
   > tell them you're trying to install your own `gene.iobio` stack and ask for assistance.  
   > This might at least avoid a number of the many hours/days spent going round and round  
   > with incorrect solutions to meet your own needs. Since this stack was built on a very  
   > resource constrained system: A system with equal or better resources might deploy much  
   > easier and this instruction build guide repository could still be valuable to help get  
   > you going.  

These documents are far from perfect, but I wanted to try to get them out as soon as possible to help others in any way possible as I dig out from under all the proofreading and formatting issues.  
While I'm able, I'm trying to get help out there for anyone who may need it.  
I sincerely hope you find these documents more helpful than starting from zero to create your own stack as a newbie.  
I also hope I've managed to include some of the 'whys' for choices made or actions taken so you can learn throughout the process as I have.  

## License  
This documentation is provided as-is for educational and research purposes.  
The underlying tools (gru-backend, gene.iobio, VEP, gnomAD, Phenolyzer, etc.) are governed by their respective licenses. See `LICENSE.md`    

## Why This Exists  
Because understanding your own genome shouldn't require accepting someone else's analysis choices.  
Because rare genetic disorders deserve better tools.  
Because self-advocacy with medical data is hard enough without also fighting closed platforms.  
Because some people live in areas that still don't have great internet access, even around larger cities.  
Because some people don't have rapid access to professional genetic or genetic-metabolic clinical resources.  
This stack was built by someone managing a rare metabolic disorder (LPI), other autoimmune issues, several  
other health complications and a concerning cardiac predisposition while using accessibility accommodations  
for visual and motor challenges, and validating it against their own WGS data.  
If you're in a similar situation, this repository may be for you. 🟢  

## Questions?  
See the individual documents. They're comprehensive.  
Want to contribute? Document what you learn.  
Future you will thank you.  
I recommend reading `07_OVERVIEW.md` next in sequence.  

# To:
The entire Iobio team (notably: Tony Di Sera (tonydisera) and Anders Pitman (anderspitman)),  
Kai Wang (kaichop) at WGLab Phenolyzer,  
The people at Ensembl.org,  
gnomAD, SAMtools and others who have collaborated or helped with any of the aforementioned persons/teams/products:  
Your work is reaching and helping more people than you may know.  
I could not have found my rare disease variants without any of you. Finding my variants with your help has kept me alive and out of serious trouble until I can see an actual metabolic geneticist clinician help me. Many thanks to you all.  
(There are a lot of 'moving parts' here in this stack. My apologies if I have left out anyone I should have included.)  

---

≡≡≡≡≡ END: README.md ≡≡≡≡≡
