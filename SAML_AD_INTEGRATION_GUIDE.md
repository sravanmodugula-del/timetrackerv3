# SAML and Active Directory Integration Guide
## TimeTracker Pro Enterprise Authentication

### Overview
This guide provides a comprehensive approach to integrate SAML Single Sign-On (SSO) and Active Directory (AD) authentication into TimeTracker Pro while maintaining the existing Role-Based Access Control (RBAC) system.

### Current Authentication Architecture
- **Existing**: Replit OIDC authentication with session management
- **Database**: PostgreSQL with user roles stored in users table
- **RBAC**: Four-tier system (Admin, Manager, Project Manager, Employee)
- **Session**: Express-session with PostgreSQL store

### Integration Strategy

#### Phase 1: SAML SSO Implementation
**Objective**: Add SAML 2.0 support alongside existing Replit Auth

**Required Dependencies**:
```bash
npm install passport-saml saml2-js node-saml xmldom
npm install --save-dev @types/passport-saml
```

**Key Components**:
1. **SAML Strategy Configuration**
   - Identity Provider (IdP) metadata parsing
   - Service Provider (SP) configuration
   - Certificate management for secure assertions
   - Attribute mapping for user profiles

2. **Middleware Integration**
   - Passport.js SAML strategy
   - Route handlers for SAML endpoints
   - Session management compatibility
   - Error handling and logging

#### Phase 2: Active Directory Integration
**Objective**: Enable AD authentication with group-based role mapping

**Required Dependencies**:
```bash
npm install passport-ldapauth ldapjs activedirectory2
npm install --save-dev @types/ldapjs
```

**Key Components**:
1. **LDAP/AD Configuration**
   - Connection string and credentials
   - Search base and filters
   - Group membership queries
   - User attribute mapping

2. **Role Mapping System**
   - AD groups to TimeTracker roles
   - Automatic role assignment
   - Group hierarchy support
   - Fallback role configuration

### Implementation Plan

#### 1. Authentication Strategy Extension

**File**: `server/auth/strategies/saml.ts`
```typescript
import { Strategy as SamlStrategy } from 'passport-saml';
import { Strategy as LdapStrategy } from 'passport-ldapauth';

export const configureSamlStrategy = (passport: any) => {
  passport.use('saml', new SamlStrategy({
    callbackUrl: process.env.SAML_CALLBACK_URL,
    entryPoint: process.env.SAML_ENTRY_POINT,
    issuer: process.env.SAML_ISSUER,
    cert: process.env.SAML_CERT,
    // Additional SAML configuration
  }, async (profile, done) => {
    // User processing and role mapping
  }));
};

export const configureLdapStrategy = (passport: any) => {
  passport.use('ldap', new LdapStrategy({
    server: {
      url: process.env.LDAP_URL,
      bindDN: process.env.LDAP_BIND_DN,
      bindCredentials: process.env.LDAP_BIND_CREDENTIALS,
      searchBase: process.env.LDAP_SEARCH_BASE,
      searchFilter: process.env.LDAP_SEARCH_FILTER
    }
  }, async (user, done) => {
    // AD user processing and group mapping
  }));
};
```

#### 2. Role Mapping Configuration

**File**: `server/auth/roleMapping.ts`
```typescript
interface RoleMappingConfig {
  saml: {
    roleAttribute: string;
    roleMappings: Record<string, string>;
  };
  ldap: {
    groupMappings: Record<string, string>;
    defaultRole: string;
  };
}

const roleMappingConfig: RoleMappingConfig = {
  saml: {
    roleAttribute: 'http://schemas.microsoft.com/ws/2008/06/identity/claims/role',
    roleMappings: {
      'TimeTracker-Admins': 'admin',
      'TimeTracker-Managers': 'manager',
      'TimeTracker-ProjectManagers': 'project_manager',
      'TimeTracker-Employees': 'employee'
    }
  },
  ldap: {
    groupMappings: {
      'CN=TimeTracker-Admins,OU=Groups,DC=company,DC=com': 'admin',
      'CN=TimeTracker-Managers,OU=Groups,DC=company,DC=com': 'manager',
      'CN=TimeTracker-ProjectManagers,OU=Groups,DC=company,DC=com': 'project_manager',
      'CN=TimeTracker-Employees,OU=Groups,DC=company,DC=com': 'employee'
    },
    defaultRole: 'employee'
  }
};
```

#### 3. Enhanced User Management

**Database Schema Extensions**:
```sql
-- Add authentication source tracking
ALTER TABLE users ADD COLUMN auth_source VARCHAR(20) DEFAULT 'replit';
ALTER TABLE users ADD COLUMN external_id VARCHAR(255);
ALTER TABLE users ADD COLUMN last_login TIMESTAMP;
ALTER TABLE users ADD COLUMN ad_groups TEXT[];

-- Add indexes for performance
CREATE INDEX idx_users_auth_source ON users(auth_source);
CREATE INDEX idx_users_external_id ON users(external_id);
```

**Storage Layer Updates**:
```typescript
// server/storage.ts additions
async upsertExternalUser(userData: {
  externalId: string;
  email: string;
  name: string;
  authSource: 'saml' | 'ldap';
  role: string;
  adGroups?: string[];
}): Promise<User> {
  // Implementation for external user management
}

async getUserByExternalId(externalId: string, authSource: string): Promise<User | undefined> {
  // Implementation for external user lookup
}
```

#### 4. Authentication Routes

