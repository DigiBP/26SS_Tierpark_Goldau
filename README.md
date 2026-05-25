# Automated HR Recruitment Process – Alpenblick Consulting AG

> **DigiBP Group Project** · Spring Semester 2026  
> FHNW University of Applied Sciences and Arts Northwestern Switzerland  
> School of Business · Module: Digitalisation of Business Processes

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Company Context](#2-company-context)
3. [Process Architecture](#3-process-architecture)
   - [Job Posting Process](#31-job-posting-process)
   - [Recruitment & Application Process](#32-recruitment--application-process)
4. [Technology Stack](#4-technology-stack)
5. [Decision Automation (DMN)](#5-decision-automation-dmn)
6. [CV Scoring Architecture](#6-cv-scoring-architecture)
7. [Service Integration (Make Scenarios)](#7-service-integration-make-scenarios)
8. [Repository Structure](#8-repository-structure)
9. [Live Demos & Links](#9-live-demos--links)
10. [Limitations & Ethical Considerations](#10-limitations--ethical-considerations)
11. [Team](#11-team)

---

## 1. Project Overview

This project fully automates the end-to-end HR recruitment process for **Alpenblick Consulting AG**, a fictional Swiss consulting firm. The system covers everything from publishing an open position to onboarding a hired candidate — with minimal human intervention required.

The implementation is intentionally **job-agnostic**: the same BPMN and DMN logic applies to any open role. Three demo positions are currently configured:

| Position | Department |
|---|---|
| Software Engineer | Technology |
| Business Analyst | Consulting |
| ERP Consultant | Enterprise Solutions |

**Key goals of the digitalisation:**
- Replace manual, subjective CV screening with an LLM-based, criteria-driven scoring system
- Automate all candidate communication (invitations, rejections, contracts)
- Provide a real-time public job listing website updated automatically by the process
- Ensure consistent, auditable, and bias-reduced hiring decisions through DMN

---

## 2. Company Context

**Alpenblick Consulting AG** is a fictional management and technology consulting firm headquartered in Zürich, Switzerland. The company operates across industries and regularly hires for both technical and business roles. Prior to this project, its recruitment process was entirely manual — reliant on individual judgment, ad-hoc email communication, and unstructured document handling.

The AS-IS process suffered from:
- Fully manual application collection and CV review
- Subjective, unstandardised screening decisions
- Inconsistent candidate communication and delays
- Manual contract creation and uncoordinated onboarding

The TO-BE process introduced here addresses each of these pain points through process automation, service integration, and decision logic.

---

## 3. Process Architecture

The solution consists of two interconnected BPMN processes deployed on **Camunda Platform 7.24**.

### 3.1 Job Posting Process

**Trigger:** HR Manager starts a new process instance via a Camunda User Task form.

**Flow:**
1. HR Manager selects a job title from a predefined Camunda form
2. A **DMN Business Rule Task** evaluates the selected position and outputs all relevant job details (title, description, requirements, pensum, location, contact)
3. A **Make scenario** writes the job data to **Google Sheets**
4. A second **Make scenario** generates an HTML job card and publishes it to **GitHub Pages** via the GitHub API
5. After a 30-day timer, the job is automatically closed (HTML file deleted from GitHub)

**Key artifacts:** `Job_Posting.bpmn`, `Position.form`, `Position.dmn`

---

### 3.2 Recruitment & Application Process

**Trigger:** Candidates browse open positions on the public job listing website and submit their application via a job-specific application page.

**Application Flow:**
1. Candidates visit the [Alpenblick Consulting AG careers page](https://digibp.github.io/26SS_Tierpark_Goldau/), where active job openings are displayed as tiles (rendered from Google Sheets via the gviz API)
2. Each tile shows the job details and a **"Jetzt bewerben"** button
3. Clicking the button redirects the candidate to a job-specific application page (`apply.html`), where they fill in their personal details and upload their CV
4. On submission, a **Make scenario** picks up the application, calls the Claude API to parse the CV, writes the data to Google Sheets, and triggers the Camunda process via the REST API

**Flow:**

```
Candidate clicks "Jetzt bewerben" on job listing website (index.html)
  → Redirected to job-specific application page (apply.html)
  → Candidate fills in details & uploads CV → submits form
  → Make: receive submission, call Claude API for CV extraction
  → Make: write structured data to Google Sheets
  → Make: trigger Camunda via REST API (Message Start Event)
  → DMN: CV Scoring (COLLECT SUM → cvScore)
  → [cvScore < 45]  Make: personalised rejection email (Gmail) → End
  → [cvScore ≥ 45]  Make: interview invitation with Calendly link
  → Hiring Manager: conducts interview (Camunda User Task)
  → Hiring Manager: evaluates & assesses candidate (Camunda User Task / Evaluate.form)
  → [If rejected] Make: rejection email → End
  → [If approved] HR Manager: creates contract (Camunda User Task / Contract.form)
  → Make: generate contract from Google Docs template, export as PDF, send via Gmail
  → Candidate: clicks confirmation link → Make scenario 9 calls Camunda Message API (ContractReceived)
  → End: Process complete
```

**Key artifacts:** `Application.bpmn`, `CV_Scoring.dmn`, `Evaluate.form`, `Contract.form`

---

## 4. Technology Stack

| Component | Tool / Service | Role |
|---|---|---|
| Process Engine | **Camunda Platform 7.24** | BPMN & DMN execution, User Tasks |
| Middleware / Automation | **Make (formerly Integromat)** | All service task execution, webhooks |
| LLM / CV Parsing | **Claude API (Anthropic)** | LLM-based CV extraction from PDF |
| Job Listings Backend | **Google Sheets** | Stores job data; public read via gviz API |
| Job Listings Frontend | **GitHub Pages** | Renders open positions as a public website |
| Email | **Gmail** | Candidate communication (invitations, rejections, contracts) |
| Calendar / Scheduling | **Calendly** | Self-service interview booking links |
| Contract Generation | **Google Docs + Gmail** | Template-based contract creation & PDF delivery |
| Data Storage | **Google Sheets** | Job data (Jobs tab) and application data (Applications tab) |
| Process Messaging | **Camunda REST API** | Correlating messages to running instances (`/engine-rest/message`) |

All service tasks in the BPMN are implemented as **Make scenarios** (webhook-triggered or scheduled). No external Python workers or Docker containers are used.

---

## 5. Decision Automation (DMN)

CV scoring is implemented in `CV_Scoring.dmn` as a single decision table with **Hit Policy: COLLECT (SUM)**. All matching rules fire and their point values are summed into a final `cvScore`. The score drives a gateway in the BPMN directly — no separate shortlist table is needed.

**Threshold:** `cvScore < 45` → Send Cancellation · `cvScore ≥ 45` → Invite to First Interview

The table evaluates four independent dimensions:

### Education Level × Relevance (max 40 pts)

Education relevance is determined by the Claude API during CV parsing and reflects how closely the candidate's field of study matches the open role.

| Education | high | medium | low |
|---|---|---|---|
| Master | 40 | 24 | 6 |
| Bachelor | 32 | 18 | 5 |
| Federal Diploma (Eidg. Fachausweis) | 22 | 13 | 4 |
| Higher Technical School (HF) | 18 | 11 | 3 |
| Apprenticeship (EFZ) | 12 | 7 | 2 |
| None / Other | 6 | 3 | 1 |

### Experience × Relevance (max 40 pts)

Experience relevance reflects how directly the candidate's prior work experience applies to the role.

| Years of Experience | high | medium | low |
|---|---|---|---|
| ≥ 7 years | 40 | 22 | 8 |
| 4 – 6 years | 32 | 18 | 6 |
| 2 – 3 years | 22 | 13 | 4 |
| 0 – 1 years | 10 | 5 | 0 |

### German Language Level (max 10 pts)

| Level | Points |
|---|---|
| Native | 10 |
| C2 | 9 |
| C1 | 8 |
| B2 | 6 |
| B1 | 4 |
| A2 | 2 |
| A1 | 1 |
| None | 0 |

### English Language Level (max 10 pts)

Same scale as German: Native=10, C2=9, C1=8, B2=6, B1=4, A2=2, A1=1, None=0.

---

## 6. CV Scoring Architecture

CV parsing is performed by the **Claude API (claude-sonnet)**, called from within Make scenario 4 when a new application is submitted via `apply.html`. The LLM extracts structured data from the attached CV PDF according to a defined schema:

```json
{
  "candidateName": "string",
  "candidateEmail": "string",
  "educationLevel": "none | apprenticeship | federalDiploma | higherTechnical | bachelor | master",
  "yearsExperience": "integer",
  "germanLevel": "none | A1 | A2 | B1 | B2 | C1 | C2 | native",
  "englishLevel": "none | A1 | A2 | B1 | B2 | C1 | C2 | native",
  "educationRelevance": "high | medium | low",
  "experienceRelevance": "high | medium | low"
}
```

These variables are passed to Camunda as process variables and consumed directly by `CV_Scoring.dmn`. The COLLECT SUM result (`cvScore`) is then evaluated by a gateway in `Application.bpmn`: scores below 45 route to rejection, scores of 45 and above proceed to the interview stage.

---

## 7. Service Integration (Make Scenarios)

**Job Posting Process**

| # | Scenario | BPMN Service Task | Webhook | Actions |
|---|---|---|---|---|
| 1 | **Send decision to Google Sheets** | Send decision to Make (Google Sheets) | `camunda-send-decision` | Writes job details (title, description, requirements, etc.) as a new row to Google Sheets |
| 2 | **Post Job to GitHub Pages** | Post Job to GitHub Pages | `camunda-post-job` | Reads job data from Google Sheets, generates an HTML job card, and publishes it to the GitHub Pages repo via the GitHub API |
| 3 | **Close Job Posting** | Close Job Posting | `camunda-close-job` | Updates the job status in Google Sheets to CLOSED, then deletes the job HTML file from GitHub Pages via the GitHub API |

**Recruitment & Application Process**

| # | Scenario | BPMN Task | Webhook | Actions |
|---|---|---|---|---|
| 4 | **Application apply.html** | Application received / Collect & Structure Application | `application-webhook` | Reads job details from Google Sheets, sends CV PDF to Claude API for structured extraction, writes parsed application data to Google Sheets, triggers Camunda process via `/engine-rest/message` |
| 5 | **Send Cancellation** | Send Cancellation | `camunda-send-cancellation` | Sends personalised rejection email to candidate via Gmail (*"Your application for {{jobTitle}} – Update"*) |
| 6 | **Invite to first Interview** | Invite to first Interview | `camunda-invite-interview` | Sends interview invitation email with Calendly booking link via Gmail (*"Invitation to first Interview – {{jobTitle}}"*) |
| 7 | **Invite to Assessment** | Invite to Assessment | `invite-to-assessment` | Sends assessment invitation email with Calendly booking link via Gmail (*"Invitation to Assessment – {{jobTitle}}"*) |
| 8 | **Send Contract (Public-Link Variante)** | Send Contract | `send-contract-webhook` | Creates a personalised contract from a Google Docs template, exports it as PDF, and sends it to the candidate via Gmail (*"Your contract – {{jobTitle}} – Alpenblick Consulting AG"*) |
| 9 | **Contract Confirmation** | Contract received | `contract-confirmation-webhook2` | Calls Camunda REST API (`/engine-rest/message`) to correlate the `Message_ContractReceived` event, resuming the process for onboarding; responds HTTP 200 to the candidate's browser |

> **Note:** Make scenarios must be activated via the **Scheduling toggle** on the Make overview page (not just saved in the editor) to run automatically.

---

## 8. Repository Structure

```
26SS_Tierpark_Goldau/
│
├── Job_Posting.bpmn               # Job posting process
├── Position.form                  # Camunda User Task form: select job title
├── Position.dmn                   # DMN: Decide on Position (job details lookup)
│
├── Application.bpmn               # Main recruitment & application process
├── CV_Scoring.dmn                 # DMN: CV Scoring (Knockout + COLLECT SUM + Shortlist)
├── Evaluate.form                  # Camunda User Task form: Evaluate & Assess Candidate
├── Contract.form                  # Camunda User Task form: Create Contract
│
├── Make/
│   ├── 1_Send_decision_to_Google_Sheets_blueprint.json
│   ├── 2_Post_Job_to_GitHub_Pages_blueprint.json
│   ├── 3_Close_Job_Posting_blueprint.json
│   ├── 4_Application_apply_html_blueprint.json
│   ├── 5_Send_Cancellation_blueprint.json
│   ├── 6_Invite_to_first_Interview_blueprint.json
│   ├── 7_Invite_to_Assessment_blueprint.json
│   ├── 8_Send_Contract_blueprint.json
│   └── 9_Contract_Confirmation_blueprint.json
│
├── index.html                     # GitHub Pages job listing website (job tiles)
├── apply.html                     # Job-specific application page ("Jetzt bewerben")
│
└── Old/                           # Archived / previous BPMN iterations
```

---

## 9. Live Demos & Links

| Resource | Link |
|---|---|
| 🌐 **Job Listing Website** | [https://digibp.github.io/26SS_Tierpark_Goldau/](https://digibp.github.io/26SS_Tierpark_Goldau/) |
| ⚙️ **Camunda Instance** | *(link to be added)* |
| 📊 **Presentation Slides** | *(link to be added)* |

> To start a new recruitment process instance, navigate to the Camunda Tasklist and initiate the **Job Posting** process. Once a position is live on the careers page, a candidate can apply by clicking **"Jetzt bewerben"** on the job tile and submitting the application form.

---

## 10. Limitations & Ethical Considerations

### LLM-based CV Parsing
The use of the Claude API for CV extraction introduces inherent variability — parsing quality depends on CV formatting and language. In a production context, human review of parsed fields would be recommended before they are passed to scoring logic.

### DMN Scoring Subjectivity
While the DMN replaces ad-hoc human judgment, the score weights and thresholds themselves reflect design choices made by the team. These should be reviewed and validated with domain experts (e.g., HR professionals) before real-world use.

### EU AI Act & Swiss DSG
Automated hiring decisions fall under the scope of **high-risk AI systems** as defined by the EU AI Act (Annex III). In a real deployment, the system would need to ensure transparency, human oversight, and the right to explanation for rejected candidates. Under the **Swiss Federal Act on Data Protection (DSG/nDSG)**, candidate data must be processed with a legal basis, stored for the minimum necessary duration, and candidates must be informed of automated processing.

### Demo Scope
This system is an academic prototype. Live credentials, API keys, and webhook URLs used in Make scenarios are for demonstration purposes only and should be rotated before any real-world use.

---

## 11. Team

| Name | Role |
|---|---|
| *(Team member 1)* | *(Role)* |
| *(Team member 2)* | *(Role)* |
| *(Team member 3)* | *(Role)* |
| *(Team member 4)* | *(Role)* |

**Supervisor:** Andreas Martin, FHNW School of Business

---

*DigiBP Group Project · Spring Semester 2026 · FHNW University of Applied Sciences and Arts Northwestern Switzerland*
