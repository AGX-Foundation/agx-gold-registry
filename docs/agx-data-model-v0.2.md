**AGX Open Compliance Registry**
**Data Model (v0.2) - Public Documentation**

---

### Overview

The AGX Open Compliance Registry is a foundational infrastructure component of the AGX Protocol, designed to bring clarity, accountability, and interoperability to Africa’s gold trade. As it launches in Ghana and Guinea—two countries with active artisanal and small-scale mining sectors—the Registry enables centralized verification of decentralized supply chain actors while retaining transparency and auditability.

The core design separates general identity and system functions from specialized role-based data models to ensure extensibility, legal interoperability, and ease of API integration. These models support live verification, credential tracking, and field-compatible workflows for every actor type involved in the gold trade.

This document outlines the data models used within the AGX Open Compliance Registry v0.2. The architecture separates the general application data model from specialized role-based models for key gold supply chain participants. This design supports extensibility, traceability, and role-specific workflows.

---

## 📊 Recommended Additions

### Entity Relationship Diagram (ERD)

A visual ERD would clearly show how tables like `actors`, `certifications`, `audits`, and role-specific tables relate to each other. This helps contributors and auditors understand the system architecture at a glance. A typical diagram includes foreign key lines and cardinality indicators.

### API Endpoint Overview

A high-level list of endpoints can support implementers and integrators:

* `GET /actors/:id`
* `POST /certifications`
* `GET /exporters?country=GH`
* `PUT /audits/:id`

These should map directly to the data model fields and include authentication/permission notes.

### Permissions Matrix

Clarify which system roles (`admin`, `auditor`, `external_viewer`) can view, modify, or certify records. Helps ensure internal security and external trust.

| Action                     | Admin | Auditor | External Viewer |
| -------------------------- | ----- | ------- | --------------- |
| View Actors                | ✅     | ✅       | ✅               |
| Create/Edit Certifications | ✅     | ✅       | 🚫              |
| Submit Audit Results       | ✅     | ✅       | 🚫              |
| Upload/View Documents      | ✅     | ✅       | 🚫              |

### Localization Guidance

For Ghana and Guinea, define:

* Regional naming conventions (e.g., prefectures, districts)
* License ID formats
* Language support for field agents (French, Twi, Malinké, etc.)

### Deployment Notes

Suggest setups such as:

* Supabase + Vercel for fast startup
* PostgreSQL with Hasura for API auto-generation
* Role-based auth via Clerk or Firebase

## 🔁 1. General Application Data Model

### `actors`

**What is an Actor?**
In the AGX data model, an **Actor** is any registered legal or operational entity participating in the gold supply chain. This includes individuals (e.g., field agents, assayers) and organizations (e.g., exporters, aggregators, vault operators). Each actor has a unique identity in the system and may take on one or more roles. These roles define what they are allowed to do, how they are verified, and what compliance rules they must follow.

By treating actors as a universal entity type with modular roles and metadata, the system ensures flexibility across jurisdictions, consistency in audits, and a single source of truth for verification.

Stores core identity details for all registered entities.

| Column                | Type      | Description                               |
| --------------------- | --------- | ----------------------------------------- |
| `id`                  | UUID      | Unique identifier                         |
| `legal_name`          | TEXT      | Registered business name                  |
| `registration_number` | TEXT      | Official national/company registration ID |
| `country_code`        | TEXT      | ISO 3166-1 code                           |
| `status`              | TEXT      | Enum: `active`, `suspended`, `archived`   |
| `created_at`          | TIMESTAMP | Timestamp of record creation              |

---

### `actor_roles`

**What are Actor Roles?**
In AGX, a single actor may perform multiple supply chain functions—such as being both an exporter and a vault operator. The `actor_roles` table maps these role identities to the core `actors` table. Each entry represents a verified capacity in which the actor operates. Roles can be independently certified and audited. This design enables composable compliance and supports real-time verification of an actor’s functional status in the supply chain.

Allows each actor to assume one or more certified supply chain roles.

