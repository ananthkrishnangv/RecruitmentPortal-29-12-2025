# CSIR-SERC Recruitment Portal - Walkthrough

A comprehensive government recruitment portal with advanced UI/UX using Bootstrap, TailwindCSS, and professional blue color theming.

---

## Project Structure

Recruitmet-Antigravity/

├── backend/

│   ├── package.json

│   ├── .env / .env.example

│   └── src/

│       ├── server.js                 # Express entry point

│       ├── seed.js                   # Database seeding script

│       ├── config/

│       │   ├── database.js           # MongoDB connection

│       │   ├── email.js              # Gmail SMTP service

│       │   └── jwt.js                # JWT token utilities

│       ├── models/

│       │   ├── User.js               # User with Aadhaar/Mobile login

│       │   ├── OTP.js                # OTP verification

│       │   ├── Applicant.js          # Applicant profile

│       │   ├── Post.js               # Job vacancies

│       │   └── Application.js        # Applications

│       ├── controllers/

│       │   ├── authController.js     # Authentication logic

│       │   └── adminController.js    # Admin operations

│       ├── routes/

│       │   ├── authRoutes.js

│       │   └── adminRoutes.js

│       └── middleware/

│           └── auth.js               # JWT & RBAC middleware

│

├── frontend/

│   ├── package.json

│   ├── vite.config.js

│   ├── tailwind.config.js

│   ├── postcss.config.js

│   ├── index.html

│   ├── public/

│   │   ├── csirlogo.jpg

│   │   └── CSIR-SERC Main Building.png

│   └── src/

│       ├── main.jsx

│       ├── App.jsx

│       ├── index.css                 # TailwindCSS + Custom CSS

│       ├── context/

│       │   └── AuthContext.jsx       # Authentication state

│       ├── services/

│       │   └── api.js                # Axios with interceptors

│       ├── components/

│       │   └── ProtectedRoute.jsx    # Role-based access

│       └── pages/

│           ├── auth/

│           │   ├── LoginPage.jsx     # Aadhaar/Mobile login

│           │   ├── AdminLoginPage.jsx

│           │   ├── SupervisorLoginPage.jsx

│           │   ├── RegisterPage.jsx

│           │   ├── ForgotPasswordPage.jsx

│           │   └── ResetPasswordPage.jsx

│           └── dashboard/

│               ├── ApplicantDashboard.jsx

│               ├── AdminDashboard.jsx

│               └── SupervisorDashboard.jsx

│

├── csirlogo.jpg                      # CSIR logo

├── CSIR-SERC Main Building.png       # Background image

├── serc.crt                          # SSL certificate

└── serc.key                          # SSL key

---

## Default Credentials

|Role|Email|Password|Storage|
|---|---|---|---|
|**Admin**|`admin@serc.res.in`|`Admin@SERC2025`|MongoDB `users` collection|
|**Supervisor**|`supervisor@serc.res.in`|`Supervisor@SERC2025`|MongoDB `users` collection|

> **Note**: Run `npm run seed` in the backend directory to create these users.

---

## Setup Instructions

### 1. Backend Setup

cd backend

# Install dependencies

npm install

# Create .env file (copy from .env.example)

cp .env.example .env

# Edit .env if needed (SMTP already configured)

# Seed database with default users

npm run seed

# Start development server

npm run dev

**Backend runs on**: `http://localhost:5000`

### 2. Frontend Setup

cd frontend

# Install dependencies  

npm install

# Start development server

npm run dev

**Frontend runs on**: `http://localhost:5173`

---

## Key Features Implemented

### ✅ Authentication System

- **Aadhaar-based login** (12-digit validation)
- **Mobile-based login** (10-digit, starts with 6-9)
- **Email-based login** for Admin/Supervisor
- **Password set during registration**
- **Self-service password reset** via email OTP
- **JWT tokens** with refresh token support
- **Account locking** after 5 failed attempts

### ✅ Professional UI/UX

- **Noto Sans font** throughout portal
- **Blue color palette** (CSIR professional theme)
- **Building background** with gradient overlay on login
- **Glassmorphism** login cards
- **Responsive design** for mobile/tablet/desktop
- **Bootstrap 5 + TailwindCSS** styling

### ✅ Three Portal Types

1. **Applicant Portal** (`/applicant`)
    
    - Dashboard with stats cards
    - Quick actions
    - Application tracking
2. **Admin Portal** (`/admin`)
    
    - User management
    - Post management
    - System statistics
    - Configuration panel
3. **Supervisor Portal** (`/supervisor`)
    
    - Application review
    - Screening interface
    - Interview scheduling
    - Reports generation

### ✅ SMTP Configuration

- **Email**: `ictserc@gmail.com`
- **App Password**: Configured in `.env`
- **OTP Templates**: Professional HTML emails

---

## Testing the Portal

1. **Start Backend**:
    
    cd backend && npm run dev
    
2. **Start Frontend**:
    
    cd frontend && npm run dev
    
3. **Open Browser**: Navigate to `http://localhost:5173`
    
4. **Test Admin Login**:
    
    - Click "Admin Login" button
    - Email: `admin@serc.res.in`
    - Password: `Admin@SERC2025`
5. **Test Supervisor Login**:
    
    - Click "Supervisor Login" button
    - Email: `supervisor@serc.res.in`
    - Password: `Supervisor@SERC2025`
6. **Test Applicant Registration**:
    
    - Click "Register Now"
    - Enter Aadhaar/Mobile number
    - Set password
    - Verify email via OTP

---

## API Endpoints

### Authentication

|Method|Endpoint|Description|
|---|---|---|
|POST|`/api/auth/register`|Register new applicant|
|POST|`/api/auth/login`|Login with Aadhaar/Mobile/Email|
|POST|`/api/auth/forgot-password`|Request password reset OTP|
|POST|`/api/auth/reset-password`|Reset password with OTP|
|POST|`/api/auth/verify-email`|Verify email with OTP|
|GET|`/api/auth/me`|Get current user (protected)|

### Admin

|Method|Endpoint|Description|
|---|---|---|
|GET|`/api/admin/dashboard`|Dashboard statistics|
|GET|`/api/admin/users`|List all users|
|PATCH|`/api/admin/users/:id/role`|Update user role|
|PATCH|`/api/admin/users/:id/status`|Toggle user active status|

---

## Next Steps (Phase 4-6)

The following features are scaffolded but need full implementation:

- [ ]  Complete applicant profile management
- [ ]  7-step application form
- [ ]  Document upload with S3 integration
- [ ]  Post/vacancy management in admin
- [ ]  Application screening workflow
- [ ]  Interview scheduling system
- [ ]  PDF generation
- [ ]  Helpdesk ticketing

---

## Technology Stack

| Layer              | Technology                               |
| ------------------ | ---------------------------------------- |
| **Frontend**       | React 18, Vite, TailwindCSS, Bootstrap 5 |
| **Backend**        | Node.js, Express.js                      |
| **Database**       | MongoDB with Mongoose                    |
| **Authentication** | JWT, bcrypt                              |
| **Email**          | Nodemailer with Gmail SMTP               |
| **Styling**        | CSS-in-JS, Noto Sans font                |