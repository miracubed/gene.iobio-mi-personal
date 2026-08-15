# Maintainer's 06_MEDICAL_CONTEXT.md
## Overview

This document provides the minimum clinical context needed to
understand why this stack was built and why certain architectural
decisions were made. It is intentionally high-level.

This repository is a public technical resource. This document does
not contain variant coordinates, clinical history details,
medication information, or any information that would constitute
a personal medical disclosure.

---

## Why This Stack Exists

This stack was built to support offline, self-directed analysis of
personal whole-genome sequencing (WGS) data in the context of a
confirmed rare metabolic condition.

The primary use case is:

- Loading and viewing personal WGS data (CRAM/VCF format) locally
- Annotating variants against reference databases (gnomAD, ClinVar,
  VEP) without sending data to external servers
- Filtering and exporting variants of interest for clinical review
- Supporting phenotype-driven gene analysis via Phenolyzer

**The data never leaves the local network.** This is a deliberate
architectural requirement, not a convenience choice. All annotation,
variant calling, and export happens on the local machine or LAN.

---

## The Rare Disease Context

The maintainer has a confirmed diagnosis of a rare autosomal
recessive metabolic disorder affecting amino acid transport. This
condition is:

- Underdiagnosed and frequently mismanaged in clinical settings
- Associated with multi-system complications requiring ongoing
  monitoring
- The subject of active specialist involvement

Whole-genome sequencing was pursued independently to support
clinical decision-making and specialist consultation. The stack
exists to make that data interpretable and accessible in a
controlled, private environment.

---

## Why Local Deployment Matters

Commercial genomic analysis platforms typically require data upload
to third-party servers. For a user with a rare confirmed diagnosis
and complex multi-system findings, uploading raw WGS data to
external platforms carries privacy and data-sovereignty risks that
are unacceptable.

Local deployment of gene.iobio solves this by providing the same
annotation and visualization capabilities as the hosted version,
with all data remaining on hardware under the user's direct control.

The maintainer also lives in a semi-rural area where mobile cell
and data services can be unpredictably limited, even in more populated areas. Due
to personal experience of no mobile coverage in these areas, the
maintainer wanted a standalone server to take to doctor's office
visits and hospitals as needed for needed personal medical documents
and information. Since her WGS is a big portion of her current needs,
she decided to come up with this build solution to meet those needs.
She is sharing with others in an effort to help those in similar
situations.

---

## Impact on Build Decisions

The following architectural choices were directly driven by the
medical context:

| Decision | Reason |
|---|---|
| Full local stack (not hosted gene.iobio) | WGS data privacy — data never leaves local network |
| gru_data full dataset (~128GiB) | Complete annotation coverage required for rare disease variant interpretation |
| GRCh37 as primary reference build | Source WGS data was called against GRCh37 |
| Phenolyzer integration | Phenotype-driven gene prioritization is essential for rare disease workflows |
| CSV/VCF export (Phase 8A) | Variant lists must be exportable for clinical review and specialist communication |
| LAN serving (Phase 8B) | Data needs to be accessible from a second machine on the same network without external exposure |
| Offline-first architecture | No dependency on internet connectivity for core analysis workflows |

---

## For Contributors and Forks

If you are forking or adapting this stack for your own use:

- The core build is general-purpose. Nothing in the stack is
  specific to the maintainer's diagnosis.
- The gru_data dataset, VEP cache, and gnomAD annotations support
  any human WGS/WES analysis workflow.
- The accessibility considerations documented in
  `05_ACCESSIBILITY_NOTES.md` are specific to the primary
  maintainer and do not need to carry forward to forks.
- The export and LAN serving features (Phase 8A and 8B) are broadly
  useful and not rare-disease-specific.

This stack was built by one person, for one specific use case, under
significant resource constraints. It is shared in the hope that it
saves someone else the months of trial and error that went into it.

---

## What This Document Does Not Contain

This document does not contain:

- The name of the specific diagnosis
- Variant coordinates or identifiers
- Gene names involved in the diagnosis
- Clinical history, medication, or treatment information
- Any information that would identify the maintainer

For accessibility and working-style considerations, see
`05_ACCESSIBILITY_NOTES.md`.

---

≡≡≡≡≡ END: 05_MEDICAL_CONTEXT.md ≡≡≡≡≡