| Column        | Type      | Description                     |
| ------------- | --------- | ------------------------------- |
| `id`          | UUID      | Unique role ID                  |
| `actor_id`    | UUID      | FK → `actors.id`                |
| `role_type`   | TEXT      | exporter, trader, assayer, etc. |
| `verified_at` | TIMESTAMP | When role was approved          |
| `verified_by` | UUID      | FK → `users.id`                 |

---

### `certifications`

**What is a Certification?**
Certifications validate that a specific actor-role pairing has met the necessary compliance requirements—whether AGX-issued, government-issued, or third-party validated. Certifications are time-bound and can carry various statuses (e.g. verified, expired, revoked). They serve as official credentials and may be linked to field audits, uploaded documents, or partner endorsement systems. This structure supports transparent trade, audit trails, and international acceptance of certified actors.

Track role-specific certification events.

| Column        | Type | Description                              |
| ------------- | ---- | ---------------------------------------- |
| `id`          | UUID | Primary key                              |
| `actor_id`    | UUID | FK → `actors.id`                         |
| `role_type`   | TEXT | Which role this applies to               |
| `cert_type`   | TEXT | Type of certification: AGX, gov, partner |
| `cert_status` | TEXT | verified, revoked, expired, provisional  |
| `issued_at`   | DATE | Certification issuance date              |
| `expires_at`  | DATE | Optional expiry                          |
| `issuer_id`   | UUID | FK → `users.id`                          |

---

### `audits`

**What is an Audit?**
An audit in AGX represents a formal compliance check—either remote or field-based—performed on a specific actor-role. Audits may involve document review, site visits, interviews, or technical validation. Outcomes are logged for risk assessment and certification status updates. Audit history builds the actor’s risk score and traceable reputation, helping buyers, donors, and regulators evaluate who to trust.

Track field inspections and compliance reviews.

| Column        | Type | Description                      |
| ------------- | ---- | -------------------------------- |
| `id`          | UUID | Primary key                      |
| `actor_id`    | UUID | FK → `actors.id`                 |
| `role_type`   | TEXT | Role being audited               |
| `audit_type`  | TEXT | Field visit, doc review, etc.    |
| `outcome`     | TEXT | pass, fail, flagged, conditional |
| `notes`       | TEXT | Freeform audit comments          |
| `audit_date`  | DATE | When the audit occurred          |
| `verifier_id` | UUID | FK → `users.id`                  |

---

### `documents`

**What is a Document in AGX?**
The documents module supports the uploading, indexing, and verification of official records (e.g. licenses, ID cards, inspection forms). Each document is tied to a registered actor and includes metadata such as upload time, document type, and verification status. Verified documents are used as evidence in audits and certification, and can be referenced in mobile field tools for low-connectivity enforcement.

Stores file metadata (e.g., permits, IDs).

| Column        | Type      | Description                               |
| ------------- | --------- | ----------------------------------------- |
| `id`          | UUID      | Primary key                               |
| `actor_id`    | UUID      | FK → `actors.id`                          |
| `file_url`    | TEXT      | Secure link to file storage               |
| `doc_type`    | TEXT      | Enum: ID, business license, export permit |
| `uploaded_at` | TIMESTAMP | Timestamp of upload                       |
| `uploaded_by` | UUID      | FK → `users.id`                           |
| `verified`    | BOOLEAN   | Whether document has been checked         |

---

### `users`

**Who are Users?**
Users are the individuals authorized to access, administer, and verify entries in the AGX Registry. This includes AGX administrators, government partners, auditors, and external verifiers. Each user is assigned a role and email, and may be linked to audits, certifications, or document reviews. This user management layer ensures role-based security, accountability, and data integrity across the platform.

System actors managing registry workflows.

| Column  | Type | Description                      |
| ------- | ---- | -------------------------------- |
| `id`    | UUID | Primary key                      |
| `name`  | TEXT | Full name                        |
| `role`  | TEXT | admin, auditor, external\_viewer |
| `email` | TEXT | Email address                    |

---

## 📦 2. Specialized Role-Based Data Models

---

## 🔒 Data Privacy, Localization, and Accessibility Notes

### Data Privacy

AGX ensures that sensitive data (e.g., biometric records, trade volumes, inspection findings) is stored securely and only accessible to authorized roles defined in the permissions matrix. Personally identifiable information (PII) is never made public. Where required by law or regulatory agreement, data storage may be geo-restricted (e.g., hosted in Ghana or Guinea).

