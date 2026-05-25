# 26SS_Tierpark_Goldau

> **DigiBP Group Project** · Spring Semester 2026  
> FHNW University of Applied Sciences and Arts Northwestern Switzerland  
> School of Business · Module: Digitalisation of Business Processes

---

## Team Members

| Name | Email |
|---|---|
| Ece Kaya | ece.kaya@students.fhnw.ch |
| Furkan Aydin | furkan.aydin@students.fhnw.ch |
| Dominic Däster | dominic.daester@students.fhnw.ch |
| Keethan Nathan | keethan.nathan@students.fhnw.ch |

**Supervisor:** Andreas Martin: andreas.martin@fhnw.ch, Charuta Pande: charuta.pande@fhnw.ch, Devid Montecchiari: devid.montecchiari@fhnw.chFHNW School of Business

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [AS-IS Process](#2-as-is-process)
3. [TO-BE Process](#3-to-be-process)
4. [Technology Stack](#4-technology-stack)
5. [Decision Automation (DMN)](#5-decision-automation-dmn)
6. [CV Scoring Architecture](#6-cv-scoring-architecture)
7. [Service Integration (Make Scenarios)](#7-service-integration-make-scenarios)
8. [Repository Structure](#8-repository-structure)
9. [Live Demos & Links](#9-live-demos--links)
10. [Limitations & Ethical Considerations](#10-limitations--ethical-considerations)

---

## 1. Introduction

Alpenblick Consulting AG is a Swiss consulting company with 10 employees. The company provides professional advisory services to small and medium-sized businesses in areas such as strategy, process improvement, and digital transformation. Due to its continuous growth, the company aims to optimize its recruitment process in order to attract qualified talent more efficiently, reduce administrative effort, and support future expansion.

The implementation is intentionally **job-agnostic**: the same BPMN and DMN logic applies to any open role. Three demo positions are currently configured:

| Position | Department |
|---|---|
| Software Engineer | Technology |
| Business Analyst | Consulting |
| ERP Consultant | Enterprise Solutions |

---

## 2. Repository Structure

```
26SS_Tierpark_Goldau/
│
├── Job_Posting.bpmn               # Job posting process
├── Position.form                  # Camunda User Task form: select job title
├── Position.dmn                   # DMN: Decide on Position (job details lookup)
│
├── Application.bpmn               # Main recruitment & application process
├── CV_Scoring.dmn                 # DMN: CV Scoring (COLLECT SUM, 4 dimensions)
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
├── index.html                     # GitHub Pages careers website (job tiles)
├── apply.html                     # Job-specific application page ("Jetzt bewerben")
│
└── Old/                           # Archived / previous BPMN iterations
```

---
## 3. AS-IS Process

<img width="1826" height="347" alt="grafik" src="https://github.com/user-attachments/assets/0e76de74-e07f-4740-9437-8773be933462" />

### 3.1 Process Description

The recruitment process begins when a new position is requested within the organization. The HR Manager first defines the position, specifying requirements and responsibilities, and then advertises the position in a web portal to attract potential candidates.

Once an applicant submits their application, the HR Manager receives the application and gathers it in the web portal. The HR Recruiter then reviews the submitted CV and evaluates whether the application is suitable.

If the application is deemed unsuitable, the HR Recruiter sends a cancellation notice to the applicant, and the process ends with a denied application.

If the application is suitable, the Hiring Manager invites the candidate to a first interview, which is subsequently conducted and evaluated. Based on this evaluation, the Hiring Manager decides whether to invite the candidate to an assessment.

If the candidate is not invited to the assessment, a cancellation is sent and the process ends. Otherwise, the Hiring Manager invites the candidate to an assessment, which is then conducted and evaluated in detail.

Following the assessment, the Hiring Manager decides whether to hire the candidate. If the decision is negative, a cancellation notice is sent to the applicant. If the decision is positive, the HR Manager creates an employment contract and sends it to the applicant.

Once the signed contract is received back from the applicant, the HR Manager proceeds to onboard the new employee, concluding the process with the employee being successfully onboarded into the organization.

### 3.2 Identified Challenges of the AS-IS Process

The current recruitment process relies heavily on manual work and individual judgment, creating several significant challenges.

**Fully Manual Application Handling**  
In the AS-IS process, the HR Manager manually monitors the web portal for incoming applications and individually gathers each submission. There is no automated notification or data extraction in place. This means that application data must be collected, organised, and transferred by hand, which is both time-consuming and prone to oversight, particularly when application volumes are high.

**Subjective CV Screening**  
The HR Recruiter manually reviews each CV and decides purely based on personal judgment whether an application is suitable. Without defined scoring criteria or a standardised evaluation framework, this step introduces significant subjectivity. Two recruiters may assess the same CV differently, leading to inconsistent and potentially unfair screening outcomes.

**Manual Interview Scheduling and Communication**  
The Hiring Manager manually sends interview invitations and assessment invitations to candidates via e-mail. Similarly, cancellation notices are written and sent individually by the HR Recruiter. This manual communication approach creates a risk of delays, oversights, and inconsistent messaging, which negatively affects the candidate experience and the organisation's professional image.

**Unstructured Interview and Evaluation Process**  
Both the first interview and the subsequent candidate evaluation are conducted without a standardised format. Feedback and assessments are not captured in a unified system, making it difficult to compare candidates objectively or to revisit evaluation results at a later stage. This lack of structure increases the risk of biased hiring decisions.

**Manual Contract Creation and Distribution**  
Once a hiring decision is made, the HR Manager manually creates the employment contract, typically in a word processor, and sends it to the candidate via e-mail. This process is not only slow but also susceptible to errors in contract content, version inconsistencies, and delays in delivery or follow-up.

**Manual Onboarding Coordination**  
The final step, onboarding the new employee, is coordinated entirely by hand. The HR Manager must individually arrange IT access, workspace setup, team introductions, and other onboarding activities. Without automation or a structured workflow, onboarding steps can be forgotten, delayed, or inconsistently executed depending on the individual responsible.

---

## 4. TO-BE Process

The TO-BE recruitment process introduces significant automation across all stages, reducing manual effort and standardizing decision-making through system integrations. It consists of two BPMN processes deployed on **Camunda Platform 7.24**, with all service tasks implemented as **Make (Integromat) scenarios**.

<img width="6930" height="3780" alt="TO_BE_Application_Model" src="https://github.com/user-attachments/assets/10efa578-8f13-4b8f-897b-755538a664fc" />


### 4.1 Job Posting Sub-Process

**Key artifacts:** `Job_Posting.bpmn`, `Position.form`, `Position.dmn`

The process begins when the HR Manager requests a new position. The manager selects the position from a predefined Camunda form (`Position.form`), after which a DMN-based business rule task (`Position.dmn`) automatically retrieves the relevant position details (title, description, requirements, location, contact). A Make scenario then writes this data to Google Sheets, and a second Make scenario generates an HTML job card and publishes it to the public careers website via the GitHub API. After a 30-day timer, the job posting is automatically closed and removed from the website.

**Flow:**
1. HR Manager selects position via `Position.form` (Camunda User Task)
2. DMN Business Rule Task (`Position.dmn`) outputs full job details
3. Make scenario 1 writes job data to Google Sheets
4. Make scenario 2 publishes HTML job card to GitHub Pages
5. 30-day timer fires → Make scenario 3 closes the posting (updates Sheets, deletes HTML from GitHub)

### 4.2 Recruitment & Application Process

**Key artifacts:** `Application.bpmn`, `CV_Scoring.dmn`, `Evaluate.form`, `Contract.form`

**Application Intake**  
Candidates visit the [Alpenblick Consulting AG careers page](https://digibp.github.io/26SS_Tierpark_Goldau/), where active positions are displayed as tiles. Each tile features a **"Jetzt bewerben"** button that redirects the candidate to a job-specific application page (`apply.html`). On submission, Make scenario 4 receives the application, calls the **Claude API** to extract structured data from the uploaded CV PDF, stores the result in Google Sheets, and triggers the Camunda process via the REST API.

**CV Scoring**  
Rather than manual CV review, the Claude API parses the CV and populates the variables consumed by `CV_Scoring.dmn`. The DMN evaluates education, experience, and language skills and produces a `cvScore`. Candidates scoring below 45 receive an automated rejection email (Make scenario 5). Candidates scoring 45 or above are automatically invited to a first interview (Make scenario 6).

**Interview & Assessment**  
The Hiring Manager conducts the first interview and evaluates the candidate using a structured digital form (`Evaluate.form`) within Camunda. If the candidate progresses, the system automatically sends an assessment invitation via Calendly (Make scenario 7). The Hiring Manager then conducts and assesses the candidate using the same `Evaluate.form`. If the candidate does not progress at any stage, a cancellation email is sent automatically.

**Contract & Onboarding**  
If the hiring decision is positive, the HR Manager creates the contract via `Contract.form` in Camunda. Make scenario 8 then automatically generates the contract from a Google Docs template, exports it as a PDF, and sends it to the candidate via Gmail. The candidate confirms acceptance by clicking a link, which triggers Make scenario 9 to correlate the `Message_ContractReceived` event in Camunda and resume the process. The onboarding workflow is then triggered automatically.

**Full process flow:**

```
Candidate clicks "Jetzt bewerben" on careers website (index.html)
  → Redirected to application page (apply.html)
  → Candidate fills in details & uploads CV → submits form
  → Make 4: call Claude API for CV extraction, write to Google Sheets,
             trigger Camunda via REST API (Message Start Event)
  → DMN CV_Scoring.dmn: COLLECT SUM → cvScore
  → [cvScore < 45]  Make 5: rejection email → End
  → [cvScore ≥ 45]  Make 6: interview invitation with Calendly link
  → Hiring Manager: conducts interview, evaluates candidate (Evaluate.form)
  → [Not progressing] Make 5: rejection email → End
  → [Progressing] Make 7: assessment invitation with Calendly link
  → Hiring Manager: conducts assessment, assesses candidate (Evaluate.form)
  → [Not hired] Make 5: rejection email → End
  → [Hired] HR Manager: creates contract details (Contract.form)
  → Make 8: generate contract (Google Docs template → PDF → Gmail)
  → Candidate: clicks confirmation link
  → Make 9: correlate Message_ContractReceived via Camunda REST API
  → Onboarding triggered → End: Employee onboarded
```

### 4.3 Improvements and Benefits of the TO-BE Process

**Automated Application Collection and Structuring**  
Incoming applications via `apply.html` are automatically received and processed by Make scenario 4. All relevant applicant data, such as education level, years of experience, and language skills, are extracted by the Claude API and stored in a structured format in Google Sheets. This eliminates manual data entry and significantly speeds up the intake stage.

**Objective and Standardised CV Scoring**  
Manual CV screening is replaced by an automated scoring system using the Claude API and `CV_Scoring.dmn`. Every candidate is evaluated against the same defined criteria and weighted rules, removing individual bias from the screening decision and ensuring consistent, transparent filtering.

**Automated Candidate Communication**  
All candidate-facing communications — interview invitations, assessment invitations, rejection notices, and contract delivery — are handled automatically by Make scenarios. Interview and assessment invitations include a self-service Calendly scheduling link, eliminating back-and-forth email exchanges and ensuring timely, professional communication at every stage.

**Structured and Digitalised Interview Evaluation**  
The TO-BE process introduces standardised digital forms (`Evaluate.form`) for conducting and evaluating both the interview and assessment. All feedback is captured centrally within Camunda, enabling objective comparison and a full audit trail of hiring decisions.

**Automated Contract Generation and Distribution**  
Rather than manually drafting contracts, the HR Manager completes `Contract.form` in Camunda. Make scenario 8 then automatically generates the contract from a Google Docs template, converts it to PDF, and delivers it via Gmail. The candidate confirms via a public link, which automatically resumes the Camunda process.

**Automated Onboarding Workflow**  
Upon contract confirmation, the system automatically triggers the full onboarding workflow — including creating a Google Workspace account, sending a welcome email, notifying the team via Slack, creating onboarding tasks in Asana, and scheduling calendar meetings — without any manual intervention.

**Automated Job Posting Management**  
Once the HR Manager confirms the position details, the system automatically publishes the job advertisement to the careers website. After 30 days, the posting is closed automatically, eliminating the need for manual advertising and follow-up.

---

## 5. Technology Stack

| Component | Tool / Service | Role |
|---|---|---|
| Process Engine | **Camunda Platform 7.24** | BPMN & DMN execution, User Tasks |
| Middleware / Automation | **Make (formerly Integromat)** | All service task execution, webhooks |
| LLM / CV Parsing | **Claude API (Anthropic)** | LLM-based CV extraction from PDF |
| Job Listings Backend | **Google Sheets** | Stores job data and application data |
| Job Listings Frontend | **GitHub Pages** | Public careers website (`index.html`, `apply.html`) |
| Email | **Gmail** | Candidate communication (invitations, rejections, contracts) |
| Calendar / Scheduling | **Calendly** | Self-service interview and assessment booking links |
| Contract Generation | **Google Docs + Gmail** | Template-based contract creation & PDF delivery |
| Process Messaging | **Camunda REST API** | Message correlation via `/engine-rest/message` |

All service tasks in the BPMN are implemented as **Make scenarios** (webhook-triggered). No external Python workers or Docker containers are used.

---

## 6. Decision Automation (DMN)

CV scoring is implemented in `CV_Scoring.dmn` as a single decision table with **Hit Policy: COLLECT (SUM)**. All matching rules fire and their point values are summed into a final `cvScore`. The score is evaluated directly by a gateway in `Application.bpmn` — no separate shortlist table is needed.

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

## 7. CV Scoring Architecture

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

## 8. Service Integration (Make Scenarios)

**Job Posting Process**

| # | Scenario | BPMN Service Task | Webhook | Actions |
|---|---|---|---|---|
| 1 | **Send decision to Google Sheets** | Send decision to Make (Google Sheets) | `camunda-send-decision` | Writes job details (title, description, requirements, etc.) as a new row to Google Sheets <br><br> <img width="756" height="381" alt="1-Send_decision_to_Google_Sheets" src="https://github.com/user-attachments/assets/5659a8ed-3199-4acc-be4b-7371e7022735" /> |
| 2 | **Post Job to GitHub Pages** | Post Job to GitHub Pages | `camunda-post-job` | Reads job data from Google Sheets, generates an HTML job card, and publishes it to the GitHub Pages repo via the GitHub API <br><br> <img width="807" height="212" alt="2-Post-Job-to-GitJub-Pages" src="https://github.com/user-attachments/assets/b71b8b42-4814-4685-8db1-cad2cb1b0a50" /> |
| 3 | **Close Job Posting** | Close Job Posting | `camunda-close-job` | Updates the job status in Google Sheets to CLOSED, then deletes the job HTML file from GitHub Pages via the GitHub API <br><br> <img width="804" height="170" alt="3-Close_Job_Posting" src="https://github.com/user-attachments/assets/1302959f-5a32-490b-a1ee-ab108615461b" /> |

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

## 9. Live Demos & Links

| Resource | Link |
|---|---|
| 🌐 **Careers Website** | [https://digibp.github.io/26SS_Tierpark_Goldau/](https://digibp.github.io/26SS_Tierpark_Goldau/) |
| ⚙️ **Camunda Instance** | *(link to be added)* |
| 📊 **Presentation Slides** | *(link to be added)* |

> To start a new recruitment process instance, navigate to the Camunda Tasklist and initiate the **Job Posting** process. Once a position is live on the careers page, a candidate can apply by clicking **"Jetzt bewerben"** on the job tile and submitting the application form on `apply.html`.

---

## 10. Limitations & Ethical Considerations

**LLM-based CV Parsing**  
The use of the Claude API for CV extraction introduces inherent variability — parsing quality depends on CV formatting and language. In a production context, human review of parsed fields would be recommended before they are passed to scoring logic.

**DMN Scoring Subjectivity**  
While the DMN replaces ad-hoc human judgment, the score weights and thresholds themselves reflect design choices made by the team. These should be reviewed and validated with domain experts (e.g., HR professionals) before real-world use.

**EU AI Act & Swiss DSG**  
Automated hiring decisions fall under the scope of **high-risk AI systems** as defined by the EU AI Act (Annex III). In a real deployment, the system would need to ensure transparency, human oversight, and the right to explanation for rejected candidates. Under the **Swiss Federal Act on Data Protection (DSG/nDSG)**, candidate data must be processed with a legal basis, stored for the minimum necessary duration, and candidates must be informed of automated processing.

**Demo Scope**  
This system is an academic prototype. Live credentials, API keys, and webhook URLs used in Make scenarios are for demonstration purposes only and should be rotated before any real-world use.

---

*DigiBP Group Project · Spring Semester 2026 · FHNW University of Applied Sciences and Arts Northwestern Switzerland*
