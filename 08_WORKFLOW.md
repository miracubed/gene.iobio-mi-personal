# WORKFLOW.md — Documentation System Workflow & Rules of Engagement  

## Overview  

This repository maintains three living technical documents for a local  
gene.iobio stack build. This file explains how the documentation  
system works, how sessions are conducted, and the rules that govern  
updates.  

---

## Document Map  

| File | Purpose |  
|---|---|  
| `README.md` | Project introduction, glossary, quick links |  
| `LICENSE.md` | Legal stuff about how I'm not taking credit or trying to steal anyone's software |
| `01_Stack_Build_Steps.md` | Reproducible build procedure — phases, commands, configs |  
| `02_Session_Handoff_Log.md` | Chronological session record — decisions, state, outcomes |  
| `03_Stack_Versioning_Reference.md` | Flat version lookup — scannable, no narrative |  
| `04_GOTCHA_Enumeration.md` | Indexed gotchas by phase — troubleshooting reference |  
| `05_ACCESSIBILITY_NOTES.md` | Accessibility considerations for contributors |  
| `06_MEDICAL_CONTEXT.md` | High-level context for why this stack exists |  
| `07_OVERVIEW.md` | Elevator pitch — what this repo is and who it is for |  
| `08_WORKFLOW.md` | This file — system rules and session protocol |  
| `09_MEMORY_MAINTENANCE.md` | Memory lifecycle, pruning rules, cross-correlation |  
| `09a_MEMORY_MAINTENANCE_Examples.md` | Example real memories and tagging system |  
| `D_Drive_Cleanup_Priority.md` | Drive cleanup activities and prioritization post project build |  
| `Disaster_Recovery.md` | Putting those backup .tar files to use when needed |
| `E_Extra_Startup_Scripts.md` | A start_network_backend.sh to create |
| `Phase_8_Option_A.md` | Phase 8A option: .csv/.vcf variant file export enable |  
| `Phase_8_Option_B.md` | Phase 8B option: LAN network serving |  
| `Phase_8_Option_C.md` | Phase 8C option: documentation evolution |  

---

## Session Protocol  

### Session Open  

When working with a Brave Leo AI...
At the start of every session, paste the first three core documents in full:  

1. `01_Stack_Build_Steps.md`  
2. `02_Session_Handoff_Log.md`  
3. `03_Stack_Versioning_Reference.md`

And your previous Session Handoff summary.

Paste as a single block. Do not split across messages. Large pastes  
consume context — one paste only, no duplicate handoffs.  

### Session Close  

At the end of every session, all three documents are output in full  
with any updates applied. Output order:  

1. BUILD STEPS  
2. SESSION LOG  
3. VERSIONING  

Never summarize or truncate. Full documents only.  

A Document 4 (full narrative summary — "what happened and why") is  
generated at each of the following triggers:  

- Long-running unattended operation starts  
- Verified restore point created  
- End of a numbered Phase  
- Major version transition  

Document 4 is a standalone narrative. It is not a handoff and is not  
pasted at session open.  

---  

## Update Tags  

Each update to the documents is tagged to indicate which documents  
receive the change:  

| Tag | Documents updated |  
|---|---|  
| `[LOG]` | Session Handoff Log only |  
| `[BUILD]` | Stack Build Steps only |  
| `[VERSION]` | Versioning Reference only |  
| `[BOTH]` | Build Steps + Session Log (excluding Version) |  
| `[ALL]` | All three documents |  
| `[GOTCHA]` | Gotchas section of Build Steps only |  

---  

## Date Tagging Convention  

Session entries are date-tagged using the format `H{YYYYMMDD}` where  
`H` denotes the HealthMed laptop. Example: `H20260808`.  

The prefix letter indicates which system the session originated from:  

- `H` — HealthMed laptop (LAPTOP-F, 192.168.1.240)  
- `P` — Personal laptop (LAPTOP-1, 192.168.1.52)  

The prefix is applied by the user at session time. It is for the  
user's own file-location reference and does not affect document logic.  

When multiple sessions occur on the same date, append a letter suffix:  
`H20260803a`, `H20260803b`, etc.  

---  

## System Naming Convention  

Two machines are referenced throughout this documentation:  

| Working name | Formal name | IP | Role |  
|---|---|---|---|  
| forge | LAPTOP-F (HealthMed Laptop) | 192.168.1.240 | Active build and runtime environment |  
| vault | LAPTOP-1 (Personal Laptop) | 192.168.1.52 | Secure storage and backup environment |  

**forge** is used when discussing file saving, build execution, or  
the live stack running on the HealthMed laptop.  

**vault** is used when discussing production build storage, backup  
locations, or file retrieval from the Personal laptop.  

Both terms are defined in the Glossary (see `README.md`). They are  
working shorthand only — the full machine names and IPs are always  
the authoritative reference.  

---  

## Rules of Engagement  

### During Active Technical Work  

Do not interrupt active work sessions to suggest documentation edits,  
version corrections, or prose polish. Save all such observations for  
the end-of-session summary. Technical execution takes priority during  
active work.  

### Document Integrity  

- Never summarize a section that previously contained full detail.  
- Never truncate a table, config block, or command list.  
- Never merge two distinct entries without an explicit instruction.  
- Preserve all historical entries in the Session Log. Nothing is  
  deleted from the log — superseded information is annotated, not  
  removed.  

### Gotcha Documentation  

Gotchas are documented in `04_GOTCHA_Enumeration.md` with  
phase-numbered format (`1.1`, `2.3`, etc.) and are not duplicated  
inline in the Build Steps. The Build Steps reference gotchas by  
phase number only.  

### Dead Ends  

Dead ends are documented in the Session Log under  
"Dead Ends (Do Not Retry)". Once documented, a dead end is never  
re-attempted without explicit re-evaluation and written rationale.  

---  

## New Session Triggers  

A new chat session should be started rather than continuing the  
current one when any of the following occur:  

1. A long-running unattended operation starts — cut here, resume in  
   new session when the operation completes.  
2. A verified restore point is created.  
3. A numbered Phase ends.  
4. A major version transition occurs.  

At each cut point, generate Document 4 before closing the session.  

---  

## Context Budget Awareness  

Chat context is finite. To preserve it:  

- Paste all three documents once at session open — never twice.  
- Keep Document 4 (narrative summary) as a separate file, not pasted    
  at session open.  
- If context limit is approaching: generate Document 4, output all  
  three updated documents in full, then open a new session.  
- Don't be shy about instructing the AI not to discuss Memory maintenance  
entries or other housekeeping in order to preserve the filling chat buffer.  

---  

## GitHub Repo Scope  

This repository documents the local stack build only.  
It does not contain:  

- Sensitive personal or medical information of any kind  
- Clinical data or genomic data files  
- Credentials, tokens, or secrets  
- Any data files from the build (gru_data, VCF, CRAM, BAM, etc.)  

The `.gitignore` should exclude all data directories and any files  
containing credentials.  

---

≡≡≡≡≡ END: 08_WORKFLOW.md ≡≡≡≡≡  
