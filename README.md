# Lola - HR Contract Management Chatbot

An AI-powered HR assistant that guides users through employment contract workflows, generates documents, and maintains audit trails.

## Overview

Lola is an intelligent chatbot designed to assist HR and Management with:

- Guiding users through structured questions
- Determining applicable HR scenarios (roadmaps)
- Generating new or amended employment contracts (Word/PDF)
- Documenting salary or workload changes in tracked records
- Sending drafts via email for review
- Maintaining governance with human-in-the-loop control

## Features

- **Role-based Access Control**: Admin and HR/CU user roles
- **Multi-language Support**: German (default), English, and Spanish
- **AI-Guided Workflow**: LLM-powered conversational interface
- **Document Generation**: Automated contract and amendment creation
- **Audit Trail**: Full traceability with timestamps and actors
- **Excel Export**: Downloadable reports in Excel format
- **Email Integration**: Draft sending for approval workflows
- **PDF Processing**: Upload and extract data from existing contracts

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │  Login   │  │  Admin   │  │  HR/CU   │  │    Chatbot UI    │ │
│  │  Page    │  │  Panel   │  │  Panel   │  │   (Lola Agent)   │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Flask Backend                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │   Auth   │  │ Workflow │  │ Document │  │   Email Service  │ │
│  │ Service  │  │  Engine  │  │ Generator│  │                  │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      AI/LLM Layer                               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Lola Agent                            │   │
│  │  - Guided question flow                                  │   │
│  │  - Input validation                                      │   │
│  │  - Scenario determination                                │   │
│  │  - Template population                                   │   │
│  │  - Summary generation                                    │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Data Layer (SQLite)                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │  Users   │  │Contracts │  │ Changes  │  │   Audit Logs     │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
lola-hr-chatbot/
├── app/
│   ├── __init__.py              # Flask app factory
│   ├── config.py                # Configuration settings
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py              # User model (roles, auth)
│   │   ├── employee.py          # Employee data model
│   │   ├── contract.py          # Contract model
│   │   ├── change_request.py    # Salary/workload changes
│   │   ├── audit_log.py         # Audit trail model
│   │   └── contract_template.py # DOCX templates for Lola
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── auth.py              # Login/logout views
│   │   ├── admin.py             # Admin panel views
│   │   ├── hr.py                # HR/CU panel views
│   │   ├── chatbot.py           # Chatbot interaction views
│   │   └── documents.py         # Document generation/download views
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth_service.py      # Authentication logic
│   │   ├── workflow_service.py  # Decision tree logic
│   │   ├── document_service.py  # Document generation
│   │   ├── email_service.py     # Email sending
│   │   ├── excel_service.py     # Excel export
│   │   └── pdf_service.py       # PDF processing
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── lola_agent.py        # Main AI agent
│   │   ├── prompts.py           # LLM prompt templates
│   │   └── tools.py             # Agent tools (DB access, etc.)
│   └── templates/
│       ├── contracts/           # Contract Word templates
│       └── emails/              # Email templates
├── static/
│   ├── css/
│   └── js/
├── templates/                   # Jinja2 HTML templates
│   ├── base.html
│   ├── auth/
│   ├── admin/
│   ├── hr/
│   └── chatbot/
├── migrations/                  # Database migrations
├── tests/
├── requirements.txt
├── .env.example
└── run.py
```

## User Roles

### Admin
- View transaction history
- Add new users
- System configuration
- Full audit access

### HR/CU (HR and Management)
- Upload employee contract PDFs
- Start chatbot interaction
- View employees contract table
- Generate and download documents
- Send drafts via email

## Core Decision Logic

### Step 1: Type of Change
Lola asks: "What do you want to do?"

Options:
1. **New employment**
2. **Change to existing employment**
   - Salary change only
   - Workload (Stellenprozent) change
   - Salary and workload change
   - Contractual changes (role, notice period, etc.)

### Scenario to Output Mapping

| Scenario                | Contract Generated?     | Excel Updated? |
|-------------------------|------------------------|----------------|
| New employment          | Yes (new contract)     | No             |
| Salary change only      | No                     | Yes            |
| Workload change         | No                     | Yes            |
| Salary + Workload       | Yes (amendment)        | Yes            |
| Legal/structural change | Yes (new contract)     | Optional       |

## Chatbot Question Flow

1. Who is the employee?
2. Is this a new employment or a change?
3. If change: What kind of change?
4. Effective date?
5. Old values (salary, %, contract reference)
6. New values
7. Should a contract/amendment be issued?
8. Final confirmation summary (human-readable)

**After confirmation:**
- Generate document
- Update database/Excel
- Send email
- Download document

## Data Models

### Contract Amendment Tracking
| Field                  | Description                              |
|------------------------|------------------------------------------|
| Name                   | Employee name                            |
| Old Contract           | Previous contract reference              |
| New Contract           | New contract reference                   |
| Old Start Date         | Original employment start date           |
| New Contract Start Date| Effective date of new contract           |
| Old FTE Percentage     | Previous full-time equivalent percentage |
| New FTE Percentage     | New full-time equivalent percentage      |

### Salary/Workload Change Tracking
| Field                  | Description                              |
|------------------------|------------------------------------------|
| Name                   | Employee name                            |
| New Salary Effective   | Date when new salary takes effect        |
| Change Type            | Salary or percentage change              |
| Old FTE Percentage     | Previous full-time equivalent percentage |
| New FTE Percentage     | New full-time equivalent percentage      |
| Old Pro-rata Salary    | Salary proportional to old FTE           |
| New Pro-rata Salary    | Salary proportional to new FTE           |

## Contract Template Variables

The LLM populates these placeholders in contract templates:

```
{{Employee_Name}}
{{Role}}
{{Entry_Date}}
{{Effective_Date_Change}}
{{Workload_Percent}}
{{Weekly_Hours}}
{{Annual_Salary}}
{{Monthly_Salary}}
{{Gender}}                  # For correct salutation
{{Job_Title}}
```

**Static content** (never modified by AI):
- Legal clauses
- Insurance wording
- Pension terms
- Confidentiality agreements
- Notice periods

## AI Agent Guidelines

The LLM (Lola) operates as a **guided assistant**, not a decision-maker:

- Guides users through structured, predefined questions
- Collects and validates input data
- Determines applicable HR scenario
- Populates predefined data schemas and templates
- Generates human-readable summaries for review

**Important constraints:**
- LLM does NOT create or modify contractual clauses
- Only fills approved placeholders
- All legal text and business rules remain deterministic
- Final decisions require human approval

## Governance & Control

- **Draft-only generation**: Documents are always drafts until approved
- **Email-based approval**: No automatic sending to employees
- **Status workflow**: Only allow "send" when status = APPROVED
- **Audit trail**: Every action logged with:
  - Unique change ID per case
  - Timestamps
  - Actor (user who performed action)
  - "Created by Lola" vs "Manually edited" flags

## Technology Stack

- **Backend**: Flask (Python)
- **Database**: SQLite (single source of truth)
- **AI/LLM**: OpenAI API / Anthropic Claude
- **Document Generation**: python-docx, reportlab
- **Excel Export**: openpyxl
- **PDF Processing**: PyPDF2, pdfplumber
- **Email**: Flask-Mail
- **Authentication**: Flask-Login
- **Internationalization**: Flask-Babel
- **Frontend**: Jinja2, Bootstrap, JavaScript

## Language Selection (i18n)

The application supports multiple languages on the login page:

| Language | Code | Status |
|----------|------|--------|
| Deutsch (German) | `de` | Default |
| English | `en` | Supported |
| Espanol (Spanish) | `es` | Supported |

### How It Works

1. **Login Page**: Users can select their preferred language using the language selector buttons
2. **Session Storage**: The selected language is stored in the user's session
3. **Default Language**: German (`de`) is the default language if no preference is set
4. **Persistence**: The language preference persists throughout the session

### Language Selection Route

```
GET /language/<lang>
```

Where `<lang>` is one of: `de`, `en`, `es`

### Adding More Languages

To add a new language:

1. Update `LANGUAGES` dictionary in `app/__init__.py`:
   ```python
   LANGUAGES = {
       'de': 'Deutsch',
       'en': 'English',
       'es': 'Espanol',
       'fr': 'Francais',  # Add new language
   }
   ```

2. Add translations in `app/templates/auth/login.html`:
   ```python
   {% set translations = {
       ...
       'fr': {
           'welcome': 'Bienvenue a Lola',
           'subtitle': 'Systeme de Gestion des Contrats RH',
           ...
       }
   } %}
   ```

## Installation

```bash
# Clone repository
git clone <repository-url>
cd lola-hr-chatbot

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Initialize database (first time only)
flask db upgrade

