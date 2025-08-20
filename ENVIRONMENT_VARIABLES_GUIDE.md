# Environment Variables Configuration Guide
## TimeTracker Pro Production Deployment

### Overview
This guide provides comprehensive documentation for all environment variables required for TimeTracker Pro deployment, including security considerations and configuration examples.

## Critical Environment Variables

### NODE_ENV (CRITICAL)
**Purpose**: Controls application behavior and security features
**Required**: Yes
**Default**: None (must be explicitly set)
**Production Value**: `production`
**Security Impact**: HIGH

```bash
NODE_ENV=production
```

**Why Critical**:
- Controls development authentication bypass (Lines 222-262 in server/replitAuth.ts)
- Determines session security settings (secure cookies, etc.)
- Controls error message detail exposure
- Affects static file serving vs Vite dev server

**Security Risk**: If not set to 'production', development authentication bypass allows unrestricted admin access

### SESSION_SECRET (CRITICAL)
**Purpose**: Encrypts user session data
**Required**: Yes
**Default**: None (application will crash)
**Security Impact**: HIGH

```bash
SESSION_SECRET=your-cryptographically-secure-random-string-minimum-32-characters
```

**Generation Example**:
```bash
# Generate secure session secret
openssl rand -base64 32
```

**Requirements**:
- Minimum 32 characters
- Cryptographically secure random generation
- Never reuse between environments
- Store securely (never in source code)

### REPL_ID (CRITICAL)
**Purpose**: Replit application identifier for OAuth
**Required**: Yes
**Default**: None (authentication will fail)
**Security Impact**: HIGH

```bash
REPL_ID=your-replit-application-id
```

**How to Obtain**:
- Available in Replit project settings
- Used for OAuth client identification
- Must match registered Replit application

### REPLIT_DOMAINS (CRITICAL)
**Purpose**: Authorized domains for OAuth callbacks
**Required**: Yes
**Default**: None (authentication will fail)
**Security Impact**: HIGH

```bash
# Single domain
REPLIT_DOMAINS=myapp.replit.app

# Multiple domains (comma-separated)
REPLIT_DOMAINS=myapp.replit.app,myapp.customdomain.com
```

**Requirements**:
- Must exactly match production domain(s)
- No trailing slashes
- HTTPS assumed for production
- Comma-separated for multiple domains

### DATABASE_URL (CRITICAL)
**Purpose**: PostgreSQL connection string
**Required**: Yes
**Default**: None (application will crash)
**Security Impact**: HIGH

```bash
DATABASE_URL=postgresql://username:password@hostname:port/database?sslmode=require
```

**Production Requirements**:
- SSL enabled (`sslmode=require`)
- Secure credentials
- Connection pooling recommended
- Network security (firewall, VPC)

## Optional Environment Variables

### PORT
**Purpose**: Application server port
**Required**: No
**Default**: `5000`
**Security Impact**: LOW

```bash
PORT=5000
```

**Considerations**:
- Use port 80/443 for production with reverse proxy
- Ensure port not conflicting with other services
- Configure firewall rules appropriately

### TZ (Timezone)
**Purpose**: Server timezone setting
**Required**: No
**Default**: `America/Los_Angeles` (hardcoded)
**Security Impact**: NONE

```bash
TZ=America/Los_Angeles
```

**Options**:
- Any valid timezone identifier
- Affects log timestamps and date calculations
- Should match business operational timezone

## Environment-Specific Configurations

### Development Environment
```bash
NODE_ENV=development
SESSION_SECRET=dev-secret-not-for-production
REPL_ID=your-dev-repl-id
REPLIT_DOMAINS=localhost:5000,your-dev-domain.replit.dev
DATABASE_URL=postgresql://localhost/timetracker_dev
PORT=5000
TZ=America/Los_Angeles
```

**Development Features Enabled**:
- Authentication bypass with test admin user
- Detailed error messages with stack traces
- Vite development server for hot reloading
- Less restrictive session security settings

