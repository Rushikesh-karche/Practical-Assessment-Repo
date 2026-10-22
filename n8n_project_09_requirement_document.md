# n8n Automation Engineering - Assessment Project 9
## Event Registration & Dynamic Ticket Dispatcher

---

## 1. Project Understanding: What Needs to be Built Using the System

### 1.1 The Business Problem & Automation Goal
Corporate events, developer summits, and training webinars attract hundreds or thousands of attendees. When event planners rely on manual CSV downloads and delayed bulk emailing to distribute attendee tickets, attendees experience delays, ticket lost-in-transit issues, and check-in chaos at the venue. Furthermore, VIP attendees who pay for executive networking sessions frequently receive generic passes rather than customized credentials with exclusive lounge access details.

The goal of this project is to build an automated, real-time **Event Registration and Ticket Dispatch Engine in n8n**. The system captures registration submissions in real time via an HTTP webhook, cleanses attendee information, generates a cryptographically unique ticket reference code, records the attendee in a centralized Google Sheets registration database, uses an `IF Node` to differentiate between VIP Pass and General Admission tiers, and dispatches a responsive, beautifully styled HTML ticket pass via Gmail directly to the attendee while returning an instant confirmation to the webform submitter.

---

### 1.2 System Components & Architecture
* **Intake Layer (Webhook Node):** Captures incoming attendee registration payloads (`full_name`, `email`, `company`, `ticket_type`) via HTTP `POST` at `/v1/event-registration`.
* **Data Sanitization & Ticket Generator (Edit Fields / Set Node):**
  * Capitalizes and trims attendee name and lowercases email.
  * Generates an auditable, human-readable ticket reference code: `TKT-[TIER]-[RANDOM6]`.
  * Generates registration timestamp using `$now`.
* **Persistent Database (Google Sheets Node):** Appends the attendee record into `Event_Registration_Master` with status `Registered`.
* **Tier Qualification Logic (IF Node):** Evaluates `{{ $json.ticket_type }}`:
  * **VIP Pass (TRUE):** Routes to VIP ticket generation (includes VIP Keynote access, Executive Lounge credentials, and dedicated concierge contact).
  * **General Admission (FALSE):** Routes to Standard ticket generation (includes general session schedule and regular access credentials).
* **Communication Layer (Gmail Node):** Dispatches a mobile-responsive HTML ticket pass with badge headers, ticket codes, and event calendar details.
* **Immediate Client Response (Respond to Webhook Node):** Returns an immediate HTTP 200 JSON payload containing `{"status": "confirmed", "ticket_code": "TKT-...", "name": "..."}` to the registration webpage.

---

### 1.3 Architecture & Data Flow Diagram

```text
[ Event Registration Webpage ]
       | (HTTP POST Attendee JSON)
       v
[ Webhook Node: /v1/event-registration ]
       |
       v
[ Edit Fields: Data Cleansing & Code Generation ]
  Expressions:
  * TicketCode = "TKT-" + Tier.slice(0,3).toUpperCase() + "-" + Random6
  * RegisteredAt = $now
       |
       v
[ Google Sheets: Append Row ] --> Insert row in Event_Registration_Master
       |
       v
[ IF Node: Ticket_Type == "VIP"? ]
  +-- TRUE (VIP)  --> [ Gmail: Dispatch VIP Pass with Lounge Credentials ] ---+
  +-- FALSE (Gen) --> [ Gmail: Dispatch Standard Admission Ticket ] --------+
                                                                            |
                                                                            v
[ Respond to Webhook Node ] <-- Return HTTP 200 {"status": "confirmed", "ticket_code": "..."}
```

---

## 2. Assessment Governance & Execution Charter

### 2.1 Key Assessment Parameters

| Parameter | Specification |
| :--- | :--- |
| **Project Code** | `N8N-PRJ-09` |
| **Allocated Timeframe** | 1.5 Hours (Time-boxed practical assessment) |
| **Lead Assessor & Reviewer** | Deepesh Jain (Course Lead / Lead Evaluator) |
| **Passing Score Threshold** | Minimum 70 / 100 points (100% compliance on Zero-Tolerance Gates) |
| **Submission Format** | Exported workflow `.json`, Google Sheets link, test execution matrix, and walkthrough video |

