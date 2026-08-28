# 20260827_DECISION_LOG.md  



**20260826 gru-backend updates:**  

A consolidation merge. PR #130 took 57 accumulated commits from the release-2.0.0 branch (dating back months, some to April 2026) and merged them into main on Aug 26. The version number stayed 2.0.0, but the code inside got substantially refactored and bug-fixed.  


---  

**What actually happened:** Consolidation merge of release-2.0.0 branch into main (PR #130, 57 commits, Aug 26, 2026)  

---  

**Key changes relevant to stack as built in this repository:**  

   - `ES modules` port + `koa` removal (internal refactor, no behavioral change for end users)  
   - `Toml` fallthrough bugfix (April 2026, predates 20260808 baseline build — already in your instance)  
   - `Config.json` + runtime environment variable overrides (new formal configuration layer)  
   - `Podman` build script formalization  
   - `ClinVar` `V3` script addition  
   - Minimum data directory version bump is necessary for `v2.0.0`  
   - `.env` file is not relevant to this local build and configuration as-is still works since we do not need to configure `gene.iobio` to reach out to external sources for data in this implementation  
   - Current `v2.0.0` `gru\_data` version satisfies the new minimum  

---  

**Decision point:**  

Evaluated for rebuild: 

  - `Toml` fix predates current build. 

  - Config changes backward-compatible. 

  - **No immediate action required.**  

---  

**Continue monitoring repositories:**  

  - https://github.com/iobio/iobio-gru-backend

  and   

  - https://github.com/iobio/gene.iobio

via `Github` notifications, `GitWatcher` and `PageCrawl.io` for future updates.

---  

≡≡≡≡≡ END: Upgrades/02_Backend_Upstream_Monitoring_20260826/20260827_DECISION_LOG.md ≡≡≡≡≡