**File**: `server/routes/auth.ts`
```typescript
// SAML routes
router.get('/auth/saml', passport.authenticate('saml'));
router.post('/auth/saml/callback', passport.authenticate('saml'), (req, res) => {
  res.redirect('/dashboard');
});

// LDAP routes
router.post('/auth/ldap', passport.authenticate('ldap'), (req, res) => {
  res.redirect('/dashboard');
});

// Metadata endpoint for SAML SP
router.get('/auth/saml/metadata', (req, res) => {
  res.type('application/xml');
  res.send(generateSamlMetadata());
});
```

#### 5. Environment Configuration

**Required Environment Variables**:
```bash
# SAML Configuration
SAML_ENABLED=true
SAML_ENTRY_POINT=https://identity-provider.company.com/saml/sso
SAML_ISSUER=timetracker-pro
SAML_CALLBACK_URL=https://your-app.replit.app/auth/saml/callback
SAML_CERT="-----BEGIN CERTIFICATE-----..."
SAML_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----..."

# Active Directory Configuration
LDAP_ENABLED=true
LDAP_URL=ldaps://dc.company.com:636
LDAP_BIND_DN=CN=serviceaccount,OU=ServiceAccounts,DC=company,DC=com
LDAP_BIND_CREDENTIALS=password123
LDAP_SEARCH_BASE=OU=Users,DC=company,DC=com
LDAP_SEARCH_FILTER=(sAMAccountName={{username}})

# Role Mapping
ENABLE_AUTO_ROLE_MAPPING=true
DEFAULT_ROLE=employee
```

### Frontend Integration

#### 1. Login Page Updates

**File**: `client/src/pages/Login.tsx`
```typescript
interface AuthMethod {
  type: 'replit' | 'saml' | 'ldap';
  name: string;
  enabled: boolean;
}

const Login = () => {
  const authMethods: AuthMethod[] = [
    { type: 'replit', name: 'Replit', enabled: true },
    { type: 'saml', name: 'Company SSO', enabled: process.env.VITE_SAML_ENABLED === 'true' },
    { type: 'ldap', name: 'Active Directory', enabled: process.env.VITE_LDAP_ENABLED === 'true' }
  ];

  return (
    <div className="auth-methods">
      {authMethods.filter(method => method.enabled).map(method => (
        <AuthMethodButton key={method.type} method={method} />
      ))}
    </div>
  );
};
```

#### 2. User Profile Updates

**Enhanced User Context**:
```typescript
interface AuthenticatedUser extends User {
  authSource: 'replit' | 'saml' | 'ldap';
  externalId?: string;
  adGroups?: string[];
  lastLogin?: Date;
}
```

### Security Considerations

#### 1. Certificate Management
- Store SAML certificates securely in environment variables
- Implement certificate rotation procedures
- Validate certificate chains and expiration

#### 2. Session Security
- Maintain existing session security measures
- Implement session timeout for external auth
- Add logout handling for all auth methods

#### 3. Role Synchronization
- Periodic sync with AD groups
- Audit trail for role changes
- Fallback authentication methods

### Testing Strategy

#### 1. SAML Testing
```typescript
// tests/auth/saml.test.ts
describe('SAML Authentication', () => {
  it('should process SAML assertions correctly', async () => {
    // Test SAML assertion processing
  });
  
  it('should map SAML attributes to user roles', async () => {
    // Test role mapping
  });
});
```

#### 2. LDAP Testing
```typescript
// tests/auth/ldap.test.ts
describe('LDAP Authentication', () => {
  it('should authenticate against Active Directory', async () => {
    // Test AD authentication
  });
  
  it('should map AD groups to application roles', async () => {
    // Test group mapping
  });
});
```

### Deployment Considerations

#### 1. Infrastructure Requirements
- SSL/TLS certificates for SAML endpoints
- Network connectivity to AD domain controllers
- Firewall rules for LDAP/LDAPS connections

#### 2. Configuration Management
- Secure storage of AD credentials
- SAML metadata management
- Environment-specific configurations

#### 3. Monitoring and Logging
- Authentication success/failure tracking
- Role mapping audit logs
- Performance monitoring for AD queries

### Migration Strategy

#### Phase 1: Parallel Authentication (2-3 weeks)
1. Implement SAML and LDAP strategies
2. Add environment configuration
3. Update user storage layer
4. Basic testing and validation

#### Phase 2: Frontend Integration (1-2 weeks)
1. Update login interface
2. Add authentication method selection
3. User profile enhancements
4. End-to-end testing

#### Phase 3: Production Rollout (1 week)
1. Gradual rollout to user groups
2. Monitor authentication metrics
3. Performance optimization
4. Documentation and training

### Maintenance Procedures

#### 1. Regular Tasks
- Certificate renewal monitoring
- AD group membership audits
- Authentication method usage analytics
- Performance optimization

#### 2. Troubleshooting
- SAML assertion debugging
- LDAP connection issues
- Role mapping problems
- Session management issues

### Success Metrics

#### 1. Authentication Performance
- Login success rate > 99%
- Authentication time < 3 seconds
- Zero authentication bypasses

#### 2. User Experience
- Single sign-on adoption rate
- Support ticket reduction
- User satisfaction scores

#### 3. Security Compliance
- Regular security audits
- Compliance with corporate policies
- Audit trail completeness

This integration approach maintains the existing application architecture while adding enterprise-grade authentication capabilities. The implementation preserves all current RBAC functionality while extending it with enterprise identity management.