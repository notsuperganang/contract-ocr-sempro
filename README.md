# Telkom Contract Data Extractor

> **Final Assignment Project (Tugas Akhir)** | Originally developed during Digistar Class Internship at Telkom Indonesia

A full-stack web application that leverages AI-powered OCR to automatically extract and validate structured data from Indonesian Telkom contract PDF documents. Features a human-in-the-loop validation workflow, batch processing capabilities, and comprehensive contract management system.

[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-20232A?style=flat&logo=react&logoColor=61DAFB)](https://reactjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=flat&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![PaddleOCR](https://img.shields.io/badge/PaddleOCR-FF6600?style=flat&logo=paddle&logoColor=white)](https://github.com/PaddlePaddle/PaddleOCR)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Screenshots](#screenshots)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Usage Guide](#usage-guide)
- [API Documentation](#api-documentation)
- [Database Schema](#database-schema)
- [Development](#development)
- [Deployment](#deployment)
- [Technical Highlights](#technical-highlights)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## Overview

This project transforms the manual contract data entry process into an automated, AI-powered workflow. Originally developed during my **Digistar Class Internship** at **Telkom Indonesia** and subsequently adopted and enhanced as my **final college assignment (Tugas Akhir)**, this application addresses the real-world challenge of efficiently processing and managing contract documents.

### The Problem

Telkom Indonesia processes hundreds of service contracts annually, each containing critical structured data:
- Customer information (schools, companies, institutions)
- Service details (connectivity, non-connectivity, bundling packages)
- Financial terms (installation costs, subscription fees, payment schedules)
- Contract duration and contact persons

Manual data entry is time-consuming, error-prone, and difficult to track.

### The Solution

An intelligent document processing system that:
1. **Extracts** structured data from PDF contracts using PaddleOCR's PP-StructureV3 pipeline
2. **Validates** extracted data through a split-view human-in-the-loop interface
3. **Manages** confirmed contracts with a comprehensive dashboard
4. **Tracks** payment schedules with automatic status updates
5. **Exports** data to Excel for integration with existing systems

---

## Features

### 🚀 Core Capabilities

- **Batch PDF Processing**: Upload and process multiple contract PDFs simultaneously with drag-and-drop interface
- **Advanced OCR**: Leverages PaddleOCR's PP-StructureV3 pipeline optimized for Indonesian contract documents
- **Intelligent Data Extraction**: Sophisticated regex patterns and fuzzy matching for robust field extraction
- **Human-in-the-Loop Validation**: Split-view interface with PDF viewer and editable form for data verification
- **Contract Management Dashboard**: Comprehensive KPI cards, statistics, and analytics
- **Payment Tracking**: Automated termin payment status updates (PENDING → DUE → OVERDUE)
- **Excel Export**: Generate formatted spreadsheet exports for confirmed contracts
- **Real-time Processing Status**: Monitor OCR and extraction progress with detailed status updates

### 🎯 Key Highlights

- **Extraction Accuracy**: Handles complex Indonesian date formats, NPWP validation, and multi-page contracts
- **Flexible Payment Methods**: Supports one-time charge, recurring payments, and complex termin payment schedules
- **Data Integrity**: Manual save workflow prevents data loss; strict validation with Zod schemas
- **Performance Optimized**: Denormalized database columns for fast queries; CPU-optimized OCR with HPI
- **Audit Trail**: Complete processing history and user modification tracking

---

## Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend (React)                        │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │  Dashboard  │  │ Upload Page  │  │  Review Interface      │  │
│  │  (KPI Cards)│  │ (Batch Drop) │  │  (Split View: PDF +    │  │
│  │             │  │              │  │   Editable Form)       │  │
│  └─────────────┘  └──────────────┘  └────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │ REST API
┌─────────────────────────────────────────────────────────────────┐
│                      Backend (FastAPI)                          │
│  ┌──────────────┐  ┌─────────────────┐  ┌──────────────────┐    │
│  │ Upload API   │  │  OCR Service    │  │  Contract API    │    │
│  │ (Multi-file) │  │  (PP-Structure  │  │  (CRUD + Export) │    │
│  │              │  │   V3 Pipeline)  │  │                  │    │
│  └──────────────┘  └─────────────────┘  └──────────────────┘    │
│         │                  │                      │             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │          Data Extractor (Stateless Functions)            │   │
│  │  - Regex patterns for Indonesian contracts               │   │
│  │  - Fuzzy matching for OCR error handling                 │   │
│  │  - Two-page extraction strategy                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Database                          │
│  ┌─────────────┐  ┌──────────────────┐  ┌──────────────────┐    │
│  │   files     │  │ processing_jobs  │  │    contracts     │    │
│  │             │  │  (Queue + Draft) │  │  (Final Data)    │    │
│  └─────────────┘  └──────────────────┘  └──────────────────┘    │
│                          │                                      │
│                  ┌───────────────────────┐                      │
│                  │ contract_term_payments│                      │
│                  │  (Payment Tracking)   │                      │
│                  └───────────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Upload Phase**: PDFs uploaded → Stored in filesystem → `files` table created
2. **Processing Phase**: Background task → OCR with PaddleOCR → Extract with regex → Save to `processing_jobs.extracted_data`
3. **Validation Phase**: User reviews in split-view → Manual edits → Save to `processing_jobs.edited_data`
4. **Confirmation Phase**: User confirms → Create `contracts` record → Sync `contract_term_payments`
5. **Export Phase**: Generate Excel from confirmed contracts

---

## Screenshots

### Dashboard with KPI Cards
> Real-time statistics showing total contracts, revenue, service counts, and collection rates

### Upload Interface
> Drag-and-drop batch upload with file validation and processing queue

### Split-View Validation
> PDF viewer (left) synchronized with editable form (right) for human-in-the-loop verification

### Contract Management
> Paginated table with search, filters, and detailed contract views

### Payment Tracking
> Automatic termin payment status updates with manual override capabilities

---

## Technology Stack

### Backend

| Technology | Purpose | Version |
|------------|---------|---------|
| **FastAPI** | REST API framework | Latest |
| **PostgreSQL** | Primary database with JSONB support | 15+ |
| **SQLAlchemy** | ORM and database migrations | 2.x |
| **Alembic** | Database migration tool | Latest |
| **PaddleOCR** | OCR engine (PP-StructureV3) | Latest |
| **PaddlePaddle** | Deep learning framework | Latest |
| **Pydantic** | Data validation and settings | 2.x |
| **python-jose** | JWT authentication | Latest |
| **pandas** | Data processing for exports | Latest |
| **openpyxl** | Excel file generation | Latest |

### Frontend

| Technology | Purpose | Version |
|------------|---------|---------|
| **React** | UI framework | 19.x |
| **TypeScript** | Type-safe JavaScript | 5.8.x |
| **Vite** | Build tool and dev server | 7.x |
| **Tailwind CSS** | Utility-first styling | 4.x |
| **React Router** | Client-side routing | 7.x |
| **TanStack Query** | Server state management | 5.x |
| **React Hook Form** | Form state management | 7.x |
| **Zod** | Schema validation | 4.x |
| **Radix UI** | Accessible component primitives | Latest |
| **React PDF** | PDF rendering in browser | 10.x |
| **Recharts** | Data visualization | 3.x |
| **Framer Motion** | Animation library | 12.x |

### Infrastructure

- **Docker & Docker Compose**: Containerization and orchestration
- **Nginx**: Reverse proxy (production)
- **GitHub**: Version control

---

## Project Structure

```
telkom-contract-extractor/
├── backend/
│   ├── app/
│   │   ├── api/                    # API route modules
│   │   │   ├── upload.py          # File upload endpoints
│   │   │   ├── processing.py      # Job status and data management
│   │   │   ├── contracts.py       # Contract CRUD operations
│   │   │   ├── dashboard.py       # KPI and analytics endpoints
│   │   │   └── admin.py           # System statistics
│   │   ├── auth/                  # JWT authentication system
│   │   │   ├── routes.py          # Login/register endpoints
│   │   │   └── dependencies.py    # Auth guards
│   │   ├── models/                # Data models
│   │   │   ├── database.py        # SQLAlchemy ORM models
│   │   │   └── schemas.py         # Pydantic validation schemas
│   │   ├── services/              # Business logic
│   │   │   ├── ocr_service.py     # PaddleOCR singleton service
│   │   │   ├── data_extractor.py  # Extraction logic (2125 lines)
│   │   │   └── file_manager.py    # File handling utilities
│   │   ├── core/                  # Core utilities
│   │   ├── utils/                 # Helper functions
│   │   ├── main.py                # FastAPI application entry
│   │   ├── config.py              # Configuration management
│   │   └── database.py            # Database connection setup
│   ├── alembic/                   # Database migrations
│   │   └── versions/              # Migration files (5 migrations)
│   ├── storage/                   # File storage
│   │   ├── uploads/               # Uploaded PDFs
│   │   └── ocr_outputs/           # OCR results
│   ├── logs/                      # Application logs
│   ├── tests/                     # Backend tests
│   └── requirements.txt           # Python dependencies
│
├── frontend/
│   ├── src/
│   │   ├── pages/                 # Route components
│   │   │   ├── DashboardPage.tsx  # KPI dashboard
│   │   │   ├── UploadPage.tsx     # Batch upload interface
│   │   │   ├── ProcessingPage.tsx # Queue monitoring
│   │   │   ├── ReviewPage.tsx     # Split-view validation
│   │   │   ├── ContractsPage.tsx  # Contract listing
│   │   │   ├── ContractDetailPage.tsx
│   │   │   └── ContractEditPage.tsx
│   │   ├── components/            # Reusable components
│   │   │   ├── ui/                # Base UI components (Radix)
│   │   │   ├── forms/             # Form components
│   │   │   └── layout/            # Layout components
│   │   ├── services/              # API client layer
│   │   ├── context/               # React Context providers
│   │   ├── hooks/                 # Custom React hooks
│   │   ├── lib/                   # Utility functions
│   │   ├── types/                 # TypeScript definitions
│   │   └── App.tsx                # Main application component
│   ├── public/                    # Static assets
│   ├── package.json               # Node.js dependencies
│   └── vite.config.ts             # Vite configuration
│
├── data/
│   ├── test-samples/              # Sample contract PDFs (8 files)
│   └── dummy-samples/             # Additional test data
│
├── scripts/                       # Utility scripts
│   ├── test_default.py            # OCR pipeline tester
│   └── raw_pipeline_processor.py  # Batch processor
│
├── deployment/                    # Deployment configurations
├── docker-compose.yml             # Development environment
├── CLAUDE.md                      # AI assistant context
└── README.md                      # This file
```

---

## Getting Started

### Prerequisites

- **Python 3.9+** (for backend)
- **Node.js 18+** (for frontend)
- **PostgreSQL 15+** (database)
- **Docker & Docker Compose** (optional, recommended)
- **Git** (version control)

### Installation

#### Option 1: Docker Compose (Recommended)

1. **Clone the repository**
   ```bash
   git clone git@github.com:notsuperganang/contract-ocr-sempro.git
   cd contract-ocr-sempro
   ```

2. **Set up environment variables**

   Create `backend/.env`:
   ```env
   # Database
   DATABASE_URL=postgresql://postgres:postgres@localhost:5432/telkom_contracts

   # Authentication (REQUIRED - change these!)
   AUTH_USERNAME=admin
   AUTH_PASSWORD=your-secure-password
   JWT_SECRET_KEY=your-secret-key-min-32-characters-long
   JWT_ALGORITHM=HS256
   JWT_ACCESS_TOKEN_EXPIRE_HOURS=24

   # File Upload
   MAX_FILE_SIZE=52428800
   UPLOAD_DIR=storage/uploads
   OUTPUT_DIR=storage/ocr_outputs
   ```

   Create `frontend/.env`:
   ```env
   VITE_API_BASE_URL=http://localhost:8000
   ```

3. **Start PostgreSQL with Docker Compose**
   ```bash
   docker-compose up postgres -d
   ```

4. **Install and run backend**
   ```bash
   cd backend
   pip install -r requirements.txt
   alembic upgrade head  # Run database migrations
   uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
   ```

5. **Install and run frontend** (in a new terminal)
   ```bash
   cd frontend
   npm install
   npm run dev
   ```

6. **Access the application**
   - Frontend: http://localhost:5173
   - Backend API: http://localhost:8000
   - API Docs: http://localhost:8000/docs

#### Option 2: Manual Setup

<details>
<summary>Click to expand manual setup instructions</summary>

**Backend Setup:**

```bash
# Install PostgreSQL
sudo apt-get install postgresql postgresql-contrib  # Ubuntu/Debian
# or
brew install postgresql  # macOS

# Create database
sudo -u postgres psql
CREATE DATABASE telkom_contracts;
\q

# Install Python dependencies
cd backend
python -m venv venv
source venv/bin/activate  # Linux/macOS
# or
venv\Scripts\activate  # Windows

pip install -r requirements.txt

# Configure environment variables (create .env file as shown above)

# Run migrations
alembic upgrade head

# Start server
uvicorn app.main:app --reload --port 8000
```

**Frontend Setup:**

```bash
cd frontend
npm install
npm run dev
```

</details>

### First-Time Setup

1. **Register a user account**
   - Navigate to http://localhost:5173
   - Click "Register" and create an account

2. **Upload a test contract**
   - Sample PDFs are available in `data/test-samples/`
   - Go to "Upload" page
   - Drag and drop a PDF or click to browse

3. **Monitor processing**
   - Processing page shows OCR and extraction progress
   - Status updates in real-time

4. **Review and confirm**
   - Once processed, go to "Review" page
   - Verify extracted data against PDF
   - Make corrections if needed
   - Click "Simpan Draft" to save changes
   - Click "Konfirmasi Data" to finalize

5. **View dashboard**
   - Dashboard shows statistics and KPIs
   - Contract list shows all confirmed contracts

---

## Usage Guide

### 1. Uploading Contracts

**Single Upload:**
1. Navigate to "Upload" page
2. Click browse or drag PDF file
3. File is validated (max 50MB, PDF only)
4. Processing starts automatically

**Batch Upload:**
1. Select multiple PDFs (up to 10 files)
2. Drop all files at once
3. Each file creates a separate processing job
4. Monitor all jobs in Processing page

### 2. Monitoring Processing

The Processing page shows:
- **Status**: queued → processing → extracted → awaiting_review
- **Progress**: Real-time updates
- **Errors**: Clear error messages with retry option

### 3. Reviewing Extracted Data

**Split-View Interface:**
- **Left Panel**: PDF viewer with page navigation
- **Right Panel**: Editable form with extracted data

**Form Sections:**
- Customer Information (Informasi Pelanggan)
- Service Details (Layanan Utama)
- Service Breakdown (Rincian Layanan)
- Payment Method (Tata Cara Pembayaran)
- Contract Duration (Jangka Waktu)
- Contact Person (Kontak Person Telkom)

**Validation:**
- Real-time validation with error messages
- Required fields highlighted
- Format validation (dates, NPWP, amounts)

**Actions:**
- **Simpan Draft**: Save changes (validation must pass)
- **Konfirmasi Data**: Finalize and create contract record
- Buttons disabled when form is invalid

### 4. Managing Contracts

**Contract List:**
- Paginated table with search
- Filter by payment method, date range
- Sort by customer name, contract value, date

**Contract Details:**
- View all extracted data
- Download PDF or export to Excel
- Edit confirmed data
- Track payment schedules

**Payment Tracking:**
- Automatic status updates:
  - **PENDING**: Future payments
  - **DUE**: Current month
  - **OVERDUE**: Past due
- Manual actions:
  - Mark as PAID
  - Add notes
  - Cancel termin

### 5. Exporting Data

**Excel Export:**
1. Open contract detail page
2. Click "Export to Excel"
3. Formatted spreadsheet with multiple sheets:
   - Contract Overview
   - Payment Schedule
   - Service Details

**JSON Export:**
- Download raw JSON data
- Useful for integration with other systems

---

## API Documentation

### Base URL
```
http://localhost:8000
```

### Authentication

All endpoints except `/auth/login` and `/auth/register` require JWT authentication.

**Headers:**
```http
Authorization: Bearer <your_jwt_token>
```

### Authentication Endpoints

#### Register User
```http
POST /auth/register
Content-Type: application/json

{
  "username": "user@example.com",
  "password": "secure_password"
}
```

#### Login
```http
POST /auth/login
Content-Type: application/x-www-form-urlencoded

username=user@example.com&password=secure_password

Response:
{
  "access_token": "eyJ0eXAiOiJKV1...",
  "token_type": "bearer"
}
```

### Upload & Processing Endpoints

#### Upload PDF
```http
POST /api/upload
Content-Type: multipart/form-data

files: [file1.pdf, file2.pdf]

Response:
{
  "message": "2 files uploaded successfully",
  "job_ids": [1, 2],
  "file_ids": [1, 2]
}
```

#### Get Processing Status
```http
GET /api/processing/status/{job_id}

Response:
{
  "id": 1,
  "status": "awaiting_review",
  "file_id": 1,
  "filename": "contract.pdf",
  "created_at": "2025-01-15T10:30:00",
  "error_message": null
}
```

#### Get Extracted Data
```http
GET /api/processing/{job_id}/data

Response:
{
  "extracted_data": { ... },
  "edited_data": { ... }
}
```

#### Save Draft (Manual Save)
```http
PATCH /api/processing/{job_id}/data
Content-Type: application/json

{
  "edited_data": {
    "informasi_pelanggan": { ... },
    "layanan_utama": { ... },
    ...
  }
}
```

#### Confirm Contract
```http
POST /api/processing/{job_id}/confirm

Response:
{
  "message": "Contract confirmed successfully",
  "contract_id": 1
}
```

### Contract Management Endpoints

#### List Contracts
```http
GET /api/contracts?page=1&per_page=10&search=SMK&payment_method=termin

Response:
{
  "items": [...],
  "total": 45,
  "page": 1,
  "per_page": 10,
  "pages": 5
}
```

#### Get Contract Details
```http
GET /api/contracts/{id}

Response:
{
  "id": 1,
  "final_data": { ... },
  "customer_name": "SMK Negeri 1 Jakarta",
  "total_contract_value": 50000000,
  "payment_method": "termin",
  "created_at": "2025-01-15T10:30:00"
}
```

#### Update Contract
```http
PUT /api/contracts/{id}
Content-Type: application/json

{
  "final_data": { ... }
}
```

#### Export Contract to Excel
```http
POST /api/contracts/{id}/export

Response:
{
  "export_url": "/api/contracts/1/download/excel",
  "filename": "contract_1_20250115.xlsx"
}
```

#### Get Termin Payments
```http
GET /api/contracts/{id}/termin-payments

Response:
[
  {
    "termin_number": 1,
    "period_label": "Januari 2025",
    "amount": 5000000,
    "status": "PAID",
    "paid_at": "2025-01-10T14:20:00"
  },
  {
    "termin_number": 2,
    "period_label": "Februari 2025",
    "amount": 5000000,
    "status": "DUE"
  }
]
```

#### Update Termin Payment
```http
PATCH /api/contracts/{id}/termin-payments/{termin_number}
Content-Type: application/json

{
  "status": "PAID",
  "notes": "Paid via bank transfer"
}
```

### Dashboard & Statistics

#### Get Dashboard Data
```http
GET /api/admin/dashboard

Response:
{
  "total_contracts": 45,
  "total_revenue": 2500000000,
  "active_contracts": 32,
  "collection_rate": 85.5,
  "monthly_revenue": [...],
  "service_distribution": {...}
}
```

### Interactive API Documentation

Visit http://localhost:8000/docs for:
- Interactive API testing
- Request/response schemas
- Authentication testing
- Example payloads

---

## Database Schema

### Entity Relationship Diagram

```
┌─────────────┐
│    users    │
│─────────────│
│ id (PK)     │
│ username    │
│ hashed_pwd  │
│ created_at  │
└─────────────┘

┌──────────────────┐         ┌──────────────────────┐
│      files       │         │  processing_jobs     │
│──────────────────│         │──────────────────────│
│ id (PK)          │◄───────┤│ id (PK)              │
│ filename         │         │ file_id (FK)         │
│ size_bytes       │         │ status (ENUM)        │
│ uploaded_at      │         │ extracted_data JSONB │
│ pdf_path         │         │ edited_data JSONB    │
└──────────────────┘         │ ocr_artifacts JSONB  │
                             │ error_message        │
                             │ created_at           │
                             └──────────────────────┘
                                       │
                                       │
                             ┌─────────▼────────────┐
                             │    contracts         │
                             │──────────────────────│
                             │ id (PK)              │
                             │ source_job_id (FK)   │
                             │ file_id (FK)         │
                             │ final_data JSONB     │
                             │                      │
                             │ -- Denormalized --   │
                             │ customer_name        │
                             │ customer_npwp        │
                             │ period_start         │
                             │ period_end           │
                             │ payment_method       │
                             │ total_contract_value │
                             │ service_connectivity │
                             │ termin_count         │
                             │                      │
                             │ confirmed_at         │
                             └──────────────────────┘
                                       │
                                       │
                        ┌──────────────▼──────────────────┐
                        │  contract_term_payments         │
                        │─────────────────────────────────│
                        │ id (PK)                         │
                        │ contract_id (FK)                │
                        │ termin_number                   │
                        │ period_label (e.g. "Jan 2025")  │
                        │ period_year                     │
                        │ period_month                    │
                        │ amount                          │
                        │ status (ENUM)                   │
                        │ paid_at                         │
                        │ notes                           │
                        └─────────────────────────────────┘
```

### Key Tables

#### `files`
Stores uploaded PDF metadata
- `id`: Primary key
- `filename`: Original filename
- `size_bytes`: File size
- `uploaded_at`: Upload timestamp
- `pdf_path`: Storage path

#### `processing_jobs`
OCR processing queue and staging area
- `status`: queued → processing → extracted → awaiting_review → confirmed
- `extracted_data`: Raw OCR extraction results (JSONB)
- `edited_data`: User modifications (JSONB)
- `ocr_artifacts`: Paths to OCR output files (JSONB)

#### `contracts`
Final confirmed contract records
- `final_data`: Source of truth (JSONB)
- Denormalized columns for performance:
  - `customer_name`, `customer_npwp`
  - `period_start`, `period_end`
  - `payment_method`, `total_contract_value`
  - `service_connectivity`, `service_non_connectivity`, `service_bundling`
  - `termin_count`, `installation_cost`, `annual_subscription_cost`

#### `contract_term_payments`
Normalized payment tracking
- Automatically synced from `contracts.final_data`
- Status auto-updated: PENDING → DUE → OVERDUE
- PAID and CANCELLED statuses preserved (never auto-updated)

### Database Migrations

The project uses Alembic for database versioning:

```bash
# View migration history
alembic history

# Current migration
alembic current

# Upgrade to latest
alembic upgrade head

# Create new migration
alembic revision --autogenerate -m "description"
```

**Migration Files:**
1. `4c94e7565b76` - Initial schema (files, processing_jobs, contracts, users)
2. `a53e7fef9408` - Add contract denormalization columns
3. `8a6d04b41608` - Extend denormalization (all fields)
4. `6bef71f3b410` - Add contract_term_payments table
5. `6b81976f05de` - Add recurring payments support

---

## Development

### Backend Development

**Running Tests:**
```bash
cd backend
pytest                        # Run all tests
pytest -v                     # Verbose output
pytest --cov=app             # With coverage
pytest tests/test_denorm.py  # Specific test file
```

**Code Formatting:**
```bash
black app/      # Format code
isort app/      # Sort imports
```

**Debugging OCR Pipeline:**
```bash
# Test OCR with sample PDF
python scripts/test_default.py

# Test data extraction only
python backend/app/services/data_extractor.py page1 \
  --in data/dev-output/ocr_results.json \
  --out extraction_output.json
```

**Enable Debug Logging:**

Edit `backend/app/config.py`:
```python
log_processing_steps: bool = True
log_ocr_results_quality: bool = True
log_debug_model_info: bool = True
```

Logs will be written to `backend/logs/app.log`

### Frontend Development

**Development Server:**
```bash
cd frontend
npm run dev      # Start with hot reload
```

**Type Checking:**
```bash
npm run build    # Build compiles TypeScript
```

**Linting:**
```bash
npm run lint     # ESLint with TypeScript
```

**Component Development:**

UI components use Radix UI primitives with Tailwind styling:
```tsx
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
```

**API Integration:**

Services layer in `src/services/`:
```typescript
import { contractsApi } from '@/services/contracts'

const contracts = await contractsApi.getAll({ page: 1, per_page: 10 })
```

### Environment Variables

**Backend (`backend/.env`):**
```env
DATABASE_URL=postgresql://user:pass@host:port/db
AUTH_USERNAME=admin
AUTH_PASSWORD=secure_password
JWT_SECRET_KEY=min-32-chars-secret-key
MAX_FILE_SIZE=52428800
UPLOAD_DIR=storage/uploads
```

**Frontend (`frontend/.env`):**
```env
VITE_API_BASE_URL=http://localhost:8000
```

---

## Deployment

### Production Deployment (Docker)

1. **Build Docker images:**
   ```bash
   docker-compose build
   ```

2. **Start services:**
   ```bash
   docker-compose up -d
   ```

3. **Run migrations:**
   ```bash
   docker-compose exec backend alembic upgrade head
   ```

4. **Configure Nginx** (example config):
   ```nginx
   server {
       listen 80;
       server_name yourdomain.com;

       # Frontend
       location / {
           proxy_pass http://localhost:5173;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }

       # Backend API
       location /api {
           proxy_pass http://localhost:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }

       # API Docs
       location /docs {
           proxy_pass http://localhost:8000;
       }
   }
   ```

### Production Checklist

- [ ] Set strong `JWT_SECRET_KEY` (min 32 characters)
- [ ] Change default `AUTH_USERNAME` and `AUTH_PASSWORD`
- [ ] Configure PostgreSQL with persistent volumes
- [ ] Set up automated database backups
- [ ] Configure log rotation
- [ ] Set appropriate file upload limits
- [ ] Enable HTTPS with SSL certificates
- [ ] Configure CORS allowed origins
- [ ] Set up monitoring and alerting
- [ ] Document backup and recovery procedures

### Performance Optimization

**Backend:**
- Use database connection pooling (configured in SQLAlchemy)
- Enable denormalized columns for fast queries
- Implement pagination for large result sets
- Use background tasks for OCR processing

**Frontend:**
- Code splitting with Vite
- Lazy loading for routes and components
- React Query caching
- Optimized bundle size

---

## Technical Highlights

### 1. OCR Pipeline Architecture

**PaddleOCR PP-StructureV3 Configuration:**
- CPU-only inference with High Performance Inference (HPI)
- Table recognition enabled (essential for service counts)
- Seal and formula recognition disabled (contract-specific optimization)
- Automatic model selection for optimal performance

**Processing Flow:**
```python
PDF → PaddleOCR → Table Detection → Text Recognition → Layout Analysis → Structured JSON
```

### 2. Data Extraction Strategy

**Two-Page Processing:**
- **Page 1**: Customer info, service counts, costs, payment method
- **Page 2**: Contract duration, contact persons, additional terms

**Robust Pattern Matching:**
- Regex patterns for Indonesian dates (e.g., "15 Januari 2025")
- NPWP validation (15-digit format)
- Currency parsing (handles "Rp 50.000.000" formats)
- Fuzzy matching for OCR errors

**Example: Date Extraction**
```python
# Handles variations: "15 Januari 2025", "15/01/2025", "15-01-2025"
date_patterns = [
    r'\d{1,2}\s+(?:Januari|Februari|Maret|...)\s+\d{4}',
    r'\d{1,2}/\d{1,2}/\d{4}',
    r'\d{4}-\d{2}-\d{2}'
]
```

### 3. Human-in-the-Loop Validation

**Why Manual Save?**

Auto-save was intentionally removed after data loss incidents:
- Problem: User types "Februari" (incomplete), auto-save triggers, backend parser fails
- Impact: Termin payment deleted from database, PAID status lost
- Solution: Manual save with strict validation, buttons disabled when form invalid

**Form Validation:**
- React Hook Form + Zod schemas
- Real-time validation feedback
- Termin period format: `/^[A-Za-z]+\s+\d{4}$/` (e.g., "Januari 2025")
- Amount validation: positive numbers only

### 4. Payment Status Automation

**Automatic Status Updates:**
```python
# Calculate status based on period date vs current date
if period_date > current_date:
    status = "PENDING"
elif period_date == current_date.replace(day=1):
    status = "DUE"
elif period_date < current_date:
    status = "OVERDUE"
```

**Status Preservation:**
- PAID and CANCELLED statuses never auto-updated
- Row-level locking prevents race conditions
- Manual override always available

### 5. Database Optimization

**Denormalization Strategy:**
- JSONB `final_data` as source of truth
- Denormalized columns for frequent queries:
  - Fast filtering by customer name, payment method
  - Quick calculations for KPI dashboard
  - Indexed columns for performance

**Query Performance:**
```sql
-- Fast KPI calculation
SELECT
    COUNT(*) as total_contracts,
    SUM(total_contract_value) as total_revenue,
    AVG(total_contract_value) as avg_contract_value
FROM contracts
WHERE payment_method = 'termin';
```

### 6. Singleton OCR Service

**Lazy Loading Pattern:**
```python
_ocr_service_instance = None

def get_ocr_service():
    global _ocr_service_instance
    if _ocr_service_instance is None:
        _ocr_service_instance = OCRService()
    return _ocr_service_instance
```

**Benefits:**
- Single model initialization (saves memory)
- Shared across all requests
- Graceful degradation if OCR unavailable

---

## Contributing

While this is a final assignment project, contributions for improvements are welcome:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Guidelines

- Follow existing code style (Black for Python, ESLint for TypeScript)
- Write tests for new features
- Update documentation as needed
- Keep commits atomic and well-described

---

## License

This project is part of an academic final assignment (Tugas Akhir). For licensing inquiries, please contact the author.

---

## Acknowledgments

### Organizations
- **Telkom Indonesia** - For the internship opportunity and real-world problem domain
- **Digistar Class Program** - For training and mentorship during the internship

### Technologies
- **PaddleOCR Team** - For the excellent PP-StructureV3 OCR pipeline
- **FastAPI** - For the modern, fast Python web framework
- **React Team** - For the powerful UI library
- **Radix UI** - For accessible component primitives

### Special Thanks
- Digistar mentors and Telkom Indonesia supervisors for guidance
- Academic advisors for support throughout the final assignment
- Open source community for the amazing tools and libraries

---

## Contact & Support

**Developer**: [notsuperganang](https://github.com/notsuperganang)

**Project Repository**: [contract-ocr-sempro](https://github.com/notsuperganang/contract-ocr-sempro)

For questions, issues, or suggestions:
- Open an issue on GitHub
- Check existing documentation in `CLAUDE.md`
- Review API documentation at `/docs` endpoint

---

## Project Timeline

- **Phase 1**: Digistar Class Internship at Telkom Indonesia (Initial CLI tool development)
- **Phase 2**: Final Assignment Adoption (Web application transformation)
- **Phase 3**: Enhancement & Features (Human-in-the-loop validation, payment tracking)
- **Phase 4**: Completion & Documentation (Comprehensive testing and documentation)

---

<div align="center">

**Built with ❤️ as a Final College Assignment**

*Originally developed during Digistar Class Internship at Telkom Indonesia*

[⭐ Star this repo](https://github.com/notsuperganang/contract-ocr-sempro) | [🐛 Report Bug](https://github.com/notsuperganang/contract-ocr-sempro/issues) | [💡 Request Feature](https://github.com/notsuperganang/contract-ocr-sempro/issues)

</div>
