# Advanced Local Civil Registration Information System (ALCRIS)

An automated web-based civil registry application designed to streamline vital statistics management—specifically Birth and Death Registrations—featuring a role-based approval pipeline between Registry Staff and System Administrators.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Technology Stack & Architecture](#2-technology-stack--architecture)
3. [Core Modules & Permissions](#3-core-modules--permissions)
   * [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
   * [Civil Document Pipeline](#civil-document-pipeline)
4. [Database Schema Specification](#4-database-schema-specification)
   * [`birth_records` Table](#birth_records-table)
   * [`death_records` Table](#death_records-table)
5. [Development Increments & Milestones](#5-development-increments--milestones)
6. [Git Branching & Project Management](#6-git-branching--project-management)
   * [Branching Strategy](#branching-strategy)
   * [Workflow Protocol](#workflow-protocol)
   * [Task Management (Trello Workflow)](#task-management-trello-workflow)
7. [Local Setup & Installation](#7-local-setup--installation)

---

## 1. System Overview

The system standardizes municipal document filings by enforcing a two-tier review hierarchy:

* **Staff Members:** Draft civil registry records (categorized as On-time or Delayed), track review statuses in real time, and submit applications for verification.
* **Administrators:** Oversee staff accounts, examine and audit submissions, provide revision feedback, and approve or reject registry documents.

The user interface uses the official municipal palette—Forest/Emerald Green and Golden Yellow/Amber—complete with status badges, interactive forms, and analytical area/doughnut charts.

---

## 2. Technology Stack & Architecture

* **Frontend:** Vue.js (Composition API / `<script setup>`), Tailwind CSS, Chart.js / `vue-chartjs`
* **Backend:** Laravel (PHP), Eloquent ORM
* **Inertia.js / API Layer:** Inertia.js for server-driven single-page architecture and Laravel Sanctum for session/token authorization
* **Database:** MySQL / MariaDB
* **Mobile Prototype:** FlutterFlow (demo for client presentation)
* **Development Model:** Incremental Development Model

```text
┌────────────────────────────────────────────────────────┐
│               Vue 3 Frontend (Inertia)                 │
│  - Staff Dashboard (Green & Yellow Theme)              │
│  - Dynamic Birth/Death Registration Modals             │
│  - Interactive Area Trend & Pipeline Ratio Charts      │
└─────────────────────────▲──────────────────────────────┘
                          │ HTTP / Inertia Props & Forms
┌─────────────────────────▼──────────────────────────────┐
│                    Laravel Backend                     │
│  - Role Middleware (Admin vs. Staff)[cite: 1]         │
│  - BirthRecordController & DeathRecordController       │
│  - Sanctum Session & Auth Handler[cite: 1]            │
└─────────────────────────▲──────────────────────────────┘
                          │ Eloquent ORM
┌─────────────────────────▼──────────────────────────────┐
│                    MySQL Database                      │
│  - users (roles: admin, staff)[cite: 1]               │
│  - birth_records (tracking_id, type, status, feedback) │
│  - death_records (tracking_id, cause, status, etc.)    │
└────────────────────────────────────────────────────────┘
```

---

## 3. Core Modules & Permissions

### Role-Based Access Control (RBAC)

* **Admin:** Full CRUD privileges over all registrations, account creation/management for staff members, approval/rejection decision engine, and system audit monitoring.
* **Staff:** Limited CRUD (Create drafts, Read own filings, Update rejected records based on feedback), submission for Admin approval, and profile management.

### Civil Document Pipeline

Every document moves through a 4-step state machine:

```text
[ New Filing ] ──► [ Draft (Saved) ] ──► [ Pending Approval ] ──┬──► [ Approved (Official) ]
                                                ▲                │
                                                └── [ Rejected ] ◄
                                                  (Needs Revision)
```

* **Draft:** Saved work-in-progress, editable by the creator.
* **Pending Approval:** Submitted to the Admin review queue; locked for staff editing.
* **Approved:** Verified and cleared for municipal issuance.
* **Rejected / Returned:** Flagged with administrative notes explaining required revisions (e.g., missing attachments, misspelled names).

---

## 4. Database Schema Specification

### `birth_records` Table

| Column | Type | Attributes / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | Auto Increment, Primary Key | Unique record ID |
| `tracking_id` | `VARCHAR(50)` | Unique, Indexed | Formatted tracking code (e.g., `BR-2026-0091`) |
| `user_id` | `BIGINT` | Foreign Key (`users.id`), Cascades | Staff member who created the record |
| `child_first_name` | `VARCHAR(100)` | Required | Child's given name |
| `child_middle_name` | `VARCHAR(100)` | Nullable | Child's middle name |
| `child_last_name` | `VARCHAR(100)` | Required | Child's surname |
| `gender` | `ENUM` | `'male'`, `'female'` | Biological sex |
| `date_of_birth` | `DATE` | Required | Birth date |
| `registration_type` | `ENUM` | `'On-time'`, `'Delayed'` | Filing category |
| `status` | `ENUM` | `'draft'`, `'pending'`, `'approved'`, `'rejected'` | Current lifecycle stage |
| `admin_feedback` | `TEXT` | Nullable | Administrative remarks on rejection |
| `created_at` / `updated_at` | `TIMESTAMP` | Default current timestamp | Audit timestamps |

### `death_records` Table

| Column | Type | Attributes / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | Auto Increment, Primary Key | Unique record ID |
| `tracking_id` | `VARCHAR(50)` | Unique, Indexed | Formatted tracking code (e.g., `DR-2026-0042`) |
| `user_id` | `BIGINT` | Foreign Key (`users.id`), Cascades | Staff member who created the record |
| `deceased_first_name` | `VARCHAR(100)` | Required | Deceased individual's given name |
| `deceased_middle_name` | `VARCHAR(100)` | Nullable | Deceased individual's middle name |
| `deceased_last_name` | `VARCHAR(100)` | Required | Deceased individual's surname |
| `gender` | `ENUM` | `'male'`, `'female'` | Biological sex |
| `date_of_death` | `DATE` | Required | Date of death |
| `cause_of_death` | `VARCHAR(255)` | Required | Certified medical cause of death |
| `registration_type` | `ENUM` | `'On-time'`, `'Delayed'` | Filing category |
| `status` | `ENUM` | `'draft'`, `'pending'`, `'approved'`, `'rejected'` | Current lifecycle stage |
| `admin_feedback` | `TEXT` | Nullable | Administrative remarks on rejection |
| `created_at` / `updated_at` | `TIMESTAMP` | Default current timestamp | Audit timestamps |

---

## 5. Development Increments & Milestones

### Increment 1: Authentication & User Accounts
* Multi-role login screen (Admin vs. Staff)
* Account creation & role assignment
* Password recovery flow
* Laravel Sanctum token/session integration

### Increment 2: Document Registration (Birth & Death Records)
* Document drafting engine for birth and death certificates
* Classification of submissions into On-time or Delayed
* Request submission pipeline to Administrator queue
* Status tracking table with dynamic search and type filters

### Increment 3: Data Analytics & Reporting
* Dashboard visualizations (Area trend charts, workflow distribution doughnut charts)
* PDF and Excel exports for municipal census and reporting
* Multi-variable filtering (Record Type, Year, Registration Status)

### Increment 4: Registry Expansion
* Marriage Registration drafting and verification workflows
* Cross-registry reporting analytics

### Increment 5: Advanced Features & Client Demo
* Automated SMS/Email alerts for approval/rejection updates
* Audit logging tracking every document mutation
* Mobile Prototype demo built in FlutterFlow

---

## 6. Git Branching & Project Management

### Branching Strategy

* `main`: Production-ready, stable codebase. Merge to `main` strictly after a complete milestone/increment is tested.
* `dev`: Integration branch for active feature development.
* `feature-*`: Dedicated feature branches branched from `dev` (e.g., `feature-login`, `feature-staff-dashboard`).

### Workflow Protocol

1. Create a task branch: `git checkout -b feature-task-name dev`
2. Commit code following standard descriptive commit syntax.
3. Push to GitHub and submit a Pull Request (PR) targeted at `dev`.
4. Perform code review and resolve merge conflicts prior to merging into `dev`.
5. After completing an entire increment, the Lead/Admin creates a PR from `dev` to `main`.

### Task Management (Trello Workflow)

The project tracks development progress using a Kanban-style Trello board aligned with the Incremental Development Model:

```text
Backlog ──► To Do ──► In Progress ──► Code Review ──► Testing ──► Done
```

* **Backlog:** Master holding list for future increment epics, admin features, and unassigned system components.
* **To Do:** Sprint tasks approved for immediate development.
* **In Progress:** Tasks currently being coded (each card is paired with an active Git feature branch).
* **Code Review:** Triggered when a pull request is submitted to `dev` for peer inspection.
* **Testing:** Verification stage for UI consistency, field validations, and database entries.
* **Done:** Code tested and merged into `dev`.

| Trello Column | Associated Cards | Associated Git Branch |
| :--- | :--- | :--- |
| **Done** | • Modified Welcome Screen<br>• Login Page UI & Auth Integration | `feature-welcome`<br>`feature-login` |
| **In Progress** | • Staff Dashboard (UI, Area Chart, & Table)<br>• Sign Up Page & Account Validation<br>• Forgot Password Token Link | `feature-staff-dashboard`<br>`feature-signup`<br>`feature-forgot-password` |
| **To Do** | • Role Management (`admin` vs `staff`)<br>• Session Handling (Laravel Sanctum Tokens)<br>• Birth & Death Registration Modals | `feature-role-management`<br>`feature-session-handling`<br>`feature-doc-registration` |
| **Backlog** | • Admin Document Control (Approve/Reject Workflow)<br>• Audit Logs & Compliance Tracking<br>• PDF/Excel Report Exporter | *Assigned per milestone* |

---

## 7. Local Setup & Installation

### Prerequisites

* PHP >= 8.2 with OpenSSL, PDO, and Mbstring extensions
* Composer >= 2.x
* Node.js >= 18.x & NPM
* MySQL / MariaDB Server

### Installation Steps

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/ArmieJoy/Advanced-Local-Civil-Registration-Information-System.git]
   cd Advanced-Local-Civil-Registration-Information-System
   ```

2. **Install backend dependencies:**
   ```bash
   composer install
   ```

3. **Install frontend dependencies (including Chart.js):**
   ```bash
   npm install
   npm install chart.js vue-chartjs
   ```

4. **Environment configuration:**
   ```bash
   cp .env.example .env
   php artisan key:generate
   ```

5. **Configure database variables in `.env`:**
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=alcris_db
   DB_USERNAME=root
   DB_PASSWORD=
   ```

6. **Run database migrations and seeders:**
   ```bash
   php artisan migrate --seed
   ```

7. **Start local development servers:**
   ```bash
   # Terminal 1: Backend
   php artisan serve

   # Terminal 2: Vite Frontend
   npm run dev
   ```
