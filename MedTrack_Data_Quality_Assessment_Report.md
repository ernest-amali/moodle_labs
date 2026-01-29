# Data Quality Assessment Report for MedTrack Ghana
Dataset Sample: 

PatientID, PatientName, PhoneNumber, AppointmentDate, DoctorName, PaymentStatus P001, Kwame Mensah, 0244123456, 2025-10-15, Dr. Osei, Paid P002, Ama Serwa, 244789012, 15/10/2025, dr. osei, paid P001, Kwame Mensah, 0244123456, 2025-10-20, Dr. Adjei, Pending P003, Kofi Annan, 0244567890, 10/16/2025, Dr. Osei, Failed P004,, 0555234567, 2025-10-17, Dr. Mensah, Paid P002, Ama Serwa, 0244789012, 2025-10-15, Dr. Osei, Paid


## Executive Summary

This assessment identifies critical data quality failures across all six standard dimensions in MedTrack London's appointment database. Three issues are causing immediate operational damage: an invalid date (10/16/2025) that systems cannot parse, a malformed phone number (244789012) missing the required format, and duplicate patient records (P001 and P002). These defects directly cause the reported problems: reminder SMS failures, incorrect appointment report counts and generation, and billing processes failures. Immediate fixes include correcting the invalid date and standardizing phone number formats, removing duplicate records via running deduplication script by Database Administrator and Implementing validation rules and unique constraints to prevent recurrence and remove financial and reporting risks.

## Dataset overview
The provided sample rows show the following notable issues used in this assessment:
- Patient P003: Appointment Date `10/16/2025` (impossible under DD/MM/YYYY dataset policy)
- Patient P002: duplicate entries, and one entry has Phone Number `244789012`
- Patient P001: duplicate entries as well
- Patient P004: missing Patient Name; phone `0555234567` present
- Appointment dates use mixed formats: `2025-10-15`, `15/10/2025`, and `10/16/2025`
- Doctor names and Payment Status show inconsistent casing (`Dr. Osei` vs `dr. osei`, `Paid` vs `paid`)

## Findings and Operational Impact

### Accuracy
- Example: Appointment Date `10/16/2025`. Interpreting the dataset’s dominant format as DD/MM/YYYY yields month = 16, which is impossible and therefore factually incorrect.
- Operational impact: SMS scheduler and billing pipelines that parse dates can fail, skip the record, or produce incorrect reports causing missed reminders. Also, billing cycles referencing appointment dates would fail to process Patient P003's payment.
- Primary impacted function: Operations

### Completeness
- Example: Patient P004 has no Patient Name.
- Operational impact: Missing identity prevents reliable personalized communications, blocks automated invoicing and demographic reporting, and forces manual resolution at registration or front desk.
- Primary impacted function: Operations

### Consistency
- Examples: Mixed appointment date representations, inconsistent doctor name casing, and inconsistent payment status casing.
- Operational impact: SMS generation based on filtering Payment Status = “Paid” fails for this “paid” status. Report generation per doctor name might exclude the “dr. osei" appointment, distorting the workload distribution analysis. Billing queries filtering Payment Status = “Paid” will incorrectly flag patient P002 as unpaid.  
- Primary impacted function: Operations

### Timeliness
- Example: All sample appointment dates are in October 2025 (in the past and outdated).
- Operational impact: Reminder jobs and “upcoming appointments” dashboards will be inaccurate if historical records are not separated; billing windows may be misapplied.
- Primary impacted function: Finance

### Validity
- Example: Phone number `244789012` violates the expected local Ghanaian 10-digit phone number pattern starting with zero.
- Operational impact: SMS gateway rejects malformed numbers; messages are not delivered, directly causing communication failures. Payment confirmation and billing receipts would fail to deliver, leading to payment disputes.
- Primary impacted function: Operations

### Uniqueness
- Example: Patient P002 appears more than once for the same appointment date and provider; P001 appears multiple times for different dates (which may be valid if multiple appointments exist).
- Operational impact: Duplicate records inflate patient counts, cause duplicate reminders, and risk duplicate invoices or billing doubly.
- Primary impacted function: Finance

## Root cause synthesis
The root causes are:
- Lack of canonical data format policy at ingestion (dates, phones, provider identifiers)
- Missing field-level validation and normalization
- Absence of uniqueness constraints and deduplication during ETL
These gaps permit malformed, inconsistent, and duplicated records into production where downstream automation assumes canonical types.

## Recommended remediation, responsibilities, and verification


### Critical technical fixes 
1. Date normalization and validation
   - Implement an ETL normalization script that treats slash-separated dates as DD/MM/YYYY per dataset policy, parses ISO strings (YYYY-MM-DD), and quarantines impossible values (e.g., month 16).
   - Store appointment dates in a typed DATE column (ISO 8601 or dataset policy) and enforce schema validation at ingest.
   - Owner: Backend Developer
   - Verification: Attempt to enter a non-existent date (e.g., 13/25/2025 or 32/10/2025) in the booking form – system should reject it.

2. Phone normalization and validation
   - Use a phone-parsing library (libphonenumber or equivalent) to normalize to a canonical format or implement a phone number format validation requiring exactly 10 digits starting with "0"
   - Owner: Backend Developer
   - Verification: Attempt to enter 9-digit phone number in booking form – system should reject it. 

3. Uniqueness enforcement and deduplication
   - Define a canonical uniqueness key, for example PatientID + appointment_date_parsed + canonical doctor identifier.
   - Run deterministic deduplication that keeps the most complete or most recent record; flag near-duplicates for human review.
   - Owner: Database Administrator
   - Verification: Zero groups with COUNT > 1 for the canonical key; billing exports produce a single invoice per canonical appointment.


### Monitoring and acceptance criteria
- Zero non-quarantined records with appointment_date_parsed null.
- Zero non-quarantined records with phone_normalized null.
- Zero duplicate groups on canonical uniqueness key.
- SMS scheduler test run produces no parse errors and gateway logs indicate successful enqueue for test messages.
- Operations and Finance sign off on reconciliation reports and billing exports.

## Prioritized action plan for MedTrack London
- Phase 1 (immediate): Quarantine P004 and contact patient to populate missing name. Owner: Operations.
- Phase 2 (same day, parallel): Run date and phone normalization scripts and produce quarantine lists. Owner: Data Engineering.
- Phase 3 (next day): Deduplicate using canonical keys and validate billing exports. Owner: Data Engineering and Finance.
- Phase 4 (short term): Deploy ingest validation, monitoring metrics (date parse failures, invalid phones, duplicate rate), and runbooks for remediation. Owners: Platform, DevOps, Data Steward.

## Conclusion
Remediating the impossible date, invalid phone format, and duplicate records will remove the primary causes of SMS delivery failures, incorrect reports, and billing issues. After normalization, validation, and deduplication are in place, run the verification checks and obtain sign-off from Operations and Finance before returning the dataset to production pipelines.

Prepared by: Junior Developer