### Language and Literacy Support

Field tools and registry interfaces are designed with multilingual support for English, French, Twi, and Malinké. This supports accessibility for field agents and rural users in Ghana and Guinea. Text interfaces are paired with icons, pictograms, and optional audio guidance in the mobile versions.

### Field Accessibility

To ensure the registry operates effectively in low-connectivity environments, all field apps and mobile verification tools include:

* Offline sync support with local caching
* QR code-based verification of TradePass and certification
* Lightweight interface layers for low-literacy users

These accessibility features enable trusted registry use in informal markets, remote gold fields, and rural supply chain hubs.

---

Each role in the AGX ecosystem plays a distinct and verifiable function within the gold supply chain. The following contextual summaries clarify the technical and operational rationale behind each specialized data model.

### What is a Gold Trader?

Gold traders act as intermediaries who purchase gold from upstream sources—often aggregators or miners—and resell it into downstream markets. In West Africa, this function is often informal, but AGX formalizes and verifies it through licensing, transaction logging, and partner visibility. Trader records must include counterparties, licensed brokerage status, and operational regions to allow compliance scoring and traceability.

### What is an Exporter?

Exporters are legally authorized entities that ship gold internationally. They must hold valid government permits, maintain customs compliance, and interface with vault and assay infrastructure. Exporter records on AGX include export volume limits, TradePass ID (a global identity passport for traceable gold), and customs partner affiliations. Ghana and Guinea both require tight oversight of exporters, making this a critical verification point.

### What is an Aggregator?

Aggregators serve as localized collection points for small-scale miners. They receive, document, and transport gold to central processing or vaulting locations. AGX requires aggregators to register the number of miners they source from, document transport methods, and define geographic scope. This helps close compliance gaps at the point where informal supply enters formal trade.

### What is an Assayer?

Assayers test the quality and purity of gold. AGX ensures that only certified, technically equipped labs with SOPs and calibration records are authorized. Given the significance of accurate grading to pricing and trust, assayers must also document their ISO or national certification. In Ghana and Guinea, AGX interfaces with approved assay labs to sync verification outputs into the Registry.

### What is a Vault Operator?

Vault operators offer secured physical storage for gold prior to export or domestic sale. Their infrastructure is vital to AGX chain-of-custody protocols. Vault operators must register their storage capacity, security procedures (human and technical), and exact GPS location. Their compliance ensures custody risk is minimized.

### What is an Inspector?

Inspectors are trained individuals or institutional agents who perform field audits, site visits, or document reviews. AGX logs their assigned zones, inspection methods, and affiliated regulatory or donor agencies. Inspectors enable real-time, field-level enforcement and are especially critical in remote or high-risk collection areas.

### What is a Field Agent?

Field agents serve as AGX’s on-the-ground representatives. They assist in onboarding, verifying transactions, collecting documentation, and syncing mobile compliance data. AGX equips field agents with mobile ID, geofenced task zones, and localized language preferences. Their logs help bridge low-literacy, low-connectivity environments in Ghana and Guinea, enabling real-time Registry updates from rural sites.

The following are explicit data models for each specialized role in the AGX registry. These can be implemented as separate tables or modularized as extended schemas depending on your backend architecture.

### A. Gold Traders (`gold_traders`)

**Who are Gold Traders?**
Gold traders are commercial intermediaries who buy and resell gold within national or cross-border markets. In West Africa, traders play a central role in aggregating supply from artisanal sources and connecting it to exporters or refineries. AGX tracks their trading volume, counterparties, operational regions, and licensing status to evaluate their legitimacy, risk level, and role in formalized trade routes.

