# projectAppdev_eme_documentary

Advanced-Local-Civil-Registration-Infromation-System (ALCRIS)

An automated web-based civil registry application designed to streamline vital statistics management—specifically Birth and Death Registrations—featuring a role-based approval pipeline between Registry Staff and System Administrators.  

1. System Overview
The system standardizes municipal document filings by enforcing a two-tier review hierarchy:  

Staff Members: Draft civil registry records (categorized as On-time or Delayed), track review statuses in real time, and submit applications for verification.  

Administrators: Oversee staff accounts, examine and audit submissions, provide revision feedback, and approve or reject registry documents.  

The user interface uses the official municipal palette—Forest/Emerald Green and Golden Yellow/Amber—complete with status badges, interactive forms, and analytical area/doughnut charts.

2. Technology Stack & Architecture
Frontend: Vue 3 (Composition API / <script setup>), Tailwind CSS, Chart.js / vue-chartjs.  

Backend: Laravel 13.29 (PHP), Eloquent ORM.  

Inertia.js / API Layer: Inertia.js for server-driven single-page architecture and Laravel Sanctum for session/token authorization.  

Database: MySQL / MariaDB.  

Development Model: Incremental Development Model.  

┌────────────────────────────────────────────────────────┐
│               Vue 3 Frontend (Inertia)                 │
│  - Staff Dashboard (Green & Yellow Theme)              │
│  - Dynamic Birth/Death Registration Modals             │
│  - Interactive Area Trend & Pipeline Ratio Charts      │
└─────────────────────────▲──────────────────────────────┘
                          │ HTTP / Inertia Props & Forms
┌─────────────────────────▼──────────────────────────────┐
│                    Laravel Backend                     │
│  - Role Middleware (Admin vs. Staff)         │
│  - BirthRecordController & DeathRecordController       │
│  - Sanctum Session & Auth Handler            │
└─────────────────────────▲──────────────────────────────┘
                          │ Eloquent ORM
┌─────────────────────────▼──────────────────────────────┐
│                    MySQL Database                      │
│  - users (roles: admin, staff)               │
│  - birth_records (tracking_id, type, status, feedback) │
│  - death_records (tracking_id, cause, status, etc.)    │
└────────────────────────────────────────────────────────┘
3. Core Modules & Permissions
Role-Based Access Control (RBAC)
Admin: Full CRUD privileges over all registrations, account creation/management for staff members, approval/rejection decision engine, and system audit monitoring.  

Staff: Limited CRUD (Create drafts, Read own filings, Update rejected records based on feedback), submission for Admin approval, and profile management.  

Civil Document Pipeline
Every document moves through a 4-step state machine:  

[ New Filing ] ──► [ Draft (Saved) ] ──► [ Pending Approval ] ──┬──► [ Approved (Official) ]
                                                ▲                │
                                                └── [ Rejected ] ◄
                                                  (Needs Revision)
Draft: Saved work-in-progress, editable by the creator.  

Pending Approval: Submitted to the Admin review queue; locked for staff editing.  

Approved: Verified and cleared for municipal issuance.  

Rejected / Returned: Flagged with administrative notes explaining required revisions (e.g., missing attachments, misspelled names).  

4. Database Schema Specification
birth_records Table
Column	Type	Attributes / Constraints	Description
id	BIGINT	Auto Increment, Primary Key	Unique record ID
tracking_id	VARCHAR(50)	Unique, Indexed	Formatted tracking code (e.g., BR-2026-0091)
user_id	BIGINT	Foreign Key (users.id), Cascades	Staff member who created the record
child_first_name	VARCHAR(100)	Required	Child's given name
child_middle_name	VARCHAR(100)	Nullable	Child's middle name
child_last_name	VARCHAR(100)	Required	Child's surname
gender	ENUM	'male', 'female'	Biological sex
date_of_birth	DATE	Required	Birth date
registration_type	ENUM	'On-time', 'Delayed'	
Filing category  

status	ENUM	'draft', 'pending', 'approved', 'rejected'	
Current lifecycle stage  

admin_feedback	TEXT	Nullable	
Administrative remarks on rejection  