### Staging Environment
```bash
NODE_ENV=production
SESSION_SECRET=staging-specific-secure-secret
REPL_ID=your-staging-repl-id
REPLIT_DOMAINS=staging.yourdomain.com
DATABASE_URL=postgresql://staging_user:secure_password@staging_host:5432/timetracker_staging?sslmode=require
PORT=5000
TZ=America/Los_Angeles
```

### Production Environment
```bash
NODE_ENV=production
SESSION_SECRET=production-cryptographically-secure-secret-key
REPL_ID=your-production-repl-id
REPLIT_DOMAINS=app.yourdomain.com,www.yourdomain.com
DATABASE_URL=postgresql://prod_user:ultra_secure_password@prod_host:5432/timetracker_production?sslmode=require
PORT=5000
TZ=America/Los_Angeles
```

## Security Best Practices

### Secret Management
1. **Never commit secrets to source control**
2. **Use secure secret management systems** (AWS Secrets Manager, Azure Key Vault, etc.)
3. **Rotate secrets regularly** (quarterly minimum)
4. **Monitor for secret exposure** in logs and error messages
5. **Use different secrets for each environment**

### Environment Validation
```javascript
// Recommended startup validation
const requiredEnvVars = [
  'NODE_ENV',
  'SESSION_SECRET', 
  'REPL_ID',
  'REPLIT_DOMAINS',
  'DATABASE_URL'
];

const missingVars = requiredEnvVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  console.error(`❌ Missing required environment variables: ${missingVars.join(', ')}`);
  process.exit(1);
}

if (process.env.NODE_ENV !== 'production') {
  console.warn('⚠️  WARNING: Not running in production mode');
}
```

## Troubleshooting Environment Issues

### Authentication Failures
**Symptoms**: Users cannot log in, OAuth errors
**Common Causes**:
- REPL_ID incorrect or missing
- REPLIT_DOMAINS doesn't match actual domain
- OAuth callback URL misconfiguration

**Debugging Steps**:
1. Verify REPL_ID matches Replit application
2. Check REPLIT_DOMAINS exactly matches URL
3. Ensure no trailing slashes in domain configuration
4. Validate OAuth callback URL registration

### Session Problems
**Symptoms**: Users logged out unexpectedly, session errors
**Common Causes**:
- SESSION_SECRET missing or incorrect
- Database connection issues (for production session store)
- Cookie security settings blocking sessions

**Debugging Steps**:
1. Verify SESSION_SECRET is set and secure
2. Check database connectivity for session storage
3. Validate cookie settings for domain/HTTPS
4. Review session middleware configuration

### Database Connection Issues
**Symptoms**: Application crashes on startup, database errors
**Common Causes**:
- DATABASE_URL format incorrect
- Database credentials invalid
- SSL configuration mismatch
- Network connectivity problems

**Debugging Steps**:
1. Test database connection independently
2. Validate connection string format
3. Check SSL requirements and certificates
4. Verify network access and firewall rules

### Development vs Production Issues
**Symptoms**: Works in development but fails in production
**Common Causes**:
- NODE_ENV not set to 'production'
- Environment-specific configurations missing
- Static file serving vs Vite dev server differences

**Debugging Steps**:
1. Verify NODE_ENV=production is set
2. Check all production environment variables
3. Test static file serving configuration
4. Validate build process completed successfully

## Environment Variable Checklist

### Pre-Deployment Validation
- [ ] All required environment variables documented
- [ ] Secrets generated with cryptographic security
- [ ] Database connection string tested
- [ ] OAuth configuration verified
- [ ] Environment-specific values set correctly

### Security Validation
- [ ] NODE_ENV=production in production environment
- [ ] SESSION_SECRET is cryptographically secure
- [ ] DATABASE_URL uses SSL in production
- [ ] No secrets committed to source control
- [ ] Secret rotation schedule established

### Production Readiness
- [ ] Startup validation script implemented
- [ ] Error handling for missing variables
- [ ] Monitoring for configuration issues
- [ ] Documentation updated and accessible
- [ ] Team training on environment management

This guide ensures secure and reliable environment configuration for all deployment scenarios of TimeTracker Pro.