* **Monthly Trade Volume (kg):** A self-reported or verified figure reflecting how much gold a trader handles on average per month. Used for tiering and capacity assessment.
* **Typical Counterparties:** A list of regular buyers or sellers the trader deals with. Helps identify supply chain networks and detect anomalous transactions.
* **Country or Region of Operation:** Indicates where the trader is actively sourcing or selling gold. This supports localized regulatory enforcement.
* **Licensed Brokerage Status:** Captures whether the trader is officially licensed, provisionally registered, or operating informally. Used for compliance scoring and certification pathways.
  Gold traders are commercial intermediaries who buy and resell gold within national or cross-border markets. In West Africa, traders play a central role in aggregating supply from artisanal sources and connecting it to exporters or refineries. AGX tracks their trading volume, counterparties, operational regions, and licensing status to evaluate their legitimacy, risk level, and role in formalized trade routes.

| Column                    | Type    | Description                               |
| ------------------------- | ------- | ----------------------------------------- |
| `id`                      | UUID    | Primary key                               |
| `actor_id`                | UUID    | FK → `actors.id`                          |
| `monthly_trade_volume_kg` | DECIMAL | Self-reported or verified monthly average |
| `brokerage_status`        | TEXT    | licensed / provisional / unlicensed       |
| `main_counterparties`     | TEXT\[] | Known buyer/seller relationships          |
| `regions_active`          | TEXT\[] | Countries or provinces traded in          |

### B. Exporters (`exporters`)

**Who are Exporters?**
Exporters are legally authorized entities that ship gold internationally. They act as the final link between local aggregation and international sale, and their compliance is critical to AGX’s verified export model. Exporters must register government-issued permits, disclose customs partners, and operate within permitted volume limits. Their records form the bridge between field-verified gold and the TradePass™ system.

* **Export Permit Reference:** Official license ID or document number issued by the relevant mining or trade authority.
* **Monthly Tonnage Limits (kg):** The capped amount of gold they are legally authorized to export monthly.
* **AGX TradePass™ ID:** A persistent digital identity that links this exporter to verified trades.
* **Customs Clearance Partner:** The logistics or customs agent that processes and clears the gold through official ports.

| Column                    | Type    | Description                             |
| ------------------------- | ------- | --------------------------------------- |
| `id`                      | UUID    | Primary key                             |
| `actor_id`                | UUID    | FK → `actors.id`                        |
| `export_permit_ref`       | TEXT    | Government-issued export license ID     |
| `monthly_export_limit_kg` | DECIMAL | Maximum allowed quantity                |
| `customs_partner`         | TEXT    | Associated customs clearing agent       |
| `tradepass_id`            | TEXT    | Link to TradePass™ or exporter passport |

### C. Aggregators (`aggregators`)

**Who are Aggregators?**
Aggregators are operational agents who gather gold from small-scale miners before transporting it to assay labs, vaults, or exporters. They play a critical compliance role at the interface between informal mining and formal trade. AGX tracks their source network, transport methods, and capacity to ensure verified collection and prevent leakage into illicit channels.

* **Linked Miners Count:** The number of miners who directly supply to this aggregator, either officially registered or mapped through field agents.
* **Collection Region:** Geographic zones where aggregation occurs. Supports risk zoning and certification audits.
* **Transport Method:** Means of physical transfer—motorbike, vehicle, etc.—used to move gold from source to checkpoint.
* **Storage Capacity (kg):** Onsite or local holding capability, which impacts risk and insurance protocols.

| Column                | Type    | Description                         |
| --------------------- | ------- | ----------------------------------- |
| `id`                  | UUID    | Primary key                         |
| `actor_id`            | UUID    | FK → `actors.id`                    |
| `linked_miners_count` | INTEGER | Number of miners directly supplying |
| `collection_region`   | TEXT    | Primary geographic sourcing area    |
| `transport_method`    | TEXT    | Truck, motorbike, on-foot, etc.     |
| `storage_capacity_kg` | DECIMAL | Local storage capacity in kilograms |

### D. Assayers (`assayers`)

**Who are Assayers?**
Assayers analyze the composition and purity of gold. Their measurements form the basis for pricing, taxation, and compliance. AGX registers technical infrastructure, procedural quality, and certification validity to ensure accuracy and reduce disputes.

* **Lab Equipment List:** Machines and analytical tools used to perform assays (e.g., XRF, fire assay).
* **Standard Operating Procedures (SOPs):** Documented workflows for testing methodology and safety.
* **ISO Certification or Accreditation Number:** External validation of lab standards via recognized national or global bodies.
* **Last Calibration Date:** Most recent technical maintenance check to ensure instrument accuracy.

