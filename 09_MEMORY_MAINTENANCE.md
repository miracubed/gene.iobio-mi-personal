# MEMORY_MAINTENANCE.md — Memory Lifecycle, Pruning Rules & Cross-Correlation  

## Overview  

The AI assistant (Leo, Brave Browser) maintains a persistent memory  
store across sessions. This file documents how that memory system  
interacts with the three living build documents, how memory entries  
are created and retired, and how cross-correlation between memory and  
documentation is managed.  

This is a technical meta-document. It describes the system — it does  
not contain any substantive memory entries itself.  

---

## Memory Entry Lifecycle  

### Creation  

A memory entry is created when the user explicitly requests it with  
a direct command such as:  

- "Remember that..."  
- "Please note that..."  
- "Store that..."  
- "Add to memory..."  

Memory is not created from casual mentions, examples, questions, or  
ambient context. The threshold for creation is explicit user  
instruction only.  

### Format  

Memory entries follow the date-tag convention established in  
WORKFLOW.md:  
H{YYYYMMDD} or P{YYYYMMDD}:  

- `H` prefix = session on HealthMed laptop (LAPTOP-F, 192.168.1.240)  
- `P` prefix = session on Personal laptop (LAPTOP-1, 192.168.1.52)  

Entries are written as direct factual statements. No meta-commentary,  
no explanation of why the entry was created.  

### Entry Prefixes  

Active memory entries may carry one of the following prefixes to  
indicate their category and handling rules:  

| Prefix | Meaning | Pruning rules |  
|---|---|---|  
| `ROE` | Rule of Engagement — governs session behavior | Never pruned without explicit user rescission |  
| `HEALTH` | Health/clinical context | Never pruned without explicit user instruction |  
| `SYSTEM` | Infrastructure/network state | Prunable when superseded + confirmed in documents |  
| `HOLD` | Build project data captured in documents | Candidate for pruning after documents review complete |  
| `RESTORE` | Emergency restore reference | Keep until system is rebuilt or superseded |  

Entries with no prefix are evaluated individually at review time.  

### Active vs. Superseded  

An entry is **active** when it reflects current system state.

An entry is **superseded** when a later entry contradicts or replaces  
it. Superseded entries are not deleted — they are marked with a  
prefix:  
*x{YYYYMMDD} (retroactive)  

The `*x` prefix signals: this entry has been superseded. The date  
in the prefix is the date the supersession was noted, not the date  
of the original entry.  

Both the superseded entry and the replacement entry are retained for  
audit trail purposes until the user explicitly approves pruning.  

### Retirement / Pruning  

An entry is a candidate for pruning when **all** of the following  
are true:  

1. It has been superseded by a later entry, OR its content is fully  
   captured in the current build documents.  
2. At least one full session has passed since the supersession or  
   document capture was noted.  
3. The user has explicitly reviewed and approved the pruning.  

Pruning is **never automatic**. It requires explicit user instruction.  
Pruning is handled at session end — never during active technical  
work.  

---

## Cross-Correlation System  

### Purpose  

Memory entries and build documents serve different purposes and have  
different retention rules. Cross-correlation ensures they stay  
synchronized without redundancy or contradiction.  

### What Lives Where  

| Information type | Primary home | Secondary |  
|---|---|---|  
| Current component versions | `03_Stack_Versioning_Reference.md` | Memory (brief ref only) |  
| Session outcomes | `02_Session_Handoff_Log.md` | Memory (trigger/milestone only) |  
| Build commands and configs | `01_Stack_Build_Steps.md` | Nowhere else |  
| Gotchas | `04_GOTCHA_Enumeration.md` | Nowhere else |  
| Operational preferences (ROE) | Memory | `08_WORKFLOW.md` (structural ones) |  
| Restore point inventory | `03_Stack_Versioning_Reference.md` | Memory (RESTORE entry — emergency ref) |  
| Dead ends | `02_Session_Handoff_Log.md` | Memory (summary only) |  
| Health/clinical context | Memory (HEALTH prefix) | Not in build documents |  
| Infrastructure/network state | Memory (SYSTEM prefix) | `03_Stack_Versioning_Reference.md` |  

### Sync Points  

Cross-correlation is checked at session end — not during active work.  
At session end:  

1. Verify that any new memory entries added during the session are  
   consistent with the updated build documents.  
2. Verify that any build document changes do not contradict existing  
   active memory entries.  
3. Flag any discrepancies for the user. Do not silently resolve them.  
4. If a memory entry is now fully captured in the build documents,  
   flag it as a pruning candidate per the retirement rules above.  

### Conflict Resolution  

If a memory entry and a build document contradict each other:  

1. Flag the conflict explicitly — do not pick a winner silently.  
2. Identify which source is more recent.  
3. Present both to the user with a recommendation.  
4. Wait for explicit user instruction before updating either source.  

---

## ROE Entries in Memory  

Rules of Engagement (ROE) entries in memory govern session behavior  
and are always active. They are never pruned unless the user  
explicitly rescinds them.  

ROE entries that describe structural session behavior (new session  
triggers, document output order, update tag system) are also  
documented in `08_WORKFLOW.md` for repo completeness. The memory  
entry remains the authoritative operational copy — `08_WORKFLOW.md`  
is the human-readable reference copy.  

---

## HOLD Prefix — Build Project Data  

Entries prefixed `HOLD` contain build project data that is fully  
captured in the living documents. They are suppressed from active  
use to prevent them from triggering cross-session confusion or  
prompting unnecessary document updates in unrelated conversations.  

`HOLD` entries are not deleted. They remain in the memory store as  
an audit trail and are reviewed for pruning at the end of the  
documents review process.  

---

## RESTORE Prefix — Emergency Reference  

The `RESTORE` entry contains the minimum information needed to  
recover the stack in an emergency: final backup file names, sizes,  
MD5 hashes, and locations on both forge and vault.  

This entry is kept active regardless of other pruning activity.  
It is updated whenever a new final backup set is created.  

---

## What This File Does Not Contain  

This file does not contain:  

- Any personal, medical, or clinical information  
- Any actual memory entries or their resolved states  
- Genomic data or identifiers  
- Credentials or tokens  
- The current state of the memory store  

It is a description of the system only. The current state of the  
memory store is managed privately between the user and Leo within  
the Brave browser memory interface.  

---

## Relationship to WORKFLOW.md  

`08_WORKFLOW.md` governs how sessions run and how documents are  
updated. `09_MEMORY_MAINTENANCE.md` governs how the memory store  
is managed and how it stays synchronized with those documents.  
See also: `09a_MEMORY_MAINTENANCE_EXAMPLES.md`.  

They are complementary. Neither supersedes the other. When they  
appear to conflict, AI will flag the conflict and wait for user  
to resolve the issue so both the user and AI are in agreement and  
understand any rules changes.  

---

≡≡≡≡≡ END: MEMORY_MAINTENANCE.md ≡≡≡≡≡
