## **Phase 8 Option A — `CSV`/`VCF` File Export Feature Unlock**

> **Before starting Phase 8 Option A:** **Phase 7** must be complete.  
> The stack is fully functional at `http://localhost:3000`, and  
> Backend is verified healthy. Frontend is built and serving.  

> **What this phase does:** Unlocks the CSV and VCF file export of variant options in the `gene.iobio` webpage UI (user interface).
> These features were fully implemented in `v4.13.1` but inaccessible due to two broken conditional gates in the frontend code.
> **Three one-line edits and a rebuild are all that is required. No new code is written.**

> **Phase 8 (Options A and B) are optional but recommended.** A fully working local stack exists at the end of **Phase 7**.
> **Phase 8 Option A** adds .csv/.vcf file export functionality.
> **Phase 8 Option Ba** adds LAN network (local area network) serving.
> **Phase 8 Option Bb** locks the WSL IP statically. Each option builds on the previous — apply them in order if applying more than one.

| File | Line | Original Gate | Problem |  
|---|:---:|---|---|  
| `client/app/components/viz/FlaggedVariantsCard.vue` | 843 | `cohortModel.flaggedVariants && cohortModel.flaggedVariants.length \> 0` | Condition never satisfied — auto-populated variants land in a different data structure |  
| `client/app/components/viz/Navigation.vue` | 952 | same | same |  
| `client/app/models/CohortModel.js` | 2903 | `reject(error)` | Blocks export when BAM depth refresh times out on large variant sets |  



### **Step 1 — Back Up All Three Files Before Editing**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

> **Always back up before editing source files.**
> These `.bak` files are your safety net.
> If an edit goes wrong, you can restore the original and try again without rebuilding from scratch.
> See **`Gotcha 8A.4`**.
Cp/Copy `FlaggedVariantsCard.vue` file as `*.bak` file  
bash  
```  
cp ~/gene.iobio/client/app/components/viz/FlaggedVariantsCard.vue ~/gene.iobio/client/app/components/viz/FlaggedVariantsCard.vue.bak
```  

Cp/Copy `Navigation.vue` file as `*.bak` file  
bash  
```  
cp ~/gene.iobio/client/app/components/viz/Navigation.vue ~/gene.iobio/client/app/components/viz/Navigation.vue.bak
```  
Cp/Copy `CohortModel.js` file as `*.bak` file  
bash  
```  
cp ~/gene.iobio/client/app/models/CohortModel.js ~/gene.iobio/client/app/models/CohortModel.js.bak
```  

Verify all three backups exist:  
bash  
```  
ls ~/gene.iobio/client/app/components/viz/*.bak
```  
bash  
```  
ls ~/gene.iobio/client/app/models/*.bak
```  
Expected: all three `.bak` files listed.  


### **Step 2 — Edit `FlaggedVariantsCard.vue`**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory → **`Glossary`: `nano line navigation`** — see **`07_OVERVIEW.md`**  
bash  
```  
nano +843 ~/gene.iobio/client/app/components/viz/FlaggedVariantsCard.vue
```  
> The `+843` flag opens `nano` directly at `line 843`, saving you from scrolling through a large file manually.
> This is especially useful given the file size.  
At `line 843`, find:  
```  
cohortModel.flaggedVariants && cohortModel.flaggedVariants.length > 0
```  
Replace it with:  
Copy and paste into `nano` replacing `cohortModel.flaggedVariants && cohortModel.flaggedVariants.length > 0` using the method that works for your system with:
```  
cohortModel.isLoaded
```  
> **Make this change only on `line 843`. Do not change any other occurrences of similar text elsewhere in the file**.  
Save: **Ctrl+O**, Enter, **Ctrl+X**  

Verify the change:  
bash  
```  
grep -n "cohortModel.isLoaded" ~/gene.iobio/client/app/components/viz/FlaggedVariantsCard.vue
```  
Expected: output shows **`line 843`** containing **`cohortModel.isLoaded`**