# Run application
flask run
```

## Running the Application

### Start the server

```bash
# Activate virtual environment
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Option A: Using flask run
flask run

# Option B: Using python directly
python run.py
```

The app will run at `http://127.0.0.1:5000`

### Stop the server

Press **`Ctrl + C`** in the terminal where the server is running.

## Database Migrations

This project uses Flask-Migrate (Alembic) to handle database schema changes. **Always use migrations when modifying models** to avoid data loss.

### When you modify a model:

```bash
# 1. Create a new migration
flask db migrate -m "Description of the change"

# 2. Review the generated migration file in migrations/versions/

# 3. Apply the migration
flask db upgrade
```

### Useful migration commands:

```bash
# View current migration status
flask db current

# View migration history
flask db history

# Rollback the last migration
flask db downgrade

# Rollback to a specific revision
flask db downgrade <revision_id>
```

## Environment Variables

```env
FLASK_APP=run.py
FLASK_ENV=development
SECRET_KEY=your-secret-key
DATABASE_URL=sqlite:///lola.db

# LLM Configuration
OPENAI_API_KEY=your-openai-key
# or
ANTHROPIC_API_KEY=your-anthropic-key

# Email Configuration
MAIL_SERVER=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your-email
MAIL_PASSWORD=your-password
```

## Application Pages