### 2.2 What Gets Assessed
* Webhook configuration and synchronous JSON response generation.
* Random string generation and prefix logic in Edit Fields.
* Accurate Google Sheets column mapping and attendee tracking.
* Conditional branching separating VIP from General Admission passes.
* Responsive, high-aesthetic HTML ticket formatting in Gmail.
* Verification of instant delivery within 2 seconds.

---

### 2.3 Official 100-Point Scoring Rubric

| Evaluation Dimension | Points | Scoring Criteria |
| :--- | :--- | :--- |
| **Functional Delivery** | 35 | Captures registration, generates ticket code, appends row to Google Sheets, branches by tier, and delivers HTML ticket email. |
| **Data Formatting & Expressions** | 20 | Clean expressions generating unique ticket numbers; robust string trimming and email sanitization. |
| **Conditional Logic & Routing** | 15 | IF node accurately differentiates VIP vs General Admission without routing errors. |
| **Google Sheets & Gmail Integration** | 15 | Spreadsheet columns correctly match schema; Gmail tickets are mobile-friendly and visually appealing. |
| **Testing Evidence & Traceability** | 10 | Documented test execution runs (TC-001 to TC-003) with verifiable n8n execution IDs. |
| **Code Hygiene & Oral Defense** | 5 | Clean visual arrangement, descriptive node names, sticky notes, and clear explanation during evaluator review. |
| **Total** | **100** | **Passing Score: 70.0 / 100.0** |

---

### 2.4 Zero-Tolerance Quality Gates (Immediate Auto-Fail)
* **Gate 1: Duplicate Ticket Codes:** Generating static or hardcoded ticket codes resulting in duplicate reference IDs.
* **Gate 2: Webhook Response Timeout:** Failing to return an HTTP response to the caller, resulting in frontend registration timeouts.
* **Gate 3: Hardcoded Personal Credentials:** Exposing Gmail account passwords or private tokens in cleartext.

---

## 3. Project Requirements Specification

### 3.1 Document Control

| Field | Value |
| :--- | :--- |
| **Document Title** | Requirement Document: Event Registration & Dynamic Ticket Dispatcher |
| **Course** | n8n Hands-On Automation Course (Topics 1-15) |
| **Author / Reviewer** | Deepesh Jain (Course Lead / Lead Evaluator) |
| **Timeframe** | 1.5 Hours |
| **Status** | Approved for Learner Assessment |

---

### 3.2 Business Requirements Document (BRD)
* **BRD-001 (Real-Time Ingestion):** Registrations must be captured instantly upon form submission.
* **BRD-002 (Unique Identification):** Every attendee must receive a unique, non-duplicable ticket reference code.
* **BRD-003 (Attendee Database):** All registered attendees must be recorded in Google Sheets for on-site badge printing and check-in.
* **BRD-004 (Tiered Experiences):** VIP attendees must receive special access badges and executive lounge details.

---

### 3.3 Product Requirements Document (PRD)
* **PRD-001:** Webhook node listening on `/v1/event-registration` for `POST` requests.
* **PRD-002:** Edit Fields node generating `TKT-[TIER]-[RANDOM6]` codes.
* **PRD-003:** Google Sheets node appending row to `Event_Registration_Master`.
* **PRD-004:** IF node evaluating `ticket_type == 'VIP'`.
* **PRD-005:** Gmail nodes dispatching customized HTML ticket passes.
* **PRD-006:** Respond to Webhook node returning HTTP 200 with ticket code.

---

### 3.4 Software Requirements Specification (SRS)

#### 4.1 Functional Requirements
* **FR-001 (Intake Webhook):** Accepts `full_name`, `email`, `company`, `ticket_type` (VIP or General).
* **FR-002 (Ticket Code Generation):**
  ```javascript
  ticket_code: ='TKT-' + ($json.ticket_type.toUpperCase().includes('VIP') ? 'VIP' : 'GEN') + '-' + Math.floor(100000 + Math.random() * 900000)
  ```