### **Step 3 — Edit `Navigation.vue`**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
nano +952 ~/gene.iobio/client/app/components/viz/Navigation.vue
```  
At `line 952`, find:
```
cohortModel.flaggedVariants && cohortModel.flaggedVariants.length > 0
```
Replace it with:  
Copy and paste into `nano` replacing `cohortModel.flaggedVariants && cohortModel.flaggedVariants.length > 0` using the method that works for your system with:
```  
cohortModel.isLoaded
```  
Save: **Ctrl+O**, Enter, **Ctrl+X**  

Verify the change:  
bash  
```  
grep -n "cohortModel.isLoaded" ~/gene.iobio/client/app/components/viz/Navigation.vue
```  
Expected: output shows **`line 952`** containing **`cohortModel.isLoaded`**  


### **Step 4 — Edit `CohortModel.js`**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  
bash  
```  
nano +2903 ~/gene.iobio/client/app/models/CohortModel.js
```  
At `line 2903`, find:  
bash  
```  
reject(error)
```  
Replace it with:  
Copy and paste into `nano` replacing `reject(error)` using the method that works for your system with:
```  
resolve()
```  
Save: **Ctrl+O**, Enter, **Ctrl+X**  
> **Context:** This line is inside a coverage refresh catch block.  
> The original `reject(error)` caused export to fail when BAM depth enrichment timed out on large variant sets.  
> Changing it to `resolve()` allows export to proceed even if coverage refresh fails.  
> See **`Gotcha 8A.3`**.  

Verify the change:  
bash  
```  
grep -n "resolve()" ~/gene.iobio/client/app/models/CohortModel.js | grep 2903
```  

Expected: **`line 2903`** containing **`resolve()`  **  


### **Step 5 — Verify Only Three Edits Were Made**  

**Where:** WSL Ubuntu terminal — user: `mir`, any directory  

Before rebuilding, confirm that only the three intended changes are present and no leftover edits from prior sessions exist:  
bash   
```   
diff ~/gene.iobio/client/app/components/viz/FlaggedVariantsCard.vue ~/gene.iobio/client/app/components/viz/FlaggedVariantsCard.vue.bak
```  
bash  
```  
diff ~/gene.iobio/client/app/components/viz/Navigation.vue ~/gene.iobio/client/app/components/viz/Navigation.vue.bak
```  
bash  
```  
diff ~/gene.iobio/client/app/models/CohortModel.js ~/gene.iobio/client/app/models/CohortModel.js.bak
```  

> Each `diff` should show (the difference between the files) exactly one changed line per file — the edits you just made.  
> If `diff` shows more than one change per file, review carefully before proceeding.  
> Unexpected changes could indicate a prior edit session left something behind.
> See **`Gotcha 8A.4`**.  


### **Step 6 — Rebuild the Frontend**  

**Where:** WSL Ubuntu terminal — user: `mir`, directory `~/gene.iobio`  
bash  
```  
cd ~/gene.iobio
```
bash  
```  
npm run build
```  
Expected: clean build, approximately 29 seconds, output `client/dist/build.js` approximately 30.5MB. No errors.  

Verify the build output:  
bash  
```  
ls -lh ~/gene.iobio/client/dist/build.js
```  
Expected: file present, approximately 30.5MB, timestamp updated to just now.  

> **`build.js` ordering note:** If you are applying both **Phase 8 Option A** and **Phase 8 Option B**,  
> always apply **Phase 8 Option A** first and rebuild, then apply **Phase 8 Option B** and rebuild again.  
> Never substitute a saved **`build.js`** from a prior session when applying both options —  
> the final build must incorporate all changes from both phases in sequence.  
> See **`Gotcha 8A.5`**.  


### **Step 7 — Restart and Validate**  

**Where:** Two WSL Ubuntu terminals — user: `mir`  

**Terminal 1 — confirm backend is running:**  
bash  
```  
curl -s localhost:9001
```  
Expected: JSON health response. If the backend is not running, start it:  
bash  
```  
podman start gru-backend-2.0.0
```  

Wait a moment, then verify:  
bash  
```  
curl -s localhost:9001
```  

**Terminal 2 — restart the frontend:**  
bash  
```  
cd ~/gene.iobio/client
```
bash  
```  
serve -s . -l 3000
```  

**Open browser and navigate to `http://localhost:3000`**


### **Step 8 — Validate Export Functionality**

**Where:** Windows browser — `http://localhost:3000`  

Load a `VCF` file using the **Load** button. Select a gene. Allow variants to populate.  

Test the export dialog:  

| Test | Expected Result |  
| - | - |  
| Export dialog appears when variants loaded | ✅ |  
| `CSV` export option is selectable | ✅ |  
| `CSV` export produces valid file with header + variant rows | ✅ |  
| `VCF` export option is selectable | ✅ |  
| `VCF` export produces valid VCF with CHROM/POS/REF/ALT columns | ✅ |  

Depending on whether you select `.csv` or `.vcf` to export as a file, by default the exported file is named:
> `gene-iobio-flagged-variants.csv`
> or
> `gene-iobio-flagged-variants.vcf`

> **Performance note on large variant sets:**
> Exports of 60+ unfiltered variants may be slow due to BAM depth coverage refresh overhead.
> This is expected behavior, not a bug.
> Filter to a manageable variant set before exporting for best results.
> See **`Gotcha L.2`**.


### **Step 9 — Final Backup: Post Phase 8 Option A**

**Where:** WSL Ubuntu terminal — user: `mir`, any directory

