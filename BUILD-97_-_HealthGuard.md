# BUILD-97 — HealthGuard

> Source: [https://notion.so/b2a93751fabf43cd9172ba52e931708b](https://notion.so/b2a93751fabf43cd9172ba52e931708b)
> Created: 2026-04-20T14:29:00.000Z | Last edited: 2026-04-20T20:12:00.000Z



---
> **ℹ **Tier 11 · Applications / Bonus · Priority: HIGH****

  Medical diagnostic application for the HealthGuard vertical. Multi-modal patient data fusion, differential diagnosis generation, risk stratification, and clinician-in-the-loop recommendations.

## Purpose

HealthGuard applies NeuroLoom to clinical decision support. Ingests structured EHR data, labs, imaging, vitals, and clinical notes; generates differential diagnoses with evidence chains; stratifies patient risk; surfaces recommendations to clinicians. HIPAA-compliant, fully auditable, clinician-in-loop by default.

## Dependencies

- **BUILD-55 (Omnispectral Intelligence)** — Multi-modal fusion
- **BUILD-29 (Candle ML)** — Inference substrate
- **BUILD-67 (NestNet)** — Specialty-specialized sub-models
- **BUILD-42 (Mahalanobis)** — Anomaly detection (unusual presentations)
- **BUILD-58 (Reality Rupture Detector)** — Catastrophic diagnosis check
- **BUILD-25 (L0 Constitution)** — Safety invariants
## File Structure

```javascript
apps/healthguard/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── ingestion/
│   │   ├── fhir.rs           # HL7 FHIR R4 ingestion
│   │   ├── hl7v2.rs          # HL7 v2 legacy feeds
│   │   ├── dicom.rs          # DICOM imaging
│   │   ├── vitals.rs         # Real-time vitals
│   │   └── notes.rs          # Clinical notes NLP
│   ├── patient/
│   │   ├── record.rs         # Unified patient record
│   │   ├── timeline.rs       # Temporal events
│   │   └── context.rs        # Clinical context
│   ├── diagnosis/
│   │   ├── differential.rs   # DDx generation
│   │   ├── evidence.rs       # Evidence chain
│   │   ├── likelihood.rs     # Bayesian likelihood
│   │   └── specialties/      # NestNet specialist networks
│   │       ├── cardio.rs
│   │       ├── onco.rs
│   │       ├── neuro.rs
│   │       └── infectious.rs
│   ├── risk/
│   │   ├── stratification.rs # Severity scoring
│   │   ├── deterioration.rs  # Early warning scores
│   │   └── readmission.rs
│   ├── recommendations/
│   │   ├── orders.rs         # Suggested orders/tests
│   │   ├── treatment.rs      # Treatment suggestions
│   │   └── followup.rs       # Follow-up scheduling
│   ├── safety/
│   │   ├── contraindication.rs
│   │   ├── drug_interaction.rs
│   │   ├── allergy.rs
│   │   └── black_swan.rs     # Rare-but-deadly checklist
│   ├── compliance/
│   │   ├── hipaa.rs          # HIPAA controls
│   │   ├── phi_deid.rs       # PHI de-identification
│   │   ├── audit.rs          # Clinical audit log
│   │   └── consent.rs        # Consent management
│   ├── ui/
│   │   ├── clinician_view.rs # Clinician interface
│   │   └── evidence_panel.rs # Show reasoning
│   ├── types.rs
│   └── config.rs
└── tests/
```

## Interfaces & Types

```rust
pub struct PatientRecord {
    pub patient_id: String,                // Internal ID, never external PHI
    pub demographics: Demographics,
    pub encounters: Vec<Encounter>,
    pub labs: Vec<LabResult>,
    pub imaging: Vec<ImagingStudy>,
    pub medications: Vec<Medication>,
    pub allergies: Vec<Allergy>,
    pub vitals: Vec<VitalsSnapshot>,
    pub notes: Vec<ClinicalNote>,
    pub problems: Vec<Problem>,
}

pub struct Demographics {
    pub age_years: u32,
    pub sex_at_birth: Sex,
    pub weight_kg: Option<f64>,
    pub height_cm: Option<f64>,
    pub pregnancy_status: Option<PregnancyStatus>,
}

pub struct DifferentialDiagnosis {
    pub patient_id: String,
    pub generated_at: u64,
    pub diagnoses: Vec<RankedDiagnosis>,
    pub unknowns: Vec<MissingData>,
    pub safety_alerts: Vec<SafetyAlert>,
}

pub struct RankedDiagnosis {
    pub icd10_code: String,
    pub name: String,
    pub probability: f64,
    pub likelihood_ratio: f64,
    pub evidence: Vec<Evidence>,
    pub required_workup: Vec<TestRecommendation>,
    pub red_flags: Vec<String>,
}

pub struct Evidence {
    pub finding: String,
    pub source: EvidenceSource,
    pub weight: f64,
    pub confidence: f64,
}

pub enum EvidenceSource {
    Lab { test: String, value: String, reference_range: String },
    Imaging { study: String, finding: String, radiologist_confidence: f64 },
    Symptom { description: String, duration: String },
    History { past_condition: String },
    ExamFinding { finding: String },
    VitalsAnomaly { measure: String, value: f64 },
}

pub struct SafetyAlert {
    pub severity: AlertSeverity,
    pub category: SafetyCategory,
    pub message: String,
    pub action_required: String,
}

pub enum AlertSeverity { Info, Caution, Warning, Critical, Emergency }

pub enum SafetyCategory {
    DrugInteraction,
    ContraindicatedMedication,
    AllergyConflict,
    BlackSwanDiagnosis,
    CriticalLabValue,
    DeteriorationRisk,
}

pub struct RiskScore {
    pub score_name: String,                // e.g., "MEWS", "qSOFA"
    pub value: f64,
    pub interpretation: String,
    pub components: Vec<(String, f64)>,
}

pub struct HealthGuardConfig {
    pub hipaa_mode: bool,                  // true
    pub clinician_in_loop_required: bool,  // true
    pub max_ddx_size: usize,               // 10
    pub min_ddx_probability: f64,          // 0.02
    pub black_swan_always_check: bool,     // true
    pub audit_retention_years: u32,        // 7+
}
```

## Implementation SOP

### Step 1: Data ingestion (`ingestion/`)

- FHIR R4 for modern EHRs (Epic, Cerner)
- HL7 v2 for legacy feeds
- DICOM for imaging (use dcm4che or direct parsing)
- Clinical NLP: extract structured findings from notes
- ⚠️ Never log raw PHI — de-identify before any processing
### Step 2: Patient record (`patient/`)

- Unified longitudinal timeline
- Temporal reasoning: find trends, sudden changes
- Context: ER vs inpatient vs outpatient changes differential
### Step 3: Diagnosis (`diagnosis/`)

- NestNet with specialty specialists (cardio, onco, neuro, infectious, etc.)
- Router selects 2-3 likely specialties
- Each specialist produces ranked DDx with evidence
- Bayesian likelihood from priors (prevalence) + likelihood ratios
- ⚠️ Always include "can't miss" diagnoses even if low probability
### Step 4: Risk stratification (`risk/`)

- Standard scores: MEWS, qSOFA, APACHE II
- ML-augmented deterioration prediction
- Readmission risk for discharge planning
### Step 5: Safety (`safety/`)

- Contraindication: medication-condition conflicts
- Drug-drug interactions (use RxNorm, pharmacist DB)
- Allergies: hard block
- Black swan: explicit checklist of rare-deadly diagnoses (e.g., aortic dissection, meningitis, stroke)
- ⚠️ Safety checks must have zero false negatives for severe categories
### Step 6: Compliance (`compliance/`)

- HIPAA: access controls, audit, encryption at rest + in transit
- PHI de-identification: Safe Harbor + expert determination
- Audit log: every access, query, recommendation
- Consent: respect patient opt-outs
- ⚠️ BAA required with any cloud provider
### Step 7: Clinician UI (`ui/`)

- Recommendations always surface as suggestions, not commands
- Evidence panel: click any diagnosis to see reasoning chain
- Explicit override tracking
- ⚠️ UI must not automate clinical decisions — FDA/regulatory risk
## Gotchas & Warnings

⚠️ Medical errors harm patients — conservative design mandatory

⚠️ FDA regulates AI/ML clinical decision support — check applicable guidance

⚠️ HIPAA violations carry large penalties — rigorous access control + audit

⚠️ Training data bias amplifies health disparities — explicit fairness monitoring

⚠️ LLMs hallucinate diagnoses — always ground in structured evidence

⚠️ Clinicians bear final responsibility — never automate, always suggest

⚠️ Emergency cases may bypass normal workflow — fast-path safety checks

⚠️ International: different regulatory regimes (GDPR, NHS, etc.)

## Testing Requirements

- **Unit:** FHIR parser handles all resource types used
- **Unit:** DDx generator produces valid ICD-10 codes
- **Unit:** Safety checks catch all test contraindications
- **Unit:** PHI de-identification passes Safe Harbor checklist
- **Integration:** End-to-end case studies (100+ de-identified cases)
- **Clinical validation:** Retrospective cohort review by clinicians
- **Bias audit:** Per-demographic performance analysis
- **Security:** Pen-test, HIPAA security audit
- **Regulatory:** FDA Class II SaMD pathway if applicable
## Acceptance Criteria

- [ ] FHIR R4 and HL7 v2 ingestion
- [ ] DICOM imaging integration
- [ ] Clinical notes NLP extracts structured findings
- [ ] Unified patient timeline with temporal reasoning
- [ ] NestNet specialist-based DDx generation
- [ ] Bayesian likelihood with evidence chains
- [ ] Standard risk scores (MEWS, qSOFA, etc.)
- [ ] ML-augmented deterioration prediction
- [ ] Comprehensive safety checks with zero-false-negative goals for severe categories
- [ ] Black swan checklist for can't-miss diagnoses
- [ ] HIPAA controls, PHI de-identification, audit log
- [ ] Clinician-in-loop UI with evidence transparency
- [ ] Retrospective validation by clinicians
- [ ] Bias/fairness audit
- [ ] All tests pass

