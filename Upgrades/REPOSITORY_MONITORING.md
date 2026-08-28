# Note About Monitoring Repositories  

## Why Two Different Monitoring Tools?  

`gene.iobio` and `iobio-gru-backend` are two different repositories, are  
maintained separately and use different release conventions, requiring  
different monitoring approaches.  

## Frontend: gene.iobio  
- **Tool:** [GitWatcher](https://www.gitwatcher.com) https://www.gitwatcher.com  
- **Why:** `gene.iobio` uses GitHub's formal Releases UI, which GitWatcher  
  monitors natively  
- **What it catches:** New releases, version bumps, release notes  

## Backend: iobio-gru-backend  
- **Tool:** [PageCrawl.io](https://www.pagecrawl.io) https://www.pagecrawl.io  
- **Why:** `iobio-gru-backend` uses git tags for versioning rather than  
  GitHub's formal Releases UI — a perfectly valid development convention,  
  but one that falls outside GitWatcher's monitoring scope  
- **What it catches:** Branch status changes, tag count increases,  
  commit message changes, language breakdown shifts  
- **Threshold:** Score 30+ triggers notification (filters cosmetic changes)  

## Backstop: GitHub Email Notifications  
- Subscribe directly via your GitHub account settings  
- https://github.com/iobio/gene.iobio  
- https://github.com/iobio/iobio-gru-backend  
- Summary format only — useful as a secondary layer, not primary  

## Notes  
- PageCrawl.io free tier is sufficient for gru-backend monitoring  
- GitWatcher free tier is sufficient for frontend monitoring  
- Neither tool requires access to private repository data  

---  

≡≡≡≡≡ END: Upgrades/REPOSITORY_MONITORING.md ≡≡≡≡≡