created_at / updated_at	TIMESTAMP	Default current timestamp	Audit timestamps
death_records Table
Column	Type	Attributes / Constraints	Description
id	BIGINT	Auto Increment, Primary Key	Unique record ID
tracking_id	VARCHAR(50)	Unique, Indexed	Formatted tracking code (e.g., DR-2026-0042)
user_id	BIGINT	Foreign Key (users.id), Cascades	Staff member who created the record
deceased_first_name	VARCHAR(100)	Required	Deceased individual's given name
deceased_middle_name	VARCHAR(100)	Nullable	Deceased individual's middle name
deceased_last_name	VARCHAR(100)	Required	Deceased individual's surname
gender	ENUM	'male', 'female'	Biological sex
date_of_death	DATE	Required	Date of death
cause_of_death	VARCHAR(255)	Required	Certified medical cause of death
registration_type	ENUM	'On-time', 'Delayed'	
Filing category  

status	ENUM	'draft', 'pending', 'approved', 'rejected'	
Current lifecycle stage  

admin_feedback	TEXT	Nullable	
Administrative remarks on rejection  

created_at / updated_at	TIMESTAMP	Default current timestamp	Audit timestamps
5. Development Increments & Milestones
Increment 1: Authentication & User Accounts

Multi-role login screen (Admin vs. Staff).  

Account creation & role assignment.  

Password recovery flow.  

Laravel Sanctum token/session integration.  

Increment 2: Document Registration (Birth & Death Records)

Document drafting engine for birth and death certificates.  

Classification of submissions into On-time or Delayed.  

Request submission pipeline to Administrator queue.  

Status tracking table with dynamic search and type filters.  

Increment 3: Data Analytics & Reporting

Dashboard visualizations (Area trend charts, workflow distribution doughnut charts).  

PDF and Excel exports for municipal census and reporting.  

Multi-variable filtering (Record Type, Year, Registration Status).  

Increment 4: Registry Expansion

Marriage Registration drafting and verification workflows.  

Cross-registry reporting analytics.  

Increment 5: Advanced Features & Client Demo

Automated SMS/Email alerts for approval/rejection updates.  

Audit logging tracking every document mutation.   

6. Git Branching & Project Management
Branching Strategy
main: Production-ready, stable codebase. Merge to main strictly after a complete milestone/increment is tested.  

dev: Integration branch for active feature development.  

feature-*: Dedicated feature branches branched from dev (e.g., feature-welcome).  

Workflow Protocol
Create a task branch: git checkout -b feature-welcome dev.  

Commit code following standard descriptive commit syntax.

Push to GitHub and submit a Pull Request (PR) targeted at dev.  

Perform code review and resolve merge conflicts prior to merging into dev.  

After completing an entire increment, the Lead/Admin creates a PR from dev to main.  

Task Management (Trello Board)
Backlog: Master backlog of system features and upcoming registry document types.  

To Do: Sprint-specific components (e.g., "Implement Death Certificate Validation").

In Progress: Features under active local development.

Review: Code undergoing PR reviews, testing, or feedback verification.

Done: Merged into dev or deployed.

7. Local Setup & Installation
Prerequisites
PHP >= 8.2 with OpenSSL, PDO, and Mbstring extensions

Composer >= 2.x

Node.js >= 18.x & NPM

MySQL / MariaDB Server

Installation Steps
Clone the repository:

Bash
https://github.com/ArmieJoy/Advanced-Local-Civil-Registration-Information-System.git

cd Advanced-Local-Civil-Registration-Information-System
Install backend dependencies:

Bash
composer install
Install frontend dependencies (including Chart.js):

Bash
npm install
npm install chart.js vue-chartjs
Environment configuration:

Bash
cp .env.example .env
php artisan key:generate
Configure database variables in .env:

Code snippet
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=alcris_db
DB_USERNAME=root
DB_PASSWORD=
Run database migrations and seeders:

Bash
php artisan migrate --seed
Start local development servers:

Bash
# Terminal 1: Backend
php artisan serve

# Terminal 2: Vite Frontend
npm run dev
