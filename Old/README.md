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
The current recruitment process relies heavily on manual work and individual judgment, creating several significant challenges.
### Fully Manual Application Handling
In the AS-IS process, the HR Manager manually monitors the web portal for incoming applications and individually gathers each submission. There is no automated notification or data extraction in place. This means that application data must be collected, organised, and transferred by hand, which is both time-consuming and prone to oversight — particularly when application volumes are high.
### Subjective CV Screening
The HR Recruiter manually reviews each CV and decides purely based on personal judgment whether an application is suitable. Without defined scoring criteria or a standardised evaluation framework, this step introduces significant subjectivity. Two recruiters may assess the same CV differently, leading to inconsistent and potentially unfair screening outcomes.
### Manual Interview Scheduling and Communication
The Hiring Manager manually sends interview invitations and assessment invitations to candidates via e-mail. Similarly, cancellation notices are written and sent individually by the HR Recruiter. This manual communication approach creates a risk of delays, oversights, and inconsistent messaging, which negatively affects the candidate experience and the organisation's professional image.
### Unstructured Interview and Evaluation Process
Both the first interview and the subsequent candidate evaluation are conducted without a standardised format. Feedback and assessments are not captured in a unified system, making it difficult to compare candidates objectively or to revisit evaluation results at a later stage. This lack of structure increases the risk of biased hiring decisions.
### Manual Contract Creation and Distribution
Once a hiring decision is made, the HR Manager manually creates the employment contract, typically in a word processor, and sends it to the candidate via e-mail. This process is not only slow but also susceptible to errors in contract content, version inconsistencies, and delays in delivery or follow-up.
### Manual Onboarding Coordination
The final step, onboarding the new employee, is coordinated entirely by hand. The HR Manager must individually arrange IT access, workspace setup, team introductions, and other onboarding activities. Without automation or a structured workflow, onboarding steps can be forgotten, delayed, or inconsistently executed depending on the individual responsible.


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
Following the assessment, a decision is made on whether to hire the candidate. If the decision is negative, a cancellation is sent automatically. If positive, the HR Manager creates the employment contract via a Camunda form, entering details such as salary, start date, and working hours. The system then automatically generates and sends the contract to the candidate, using a Google Docs template and a DocuSign link.
Once the signed contract is received back from the applicant, the system automatically triggers the full onboarding workflow, including creating a Google Workspace account, sending a welcome e-mail, notifying the team via Slack, creating onboarding tasks in Asana, and scheduling calendar meetings. The process concludes with the employee being successfully onboarded.

## 3.2 Improvements and Benefits of the TO-BE Recruitment Process
### Automated Application Collection and Structuring
In the TO-BE process, incoming applications are automatically detected and collected by the system, replacing the manual monitoring and gathering performed by the HR Manager in the AS-IS process. All relevant applicant data, such as years of experience, education level, and language skills, are automatically extracted and stored in a structured format. This eliminates manual data entry, reduces the risk of lost or overlooked applications, and significantly speeds up the early stages of the recruitment process.
### Objective and Standardised CV Scoring
One of the most significant improvements is the replacement of manual CV screening with an automated CV scoring system. A Python-based script evaluates each application against defined, weighted criteria and produces a score between 0 and 100. This removes individual bias from the screening decision, ensures every candidate is assessed against the same standard, and enables faster, more consistent filtering of unsuitable applications. The result is a fairer and more transparent initial selection process.
### Automated Candidate Communication
All candidate-facing communications, including interview invitations, assessment invitations, cancellation notices, and contract delivery, are handled automatically by the system. Interview invitations include a self-service scheduling link, allowing candidates to book their preferred time slot without back-and-forth e-mail exchanges. Cancellations are sent as personalised, automatically generated messages. This ensures timely, consistent, and professional communication at every stage, greatly improving the overall candidate experience.
### Structured and Digitalised Interview Evaluation
The TO-BE process introduces standardised digital forms for conducting and evaluating interviews. The Hiring Manager captures structured feedback, including scores for technical skills, cultural fit, and an overall recommendation, directly within the system. This replaces informal notes on paper or in personal spreadsheets, ensuring that all evaluation data is centrally stored, comparable, and auditable. Hiring decisions are therefore better informed and more defensible.
### Automated Contract Generation and Distribution
Rather than manually drafting contracts in a word processor, the HR Manager completes a structured digital form with the relevant contract details. The system then automatically generates the contract using a predefined template, creates a digital signing link, and sends it directly to the candidate. This reduces the time between hiring decision and contract delivery, minimises the risk of errors in contract content, and provides a seamless digital signing experience for the candidate.
### Automated Onboarding Workflow
Upon receipt of the signed contract, the system automatically triggers a comprehensive onboarding workflow. This includes creating the new employee's work account, sending a welcome e-mail, notifying the team, creating onboarding tasks, and scheduling introductory meetings. All without manual intervention. This ensures that every new employee receives a consistent and complete onboarding experience, regardless of who is responsible for the process on a given day.
### Automated Job Posting Management
The TO-BE process also introduces an automated job posting sub-process. Once the HR Manager selects and confirms the position details, the system automatically publishes the job advertisement to the relevant portals. After a defined period of 30 days, the postings are closed automatically. This eliminates the need for manual advertising and follow-up, reducing administrative overhead and ensuring postings are always managed consistently.


## Key Features
- Supports **structured** and **unstructured** recruitment activities
- Enables **team collaboration** and **stakeholder decision-making**
- Models real-world hiring scenarios end-to-end
