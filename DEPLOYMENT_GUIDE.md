# TimeTracker Pro - Production Deployment Guide

## Overview
This guide provides comprehensive instructions for deploying TimeTracker Pro to production on Replit with full PST timezone support and enterprise-ready configuration.

## Prerequisites

### Required Environment Variables
- `DATABASE_URL` - PostgreSQL connection string (auto-provided by Replit)
- `SESSION_SECRET` - Session encryption key (auto-provided by Replit)
- `REPL_ID` - Replit application ID (auto-provided)
- `REPLIT_DOMAINS` - Deployment domain(s) (auto-provided)

### Optional Configuration
- `NODE_ENV=production` - Automatically set in production deployments
- `TZ=America/Los_Angeles` - PST timezone (configured in code)

## Deployment Steps

### 1. Pre-Deployment Verification
```bash
# Verify build process
npm run build

# Check database schema
npm run db:push

# Run development tests
npm run dev
```

### 2. Database Migration
The application uses Drizzle ORM with automatic schema synchronization:
- Development and production databases are separate
- Schema changes pushed via `npm run db:push`
- No manual migrations required

### 3. Production Deployment

#### Option A: Autoscale Deployment (Recommended)
1. Click "Deploy" button in Replit workspace
2. Select "Autoscale Deployment"
3. Configure machine power based on expected load
4. Set maximum instances for scaling
5. Deploy

#### Option B: Reserved VM Deployment
1. Click "Deploy" button in Replit workspace
2. Select "Reserved VM Deployment"
3. Choose consistent machine specifications
4. Deploy for 24/7 availability

### 4. Post-Deployment Configuration

#### Database Setup
- PostgreSQL database auto-provisioned
- Session table created automatically
- Schema synchronized on first deployment

#### Authentication
- Replit Auth configured automatically
- Session management via PostgreSQL store
- Secure cookie settings enabled

#### Domain Configuration
- Default: `[your-repl-name].replit.app`
- Custom domain: Configure in deployment settings
- SSL/TLS certificates managed automatically

## Environment-Specific Features

### Development Environment
- Memory-based session storage for stability
- Relaxed cookie security settings
- Hot module reloading enabled
- Debug logging active

### Production Environment
- PostgreSQL session storage
- Secure HTTP-only cookies
- SameSite=strict security
- Optimized asset bundles
- PST timezone enforcement

## Critical Production Fixes Applied

### Session Management
```typescript
// Production: PostgreSQL session store
const pgStore = connectPg(session);
const sessionStore = new pgStore({
  conString: process.env.DATABASE_URL,
  createTableIfMissing: false,
  ttl: sessionTtl,
  tableName: "sessions",
});

// Secure cookie configuration
cookie: {
  httpOnly: true,
  secure: true, // HTTPS required
  maxAge: sessionTtl,
  sameSite: 'strict',
}
```

### PST Timezone Configuration
```typescript
// Server startup
process.env.TZ = "America/Los_Angeles";

// Client-side PST handling
const pstDate = now.toLocaleDateString('en-CA', { 
  timeZone: 'America/Los_Angeles' 
});
```

## Monitoring and Troubleshooting

### Accessing Production Logs
1. Navigate to Deployments tab
2. Select your deployment
3. Click "View deploy logs"
4. Use filters: "Errors only", date range, search

### Common Issues and Solutions

#### Authentication Failures
- **Symptom**: Users cannot log in or stay logged in
- **Cause**: Session configuration mismatch
- **Solution**: Verify PostgreSQL session store enabled in production

#### Time Zone Issues
- **Symptom**: Incorrect date/time calculations
- **Cause**: UTC vs PST timezone mismatch
- **Solution**: Verify PST timezone configured server and client-side

#### Database Connection Errors
- **Symptom**: 500 errors on data operations
- **Cause**: Missing DATABASE_URL or connection issues
- **Solution**: Check environment variables and database status

### Performance Optimization

#### Bundle Size Optimization
Current production build:
- Main bundle: 693.48 kB (gzipped: 196.59 kB)
- CSS bundle: 70.00 kB (gzipped: 12.46 kB)

Consider implementing:
- Dynamic imports for code splitting
- Lazy loading for large components
- Service worker for caching

#### Database Performance
- Connection pooling enabled via Neon
- Proper indexing on frequently queried columns
- Efficient query patterns with Drizzle ORM

## Security Considerations

### Authentication Security
- OIDC-based authentication via Replit Auth
- Secure session management with PostgreSQL
- Automatic token refresh handling
- Role-based access control enforcement

### Data Protection
- HTTPS enforcement in production
- Secure cookie configuration
- SQL injection prevention via parameterized queries
- XSS protection via proper data sanitization

### Access Control
- Role-based permissions system
- Project-level access restrictions
- Department-scoped data access
- Audit trails for all operations

## Backup and Recovery

### Database Backups
- Automatic backups managed by Replit
- Point-in-time recovery available
- Manual backup exports via database tools

### Code Repository
- Export complete codebase as ZIP
- Include all source files and configuration
- Exclude node_modules and build artifacts

### Rollback Procedures
- Use Replit checkpoints for quick rollbacks
- Redeploy previous working version
- Database rollback via Replit tools

## Scaling Considerations

### Horizontal Scaling
- Autoscale deployment handles traffic spikes
- Load balancing managed automatically
- Session persistence via PostgreSQL

### Vertical Scaling
- Upgrade machine specifications as needed
- Monitor CPU and memory usage
- Database performance scaling via Replit

### Feature Scaling
- Modular architecture supports feature additions
- Role-based permissions accommodate organizational growth
- Multi-tenant ready organizational structure

## Support and Maintenance

### Regular Maintenance Tasks
- Monitor application logs for errors
- Review performance metrics
- Update dependencies as needed
- Backup verification

### Emergency Procedures
- Contact Replit support for platform issues
- Use rollback features for quick recovery
- Monitor error rates and response times

This deployment guide ensures a smooth transition from development to production with enterprise-ready security, performance, and reliability features.