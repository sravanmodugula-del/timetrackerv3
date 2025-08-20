# TimeTracker Pro - Complete Source Code Package

## Package Contents

This archive contains the complete source code for TimeTracker Pro, a comprehensive employee time tracking application with PST timezone support.

### 📁 Project Structure

```
timetracker-pro/
├── client/                    # React frontend application
│   ├── src/
│   │   ├── components/       # Reusable UI components
│   │   ├── hooks/           # Custom React hooks
│   │   ├── lib/             # Utility libraries
│   │   ├── pages/           # Application pages
│   │   ├── App.tsx          # Main application component
│   │   └── main.tsx         # Application entry point
│   └── index.html           # HTML template
├── server/                   # Express.js backend
│   ├── auth/                # Authentication middleware
│   ├── db.ts                # Database connection
│   ├── index.ts             # Server entry point
│   ├── replitAuth.ts        # Replit Auth integration
│   ├── routes.ts            # API route handlers
│   ├── storage.ts           # Database operations
│   └── vite.ts              # Vite integration
├── shared/                   # Shared types and utilities
│   ├── schema.ts            # Database schema & types
│   └── timezone.ts          # PST timezone utilities
├── attached_assets/          # User-uploaded assets
├── components.json           # shadcn/ui configuration
├── drizzle.config.ts        # Database configuration
├── package.json             # Dependencies & scripts
├── tailwind.config.ts       # Tailwind CSS configuration
├── tsconfig.json            # TypeScript configuration
├── vite.config.ts           # Vite build configuration
├── DATABASE_SCHEMA.md       # Database documentation
├── DEPLOYMENT_GUIDE.md      # Production deployment guide
├── BRANDING_GUIDE.md        # UI/UX branding guidelines
└── replit.md                # Project overview & recent changes
```

### 🚀 Quick Start

1. **Install Dependencies**
   ```bash
   npm install
   ```

2. **Set Up Environment Variables**
   ```bash
   # Required for Replit deployment
   DATABASE_URL=your_postgresql_connection_string
   SESSION_SECRET=your_session_secret
   REPL_ID=your_repl_id
   REPLIT_DOMAINS=your_domain
   ```

3. **Initialize Database**
   ```bash
   npm run db:push
   ```

4. **Start Development Server**
   ```bash
   npm run dev
   ```

5. **Build for Production**
   ```bash
   npm run build
   ```

### 🏗️ Architecture Overview

#### Frontend (React + TypeScript)
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite for fast development and optimized builds
- **UI Library**: shadcn/ui components built on Radix UI
- **Styling**: Tailwind CSS with custom design system
- **Routing**: Wouter for client-side navigation
- **State Management**: TanStack Query for server state
- **Forms**: React Hook Form with Zod validation

#### Backend (Node.js + Express)
- **Runtime**: Node.js with Express.js framework
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: Replit Auth with OpenID Connect
- **Session Management**: PostgreSQL-backed sessions
- **API Design**: RESTful endpoints with role-based access

#### Database (PostgreSQL)
- **ORM**: Drizzle ORM for type-safe database operations
- **Schema**: Comprehensive organizational hierarchy
- **Timezone**: PST (Pacific Standard Time) support
- **Migrations**: Automatic schema synchronization

### ⭐ Key Features

#### Time Tracking
- ✅ Dual input modes: Start/End Time and Manual Duration
- ✅ PST timezone support throughout application
- ✅ Historical time entry editing capabilities
- ✅ Quick date selection with PST-aware calculations
- ✅ Project and task association with time entries

#### User Management
- ✅ Role-based access control (Admin, Manager, Employee, Viewer)
- ✅ Replit Auth integration for secure authentication
- ✅ Employee profile management with organizational hierarchy
- ✅ Department and organization structure support

#### Project Management
- ✅ Enterprise-wide and restricted project access
- ✅ Task management within projects
- ✅ Project assignment and team collaboration
- ✅ Color-coded project organization

