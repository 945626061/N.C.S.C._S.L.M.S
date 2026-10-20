# N.C.S.C._S.L.M.S
N.C.S.C. Smart Leave Management System
National Cement Share Company (N.C.S.C.) • Dire Dawa / Addis Ababa, Ethiopia
Compliant with the Federal Democratic Republic of Ethiopia Labour Proclamation No. 1156/2019
1. System Overview & Architectural Guardrails
The N.C.S.C. Smart Leave Management System is a mission-critical, enterprise-grade leave and workforce governance platform engineered specifically for the industrial operations of National Cement Share Company (N.C.S.C.) in Ethiopia.
Core Non-Negotiables:
Strict Institutional Branding: Display name is strictly "N.C.S.C." across all user interfaces, headers, audit stamps, and export dossiers.
Safe Atomic Sequential Employee ID Generator: Formatted strictly as STF_00001, STF_00002, ... Generated atomically using InnoDB row locks and sequence ledger tables. Never uses COUNT(*) or row counts, and decommissioned IDs are never reused.
Username-Based Authentication & First-Login Security:
Single user accounts (no shared accounts).
Temporary initial credentials follow strict format patterns:
Employee: NCSC@2026-<random-hex> (e.g., NCSC@2026-a1b2c3)
HR / System Admin: NCSC@2027-<random-hex> (e.g., NCSC@2027-e4f5a6)
Mandatory first-login flag (must_change_password=true) enforced server-side. Users are strictly blocked from business routes and APIs until they change credentials.
Immutable Append-Only Leave Ledger: Leave balances are never mutated directly in historical rows. Every transaction is appended to an immutable double-entry ledger (OPENING_BALANCE, ANNUAL_ACCRUAL, LEAVE_RESERVED, LEAVE_COMMITTED, LEAVE_REVERTED, STATUTORY_ADJUSTMENT) protected by tamper-evident SHA-256 hash chains.
FDRE Labour Proclamation No. 1156/2019 Legal Guardrails:
Articles 77 & 78: 16 working days for year 1 of service + 1 additional working day for every 2 additional years.
Article 79: Maximum postponement limit of 2 consecutive budget years. Generates alert warnings without automatic silent forfeiture (which is strictly illegal under Ethiopian law).
Article 81: Cash compensation in lieu of annual leave is prohibited during active employment.
Articles 85 & 86: 6-month tiered sick leave (Month 1 @ 100%, Months 2-3 @ 50%, Months 4-6 unpaid).
Article 88: 120 consecutive calendar days paid maternity leave (30 prenatal + 90 postnatal) & 3 consecutive days paid paternity leave.
Multi-Tier Approval Hierarchy with Separation of Duties:
Employee Submit -> Immediate Supervisor -> Division Manager -> HR Queue. Users are cryptographically blocked from approving their own leave requests.
Tri-Lingual Localization & Default Timezone: Full UI and notification parity in English (en), Amharic (am), and Simplified Chinese (zh). Server and display time is set to Africa/Addis_Ababa (UTC+3).
2. Technology Stack
Frontend & App Shell: React 19, TypeScript 5.8, Tailwind CSS v4, Lucide Icons
Backend / ORM: Next.js / Node.js, Prisma ORM
Database Engine: MySQL 8.x (InnoDB, utf8mb4_unicode_ci)
Session & Auth Guard: Server-side RBAC with IDOR protection & Iron-session / NextAuth token validation
Testing: Node.js native test runner (node:test) with 51 comprehensive unit and concurrency stress tests
3. Synthetic Seed Test Accounts
The system is pre-seeded with 5 realistic industrial manufacturing personas reflecting plant operations:
Employee ID	Username	Full Name / Amharic / Chinese	Role	Department	Shift Mode	Temporary Initial Password	Must Change Password
STF_00001	abebe.kebede	Abebe Kebede / አበበ ከበደ / 阿贝贝·科贝德	EMPLOYEE	ENG (Engineering)	5-Day Office	NCSC@2026-a1b2c3	Yes (true)
STF_00002	tigist.haile	Tigist Haile Mariam / ትዕግስት ኃይለማርያም / 蒂吉斯特·海莱	HR_ADMIN	HR (Human Resources)	5-Day Office	NCSC@2027-e4f5a6	Yes (true)
STF_00003	dawit.mengistu	Dawit Mengistu / ዳዊት መንግስቱ / 达维特·门吉斯图	LINE_MANAGER	PROD (Clinker Plant)	6-Day Plant Shift	NCSC@2026-c7d8e9	No (false)
STF_00004	yohanes.admin	Yohanes Tadesse / ዮሐንስ ታደሰ / 约翰内斯·塔德塞	SYSTEM_ADMIN	ELEC (Instrumentation)	5-Day Office	NCSC@2027-f0a1b2	No (false)
STF_00005	mulugeta.bekele	Mulugeta Bekele / ሙሉጌታ በቀለ / 穆卢盖塔·贝克勒	EMPLOYEE	PROD (Kiln Operations)	6-Day Plant Shift	NCSC@2026-b3c4d5	No (false)
4. Environment Variables Configuration
Copy .env.example to .env:
code
Bash
cp .env.example .env
Key environment variables:
code
Ini
# Database: MySQL 8.x (InnoDB engine, utf8mb4 collation)
DATABASE_URL="mysql://ncsc_admin:ncsc_prod_2026_pass@localhost:3306/ncsc_leave_management?charset=utf8mb4"

