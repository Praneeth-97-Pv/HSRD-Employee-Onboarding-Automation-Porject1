# 🧩 HRSD Employee Onboarding — Scoped Application

---

## 📌 Project Overview

This application automates the employee onboarding lifecycle using ServiceNow's HR Service Delivery (HRSD) module. It orchestrates provisioning tasks across IT, Facilities, and Payroll departments using HR Cases, HR Tasks, and Lifecycle Events — all triggered automatically upon new hire approval.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| **Scoped Application** | Isolated namespace (`x_hrsd_onboarding`) with controlled dependencies |
| **HR Cases & Tasks** | Auto-created onboarding cases with department-specific task routing |
| **Lifecycle Events** | Triggers provisioning steps on hire date milestones |
| **Flow Designer** | Parallel multi-team task assignment with SLA milestone tracking |
| **COE Routing** | Centre of Excellence rules to assign work to correct HR teams |
| **Service Portal** | Employee-facing portal with real-time case status and checklist |
| **Custom Widgets** | Onboarding checklist, document upload, and case tracker widgets |
| **ACLs & RBAC** | Field-level access controls restricting salary and personal data |
| **REST Integration** | Mock Active Directory integration for auto account provisioning |

---

## 🏗️ Architecture

```
New Hire Record Created
        │
        ▼
 HR Case (Onboarding)
        │
   ┌────┴────┐
   │  Flow   │ ──── COE Routing Rules
   │Designer │
   └────┬────┘
        │
   ┌────┴──────────────────────┐
   │                           │
   ▼                           ▼
IT Tasks                  Facilities Tasks
(Account Provisioning,    (Badge, Desk,
 Laptop, Software)         Parking Access)
        │
        ▼
  REST API Call ──► Mock Active Directory
  (Account auto-provisioned on case approval)
        │
        ▼
  Payroll Tasks
  (Bank details, Tax setup)
        │
        ▼
  Employee Service Portal
  (Checklist visibility, Doc uploads, Status)
```

---

## 📁 Repository Structure

```
hrsd-onboarding/
├── README.md
├── docs/
│   ├── architecture.md          # Detailed system design
│   ├── setup-guide.md           # PDI installation steps
│   ├── acl-matrix.md            # Role & field access reference
│   └── screenshots/             # UI screenshots from PDI
├── update-sets/
│   └── hrsd_onboarding_v1.xml   # Exportable ServiceNow Update Set
├── scripts/
│   ├── business-rules/
│   │   ├── create_onboarding_tasks.js
│   │   └── notify_manager_on_completion.js
│   ├── client-scripts/
│   │   └── validate_start_date.js
│   ├── script-includes/
│   │   └── OnboardingUtils.js
│   └── rest-integration/
│       └── active_directory_provisioning.js
├── flow-designer/
│   └── onboarding_flow_spec.md  # Flow Designer configuration spec
├── widgets/
│   ├── onboarding-checklist/
│   │   ├── widget.html
│   │   ├── widget.js
│   │   └── widget.scss
│   ├── document-upload/
│   │   ├── widget.html
│   │   ├── widget.js
│   │   └── widget.scss
│   └── case-status-tracker/
│       ├── widget.html
│       ├── widget.js
│       └── widget.scss
├── acls/
│   └── acl_definitions.md
├── service-portal/
│   └── portal_config.md
└── .github/
    └── workflows/
        └── validate-xml.yml
```

---

## 🚀 Getting Started

### Prerequisites
- ServiceNow Personal Developer Instance (PDI) — [Request free at developer.servicenow.com](https://developer.servicenow.com)
- HRSD plugin activated on your PDI
- GitHub account for source control integration

### Installation via Update Set

1. Clone this repo
2. In your PDI: **System Update Sets → Retrieved Update Sets → Import Update Set from XML**
3. Upload `update-sets/hrsd_onboarding_v1.xml`
4. Preview and Commit the Update Set
5. Navigate to **HRSD → Administration** to verify the app loaded

### Installation via Studio Source Control (Recommended)

1. In your PDI, open **Studio**
2. Go to **Source Control → Import from Source Control**
3. Enter this repository URL
4. Branch: `main`
5. Studio will pull all application files automatically

---

## 🔐 Security Design

### Role Hierarchy

| Role | Access |
|---|---|
| `x_hrsd_onboarding.hr_admin` | Full access to all HR Cases, salary, personal data |
| `x_hrsd_onboarding.hr_agent` | HR Cases and Tasks — no salary fields |
| `x_hrsd_onboarding.it_agent` | IT Tasks only |
| `x_hrsd_onboarding.facilities_agent` | Facilities Tasks only |
| `x_hrsd_onboarding.payroll_agent` | Payroll Tasks + salary fields |
| `x_hrsd_onboarding.employee` | Own onboarding portal view only |

### Field-Level ACLs
- `u_salary`, `u_bank_account`, `u_tax_id` — restricted to `hr_admin` and `payroll_agent` only
- `u_personal_address`, `u_emergency_contact` — restricted from IT and Facilities agents
- See [`acls/acl_definitions.md`](acls/acl_definitions.md) for the full matrix

---

## 🔗 REST Integration — Mock Active Directory

The app calls a mock AD REST endpoint on **HR Case approval**:

```
POST /api/mock-ad/provision-user
Authorization: Basic (stored in ServiceNow Credential Store)
Content-Type: application/json

{
  "employee_id": "EMP-001",
  "first_name": "Manasa",
  "last_name": "Reddy",
  "department": "Engineering",
  "start_date": "2025-08-01",
  "manager_email": "manager@company.com"
}
```

Response triggers account creation status update back on the HR Case.

See [`scripts/rest-integration/active_directory_provisioning.js`](scripts/rest-integration/active_directory_provisioning.js) for the full Script Include.

---

## 📊 Flow Designer — Onboarding Flow

The main flow (`Onboarding Orchestration Flow`) runs on HR Case creation with subject `New Hire Onboarding`:

1. **Trigger**: HR Case created with Lifecycle Event = `New Hire`
2. **Stage 1 — Parallel**: Create IT Tasks + Facilities Tasks simultaneously
3. **Stage 2 — Wait**: SLA milestone — all Stage 1 tasks complete
4. **Stage 3**: REST call to Active Directory for account provisioning
5. **Stage 4**: Create Payroll Tasks
6. **Stage 5**: Notify employee via email + Service Portal notification
7. **Stage 6 — Wait**: All tasks complete
8. **Stage 7**: Close HR Case, trigger Lifecycle Event `Onboarding Complete`

---

## 🖥️ Service Portal

**URL**: `[https://[your-pdi].service-now.com/onboarding](https://dev400294.service-now.com/now/servicenow-studio/builder%3Ftable%3Dsys_app%26sysId%3De00ded0e971d0310d0513f2ad053af71%26)`

### Pages
| Page | Description |
|---|---|
| `/onboarding/home` | Employee welcome page with progress overview |
| `/onboarding/checklist` | Interactive task checklist widget |
| `/onboarding/documents` | Document upload widget (ID proof, tax forms) |
| `/onboarding/status` | Real-time HR Case status tracker |


## 🛠️ Tech Stack

- **Platform**: ServiceNow (Washington DC / Xanadu release)
- **Module**: HR Service Delivery (HRSD)
- **Tools Used**: Flow Designer, Studio, Service Portal, REST Message, Update Sets
- **Languages**: JavaScript (ServiceNow GlideScript), AngularJS (Service Portal widgets), HTML/SCSS

---


