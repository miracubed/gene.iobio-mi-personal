# UPGRADE_CHRONICLE.md for gene.iobio full backend/frontend stack  
# Gene.iobio Stack Upgrade Chronicle  


Last Update: 20260827  


## Open Note To Upstream Maintainers (Tony Di Sera, Anders Pitman, iobio team)  

If any of this documentation, structure, or approach is useful for your own documentation efforts,  
you're welcome to use, adapt, or build on it. No attribution needed. The goal is to make `gene.iobio`   
more accessible to people outside traditional bioinformatics workflows—if this helps you do that, take what works and leave the rest.  


---  


##  Scheme of the Upgrades directory of this repository:  



Aug 8, 2026 - Initial deployment: frontend v4.13.1 + backend gru 2.0.0 (validated)  
Aug 19, 2026 - Frontend upgrade: v4.13.1 → v4.13.2 (validated, backend unchanged)  
Aug 26, 2026 - Backend upstream monitoring: PR #130 (evaluated, no rebuild needed)  


---  


## Actual Repository directories:  

```  
`Upgrades/`
  ├── `UPGRADE_CHRONICLE.md` ← Single master document, chronological
  ├── `00_Initial Full Stack Deployment Baseline_20260808` - no directory - work lives in `main` documents
  └── `01_Frontend_Upgrade_20260819/`
        ├── `20260818_Change_Alert_Summary.md`
        ├── `Upgrade_v4.13.1_to_v4.13.2.md`
        └── `v4.13.2_Stack_Versioning_Reference.md
  └── `02_Backend_Upstream_Monitoring_20260826/`
        ├── `20260826_Change_Alert_Summary.md`
        └── `20260827_DECISION_LOG.md`
```  


> Initial build can/should start with '01 - Frontend Upgrade' as this current full local build of:  
> `gene.iobio` `gru-backend` `v2.0.0` and frontend `v4.13.2`
> See `v4.13.2 Release Notes` at https://github.com/iobio/gene.iobio/releases/tag/gene_4.13.2  


---  


## 00 - Initial Full Stack Deployment Baseline | Aug 8, 2026 (no directory - work lives in `main` documents)  
   - **Frontend:** `v4.13.1`  
   - **Backend:** `gru-backend` `v1.17.1`, immediately updated to `v2.0.0`  
   - **Status:** Deployed and validated  

## 01 - Frontend Upgrade | Aug 19, 2026  
   - **Component:** gene.iobio frontend  
   - **Upgrade:** v4.13.1 → v4.13.2  
   - **Backend:** gru-backend v2.0.0 (unchanged)  
   - **Key Change:** Node.js v16 → v20 (sass dependency)  
   - **Status:** Deployed and validated  
   - **Refer to "01_Build_Stack_Steps.md"** for validation  
   - **Details:** See `Upgrades/01_Frontend_Upgrade_20260819/`  

## 02 - Backend Upstream Monitoring | Aug 26, 2026  
   - **Component:** gru-backend upstream  
   - **Event:** PR #130 consolidation merge (57 commits)  
   - **Current Stack:** v4.13.2 frontend + v2.0.0 backend  
   - **Status:** Evaluated, no action required  
   - **Details:** See `Upgrades/02_Backend_Upstream_Monitoring_20260826/`  

 
---  


≡≡≡≡≡ END: Upgrades/UPGRADE_CHRONICLE.md ≡≡≡≡≡