### Authentication
- **Login Page**: User authentication with role-based redirect and language selection

### Admin Panel
- **Dashboard**: Overview of system activity
- **User Management**: Add/edit users and assign roles
- **Transaction History**: View complete audit log

### HR/CU Panel
- **Main Dashboard**: Quick access to all HR functions
- **Upload Contracts**: Upload existing employee contract PDFs for processing
- **Employees Table**: View all employees and their contract status
- **Contract History**: Track all contract changes and amendments

### Settings
- **Company Information**: Configure company details for contracts
- **Company Representatives**: Manage authorized signers
- **Contract Templates**: Upload DOCX templates for contract generation (see below)

### Chatbot Interface
- **Chat with Lola**: Interactive conversation for HR workflows
- **Confirmation Summary**: Review before generating documents
- **Document Preview**: View generated draft before approval

### Document Management
- **Download Center**: Download generated contracts (Word/PDF)
- **Email Draft**: Send documents for review and approval
- **Excel Export**: Export tracking data to Excel format

## Contract Templates

HR users can upload DOCX templates that Lola uses to generate contracts. Each template has a type and description to help Lola select the appropriate template.

### Managing Templates

Access via: **Settings → Contract Templates**

| Route | Description |
|-------|-------------|
| `/hr/settings/contract-templates` | View all uploaded templates |
| `/hr/settings/contract-templates/upload` | Upload a new DOCX template |

### Template Fields

| Field | Description |
|-------|-------------|
| **File** | DOCX template file (max 16MB) |
| **Contract Type** | Template category (e.g., `new`, `amendment`, `termination`, `probation`) |
| **Description** | Explains when this template should be used |

### API for Lola Agent

Lola can retrieve available templates via the API:

```
GET /hr/api/contract-templates
```

Returns JSON with all active templates:
```json
{
  "templates": [
    {
      "id": 1,
      "name": "new_contract_v2.docx",
      "contract_type": "new",
      "description": "Standard new employment contract",
      "file_path": "contract_templates/new_contract_v2.docx",
      "is_active": true,
      "created_at": "2026-01-15T10:30:00",
      "created_by": "hr_user"
    }
  ]
}
```

### Storage

Templates are stored using the Storage Service:
- **Local (development)**: `./uploads/contract_templates/`
- **S3 (production)**: `uploads/contract_templates/`

## Recent Changes (January 2026)

### PDF Processing with LLM
- Implemented automatic PDF contract processing using LLM (OpenAI/Anthropic)
- Added `process_and_create_employee()` pipeline in `pdf_service.py`
- Extracts employee data from uploaded contracts and creates database records
- Added "Process Uploaded PDFs" button in HR Dashboard

### Security Improvements
- Moved hardcoded credentials to environment variables
- `ADMIN_PASSWORD` and `HR_PASSWORD` are now required in `.env`
- No default passwords in code - application will refuse to initialize without proper configuration

### Chat Interface Improvements
- Added Markdown processor for chat messages
- Chat now properly renders **bold**, *italic*, lists, and line breaks
- Custom Jinja2 `markdown` filter with `nl2br` extension

### Bug Fixes
- Fixed `chatbot.reset` endpoint naming issue
- Updated Anthropic library from 0.8.1 to 0.75.0

### New Dependencies
- `markdown==3.9` - For rendering Markdown in chat interface



