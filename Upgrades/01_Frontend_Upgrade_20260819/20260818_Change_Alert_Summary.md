# 20260818_Change_Alert_Summary.md
# 20260818  

---  
---  

[@tonydisera](https://github.com/tonydisera) https://github.com/tonydisera pushed 2 commits.  

   • [cb28d29](https://github.com/iobio/gene.iobio/commit/cb28d29a5f6f22e794d61a982078b2e4b2a85339) Move Nebula settings to runtime config  
   • [3c42413](https://github.com/iobio/gene.iobio/commit/3c424130eba731d7c82ae08d465b2b6b8ab8ab8e) Merge `pull` request [#1166](https://github.com/iobio/gene.iobio/pull/1166) from `iobio/v4.13.1-runtime-config`  

---  
---  

Merged [#1166](https://github.com/iobio/gene.iobio/pull/1166) into master.  

---  
---  

**You can view, comment on, or merge this pull request online at:**  
https://github.com/iobio/gene.iobio/pull/1167  

**Commit Summary**  
   • [cb28d29](https://github.com/iobio/gene.iobio/pull/1167/commits/cb28d29a5f6f22e794d61a982078b2e4b2a85339) Move Nebula settings to runtime config  

**File Changes** **([18 files](https://github.com/iobio/gene.iobio/pull/1167/files))**  
   • M [.env.template](https://github.com/iobio/gene.iobio/pull/1167/files#diff-749e06f64632f62a0c0dfbf4c4f3850e27e94ac109aa121fabd5c29469ae88de) (4)  
   • M [.envTemplateExhibit](https://github.com/iobio/gene.iobio/pull/1167/files#diff-404c133d941a07a910dc49a5efc65e51e5b55da727493f30f714afad48c9b392) (3)  
   • D [.envTemplateNebula](https://github.com/iobio/gene.iobio/pull/1167/files#diff-9859c30623066205ef52a1c61359989a00ffdedfb13d2ccd7ee88d576288439b) (10)  
   • M [README.md](https://github.com/iobio/gene.iobio/pull/1167/files#diff-b335630551682c19a781afebcf4d07bf978fb1f8ac04c6bf87428ed5106870f5) (6)  
   • M [RELEASE_NOTES.md](https://github.com/iobio/gene.iobio/pull/1167/files#diff-68d24fa81558aae3d8c59e2aa57a4fa719ea3b04d7fa14beff45f16f00858f50) (6)  
   • M [build.sh](https://github.com/iobio/gene.iobio/pull/1167/files#diff-4d2a8eefdf2a9783512a35da4dc7676a66404b6f3826a8af9aad038722da6823) (6)  
   • M [client/app/components/pages/GeneHome.vue](https://github.com/iobio/gene.iobio/pull/1167/files#diff-a6b9776eef7e07e4a74c7867647dc4a32af9c0dfe8841f46454c646f388a4890) (19)  
   • M [client/app/components/viz/IntroCard.vue](https://github.com/iobio/gene.iobio/pull/1167/files#diff-dedbee3b6d9f2f95414fb5259eff0c0ad18a447044653c0b1fb3c51714c59011) (31)  
   • M [client/app/components/viz/Welcome.vue](https://github.com/iobio/gene.iobio/pull/1167/files#diff-9e849d0e49bb21789c49b2d6d855a6b2b9af58dee12a2b753cf368b6ac18a2e7) (2)  
   • M [client/app/globals/GlobalApp.js](https://github.com/iobio/gene.iobio/pull/1167/files#diff-4b66c9c9dc5961a0862e7a06bc60303eb04c83b0f287dbee541c006209440cae) (2)  
   • M [client/app/models/GeneModel.js](https://github.com/iobio/gene.iobio/pull/1167/files#diff-f9689121fe4bb934c0a1bb029c1a9ddb70cee2d920c33e2010b6cc5818c01fa3) (11)  
   • M [client/app/routes.js](https://github.com/iobio/gene.iobio/pull/1167/files#diff-4276aaa0b62efa28cae5aa15a41e093f0e996b34c019bbb60020aece5972063a) (8)  
   • M [client/config.json](https://github.com/iobio/gene.iobio/pull/1167/files#diff-d9cbba54944b7cda3d0151bf1a5353c8e14d6bfd271ee36b5e9dfe254a553e41) (15)  
   • A [client/config.nebula.json](https://github.com/iobio/gene.iobio/pull/1167/files#diff-7b33edc969de8103653c05502d5e1e20e93705d65255934e7d44eded5fa9459b) (23)  
   • M [client/js/appConfig.js](https://github.com/iobio/gene.iobio/pull/1167/files#diff-7b33edc969de8103653c05502d5e1e20e93705d65255934e7d44eded5fa9459b) (24)  
   • M [package-lock.json](https://github.com/iobio/gene.iobio/pull/1167/files#diff-053150b640a7ce75eff69d1a22cae7f0f94ad64ce9a855db544dda0929316519) (4)  
   • M [package.json](https://github.com/iobio/gene.iobio/pull/1167/files#diff-7ae45ad102eab3b6d7e7896acd08c427a9b25b346470d7bc6507b6481575d519) (2)  
   • M [server/express-app.js](https://github.com/iobio/gene.iobio/pull/1167/files#diff-499177b6addc0aae3a868ca42756c7b22ddfda2e11d65e45e53e6ba3787399fa) (2)  


**Patch Links:**  
   • https://github.com/iobio/gene.iobio/pull/1167.patch  
   • https://github.com/iobio/gene.iobio/pull/1167.diff  

---  
---  

Merged [#1167](https://github.com/iobio/gene.iobio/pull/1167) into `v4.13.summer2026`. 

---  
---  

**Summary**  
Fix critical severity security issue in 'client/config.json'.  

**Vulnerability**  

| Field | Value |   
|---|---|  
| ID | V-001 |  
| Severity | CRITICAL |  
| Scanner | multi_agent_ai |  
| Rule | V-001 |  
| File | client/config.json:1 |  
| Assessment | Likely exploitable |  

**Description:**  API keys and sensitive configuration values are stored in client-side JSON configuration files that are served to the browser. The `omim_api_key` field creates a credential exposure vector, and external API endpoints in `GeneModel.js` may require authentication credentials that could be compromised.

**Evidence**
**Exploitation scenario:** Attacker accesses browser developer tools to download `client/config.json` or `client/config.nebula.json`, extracts the `omim_api_key` value, and uses it to make unauthorized requests to the `OMIM API`.  
**Scanner confirmation:** `multi_agent_ai` `rule V-001` flagged this pattern.  
**Production code:** This file is in the production codebase, not test-only code.  
**Threat Model Context**  
This is a web service - vulnerabilities in request handlers are directly exploitable by remote attackers.

**Changes**
   • `client/config.json`
   • `client/config.nebula.json`

**Behavior Preservation**
The change is scoped to 2 files on the vulnerable path; it only tightens handling of untrusted input and leaves valid inputs unaffected.

Automated security fix by **OrbisAI Security** https://orbisappsec.com/

**You can view, comment on, or merge this pull request online at:**
https://github.com/iobio/gene.iobio/pull/1168
https://github.com/iobio/gene.iobio/pull/1168

**Commit Summary**  
   • [f52555c](https://github.com/iobio/gene.iobio/pull/1168/commits/f52555ceeeedba0028a04b16cc58c1d905370f96) fix: `V-001` security vulnerability

**File Changes** **([2 files](https://github.com/iobio/gene.iobio/pull/1168/files))**  
   • M [client/config.json](https://github.com/iobio/gene.iobio/pull/1168/files#diff-d9cbba54944b7cda3d0151bf1a5353c8e14d6bfd271ee36b5e9dfe254a553e41) (1)  
   • M [client/config.nebula.json](https://github.com/iobio/gene.iobio/pull/1168/files#diff-7b33edc969de8103653c05502d5e1e20e93705d65255934e7d44eded5fa9459b) (1)  

**Patch Links:**
   • https://github.com/iobio/gene.iobio/pull/1168.patch
   • https://github.com/iobio/gene.iobio/pull/1168.diff

---  
---  

**You can view, comment on, or merge this pull request online at:**
https://github.com/iobio/gene.iobio/pull/1169

**Commit Summary**
    • [2fbff05](https://github.com/iobio/gene.iobio/pull/1169/commits/2fbff05ca93decc02a6bca3569e9cdd7472c4068) Merge `pull` request [#1167](https://github.com/iobio/gene.iobio/pull/1167) from `iobio/v4.13.1-runtime-config`
    • [045cd63](https://github.com/iobio/gene.iobio/pull/1169/commits/045cd6350c94660083b06961da1d0381053b467e) Fix `ClinVar` `VCF` URL when switching genome builds.
    • [a72f523](https://github.com/iobio/gene.iobio/pull/1169/commits/a72f52304794c464d780c8fe02df463dae10669c) Release notes for `4.13.1`

**File Changes** **([3 files](https://github.com/iobio/gene.iobio/pull/1169/files))**  
    • M [RELEASE_NOTES.md](https://github.com/iobio/gene.iobio/pull/1169/files#diff-68d24fa81558aae3d8c59e2aa57a4fa719ea3b04d7fa14beff45f16f00858f50) (46)  
    • M [client/app/models/CohortModel.js](https://github.com/iobio/gene.iobio/pull/1169/files#diff-a2675cd3da76ed2870478f5a693d95fa944c57df2da67af34899dae87c3cf1ed) (5)  
    • M [client/app/models/EndpointCmd.js](https://github.com/iobio/gene.iobio/pull/1169/files#diff-b35f7999890f8393ab6d9c0a0bed4acf0ed9d669c321ac1993b70ea27d3f5874) (5)  

**Patch Links:**  
    • https://github.com/iobio/gene.iobio/pull/1169.patch  
    • https://github.com/iobio/gene.iobio/pull/1169.diff  

---  
---  

Merged [#1169](https://github.com/iobio/gene.iobio/pull/1169) into master.

---  
---  

[@tonydisera](https://github.com/tonydisera) https://github.com/tonydisera pushed 4 commits.  
   • [2fbff05](https://github.com/iobio/gene.iobio/commit/2fbff05ca93decc02a6bca3569e9cdd7472c4068) Merge pull request [#1167](https://github.com/iobio/gene.iobio/pull/1167) from `iobio/v4.13.1-runtime-config`  
   • [045cd63](https://github.com/iobio/gene.iobio/commit/045cd6350c94660083b06961da1d0381053b467e) Fix `ClinVar` `VCF` URL when switching genome builds.  
   • [a72f523](https://github.com/iobio/gene.iobio/commit/a72f52304794c464d780c8fe02df463dae10669c) `Release notes` fpr `4.13.1`  
   • [19faa21](https://github.com/iobio/gene.iobio/commit/19faa21cb151ec7b3e11a81235391a85351c32cc) Merge `pull` request [#1169](https://github.com/iobio/gene.iobio/pull/1169) from `iobio/v4.13.summer2026`  

---  
---  

**You can view, comment on, or merge this pull request online at:**  
https://github.com/iobio/gene.iobio/pull/1170  

**Commit Summary**  
   • [ff00ae7](https://github.com/iobio/gene.iobio/pull/1170/commits/ff00ae7025dfa7216ace66e56c3535cdfe1509a4) point the production `gene.iobio` to the `backend.iobio.io `http://backend.iobio.io/ gru instance. bump up the version number to `4.13.1.1`.  
   • [1854f59](https://github.com/iobio/gene.iobio/pull/1170/commits/1854f59eed42ad2a7bd7d37abd6f8f5fd15258a8) Fix minor version - now `4.13.2`  

**File Changes** **([4 files](https://github.com/iobio/gene.iobio/pull/1170/files))**  
   • M [RELEASE_NOTES.md](https://github.com/iobio/gene.iobio/pull/1170/files#diff-68d24fa81558aae3d8c59e2aa57a4fa719ea3b04d7fa14beff45f16f00858f50) (60)  
   • M [client/app/globals/GlobalApp.js](https://github.com/iobio/gene.iobio/pull/1170/files#diff-4b66c9c9dc5961a0862e7a06bc60303eb04c83b0f287dbee541c006209440cae) (2)  
   • M [client/config.json](https://github.com/iobio/gene.iobio/pull/1170/files#diff-d9cbba54944b7cda3d0151bf1a5353c8e14d6bfd271ee36b5e9dfe254a553e41) (7)  
   • M [package.json](https://github.com/iobio/gene.iobio/pull/1170/files#diff-7ae45ad102eab3b6d7e7896acd08c427a9b25b346470d7bc6507b6481575d519) (2)  


**Patch Links:**  
   • https://github.com/iobio/gene.iobio/pull/1170.patch  
   • https://github.com/iobio/gene.iobio/pull/1170.diff  

---  
---  

Merged [#1170](https://github.com/iobio/gene.iobio/pull/1170) into `master`.  

---  
---  

[@tonydisera](https://github.com/tonydisera) https://github.com/tonydisera pushed 3 commits.  

  • [ff00ae7](https://github.com/iobio/gene.iobio/commit/ff00ae7025dfa7216ace66e56c3535cdfe1509a4) point the production `gene.iobio` to the `backend.iobio.io` http://backend.iobio.io/ gru instance. bump up the version number to `4.13.1.1`.  
  • [1854f59](https://github.com/iobio/gene.iobio/commit/1854f59eed42ad2a7bd7d37abd6f8f5fd15258a8) Fix minor version - now `4.13.2`  
  • [bf161d7](https://github.com/iobio/gene.iobio/commit/bf161d75e62ade11fc95fa90c80e5badb772e547) Merge `pull` request [#1170](https://github.com/iobio/gene.iobio/pull/1170) from `iobio/v4.13.summer2026`  

---
---  

# v4.13.2 https://github.com/iobio/gene.iobio/releases/tag/gene_4.13.2
**Repository**: [iobio/gene.iobio](https://github.com/iobio/gene.iobio) https://github.com/iobio/gene.iobio·
**Tag**: [gene_4.13.2](https://github.com/iobio/gene.iobio/tree/gene_4.13.2) https://github.com/iobio/gene.iobio/tree/gene_4.13.2·
**Commit**: [1854f59](https://github.com/iobio/gene.iobio/commit/1854f59eed42ad2a7bd7d37abd6f8f5fd15258a8)·
**Released by**: [@tonydisera](https://github.com/tonydisera) https://github.com/tonydisera

# gene.iobio v4.13 Release Notes  

## Backward and forward compatibility with iobio gru backend v2.0  
   • `iobio gru backend v2.0` revamped the `gnomAD` data directory structure and annotated `INFO` field names to be more consistent and easier to maintain.  
   • `annotateVariantsV3` annotates variants with `gnomAD` population allele frequencies for `GRCh37` (`gnomAD` `v2.1.1`) and `GRCh38` (upgraded from `v4.0` to `v4.1`). `gnomAD` `INFO` fields from `annotateVariantsV3` and `getClinvarVariantsV2/V3` use uniform prefixes of `gg4` or `gg2` (`gnomad` genomes v4 or v2 for `GRCh37` and `GRCh38` respectively).  
   • We postponed adoption of the combined allele frequencies for genomes and exomes now supported in `gnomAD v4.1` for a few reasons:  
      • the `vcf` files lack the key fields such as the `NONPARS` flag and warning flags when the distribution across population groups diverges between gnomad exomes and genomes  
      • The `HAIL` tables have the comprehensive set of fields, but this will require considerable effort to port from `vcfanno` to a `python` library or `c application` supporting `HAIL`.  

## Upgraded dependencies  
   • Upgraded `Node.js` (`v13` -> `v20`)  
   • Upgraded `webpack` (`v2.7` -> `v3.12`)  
   • Replace `node-gyp` and `node-sass` with `pure-js package sass` (`1.89.2`). Upgraded `sass-loader` (`v6.0.7` -> `v7.3.1`).  
   • Downgraded `css-loader` (`1.0.1` -> `0.28.11`).  

## Run-time config  
   • Runtime paths, backend URLs, and gene.iobio deployment settings are loaded from `client/config.json` for local development. Routing now uses the `snake_case path_prefix` property.  
   • Nebula settings such as `gene.default_mode`, `gene.site_name`, `gene.show_intro`, and the intro paragraphs are runtime configuration rather than `webpack` `.env` values.  
   • When served by `GRU`, `IOBIO_GENE_*`, `IOBIO_BAM_*`, and `IOBIO_BACKEND_*` environment variables are mapped generically to `snake_case` runtime properties. Exact true and false values become JSON booleans.  

## Better error handling for local bam files  
   • When a local bam file is selected, catch any errors and show an informative error message. For example, Waygate doesn't handle spaces in the local file name. Before this fix, the error was swallowed and subsequent requests failed because the bam URL wasn't reachable.  

## ClinVar variants track loads from vcf of active genome build. (bug fix)  
   • ClinVar track was using wrong build `VCF` after genome build switch. This bug only showed up when switching from `GRCh38` to `GRCh37` or vice versa in the same session.  

## Minor releases  
   • `4.13.0`  
      • initial release of `gene.iobio` `v4.13.0`  
   • `4.13.1`  
      • Nebula runtime config support  
      • Bug fix: prevent stale `ClinVar` `vcf` url from being used after genome build switch.  
   • 4.13.2  
      • Standalone prod `gene.iobio` runtime config points to `backend.iobio.io` http://backend.iobio.io/.   

This release has 2 assets:  
    • Source code (zip)  
    • Source code (tar.gz)  
Visit the release page https://github.com/iobio/gene.iobio/releases/tag/gene_4.13.2 to download them.  

---  
---  

≡≡≡≡≡ END: Upgrades/01_Frontend_Upgrade_20260819/20260818_Change_Alert_Summary.md ≡≡≡≡≡
