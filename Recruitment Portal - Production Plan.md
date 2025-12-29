# CSIR-SERC Recruitment Portal - Full Stack Architecture & Implementation Guide

**Organization:** Structural Engineering Research Centre (SERC), CSIR, Government of India  
**Document Date:** December 2024  
**Compliance:** GIGW 3.0, CERT-In Guidelines, GOI Cyber Security Standards


#####Gmail App Password - 

```
yyhoakynckydyybm
```
---

## TABLE OF CONTENTS

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [System Architecture](#system-architecture)
4. [Database Design](#database-design)
5. [Security Compliance](#security-compliance)
6. [Portal Features & Modules](#portal-features--modules)
7. [7-Step Application Form](#7-step-application-form)
8. [User Portals](#user-portals)
9. [Admin Panel](#admin-panel)
10. [Design System & UI/UX](#design-system--uiux)
11. [Implementation Roadmap](#implementation-roadmap)
12. [Deployment & DevOps](#deployment--devops)

---

## PROJECT OVERVIEW

### Objectives
- **Streamline recruitment** across multiple CSIR-SERC laboratories and positions
- **Digital-first application** process with comprehensive applicant lifecycle management
- **Multi-role portal** for applicants, supervisors/screeners, and administrators
- **Government compliance** with GIGW, CERT-In, and ISO 27001 standards
- **Enterprise scalability** to handle 1000+ concurrent applications

### Key Requirements
✅ 7-step online form with progressive profiling  
✅ Multi-post application support (Technical, Technician, Scientist, Administrative)  
✅ Profile management with document uploads (CV, certificates, photo)  
✅ Supervisor portal with application filtering & reporting  
✅ Admin panel for form customization & system configuration  
✅ Email notifications (Gmail SMTP) with Telegram/SMS toggles  
✅ PDF generation & download (application forms, selection lists)  
✅ Built-in helpdesk ticketing system  
✅ Multilingual support (English + Hindi with GIGW compliance)  
✅ SSL/TLS with custom certificate upload  
✅ Logo & branding customization via admin panel  

---

## TECHNOLOGY STACK

### Backend
```
Runtime:          Node.js 18+ LTS (ES2020+)
Framework:        Express.js 4.18+
Authentication:   JWT (jsonwebtoken) + bcrypt
Database:         MongoDB Atlas (CERT-In compliant)
ORM/ODM:          Mongoose 7.x
Validation:       Joi, express-validator
Email Service:    Nodemailer (Gmail SMTP)
Notifications:    Telegram Bot API, Twilio SMS
PDF Generation:   pdfkit, html-pdf
File Upload:      Multer, AWS S3 (optional)
Logging:          Winston, Morgan
Rate Limiting:    express-rate-limit
Security:         Helmet, CORS, express-mongo-sanitize
Testing:          Jest, Supertest

```

### Frontend
```
Framework:        React 18.2+ (Functional Components)
UI Library:       Fluent UI (Microsoft) v9+
State Mgmt:       Redux Toolkit + RTK Query
Form Handling:    React Hook Form + Zod validation
Styling:          CSS-in-JS (Styled Components), Tailwind CSS
PDF Viewer:       React-PDF (non-downloadable viewer)
Charts:           Recharts, Chart.js
Icons:            Fluent System Icons
Internalization:  i18next + react-i18next
HTTP Client:      Axios with request/response interceptors
Build:            Vite 5.x (faster than CRA)
Testing:          Vitest, React Testing Library
Deployment:       Vercel, Netlify, or self-hosted
```

### DevOps & Infrastructure
```
CI/CD:            GitHub Actions, GitLab CI
Monitoring:       Prometheus, Grafana
Logging:          ELK Stack (Elasticsearch, Logstash, Kibana)
Reverse Proxy:    Nginx
Web Server:       Nginx/Apache
SSL/TLS:          Let's Encrypt, custom certificates
Database Backup:  MongoDB automated backups + S3
Message Queue:    Bull (Redis-based) for async tasks
Caching:          Redis
```

---

## SYSTEM ARCHITECTURE

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CDN & Load Balancer                         │
│                    (CloudFlare / AWS ALB)                       │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────▼────┐    ┌────▼────┐    ┌────▼────┐
    │ Applicant│    │Supervisor│   │ Admin    │
    │ Portal   │    │ Portal   │   │ Panel    │
    │ (React)  │    │ (React)  │   │ (React)  │
    └────┬─────┘    └────┬─────┘   └────┬─────┘
         │               │              │
         └───────────────┼──────────────┘
                         │
            ┌────────────▼────────────┐
            │   API Gateway/Proxy     │
            │    (Nginx/Express)      │
            └────────────┬────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
    ┌───▼───┐       ┌────▼────┐     ┌───▼──┐
    │Auth   │       │Routes   │     │Files │
    │Service │      │& Logic  │     │Service│
    │       │       │         │     │(S3)  │
    └───┬───┘       └────┬────┘     └──────┘
        │                │
        └────────────────┼───────────────┐
                         │               │
                    ┌────▼─────┐    ┌──▼──────┐
                    │MongoDB   │    │Redis    │
                    │(Primary) │    │(Cache)  │
                    └──────────┘    └─────────┘
```

### Microservices Approach (Optional for Scale)

For large deployments, separate into:
- **Auth Service** - User management, JWT, role-based access
- **Application Service** - Form submission, validation, storage
- **Notification Service** - Email, SMS, Telegram async queue
- **Reporting Service** - PDF generation, exports
- **Admin Service** - Configuration, user management
- **File Service** - Upload, virus scanning, S3 integration

---

## DATABASE DESIGN

### Collections/Models

#### 1. **Users Collection**
```javascript
{
  _id: ObjectId,
  email: String (unique, indexed),
  phone: String,
  password: String (bcrypt hashed),
  firstName: String,
  lastName: String,
  role: Enum ['APPLICANT', 'SUPERVISOR', 'ADMIN'],
  department: String,
  labName: String,
  isActive: Boolean,
  isVerified: Boolean,
  verificationToken: String,
  passwordResetToken: String,
  passwordResetExpiry: Date,
  loginAttempts: Number,
  lastLogin: Date,
  createdAt: Date,
  updatedAt: Date,
  
  // Audit trail
  auditLogs: [{
    action: String,
    timestamp: Date,
    ipAddress: String,
    userAgent: String
  }]
}
```

#### 2. **Applicants Collection**
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: Users),
  aadharNumber: String (encrypted, NOT linked to UIDAI),
  aadharVerificationStatus: String, // 'PENDING', 'VERIFIED', 'FAILED'
  personalDetails: {
    dateOfBirth: Date,
    gender: Enum,
    nationality: String,
    maritalStatus: String,
    pwdStatus: Boolean,
    pwdCategory: String
  },
  contactDetails: {
    phone: String,
    email: String,
    alternatePhone: String,
    correspondenceAddress: {
      street: String,
      city: String,
      state: String,
      pincode: String,
      country: String
    },
    permanentAddress: {
      street: String,
      city: String,
      state: String,
      pincode: String,
      country: String
    }
  },
  educationDetails: [{
    degree: String,
    discipline: String,
    institution: String,
    university: String,
    yearOfPassing: Number,
    marks: Number,
    gpa: String,
    documentPath: String (S3 URL),
    verified: Boolean
  }],
  experienceDetails: [{
    jobTitle: String,
    organization: String,
    department: String,
    reportingTo: String,
    startDate: Date,
    endDate: Date,
    currentlyWorking: Boolean,
    noOfCertificate: String (NOC S3 URL),
    description: String
  }],
  publications: [{
    title: String,
    journal: String,
    yearOfPublication: Number,
    doi: String,
    authorPosition: String,
    documentPath: String
  }],
  patents: [{
    title: String,
    filingNumber: String,
    filingDate: Date,
    status: String,
    inventors: [String],
    documentPath: String
  }],
  documents: {
    photoPath: String (S3 URL, 100x120 px JPG),
    cvPath: String (S3 URL),
    coverLetter: String (S3 URL),
    additionalDocuments: [{
      documentType: String,
      filePath: String,
      uploadedAt: Date
    }]
  },
  preferences: {
    preferredLocation: [String],
    willingToRelocate: Boolean,
    languageProficiency: [String]
  },
  createdAt: Date,
  updatedAt: Date,
  profileCompletionPercentage: Number
}
```

#### 3. **Posts Collection**
```javascript
{
  _id: ObjectId,
  postCode: String (unique, e.g., 'SE-2025-SCIENTIST-001'),
  postName: String,
  postCategory: Enum ['SCIENTIST', 'TECHNICIAN', 'TECHNICAL_ASSISTANT', 'ADMINISTRATIVE'],
  numberOfVacancies: Number,
  payLevel: String,
  
  essentialQualifications: [{
    degree: String,
    discipline: String,
    specialization: String,
    yearsOfExperience: Number,
    experienceType: Enum ['GENERAL', 'RESEARCH', 'INDUSTRY']
  }],
  
  desirableQualifications: [{
    criterion: String,
    weightage: Number // 1-5
  }],
  
  jobDescription: String,
  responsibilities: [String],
  labIds: [ObjectId], // Reference to multiple labs
  ageLimit: {
    general: Number,
    sc: Number,
    st: Number,
    obc: Number,
    pwd: Number
  },
  reservationDetails: {
    general: Number,
    sc: Number,
    st: Number,
    obc: Number,
    ews: Number,
    pwd: Number
  },
  applicationFee: {
    amount: Number,
    currency: String,
    waivableFor: [String] // ['SC', 'ST']
  },
  
  selectionProcess: [String], // ['WRITTEN_EXAM', 'INTERVIEW', 'PRACTICAL']
  selectionCriteria: String,
  
  applicationStartDate: Date,
  applicationDeadline: Date,
  selectionStartDate: Date,
  expectedJoiningDate: Date,
  
  isPublished: Boolean,
  isActive: Boolean,
  createdBy: ObjectId (ref: Users),
  createdAt: Date,
  updatedAt: Date,
  
  customFormFields: [ObjectId] // Reference to CustomFormField collection
}
```

#### 4. **Applications Collection**
```javascript
{
  _id: ObjectId,
  applicationNumber: String (unique, auto-generated), // APP-2025-001
  postId: ObjectId (ref: Posts),
  applicantId: ObjectId (ref: Applicants),
  applicantEmail: String,
  
  formResponses: {
    step1_personalInfo: Object,
    step2_contactAddress: Object,
    step3_educationQualifications: Array,
    step4_workExperience: Array,
    step5_researchPublications: Array,
    step6_documentsUpload: Object,
    step7_declaration: Object
  },
  
  applicationStatus: Enum ['DRAFT', 'SUBMITTED', 'UNDER_REVIEW', 'SHORTLISTED', 
                           'REJECTED', 'INTERVIEW_SCHEDULED', 'SELECTED', 'WAITLISTED'],
  
  reviewerComments: [{
    reviewerId: ObjectId (ref: Users),
    comment: String,
    timestamp: Date,
    status: String
  }],
  
  screeningStatus: {
    eligibilityCheck: Enum ['PENDING', 'ELIGIBLE', 'INELIGIBLE'],
    qualificationMatch: Number (0-100 percentage),
    experienceMatch: Number (0-100 percentage),
    overallScore: Number,
    remarks: String
  },
  
  interviewDetails: {
    interviewDate: Date,
    interviewTime: String,
    interviewLink: String (Zoom/Teams),
    panelMembers: [ObjectId],
    interviewRound: Number,
    interviewScore: Number
  },
  
  feePaid: Boolean,
  feeAmount: Number,
  feePaymentDate: Date,
  feePaymentMethod: String,
  feeTransactionId: String,
  
  pdfPath: String (S3 URL),
  
  createdAt: Date,
  submittedAt: Date,
  updatedAt: Date,
  
  // Audit trail
  activityLog: [{
    action: String,
    performedBy: ObjectId,
    timestamp: Date,
    details: String
  }]
}
```

#### 5. **CustomFormFields Collection**
```javascript
{
  _id: ObjectId,
  postId: ObjectId (ref: Posts),
  fieldName: String,
  fieldLabel: String,
  fieldType: Enum ['TEXT', 'TEXTAREA', 'RADIO', 'CHECKBOX', 'DROPDOWN', 
                   'DATE', 'FILE', 'NUMBER'],
  isRequired: Boolean,
  isMandatory: Boolean,
  fieldOrder: Number,
  stepNumber: Number (1-7),
  validationRules: {
    minLength: Number,
    maxLength: Number,
    pattern: String (regex),
    allowedFileTypes: [String],
    maxFileSize: Number
  },
  options: [String], // For dropdown, radio, checkbox
  helpText: String,
  placeholder: String,
  createdAt: Date,
  updatedAt: Date
}
```

#### 6. **NotificationSettings Collection**
```javascript
{
  _id: ObjectId,
  
  emailSettings: {
    provider: 'GMAIL_SMTP', // or 'SENDGRID'
    smtpServer: String,
    smtpPort: Number,
    senderEmail: String,
    senderName: String,
    username: String,
    password: String (encrypted),
    isConfigured: Boolean,
    lastTestedAt: Date,
    testEmailSent: Boolean
  },
  
  emailTemplates: {
    applicationSubmitted: String (template),
    applicationUnderReview: String,
    applicationShortlisted: String,
    applicationRejected: String,
    interviewScheduled: String,
    selectionNotification: String
  },
  
  smsNotifications: {
    isEnabled: Boolean,
    provider: Enum ['TWILIO', 'AWS_SNS', 'DISABLED'],
    accountSid: String,
    authToken: String (encrypted),
    fromNumber: String,
    templates: Object
  },
  
  telegramNotifications: {
    isEnabled: Boolean,
    botToken: String (encrypted),
    chatId: String,
    templates: Object
  },
  
  pushNotifications: {
    isEnabled: Boolean,
    provider: String
  },
  
  createdAt: Date,
  updatedAt: Date,
  updatedBy: ObjectId (ref: Users)
}
```

#### 7. **HelpDeskTickets Collection**
```javascript
{
  _id: ObjectId,
  ticketNumber: String (unique, auto-generated), // TICKET-2025-001
  applicantId: ObjectId (ref: Applicants),
  applicantEmail: String,
  applicationId: ObjectId (ref: Applications), // optional
  
  subject: String,
  description: String,
  category: Enum ['TECHNICAL', 'DOCUMENTATION', 'PAYMENT', 'GENERAL'],
  priority: Enum ['LOW', 'MEDIUM', 'HIGH', 'URGENT'],
  status: Enum ['OPEN', 'IN_PROGRESS', 'AWAITING_APPLICANT', 'RESOLVED', 'CLOSED'],
  
  attachments: [{
    fileName: String,
    filePath: String (S3 URL),
    uploadedAt: Date
  }],
  
  responses: [{
    responderId: ObjectId (ref: Users),
    responderRole: String,
    message: String,
    attachments: [String],
    timestamp: Date
  }],
  
  assignedTo: ObjectId (ref: Users),
  createdAt: Date,
  resolvedAt: Date,
  updatedAt: Date
}
```

#### 8. **SystemConfiguration Collection**
```javascript
{
  _id: ObjectId,
  
  branding: {
    organizationName: String,
    organizationShortName: String,
    logoUrl: String (S3 URL),
    logoAltText: String,
    faviconUrl: String,
    colorScheme: {
      primary: String, // #0077b6
      secondary: String, // #00b4d8
      tertiary: String, // #90e0ef
      background: String, // #caf0f8
      textColor: String, // #03045e
      accentColor: String
    }
  },
  
  contactDetails: {
    recruitmentDivisionPhone: [String],
    recruitmentDivisionEmail: String,
    officeAddress: String,
    mapLocationUrl: String,
    officeDays: String, // Mon-Fri, 9AM-5PM
    websiteUrl: String
  },
  
  portalSettings: {
    applicationFeeRequired: Boolean,
    defaultApplicationFee: Number,
    maxFileUploadSize: Number, // in MB
    allowedFileExtensions: [String],
    autoConfirmationEmailEnabled: Boolean,
    multiLanguageEnabled: Boolean,
    supportedLanguages: [String], // ['EN', 'HI']
    maintenanceMode: Boolean,
    maintenanceMessage: String
  },
  
  securitySettings: {
    enableSSL: Boolean,
    sslCertificatePath: String,
    sslKeyPath: String,
    sslCertificateExpiry: Date,
    corsOrigins: [String],
    sessionTimeout: Number, // in minutes
    passwordPolicy: {
      minLength: Number,
      requireUpperCase: Boolean,
      requireSpecialCharacters: Boolean,
      expiryDays: Number
    }
  },
  
  complianceSettings: {
    gigwCompliant: Boolean,
    certiInCompliant: Boolean,
    dataRetentionDays: Number,
    gdprCompliant: Boolean,
    accessibilityLevel: String // 'AA', 'AAA'
  },
  
  createdAt: Date,
  updatedAt: Date,
  updatedBy: ObjectId (ref: Users)
}
```

---

## SECURITY COMPLIANCE

### CERT-In Guidelines Implementation

#### 1. **Access Control (CERT-In Rule 6)**
```javascript
// Role-Based Access Control (RBAC)
const roles = {
  APPLICANT: {
    permissions: ['VIEW_OWN_PROFILE', 'SUBMIT_APPLICATION', 'UPLOAD_DOCUMENTS', 
                 'VIEW_APPLICATION_STATUS', 'CONTACT_HELPDESK']
  },
  SUPERVISOR: {
    permissions: ['VIEW_APPLICATIONS', 'FILTER_APPLICATIONS', 'ADD_COMMENTS', 
                 'GENERATE_REPORTS', 'SCHEDULE_INTERVIEWS', 'APPROVE_APPLICATIONS']
  },
  ADMIN: {
    permissions: ['*'] // All permissions
  }
};

// Multi-Factor Authentication (MFA)
// Email OTP + SMS OTP for sensitive operations
```

#### 2. **Password Security (CERT-In Rule 5)**
```javascript
// Password Policy
const passwordPolicy = {
  minLength: 12,
  requireUpperCase: true,
  requireLowerCase: true,
  requireNumbers: true,
  requireSpecialCharacters: true, // !@#$%^&*
  expiryDays: 90,
  historyCheck: 5 // Can't reuse last 5 passwords
};

// Implementation with bcrypt
const hash = await bcrypt.hash(password, 12); // 12 salt rounds
```

#### 3. **Session Management (CERT-In Rule 4)**
```javascript
// JWT with short expiry
const token = jwt.sign(payload, SECRET, { 
  expiresIn: '15m' // Access token: 15 minutes
});

// Refresh token: 7 days (secure, httpOnly cookie)
const refreshToken = jwt.sign(payload, REFRESH_SECRET, { 
  expiresIn: '7d'
});

// Session timeout: 15 minutes inactivity
// Secure cookies: HttpOnly + Secure flags
```

#### 4. **Logging & Monitoring (CERT-In Rule 7)**
```javascript
// Comprehensive audit trail
auditLog({
  action: 'APPLICATION_SUBMITTED',
  userId: applicant._id,
  timestamp: new Date(),
  ipAddress: req.ip,
  userAgent: req.headers['user-agent'],
  details: { applicationId, postId }
});

// Alert on suspicious activities
// - 5+ failed login attempts → account lock
// - Bulk downloads detected → alert admin
// - Unusual geographic access → flag for review
```

#### 5. **Encryption (CERT-In Rule 3)**
```javascript
// TLS 1.2+ for all communications
// AES-256 for sensitive data encryption (aadhar, bank details)
const crypto = require('crypto');

const encryptedAadhar = encrypt(aadharNumber, ENCRYPTION_KEY);
const decryptedAadhar = decrypt(encryptedAadhar, ENCRYPTION_KEY);

// Database encryption at rest (MongoDB field-level encryption)
// S3 files with server-side encryption
```

#### 6. **Backup & Disaster Recovery (CERT-In Rule 8)**
```
- Automated MongoDB backups: Daily + Weekly archives
- 30-day retention
- S3 cross-region replication
- RTO: 4 hours, RPO: 1 hour
- Monthly backup restoration tests
```

#### 7. **Incident Reporting**
```
- 6-hour reporting to CERT-In (Rule 6 of CERT-In 2013 Rules)
- Automated incident detection + manual verification
- Documented incident response plan
- Post-incident analysis (within 48 hours)
```

### GIGW 3.0 Compliance

#### 1. **Accessibility (WCAG 2.1 Level AA)**
- Color contrast ratio ≥ 4.5:1 for normal text
- Keyboard navigation (Tab order, focus indicators)
- Screen reader compatibility (ARIA labels)
- Text alternatives for images
- Responsive design (mobile-first approach)

#### 2. **Multilingual Support (Hindi + English)**
```javascript
// i18next configuration
import i18n from 'i18next';

i18n.init({
  resources: {
    en: { translation: require('./locales/en.json') },
    hi: { translation: require('./locales/hi.json') }
  },
  lng: 'en',
  fallbackLng: 'en',
  interpolation: { escapeValue: false }
});

// Usage: <Trans i18nKey="applicationSubmitted" />
```

#### 3. **API Integration with Government Platforms**
```javascript
// Single Sign-On (SSO) integration (optional for future)
// DigiLocker integration for document verification
// MyScheme API for scheme information
// India Portal integration (notification link)

// Placeholder endpoints:
POST /api/diglocker/verify - Document verification
GET /api/india-portal/register - Portal registration
```

#### 4. **Performance Standards**
- Page load time < 3 seconds
- TTFB (Time to First Byte) < 500ms
- Lighthouse score > 90
- Mobile usability: Perfect score

#### 5. **Content Standards**
- Published date on all pages
- Last updated date visible
- Document metadata (title, size, format)
- Language clarity (Flesch Reading Ease > 60)
- No jargon without explanation

---

## PORTAL FEATURES & MODULES

### Applicant Portal

#### Dashboard
- **Overview**: Application status timeline, announcements
- **Quick Actions**: Apply for new post, upload documents, edit profile
- **Statistics**: Applications submitted, shortlisted, rejected, pending

#### Profile Management
```
Personal Information
├── Name, DOB, Gender, Nationality
├── Marital Status, Category (General/SC/ST/OBC/EWS)
├── PwD status and disability type
└── Photo upload (100x120 px JPG)

Contact Details
├── Email (verified), Phone (verified)
├── Alternate contact info
├── Correspondence address
└── Permanent address

Education
├── Add/Edit degree, discipline, marks
├── University, institution
├── Year of passing
└── Document upload (certificate)

Experience
├── Job title, organization, department
├── Employment dates, currently working status
├── NOC upload (if employed in govt sector)
└── Job description

Research & Publications
├── Published papers/journals
├── Patent filings (with status)
├── Conference presentations
└── Researcher ID (ORCID, ResearchGate)
```

#### Multi-Post Application
- **Post Search**: Filter by category, location, experience level, pay scale
- **Post Details**: Full job description, eligibility, selection process
- **Apply Now**: Initiates 7-step form
- **Track Applications**: Status, reviewer comments, interview schedule
- **Download Forms**: PDF of submitted application form

#### Document Management
- **Upload**: CV/Resume, cover letter, certificates, NOC
- **Security**: Virus scanning, file type validation
- **Storage**: S3 with encryption
- **Retrieval**: Download or share via link

#### Notifications
- **Email**: Application submitted, shortlisted, interview scheduled
- **SMS**: (if enabled) Important updates
- **In-App**: Notification bell with dismissible alerts
- **Preferences**: Manage notification types and frequency

#### Helpdesk Support
- **Raise Ticket**: Category, priority, attachments
- **Track Status**: Real-time ticket updates
- **Chat**: Communicate with support team
- **Knowledge Base**: FAQs, how-to guides

### Supervisor/Screener Portal

#### Application Management
- **List View**: All applications with filtering
  - Filter by: Post, status, date, applicant name, category
  - Sort by: Date, name, score, status
  - Bulk actions: Shortlist, reject, send for interview

- **Detail View**: Full applicant profile + application
  - Applicant contact information
  - Education & experience summary
  - Document preview (embedded PDF viewer)
  - Application timeline
  - Previous applications (if any)

#### Screening & Evaluation
```
Eligibility Check
├── Qualification match (auto-check)
├── Experience verification
├── Age calculation
└── Category verification

Scoring System
├── Qualification score (auto: 0-40 points)
├── Experience score (manual: 0-30 points)
├── Additional achievements (manual: 0-20 points)
├── Interview performance (later: 0-10 points)
└── Total: 0-100

Comments & Notes
├── Internal notes (not visible to applicant)
├── Decision: Eligible/Ineligible
├── Recommendation for interview
└── Approval chain (if needed)
```

#### Reporting & Analytics
- **Dashboard Charts**:
  - Application count by post
  - Application status distribution (pie chart)
  - Category-wise applications (bar chart)
  - Submission trends (line chart over time)
  - Shortlist rate by post

- **Reports** (downloadable as Excel/PDF):
  - Eligible applicants list (with scores)
  - Rejected applications (with reasons)
  - Interview schedule
  - Category-wise statistics
  - Lab-wise application summary

#### Interview Management
- **Schedule Interview**:
  - Select applicants
  - Choose date/time
  - Generate Zoom/Teams link
  - Auto-send invitation (email + SMS)

- **Interview Feedback**:
  - Rate applicant (1-10)
  - Add comments
  - Record recommendation

#### Exports
- **Excel Export**: Filtered applications with all details
- **PDF Export**: Reports with branding, signatures
- **CSV Export**: For external tool integration

---

## ADMIN PANEL

### System Configuration

#### Branding & Portal Settings
```
Organization Details
├── Organization name & short name
├── Logo upload (with preview)
├── Color scheme customization
│   ├── Primary color (#0077b6)
│   ├── Secondary color (#00b4d8)
│   ├── Background color (#caf0f8)
│   └── Font color (#03045e)
├── Favicon
└── Website link

Portal Settings
├── Portal name & tagline
├── Enable/disable features
├── Maintenance mode toggle
├── Message of the day
└── Footer content (contact, links)

Contact Information
├── Recruitment division phone (multiple)
├── Email address
├── Office address (formatted)
├── Map location (Google Maps embed)
├── Office hours (working days/times)
└── Social media links
```

#### Post Management
```
Create/Edit Post
├── Post code, name, category
├── Number of vacancies
├── Pay level/salary
├── Qualifications (essential & desirable)
├── Job description & responsibilities
├── Lab assignment (multiple labs support)
├── Age limits by category
├── Reservation details
├── Application fee (amount, waivers)
├── Selection process
├── Important dates (start, deadline, interview, joining)
├── Publish/unpublish
└── Custom form fields (per post)

Multi-Lab Support
├── Assign post to multiple labs
├── Lab-specific form fields
├── Lab-wise application routing
└── Lab head approval flow (optional)

Post Templates
├── Create post template
├── Reuse template for similar posts
└── Version history
```

#### Form Customization

**Dynamic Form Builder**
```
Step 1: Personal Information
├── Auto-fields: Name, DOB, Gender, etc.
├── Custom fields: Add via form builder
├── Conditional fields (show if X = Y)
└── Field settings: Required, validation, help text

Step 2: Contact & Address
├── Standard fields (email, phone)
├── Address selector (same as permanent)
└── Custom address fields

Steps 3-7: Customizable
├── Drag-drop field manager
├── Field type: Text, Textarea, Radio, Checkbox, Dropdown, Date, File, Number
├── Validation rules per field
├── Help text & placeholders
├── Conditional logic
└── Field dependencies

Form Logic
├── Show/hide fields based on conditions
├── Calculate fields (age from DOB)
├── Auto-populate (from previous step)
└── Validation: Client-side + Server-side
```

#### Email Notification Settings

**SMTP Configuration (Gmail)**
```javascript
emailSettings: {
  provider: 'GMAIL_SMTP',
  smtpServer: 'smtp.gmail.com',
  smtpPort: 587,
  useSecureConnection: true,
  senderEmail: 'recruitment@csir-serc.org', // App password generated
  senderName: 'CSIR-SERC Recruitment',
  username: 'recruitment@csir-serc.org',
  password: '••••••••••' (encrypted in DB)
}

// Email templates for:
// - Application received
// - Under review
// - Shortlisted
// - Rejected
// - Interview scheduled
// - Selected
// - Waitlisted
// - Helpdesk response
// - Password reset

// Template variables:
// {{applicantName}}, {{postName}}, {{postCode}},
// {{applicationNumber}}, {{status}}, {{portalUrl}},
// {{interviewDate}}, {{contactEmail}}
```

**SMS Notifications (Optional)**
```javascript
smsSettings: {
  isEnabled: true/false,
  provider: 'TWILIO', // or AWS SNS
  accountSid: 'AC••••••••••',
  authToken: '••••••••••' (encrypted),
  fromNumber: '+91XXXXXXXXXX',
  templates: {
    applicationSubmitted: 'App submitted. Number: {{appNumber}}',
    shortlisted: 'Congratulations! You are shortlisted.',
    interview: 'Interview on {{date}} at {{time}}. Link: {{link}}'
  }
}
```

**Telegram Bot (Optional)**
```javascript
telegramSettings: {
  isEnabled: true/false,
  botToken: 'XXXXXXXXXXXXXXXXXXXX' (encrypted),
  adminChatId: '-XXXXXXXXXX',
  templates: {
    newApplication: 'New application: {{applicantName}} for {{postName}}',
    applicationSubmitted: 'Application {{appNumber}} submitted'
  }
}
```

#### SSL Certificate Management
```
SSL Settings
├── Enable SSL: Toggle
├── Current certificate status
├── Certificate expiry date (with renewal alert)
├── Upload custom certificate (.pem)
├── Upload custom key (.key)
├── CSR generation tool
├── Certificate chain (if multi-level)
└── HSTS (HTTP Strict Transport Security) configuration
```

#### User Management
```
Users List
├── Filter by role (Applicant, Supervisor, Admin)
├── Search by email/name
├── View user details
├── Edit user role
├── Reset password (force reset on next login)
├── Disable/enable user
├── View login history
└── Audit actions

User Roles
├── APPLICANT: Apply, upload documents, track status
├── SUPERVISOR: Screen, evaluate, report
├── ADMIN: All system configuration

Create New User
├── Email
├── Role
├── Initial password (sent via email)
└── Department/Lab assignment
```

#### Audit Logs & Activity Tracking
```
Audit Dashboard
├── Real-time activity feed
├── Filter by user, action, date range, object
├── Export logs (CSV, PDF)

Logged Actions
├── User login/logout + IP, time
├── Application submitted
├── Application status changed
├── Document uploaded/deleted
├── Comment added
├── Configuration changed
├── Email sent/failed
├── Admin actions
└── Failed security attempts

Search & Export
├── Advanced filters
├── Date range selection
├── Export to CSV
└── Archive old logs
```

#### System Monitoring
```
Monitoring Dashboard
├── Server uptime status
├── Database connectivity
├── Email service status
├── File storage (S3) status
├── API response time
├── Current concurrent users
├── Storage usage (applications, documents)
├── Failed job queue (emails, PDFs)
└── Error rate & trending

Alerts
├── Email alerts on critical issues
├── Dashboard warnings for manual review
└── Auto-retry failed operations
```

---

## 7-STEP APPLICATION FORM

### Form Flow & Validation

```
┌──────────────────────────────────────────────┐
│         STEP 1: Personal Information         │
│  (Name, DOB, Gender, Category, PwD status)   │
└──────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────┐
│       STEP 2: Contact & Address Details      │
│  (Email, Phone, Correspondence, Permanent)   │
└──────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────┐
│     STEP 3: Educational Qualifications      │
│  (Degree, Discipline, Marks, Year, Upload)   │
└──────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────┐
│           STEP 4: Work Experience            │
│  (Job Title, Org, Dates, NOC if Employed)    │
└──────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────┐
│      STEP 5: Publications & Patents         │
│  (Papers, Patents, Patent Status, Links)     │
└──────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────┐
│         STEP 6: Document Upload             │
│  (CV, Photo, Cover Letter, Certificates)     │
└──────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────┐
│    STEP 7: Declaration & Submission         │
│  (Aadhar, Consent, Terms, Preview, Submit)   │
└──────────────────────────────────────────────┘
                     ↓
            ✓ Submitted Successfully
            ↓
        Application Number: APP-2025-001
        Confirmation Email Sent
```

### Step-by-Step Details

#### Step 1: Personal Information
```
Form Fields (with Fluent UI components):

Personal Details Section
├── First Name* (TextField, minLength: 2, maxLength: 50)
├── Last Name* (TextField, minLength: 2, maxLength: 50)
├── Date of Birth* (DatePicker, max: 65 years, min: 18 years)
├── Gender* (Dropdown: Male, Female, Other, Prefer not to say)
├── Nationality* (Dropdown, default: Indian)
├── Marital Status (Dropdown: Single, Married, Divorced, Widowed)
│
Category & Concessions
├── Category* (RadioGroup: General, SC, ST, OBC (NCL), EWS)
│   └── Show SC/ST/OBC certificate requirement if selected
├── Person with Disability* (Checkbox + if YES, show sub-fields)
│   ├── Disability Type (Dropdown)
│   ├── Disability Percentage (NumberField: 0-100)
│   └── Certificate Upload (FileInput, .pdf/.jpg only)
│
Validation Rules
├── Name: Only alphabets + spaces
├── DOB: Must be between 18-65 years (depends on post age limit)
├── Category: Mandatory for Indian citizens
└── All fields mandatory (*)
```

#### Step 2: Contact & Address
```
Contact Information Section
├── Email Address* (TextField, type: email, validated regex)
│   └── Verification: OTP sent to email
├── Confirm Email* (TextField, must match above)
├── Phone Number* (TextField, 10 digits, starts with 6-9)
│   └── SMS OTP verification
├── Alternate Phone (TextField, optional)
│
Address Details Section
├── Address Type Selector: Correspondence / Same as Permanent
│
Correspondence Address
├── Street Address* (Textarea, maxLength: 200)
├── City/Town* (Autocomplete, from list)
├── State* (Dropdown)
├── Postal Code* (TextField, 6 digits)
├── Country* (Dropdown, default: India)
│
Permanent Address (optional)
├── Same as Correspondence (Checkbox)
├── If NO, repeat address fields above
│
Validation Rules
├── Phone: Must be numeric, 10 digits, starts with 6-9
├── Email: Valid email regex
├── Postal Code: 6-digit Indian PIN code
└── Address: Max 200 characters
```

#### Step 3: Educational Qualifications
```
Education Section (Multiple entries allowed)

For Each Education Entry:
├── Add Education Button (Creates new row)
├── Educational Degree* (Dropdown: 10th, 12th, Diploma, B.E., B.Tech, M.E., M.Tech, PhD, etc.)
├── Discipline/Subject* (Dropdown/Autocomplete, changes based on Degree)
├── Institution Name* (Autocomplete, from list)
├── University/Board* (Autocomplete)
├── Year of Passing* (NumberField/Dropdown: 1980-2024)
├── Marks Obtained (NumberField)
├── Total Marks (NumberField)
├── Percentage* (Auto-calculated or manual)
├── Grade/GPA (TextField)
├── Upload Certificate* (FileInput, .pdf max 2MB)
├── Delete Entry Button (after 1st entry is required)
│
Validation Rules
├── At least 1 education entry required
├── Marks validation: ≤ Total Marks
├── Year: ≤ current year
├── Percentage: Between 0-100
└── Certificate: PDF only, < 2MB

Notes:
- Essential qualifications auto-filled based on selected post
- Applicant must match essential criteria to proceed
- Desirable qualifications highlighted for scoring
```

#### Step 4: Work Experience
```
Experience Section (Optional, but recommended for senior posts)

If Employed in Government/Autonomous Body:
├── Upload No Objection Certificate (NOC)* (.pdf, mandatory for govt employees)

For Each Experience Entry:
├── Add Experience Button
├── Job Title/Designation* (TextField)
├── Organization Name* (Autocomplete)
├── Organization Type (Dropdown: Private, Government, Autonomous, NGO, Educational, etc.)
├── Department* (TextField)
├── Reporting Manager Name (TextField)
├── Employment Start Date* (DatePicker)
├── Employment End Date (DatePicker, if ended)
├── Currently Employed Here* (Checkbox)
│   └── If YES, hide End Date
├── Total Years* (Auto-calculated)
├── Job Description/Responsibilities (Textarea, max 500 chars)
├── Delete Entry Button
│
Validation Rules
├── At least 0 entries (optional for fresh graduates)
├── End Date ≥ Start Date
├── No overlapping employment dates
├── Total experience calculated accurately
└── Government employees MUST upload NOC

Notes:
- Post page specifies minimum experience required (e.g., "5+ years")
- Experience years matched against post eligibility
- Recent experience weighted higher in scoring
```

#### Step 5: Research Publications & Patents
```
Publications Section (Optional)

For Each Publication:
├── Add Publication Button
├── Paper Title* (TextField)
├── Journal/Conference Name* (Dropdown/Autocomplete)
├── Year of Publication* (Dropdown: 2015-2024)
├── Digital Object Identifier (DOI) (TextField, optional)
├── Your Role as Author (Dropdown: First Author, Co-Author, Corresponding Author)
├── PubMed/Google Scholar Link (TextField, optional)
├── Upload Paper/Link (TextField or FileInput, optional)
├── Delete Entry Button
│
Patents Section (Optional)

For Each Patent:
├── Add Patent Button
├── Patent Title* (TextField)
├── Patent Filing Number (TextField)
├── Filing Date (DatePicker)
├── Patent Status (Dropdown: Filed, Published, Granted, Rejected)
├── List of Inventors (TextArea, one per line)
├── Application Link (TextField, optional)
├── Delete Entry Button
│
Validation Rules
├── Publication year: ≤ current year
├── DOI format validation (optional)
├── At least 1 field filled per section
└── Both sections optional

Notes:
- Used for scientific posts (Scientist, Senior Scientist)
- Increases applicant score if present
- Quality of publications considered in screening
```

#### Step 6: Document Upload
```
Documents Section

Profile Photo*
├── FileInput (JPG/JPEG only, 100x120 px)
├── Size: < 200 KB
├── Live preview after upload
├── Crop tool if dimensions incorrect
└── Required for all applicants

Curriculum Vitae (CV/Resume)*
├── FileInput (.pdf, .doc, .docx)
├── Size: < 5 MB
├── Max 2 pages recommended
└── Must contain: Education, Experience, Contact Info

Cover Letter (Optional)
├── FileInput (.pdf, .doc, .docx)
├── Size: < 2 MB
├── Optional but recommended
└── Should explain motivation for the post

Certificate of Education*
├── FileInput (auto-required based on Step 3)
├── Already uploaded in Step 3, shown here as reference
├── Can re-upload here if needed
└── Size: < 2 MB each

Other Documents (Optional)
├── Add Multiple Files Button
├── Document Type (Dropdown: Certificate, Award, Publication, Research Paper, etc.)
├── FileInput (.pdf/.doc/.docx)
├── Max 5 additional documents
├── Size: < 2 MB each
│
Validation Rules
├── Photo: JPG only, 100x120 px, < 200 KB
├── CV: Required, < 5 MB
├── Certificates: Auto-required from Step 3
├── Other docs: Optional
└── Total upload size: < 50 MB

Notes:
- All files scanned for viruses before storage
- Files stored on S3 with encryption
- Auto-compression of large images
- Supported formats: PDF, DOC, DOCX, JPG, JPEG
```

#### Step 7: Declaration & Submission
```
Important Information Section
├── Info Box: "Review your application before submitting"
├── Link to Preview Application (opens new tab)
└── Timestamp of last saved

Applicant Declaration*
├── Checkbox: "I hereby declare that the information provided is true and correct"
└── Checkbox: "I understand that providing false information may lead to rejection"

Aadhar Details* (No UIDAI Integration - Local Storage Only)
├── Info Box: "Aadhar details are used for identity verification only"
├── "I have read and understood the Aadhar usage policy"
├── Aadhar Number Input (Formatted: XXXX XXXX XXXX, masked display)
│   └── Server-side: Stored encrypted, NOT linked to UIDAI API
├── Aadhar Upload (Optional, .pdf only)
│   └── Alternative: Email OTP verification (shown after submission)
│
Terms & Conditions*
├── Checkbox: "I have read and agree to Terms & Conditions"
├── Link to Terms (opens in modal)
├── Link to Privacy Policy
├── Link to Data Protection Policy
│
Consent Section*
├── Checkbox: "I consent to receive emails/SMS notifications"
├── Checkbox: "I allow CSIR-SERC to contact me for follow-up"
├── Checkbox: "I understand my data will be stored securely"
│
Button Section
├── Save as Draft Button (gray, saves form state locally)
├── Preview Application Button (opens PDF in new tab)
└── Submit Application Button (blue, primary action)
│   └── Disabled until all (*) fields filled
│   └── Shows spinner during submission
│   └── Confirmation: "Are you sure?" modal before final submission
│
Validation Rules
├── All checkboxes marked
├── Aadhar format: 12 digits, accepted formats XXXX XXXX XXXX
├── Email & Phone already verified in Step 2
└── All previous steps valid
```

### Form Behavior & Features

#### Auto-Save
```javascript
// Auto-save every 30 seconds or on field blur
// Save to localStorage (client-side backup)
// Save to database (auto-draft)
// Show "Saving..." indicator
// Show "Last saved at XX:YY" timestamp

// Resume from draft:
// - If user closes without submitting
// - Next login shows "Resume Application" option
// - Can continue from last saved step
```

#### Validation
```javascript
// Client-side validation (instant feedback)
// - Real-time as user types
// - Red error message below field
// - Disable Submit button until valid

// Server-side validation (security)
// - Re-validate all fields on submit
// - Prevent injection attacks
// - Ensure data integrity
// - Return 400 with error details if invalid

// Cross-field validation
// - Email match
// - Address consistency
// - No overlapping dates
// - Age based on post
```

#### Progress Indicator
```
Visual Progress Bar:
[████░░░░░░░░░░░░░] Step 1 of 7 (14%)

Current Step Indicator:
✓ Personal Information (Completed)
✓ Contact Details (Completed)
→ Education (In Progress)
○ Experience (Pending)
○ Publications (Pending)
○ Documents (Pending)
○ Declaration (Pending)

Estimated Time: 8 minutes
Completion: 43%
```

---

## DESIGN SYSTEM & UI/UX

### Color Palette (Drupal-inspired + CSIR Professional)
```
Primary Colors:
├── Primary Blue: #0077b6 (Call-to-action buttons, links)
├── Light Blue: #00b4d8 (Hover states, secondary elements)
├── Sky Blue: #90e0ef (Backgrounds, highlights)
├── Cream Background: #caf0f8 (Page backgrounds)
│
Text Colors:
├── Dark Blue/Navy: #03045e (Primary text, headings)
├── Medium Gray: #6c757d (Secondary text)
├── Light Gray: #f8f9fa (Disabled text)
│
Status Colors:
├── Success Green: #28a745
├── Error Red: #dc3545
├── Warning Orange: #ffc107
├── Info Blue: #17a2b8
│
Neutral:
├── White: #ffffff
├── Light Gray: #f1f3f5
├── Medium Gray: #adb5bd
└── Dark Gray: #212529
```

### Typography
```
Font Family: Noto Sans (Google Fonts)
- Fallback: -apple-system, BlinkMacSystemFont, Segoe UI, Helvetica Neue

Font Sizes:
├── H1: 32px, font-weight: 600
├── H2: 24px, font-weight: 600
├── H3: 20px, font-weight: 600
├── Body: 16px, font-weight: 400
├── Small: 14px, font-weight: 400
└── Tiny: 12px, font-weight: 400

Line Heights:
├── Headings: 1.2
├── Body: 1.5
└── Tight: 1.3
```

### Component Library (Fluent UI)
```
Fluent UI v9+ Components Used:
├── Button (Primary, Secondary, Outline)
├── TextField (Text, Email, Number, Password)
├── Checkbox
├── RadioButton & RadioGroup
├── Dropdown (Select)
├── DatePicker
├── TimePicker
├── Textarea
├── Card
├── Modal/Dialog
├── Toast/Notification
├── Progress Indicator
├── Spinner/Loading
├── Breadcrumb
├── Navigation
├── CommandBar
├── Pivot (Tabs)
├── Tooltip
├── Popover
└── SearchBox

Custom Components:
├── MultiStepForm
├── ApplicationCard
├── StatusTimeline
├── ReportChart
├── PDFViewer
├── FileUploadZone
└── ConfirmationModal
```

### Accessibility Standards
```
WCAG 2.1 Level AA Compliance:
✓ Color Contrast: ≥ 4.5:1 (text), ≥ 3:1 (graphics)
✓ Keyboard Navigation: Full support (Tab, Enter, Escape)
✓ Focus Indicators: Visible on all interactive elements
✓ Screen Readers: ARIA labels, role attributes
✓ Form Labels: <label> linked to inputs
✓ Alternative Text: All images have descriptive alt text
✓ Captions: Video captions (if applicable)
✓ Skip Links: "Skip to main content" link
✓ Responsive: Mobile-first design (320px+)
✓ Font Size: Minimum 14px for body text
✓ Touch Targets: Minimum 48x48px for buttons
```

### Responsive Design Breakpoints
```
Mobile (320px - 767px)
├── Single column layout
├── Hamburger menu for navigation
├── Large touch targets (48x48px)
└── Bottom navigation tabs

Tablet (768px - 1023px)
├── Two-column layout
├── Sidebar navigation collapse
├── Optimized spacing
└── Medium touch targets (40x40px)

Desktop (1024px+)
├── Full multi-column layout
├── Side-by-side panels
├── Optimal spacing and padding
└── Standard cursor targets
```

---

## IMPLEMENTATION ROADMAP

### Phase 1: Foundation & Backend (Weeks 1-6)
- [ ] Node.js + Express server setup
- [ ] MongoDB database & collections design
- [ ] JWT authentication & authorization
- [ ] API endpoints for all CRUD operations
- [ ] Input validation & sanitization
- [ ] Password hashing & security
- [ ] CORS, Helmet, rate limiting
- [ ] Logging with Winston
- [ ] Environment configuration
- [ ] Unit tests (Jest)

### Phase 2: Frontend Setup (Weeks 4-7)
- [ ] React project setup with Vite
- [ ] Fluent UI integration
- [ ] Redux Toolkit setup
- [ ] Routing structure (React Router)
- [ ] API integration (Axios interceptors)
- [ ] Authentication flow (login, register, logout)
- [ ] i18next multilingual setup
- [ ] Responsive CSS framework
- [ ] Component library structure

### Phase 3: Core Features (Weeks 8-12)
- [ ] 7-step application form (all steps)
- [ ] Form validation (client + server)
- [ ] Auto-save draft functionality
- [ ] PDF generation & download
- [ ] File upload to S3
- [ ] Applicant profile management
- [ ] Dashboard for all portals
- [ ] Status tracking system

### Phase 4: Supervisor/Admin Features (Weeks 13-16)
- [ ] Application filtering & search
- [ ] Applicant screening interface
- [ ] Scoring system
- [ ] Report generation (Excel, PDF)
- [ ] Dashboard charts (Recharts)
- [ ] Interview scheduling
- [ ] Admin form customization
- [ ] Post management interface

### Phase 5: Advanced Features (Weeks 17-20)
- [ ] Helpdesk ticketing system
- [ ] Email notification system (SMTP)
- [ ] SMS/Telegram toggles
- [ ] SSL certificate management
- [ ] Logo & branding customization
- [ ] Audit logs & monitoring
- [ ] Advanced analytics
- [ ] User role management

### Phase 6: Security & Compliance (Weeks 21-22)
- [ ] CERT-In compliance audit
- [ ] GIGW compliance check
- [ ] Penetration testing
- [ ] Security hardening
- [ ] HTTPS/SSL setup
- [ ] Data encryption implementation
- [ ] Backup & disaster recovery
- [ ] Incident response testing

### Phase 7: Testing & QA (Weeks 23-24)
- [ ] End-to-end testing (Cypress)
- [ ] Performance testing (Lighthouse)
- [ ] Load testing (k6)
- [ ] User acceptance testing (UAT)
- [ ] Browser compatibility testing
- [ ] Mobile responsiveness testing
- [ ] Accessibility audit (axe DevTools)
- [ ] Security testing (OWASP)

### Phase 8: Deployment & Documentation (Weeks 25-26)
- [ ] Production environment setup
- [ ] CI/CD pipeline (GitHub Actions)
- [ ] Database migration scripts
- [ ] Deployment documentation
- [ ] User documentation
- [ ] Admin manual
- [ ] API documentation (Swagger)
- [ ] Go-live checklist

---

## DEPLOYMENT & DEVOPS

### Linux Server Deployment

### Server Requirements

**Recommended Specifications:**
```
Production Server:
├── CPU: 4+ vCPU (Intel/AMD equivalent)
├── RAM: 8+ GB (16 GB recommended)
├── Storage: 200+ GB SSD
├── Bandwidth: 100 Mbps+
├── OS: Ubuntu 20.04 LTS / RHEL 8+
└── Uptime SLA: 99.5%

Database Server (MongoDB):
├── CPU: 4+ vCPU
├── RAM: 16+ GB
├── Storage: 500+ GB SSD (RAID-6)
├── Replica Set: 3 nodes minimum
├── Backup: Automated daily + weekly
└── Disaster Recovery: Cross-region replication

Backup Server:
├── Storage: 1+ TB SSD/HDD
├── Replication: Real-time mirroring
├── Retention: 30 days rolling
└── Testing: Monthly restore tests
```


### SSL/TLS Configuration

**Nginx Configuration with SSL:**
```nginx
server {
    listen 443 ssl http2;
    server_name recruitment.csir-serc.org;

    ssl_certificate /etc/nginx/certs/certificate.pem;
    ssl_certificate_key /etc/nginx/keys/private-key.pem;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    
    # CORS headers (if needed)
    add_header Access-Control-Allow-Origin "$http_origin" always;

    location / {
        proxy_pass http://frontend:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/ {
        proxy_pass http://backend:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen 80;
    server_name recruitment.csir-serc.org;
    return 301 https://$server_name$request_uri;
}
```

### Monitoring & Logging

**Prometheus Metrics:**
```
- Node.js metrics (CPU, memory, event loop)
- Express metrics (request count, latency, errors)
- MongoDB metrics (query performance, connection pool)
- Application metrics (custom: applications submitted, etc.)
```

**ELK Stack (Elasticsearch, Logstash, Kibana):**
```
- Centralized logging of all services
- Application logs with context
- Error tracking and alerting
- Log retention: 30 days
- Search and analysis capabilities
```

**Health Checks:**
```
GET /health - Returns 200 + JSON status
├── Database: Connected/Disconnected
├── Redis: Connected/Disconnected
├── Uptime: Seconds since start
└── Timestamp: Current server time
```

---

## API ENDPOINTS (Summary)

### Authentication
```
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh-token
POST   /api/auth/forgot-password
POST   /api/auth/reset-password
POST   /api/auth/verify-email
```

### Applications
```
GET    /api/applications
POST   /api/applications
GET    /api/applications/:id
PATCH  /api/applications/:id
DELETE /api/applications/:id
GET    /api/applications/:id/pdf
POST   /api/applications/:id/submit
```

### Posts
```
GET    /api/posts
POST   /api/posts
GET    /api/posts/:id
PATCH  /api/posts/:id
DELETE /api/posts/:id
POST   /api/posts/:id/publish
```

### Applicants
```
GET    /api/applicants/profile
PATCH  /api/applicants/profile
POST   /api/applicants/upload-document
GET    /api/applicants/documents
```

### Admin
```
GET    /api/admin/system-config
PATCH  /api/admin/system-config
POST   /api/admin/users
GET    /api/admin/audit-logs
GET    /api/admin/dashboard
POST   /api/admin/email-settings/test
```

### Notifications
```
POST   /api/notifications/send-email
POST   /api/notifications/send-sms
GET    /api/notifications/templates
POST   /api/tickets
GET    /api/tickets/:id
PATCH  /api/tickets/:id/respond
```

---

## CONCLUSION & NEXT STEPS

This comprehensive architecture document provides:

1. **Complete Technical Specification** - Database, API, security
2. **UI/UX Framework** - Design system, Fluent UI integration
3. **Compliance Roadmap** - GIGW 3.0, CERT-In, GOI standards
4. **Implementation Guide** - 26-week phased approach
5. **Security Hardening** - Encryption, access control, audit logs

### For Implementation:
1. **Hire experienced MERN developers** (4-6 team members)
2. **Set up development environment** (Node.js, MongoDB)
3. **Create Git repository** with branching strategy
4. **Establish CI/CD pipeline** early
5. **Follow security best practices** from day 1
6. **Regular security audits** during development
7. **User testing** starting Phase 3
8. **Documentation** as you build

### Key Considerations:
- **Data security** is paramount - implement encryption everywhere
- **GIGW compliance** required for government organization
- **Scalability** - design for 10,000+ concurrent users
- **Performance** - page load < 3 seconds
- **Accessibility** - WCAG 2.1 Level AA mandatory
- **Disaster recovery** - plan for business continuity

---

**Document prepared for:** CSIR-SERC Recruitment Portal  
**Technology Stack:** MERN + Fluent UI  
**Compliance:** GIGW 3.0, CERT-In 2013 Rules, ISO 27001  
**Expected Timeline:** 6-7 months (26 weeks)  
**Estimated Team:** 5-7 developers + QA + DevOps

---



use the logo file available at - [Branding ](https://drive.google.com/drive/folders/1Yo8BOOPAWyZ2O76QCl_trPVcDh6DXoCS?usp=sharing) for the portal, and show the admin login page and super login page , and show the default credentails and where the credentials are stored. try to use this colour options which is profession, use based on the professional colour grading like blue, professional colouring for portal. Use the background image 
Use noto sans fonts across portal. Use background for login page located at [Gdrive](https://drive.google.com/drive/folders/1Yo8BOOPAWyZ2O76QCl_trPVcDh6DXoCS?usp=sharing)
Use login page with background with gradient effects like professional 
Have seperate page for login, registration and password reset, use password reset and send new user password through email by smtp, 

use smtp setting as - ictserc@gmail.com & app password - yyhoakynckydyybm

make it aadhaar and mobile number as primary for login and registration, login through mobile number or 12 aadhaar number 