* **FR-003 (Google Sheets Persistence):** Append row to `Event_Registration_Master` with status `Registered`.
* **FR-004 (Tier Routing):** IF node checks `{{ $json.ticket_type.toLowerCase().includes('vip') }}`.
* **FR-005 (VIP Ticket Email):** Sends styled VIP HTML email to `{{ $json.email }}` with Executive Lounge access credentials.
* **FR-006 (General Ticket Email):** Sends styled General Admission HTML email to `{{ $json.email }}`.
* **FR-007 (Immediate Webhook Response):** Return HTTP 200 with JSON payload:
  ```json
  {
    "status": "confirmed",
    "ticket_code": "{{ $json.ticket_code }}",
    "name": "{{ $json.full_name }}"
  }
  ```

#### 4.2 Non-Functional Requirements
* **NFR-001 (Latency):** Total pipeline execution and webhook response must complete within 2,000 ms.
* **NFR-002 (Responsive Design):** Ticket email must render cleanly on both mobile and desktop email clients.

---

## 4. Technical Specifications & Integration Details

### 4.1 Google Sheets Data Schema (`Event_Registration_Master`)

| Column Name | Data Type | Required | Sample Value | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`Ticket_Code`** | String | Yes | `TKT-VIP-849201` | Unique ticket identifier |
| **`Registered_At`** | DateTime | Yes | `2026-10-16 15:00:20` | Registration timestamp |
| **`Attendee_Name`** | String | Yes | `Alex Mercer` | Full attendee name |
| **`Attendee_Email`** | String | Yes | `alex.mercer@techconf.org` | Ticket destination email |
| **`Company`** | String | Yes | `Mercer Solutions` | Organization name |
| **`Ticket_Type`** | String | Yes | `VIP` | VIP or General Admission |
| **`Check_In_Status`** | String | Yes | `Registered` | Check-in state (`Registered` / `Checked In`) |

---

### 4.2 Sample Inbound Webhook Payload

```json
{
  "full_name": "Alex Mercer",
  "email": "alex.mercer@techconf.org",
  "company": "Mercer Solutions",
  "ticket_type": "VIP"
}
```

---

### 4.3 Node Expression Formulas
* **Ticket Code Formula:**
  ```javascript
  ='TKT-' + (($json.body?.ticket_type || $json.ticket_type || 'General').toUpperCase().includes('VIP') ? 'VIP' : 'GEN') + '-' + Math.floor(100000 + Math.random() * 900000)
  ```
* **Tier Condition in IF Node:**
  ```javascript
  ={{ $json.ticket_type.toLowerCase().includes('vip') }}
  ```
* **Timestamp Formula:**
  ```javascript
  ={{ $now.toFormat('yyyy-MM-dd HH:mm:ss') }}
  ```

---

## 5. Test Execution Matrix

| Test ID | Test Scenario | Input Data | Expected Outcome | Execution Evidence Required |
| :--- | :--- | :--- | :--- | :--- |
| **TC-001** | VIP Ticket Registration | `ticket_type: "VIP"` | Unique `TKT-VIP-xxxxxx` generated; row appended to Google Sheets; VIP HTML pass received. | Screenshot of Google Sheet row + 1 VIP Gmail ticket pass |
| **TC-002** | General Admission Registration | `ticket_type: "General Admission"` | Unique `TKT-GEN-xxxxxx` generated; row appended; standard ticket received. | Screenshot of Google Sheet row + 1 standard Gmail ticket |
| **TC-003** | Immediate Webhook Response | Postman POST submission | Client receives HTTP 200 with ticket code in JSON body within 2 seconds. | Screenshot of Postman HTTP 200 response |

---

## 6. Candidate Pre-Submission Checklist
* [x] Workflow is exported as `N8N_PRJ_09_Ticket_Dispatcher.json`.
* [x] Webhook node returns HTTP 200 response with generated ticket code.
* [x] Both VIP and General Admission branches deliver customized HTML tickets.
* [x] Google Sheet schema matches table columns exactly.
* [x] Walkthrough demonstrates submission and dynamic ticket receipt.