# Application Security & NextAuth Session Encryption
NEXTAUTH_SECRET="ncsc-production-secret-token-change-in-prod-2026"
NEXTAUTH_URL="http://localhost:3000"

# Enterprise Locales & Institutional Timezone
TIMEZONE="Africa/Addis_Ababa"
DEFAULT_LOCALE="en"
SUPPORTED_LOCALES="am,en,zh"

# Storage & Statutory Medical Attachments (Article 85/86)
ATTACHMENT_MAX_SIZE_BYTES="5242880"
ATTACHMENT_ALLOWED_MIME_TYPES="application/pdf,image/png,image/jpeg"

# Statutory Labour Proclamation Guardrails (FDRE Proclamation No. 1156/2019)
PROCLAMATION_ANNUAL_LEAVE_BASE_DAYS="16"
PROCLAMATION_POSTPONEMENT_MAX_CONSECUTIVE_YEARS="2"
PROCLAMATION_SICK_LEAVE_CEILING_DAYS="180"
5. Docker Compose Deployment
To run the complete production stack (MySQL 8.x + N.C.S.C. Application) with health checks and volume persistence:
code
Bash
# Start MySQL 8.x and N.C.S.C. Application services
docker compose up -d

# Verify container health and logs
docker compose ps
docker compose logs -f
The system will be accessible on:
Application Portal: http://localhost:3000
MySQL 8.x Database: localhost:3306 (ncsc_leave_management)
6. Local Development & Database Migration
Prerequisites
Node.js 20+
MySQL 8.0+
Step-by-Step Setup:
code
Bash
# 1. Install dependencies
npm install

# 2. Run the automated test suite (51/51 specs)
npm test

# 3. Start development server on port 3000
npm run dev

# 4. Production build
npm run build
7. Automated Test Suite (51 Passed Specs)
Run all automated unit tests and concurrency stress tests with:
code
Bash
npm test
Test Suite Distribution:
Sequential ID Generator (4 tests):
Format validation (STF_00001 with 5 digits)
Parsing of ID strings to integer sequence
Sequential increment without row counts
Concurrency Stress Test: 100 concurrent worker bursts generate 100 unique, gapless IDs with 0 collisions.
Authentication & First-Login Guard (6 tests):
Temporary password generation (NCSC@2026 vs NCSC@2027)
Password complexity validation
First-login mustChangePassword=true route interception
Access restoration upon credential change
FDRE Labour Proclamation No. 1156/2019 Legal Engine (7 tests):
Articles 77 & 78 service tenure annual accrual calculation
Service proration for partial fiscal years
Article 79 2-year postponement watchdog and non-forfeiture mandate
Article 81 cash encashment prohibition during ongoing employment
Articles 85 & 86 tiered sick pay schedule
Article 88 120-day maternity & 3-day paternity leaves
Immutable Leave Ledger (5 tests):
Double-entry balance calculation
Three-phase reservation, commit debit, and rollback lifecycle
Overdraft protection and justification requirements
Tamper-evident cryptographic hash chain verification
Concurrency Stress Test: 20 concurrent workers on limited balance prevent race conditions and overdrafts.
Leave Request Workflow Engine (7 tests):
Multi-tier approval chain (Submit -> Supervisor -> Division Manager -> HR)
Supervisor rejection rollback
Separation of Duties (SoD) self-approval blocking
Temporary delegation with DFS cycle detection
Secure MIME type and size ceiling validation on medical attachments
Tri-lingual in-app notification translation engine
Master Data & Localization (6 tests):
Industrial manufacturing work unit loading
Uppercase department code validation
Dated employee assignment history immutability
Strict "N.C.S.C." branding verification
Parity across English, Amharic, and Chinese navigation
Security Hardening & RBAC (5 tests):
IDOR prevention: Ordinary employee blocked from external personnel records
Line Manager departmental scoping and cross-department blocking
HR Admin and System Admin enterprise-wide access
Operational blocking on pending first-login password change
Article 85 medical attachment confidentiality
Auditable Reporting Suite & Postponement Watchdog (3 tests):
Scope-restricted historical querying
Multi-temporal aggregation (This Month, Last 3 & 6 Months) with CSV and ASCII dossier checksums
Organization-wide leave liability and financial exposure estimation in ETB
Working Days & Gazetted Holidays (8 tests):
Distinction between 5-day office and 6-day manufacturing plant shifts
Exclusion of Ethiopian Gazetted Public Holidays without double deduction on weekends
8. License & Institutional Attribution
Proprietary software developed for National Cement Share Company (N.C.S.C.), Federal Democratic Republic of Ethiopia. All rights reserved.
