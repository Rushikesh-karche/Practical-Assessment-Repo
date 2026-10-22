# Assessment Project 9: Event Registration & Dynamic Ticket Dispatcher
**Project Code:** N8N-PRJ-09  
**Course:** n8n Automation Engineering (Hands-On Topics 1–15)  
**Lead Assessor & Reviewer:** Deepesh Jain  
**Author:** Rushikesh Karche  

---

## 📁 Repository Contents

| File | Description |
| :--- | :--- |
| **N8N_PRJ_09_Ticket_Dispatcher.json** | Complete, production-ready n8n workflow export with sticky notes, expressions, and pinData. |
| **
8n_project_09_requirement_document.md** | Official assessment requirement document with BRD, PRD, SRS, Google Sheets schema, and Test Execution Matrix. |
| **N8N_Project_09_Event_Registration_Ticket_Dispatcher.pdf** | Official assessment specification document. |

---

## 🚀 Workflow Architecture

1. **Intake Layer (Webhook Node):** Listens on POST /v1/event-registration with response mode configured as esponseNode.
2. **Data Sanitization & Ticket Generator (Edit Fields Node):** Cleanses attendee names/emails and generates unique TKT-[TIER]-[RANDOM6] codes.
3. **Persistent Database (Google Sheets Node):** Appends rows into Event_Registration_Master with all 7 columns.
4. **Tier Qualification Logic (IF Node):** Differentiates VIP vs General Admission.
5. **Immediate Client Response (Respond to Webhook Node):** Synchronously returns HTTP 200 JSON confirmation in under 2 seconds.