#### Analytics & Reporting
- ✅ Real-time dashboard with interactive charts
- ✅ Time tracking analytics by project and department
- ✅ CSV export functionality for reports
- ✅ Date range filtering with PST support

#### Production Ready
- ✅ Environment-aware configuration (dev/production)
- ✅ Secure session management with PostgreSQL
- ✅ Optimized build process with asset bundling
- ✅ Comprehensive error handling and logging

### 🔧 Technology Stack

#### Core Dependencies
```json
{
  "react": "^18.x",
  "express": "^4.x",
  "drizzle-orm": "^0.x",
  "@neondatabase/serverless": "^0.x",
  "tailwindcss": "^3.x",
  "typescript": "^5.x",
  "vite": "^5.x"
}
```

#### Authentication & Security
- Replit Auth (OIDC)
- Express Session with PostgreSQL store
- Role-based permissions system
- Secure cookie configuration

#### Database & ORM
- PostgreSQL (Neon serverless)
- Drizzle ORM with Zod validation
- Automatic schema migrations
- Connection pooling

### 🌍 PST Timezone Implementation

The application operates entirely in Pacific Standard Time:

#### Server-Side
```typescript
// Timezone configuration
process.env.TZ = "America/Los_Angeles";

// PST date calculations
const todayPST = now.toLocaleDateString('en-CA', { 
  timeZone: 'America/Los_Angeles' 
});
```

#### Client-Side
```typescript
// PST-aware date utilities
export function getCurrentPSTDate(): string {
  const now = new Date();
  return now.toLocaleDateString('en-CA', { 
    timeZone: 'America/Los_Angeles' 
  });
}
```

### 🚀 Deployment Options

#### Replit Platform (Recommended)
- **Autoscale Deployment**: Automatic scaling based on traffic
- **Reserved VM**: Consistent resources for 24/7 availability
- **Database**: Managed PostgreSQL with automatic backups
- **SSL/TLS**: Automatic certificate management

#### Self-Hosted Options
- Docker containerization ready
- Environment variable configuration
- PostgreSQL database required
- HTTPS recommended for production

### 📊 Database Schema

The application uses a comprehensive PostgreSQL schema supporting:

- **User Management**: Profiles, roles, authentication
- **Organizational Structure**: Organizations → Departments → Employees
- **Project Management**: Projects, tasks, access control
- **Time Tracking**: Entries with PST timezone support
- **Session Management**: Secure authentication sessions

See `DATABASE_SCHEMA.md` for detailed table structures and relationships.

### 🔧 Configuration Files

#### Essential Configuration
- `package.json` - Dependencies and build scripts
- `tsconfig.json` - TypeScript compilation settings
- `tailwind.config.ts` - UI styling configuration
- `vite.config.ts` - Build and development server settings
- `drizzle.config.ts` - Database connection and migration settings

#### Documentation
- `DEPLOYMENT_GUIDE.md` - Production deployment instructions
- `DATABASE_SCHEMA.md` - Complete database documentation
- `BRANDING_GUIDE.md` - UI/UX design guidelines
- `replit.md` - Project overview and recent changes

### 🔒 Security Features

- OIDC authentication via Replit Auth
- Role-based access control with granular permissions
- Secure session management with PostgreSQL storage
- SQL injection prevention via parameterized queries
- XSS protection through proper data sanitization
- HTTPS enforcement in production environments

### 📈 Performance Optimizations

- Optimized React component rendering
- Efficient database queries with proper indexing
- Asset bundling and minification for production
- Connection pooling for database efficiency
- Lazy loading for improved initial load times

### 📞 Support Information

For deployment assistance or technical support:
1. Review `DEPLOYMENT_GUIDE.md` for step-by-step instructions
2. Check `DATABASE_SCHEMA.md` for database-related questions
3. Consult `replit.md` for project-specific context and recent changes

---

**Version**: Production-ready with PST timezone support
**Last Updated**: August 15, 2025
**Build Status**: ✅ Successful (693.48 kB main bundle)
**Database Schema**: ✅ Validated and synchronized