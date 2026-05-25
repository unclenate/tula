# Problem Statement

What problem Tula solves, for whom, and why now. Required by
`management/product-lite`.

**Owner:** @unclenate (RealActivity) | **Last Updated:** 2026-05-25

---

## The problem

Personal health data is **fragmented and inaccessible to the person it
describes**. It lives in patient-portal screens, printed lab reports,
prescription bottles, hospital whiteboards, insurance mailings, discharge
instructions, imaging reports, and handwritten provider notes — most of it
never reaching a structured system the patient can use. Patients and caregivers
carry it in folders, photograph it for their own records, or lose track of it.

Existing AI health tools make this worse on two axes:

- **They take custody of the data.** Most are closed platforms that require
  handing health information to another vendor's cloud.
- **They are built for high-income, English-speaking, well-connected users**,
  excluding the populations with the greatest disease burden and least access.

The result: people cannot easily organize, understand, or act on their own
health information, and the cognitive and logistical burden falls hardest on
patients with complex illness, their caregivers, and communities in
low-resource settings.

## Who it is for

- **Patients navigating complex or serious illness** — high data volume across
  many providers.
- **Caregivers** — coordinating medications, appointments, symptoms, and
  insurance without clinical training.
- **People managing chronic conditions or hereditary risk** — longitudinal
  monitoring and trend detection.
- **Community health workers in low-resource settings** — a shared, low-bandwidth,
  no-license instance accessible from a basic smartphone.

See [`use-cases.md`](../use-cases.md) for the full set.

## Why now

- **Agent-native software is viable.** A general-purpose agent runtime
  (OpenClaw) plus frontier and purpose-built healthcare models can turn a phone
  camera and an email inbox into a universal health-data connector — no portal
  integration, no FHIR negotiation, no IT department.
- **Patient-access mandates are expanding** (US ONC / Cures Act info-blocking
  rules; the EU European Health Data Space), making patient-authorized access
  to one's own records a first-class, standards-backed path.
- **Open-weight healthcare models** (MedGemma, MedASR) now run on modest
  hardware, making a private, self-hosted, no-API-fee deployment realistic for
  global-health-equity contexts.

## The shape of the solution

A **self-hosted personal health agent**: the user forwards or photographs
health documents to a private agent that extracts structured FHIR data, organizes
it longitudinally, drafts (never sends) patient-provider communications, and
surfaces changes — all on infrastructure the user controls, with the data never
leaving their environment.

## What this is explicitly NOT

Tula is not a medical device, a diagnostic system, a treatment-recommendation
engine, an emergency-response system, or a replacement for a clinician, an EHR,
or a patient portal. It supports health literacy, organization, and
patient-provider communication. The hospital-scale, multi-tenant platform is a
separate product (Aria); see [`OPEN_CORE.md`](../../OPEN_CORE.md).