The only files that were changed/built involve the frontend,  
so this what we must capture as a backup (`~/gene.iobio directory`).  
If you are stopping with `Phase_8_Option_A.md` complete, the following is your FINAL backup,  
so you can add `FINAL` to the file name. If you are going on to complete `Phase_8_Option_B.md`,  
hold your FINALbackup until the end of that subphase.  
Create an updated frontend backup archive capturing the `Phase 8 Option A` edits:  
bash  
```  
tar -cf /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_CSVVCF_20260808.tar  ~/gene.iobio/
```  

Verify MD5:  
bash  
```  
md5sum /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_CSVVCF_FINAL_20260808.tar
```  

Note and record the hash.  

> MD5 is from maintainer's system and will likely differ on yours.  
> Keep the file naming conventions, update the date portion for your own export.  
> Verify your hash matches itself across both copy locations (`forge`/`vault`).  

Copy to `vault`:
bash  
```  
cp /mnt/d/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_CSVVCF_20260808.tar "/mnt/medlaptop/e/MedLaptop\ Mirror/gene_iobio_dir_bkps/gene_iobio_frontend_dir/"
```   

Verify `vault` MD5:  
bash  
```  
md5sum "/mnt/medlaptop/e/MedLaptop\ Mirror/gene_iobio_dir_bkps/gene_iobio_frontend_dir/gene_iobio_frontend_CSVVCF_20260808.tar"
```  

Hashes must match.


## **Phase 8 Option A Enable CSV/VCF File Export Functionality Completion**

| Component | Status |  
| - | - |  
| `FlaggedVariantsCard.vue` `line 843` | ✅ gate → `cohortModel.isLoaded` |  
| `Navigation.vue` `line 952` | ✅ gate → `cohortModel.isLoaded` |  
| `CohortModel.js` `line 2903` | ✅ `reject(error)` → `resolve()` |  
| All three originals backed up as `.bak` | ✅ |  
| `Diff` verified — three edits only, no `cruft` | ✅ |  
| Frontend rebuilt | ✅ clean, ~30.5MB build.js |  
| Export dialog accessible | ✅ |  
| `CSV` export functional | ✅ valid file |  
| `VCF` export functional | ✅ valid VCF |  
| Frontend archive updated | ✅ MD5 verified `forge` + `vault` |  

> See **`Glossary`: `cruft`** in **`07_OVERVIEW`**

> If you are not proceeding to **Phase 8 Option B:** The stack is complete.  
> A fully functional local `gene.iobio` installation with `CSV/VCF export` is operational at `http://localhost:3000`.  
> See the **`Quick Reference`** section and **`E_Extra_Startup_Scripts.md`** for day-to-day startup commands.  

> If you are proceeding to Phase 8 Option B for LAN serving and WSL IP persistence:** Continue directly.  
> **Phase 8 Option B** requires another `npm run build` after its own configuration changes —  
> do not skip it or substitute the current `build.js`.  


## **Known Limitations**  

These are expected behaviors, not bugs. Understanding them prevents unnecessary troubleshooting.  

| Limitation | Detail |  
| - | - |  
| Modifier impact section — browser rendering | Expanding the Modifier impact section on whole-genome variant lists (thousands of intronic/UTR variants) can crash the browser tab. Expected behavior on large datasets. Filter before expanding. |  
| `CSV`/`VCF` export — large unfiltered sets | Exports of 60+ unfiltered variants may be slow due to BAM depth coverage refresh overhead. Filter to a manageable variant set before exporting. |  
| WSL IP persistence | Resolved in **Phase 8 Option B** via `netplan`. If portproxy rules break unexpectedly, verify `hostname -I` returns `172.18.60.70`. |  
| Vue development mode warning | Appears in browser console. Expected — does not affect functionality. |  
| HTTPS warning on data `URI`s | Browser compliance notice only. Export files download correctly despite the warning. |  
| `netplan` `apply systemd` warning | WSL2 does not run systemd. `sudo netplan apply` warns about this every time. Expected, non-fatal, does not affect static IP assignment. |  
| `1.17.0 image` still present in `Podman` | The 1.17.0 image was pulled during the original build process. If both 1.17.0 and 2.0.0 are present in your `Podman` storage, **never run them simultaneously**. Always verify the running image with `podman ps --format "{{.Names}} {{.Image}}"` before use. **The correct image ID for 2.0.0 is `7e2c4a419267`**. |  

Next Steps:

Stop here, create your final backups unless moving to `Phase_8_Option_B.md`...  
> (Out of an abundance of caution: I did create production stack backups prior to  
> execution of `Phase_8_Option_A.md` and `Phase_8_Option_B.md`, but if I had to do it again,  
> I would just apply the fixes in `Phase 8 Option A` and `B`, and then make my  
> FINAL backups with all fixes included.) Your choice.  

Next stop:

   - Continue to LAN configuration and WSL IP Persistence in **`Phase_8_Option_B.md`**

Then make your FINAL backups to use in case **`Disaster_Recovery.md`** is needed.  

---

≡≡≡≡≡ END: Phase_8_Option_A.md ≡≡≡≡≡  
