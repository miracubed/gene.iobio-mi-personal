20260923 gene.iobio 4.13.3 (tag gene_4.13.3, release title
"v4.14.3" = upstream typo, commit 0e6b004) reviewed: Mosaic-only
init/backend_map error handling + simple-mode demo prompt fix. Standalone
unchanged. NO ACTION for standalone local installation for this repository.
4.13.2 already uses runtime client/config.json (.env obsolete); local config:
default_mode=advanced, show_files_button=true, backend 192.168.1.240:9001,
backend_map trimmed to default.
Local deltas vs tag gene_4.13.2: CohortModel.js reject->resolve (flagged
coverage), index.html favicon link, package-lock.json.
Navigation.vue / FlaggedVariantsCard.vue = upstream (.bak identical).
Patch verified live: CohortModel.js 2026-08-19 19:55:49 < dist/build.js
20:34:36; build.js line 126145 = resolve().
No overlap with 4.13.3 files. If upgrading later: checkout, reapply
CohortModel.js + config.json + index.html, npm install, rebuild dist,
re-grep build.js for resolve().
