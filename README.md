# 26SS_Tierpark_Goldau

# Team Members
| Name  | Email Adresse     |
|-------|------------------|
| Ece Kaya  | ece.kaya@students.fhnw.ch  |
| Furkan Aydin  | furkan.aydin@students.fhnw.ch  |
| Dominic Däster  | dominic.daester@students.fhnw.ch  |
| Keethan Nathan  | keethan.nathan@students.fhnw.ch  |

# KEFD
A process management system for employee recruitment – from job posting to onboarding.

# 1. Introduction
Alpenblick Consulting AG is a Swiss consulting company with 10 employees. The company provides professional advisory services to small and medium-sized businesses in areas such as strategy, process improvement, and digital transformation. Due to its continuous growth, the company aims to optimize its recruitment process in order to attract qualified talent more efficiently, reduce administrative effort, and support future expansion.

# 2. AS IS Process

<img width="1826" height="347" alt="grafik" src="https://github.com/user-attachments/assets/0e76de74-e07f-4740-9437-8773be933462" />
## 2.1 Process Description
The recruitment process begins when a new position is requested within the organization. The HR Manager first defines the position, specifying requirements and responsibilities, and then advertises the position in a web portal to attract potential candidates.

Once an applicant submits their application, the HR Manager receives the application and gathers it in the web portal. The HR Recruiter then reviews the submitted CV and evaluates whether the application is suitable.

If the application is deemed unsuitable, the HR Recruiter sends a cancellation notice to the applicant, and the process ends with a denied application.

If the application is suitable, the Hiring Manager invites the candidate to a first interview, which is subsequently conducted and evaluated. Based on this evaluation, the Hiring Manager decides whether to invite the candidate to an assessment.

If the candidate is not invited to the assessment, a cancellation is sent and the process ends. Otherwise, the Hiring Manager invites the candidate to an assessment, which is then conducted and evaluated in detail.

Following the assessment, the Hiring Manager decides whether to hire the candidate. If the decision is negative, a cancellation notice is sent to the applicant. If the decision is positive, the HR Manager creates an employment contract and sends it to the applicant.

Once the signed contract is received back from the applicant, the HR Manager proceeds to onboard the new employee, concluding the process with the employee being successfully onboarded into the organization.

## 2.2 Identified Challenges of the AS-IS Process



# 3. TO BE Process
The TO-BE recruitment process introduces significant automation across all stages, reducing manual effort and standardizing decision-making through system integrations.
<img width="1703" height="724" alt="grafik" src="https://github.com/user-attachments/assets/7023e318-3a2d-40fe-a628-3b9ddd9bedf1" />

## 3.1 Process Description
### Job Posting Sub-Process
The process begins when the HR Manager requests a new position. The manager first selects the position from a predefined form, after which a DMN-based business rule task automatically retrieves the relevant position details. The HR Manager then reviews and completes the position details before the system automatically posts the job to the relevant portals. After a 30-day timer, the job postings are automatically closed, ending this sub-process.
### Application & Screening
When an applicant submits their application, the system automatically receives and collects the application, extracting all relevant data, such as years of experience, education level, and language skills, and storing them in a structured format (e.g., Google Sheets). Rather than a manual CV review, a Python script automatically scores the CV on a scale of 0–100 based on defined criteria. The system then evaluates whether the application is suitable.
If the score is too low, the system automatically sends a personalised cancellation e-mail to the applicant via Gmail, and the process ends with a denied application.
### Interview & Assessment
If the application is suitable, the system automatically invites the candidate to a first interview, sending a scheduling link (via Calendly). The Hiring Manager conducts the first interview using a structured digital form, capturing scores for technical skills, cultural fit, and an overall recommendation. Immediately after, the Hiring Manager evaluates the candidate using a standardised rating form.
Based on this evaluation, the Hiring Manager decides whether to invite the candidate to an assessment. If not, a cancellation is sent automatically and the process ends. If yes, the system automatically sends an assessment invitation, after which the Hiring Manager conducts the assessment and subsequently assesses the candidate in detail.
### Hiring Decision & Onboarding
Following the assessment, a decision is made on whether to hire the candidate. If the decision is negative, a cancellation is sent automatically. If positive, the HR Manager creates the employment contract via a Camunda form, entering details such as salary, start date, and working hours. The system then automatically generates and sends the contract to the candidate — using a Google Docs template and a DocuSign link.
Once the signed contract is received back from the applicant, the system automatically triggers the full onboarding workflow, including creating a Google Workspace account, sending a welcome e-mail, notifying the team via Slack, creating onboarding tasks in Asana, and scheduling calendar meetings. The process concludes with the employee being successfully onboarded.




## Key Features
- Supports **structured** and **unstructured** recruitment activities
- Enables **team collaboration** and **stakeholder decision-making**
- Models real-world hiring scenarios end-to-end