| Column                     | Type    | Description                    |
| -------------------------- | ------- | ------------------------------ |
| `id`                       | UUID    | Primary key                    |
| `actor_id`                 | UUID    | FK → `actors.id`               |
| `lab_equipment`            | TEXT\[] | List of lab tools and machines |
| `standard_procedures`      | TEXT    | Documented SOP summary         |
| `iso_certification_number` | TEXT    | If applicable                  |
| `last_calibration_date`    | DATE    | Most recent calibration check  |

### E. Vault Operators (`vault_operators`)

**Who are Vault Operators?**
Vault operators provide secure physical custody of gold. AGX verifies their technical security standards, GPS location, and chain-of-custody procedures. Their function is vital to ensuring that gold cannot be swapped, stolen, or diluted between verification and export.

* **Max Storage Capacity (kg):** Total secure holding space available.
* **Security Protocols:** Human and technological measures in place (guards, surveillance, biometric access).
* **GPS or Geofencing Location:** Coordinates or structured data enabling location tracking and alerting.
* **Chain-of-Custody Documentation:** Referenced SOP or digital system for logging transfers in/out of the vault.

| Column                 | Type    | Description                             |
| ---------------------- | ------- | --------------------------------------- |
| `id`                   | UUID    | Primary key                             |
| `actor_id`             | UUID    | FK → `actors.id`                        |
| `max_capacity_kg`      | DECIMAL | Vault limit in kilograms                |
| `security_protocols`   | TEXT    | Human and tech-based security approach  |
| `location_geo`         | TEXT    | Latitude/longitude or place description |
| `chain_of_custody_doc` | TEXT    | Link or hash to documented process      |

### F. Inspectors (`inspectors`)

**Who are Inspectors?**
Inspectors are field-level validators affiliated with government agencies, NGOs, or AGX itself. They perform site audits, document inspections, and risk assessments. Their reports feed directly into audit logs and certification decisions.

* **Affiliated Agency/NGO:** Institution or organization the inspector reports to.
* **Assigned Regions:** Zones or districts covered under their mandate.
* **Inspection Tools/Methods:** Field kits, checklists, or mobile apps used for data collection.
* **Reporting Interface:** How inspections are submitted—paper, mobile form, cloud sync.

| Column                | Type    | Description                             |
| --------------------- | ------- | --------------------------------------- |
| `id`                  | UUID    | Primary key                             |
| `actor_id`            | UUID    | FK → `actors.id`                        |
| `agency_name`         | TEXT    | Name of supervising body or partner org |
| `assigned_regions`    | TEXT\[] | Provinces or sites monitored            |
| `inspection_methods`  | TEXT    | List of tools or approaches used        |
| `reporting_interface` | TEXT    | Paper, app-based, third-party           |

### G. Field Agents (`field_agents`)

**Who are Field Agents?**
Field agents are AGX's frontline representatives. They onboard new actors, verify transactions, collect documentation, and support real-time data sync from rural areas. Their presence enables the inclusion of miners and aggregators in remote, low-connectivity zones.

* **Assigned Zones:** Villages, mining camps, or market areas covered by the agent.
* **Biometric Data Reference:** Mobile ID record—often a face image or fingerprint hash.
* **Last Sync Timestamp:** Last time their mobile system synced to AGX servers.
* **Language Preference:** Interface and training language tailored for each agent.

| Column                | Type      | Description                     |
| --------------------- | --------- | ------------------------------- |
| `id`                  | UUID      | Primary key                     |
| `actor_id`            | UUID      | FK → `actors.id`                |
| `assigned_zones`      | TEXT\[]   | Villages or regions covered     |
| `biometric_data_ref`  | TEXT      | Hash or link to mobile ID/photo |
| `last_sync_time`      | TIMESTAMP | Last mobile sync with server    |
| `language_preference` | TEXT      | Preferred interface language    |

---

**License:** Open Documentation (CC BY-SA 4.0)
**Repository:** [https://github.com/agx-foundation/agx-open-compliance-registry](https://github.com/agx-foundation/agx-open-compliance-registry)
