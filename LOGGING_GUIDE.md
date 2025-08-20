# TimeTracker Pro - Comprehensive Logging Guide

## Overview
TimeTracker Pro includes comprehensive logging throughout the application stack to help your team troubleshoot issues effectively. This guide covers all logging features and how to use them.

## Server-Side Logging

### 1. Enhanced Server Logging
Location: `server/index.ts`

**Features:**
- Process-level error handlers (uncaught exceptions, unhandled rejections)
- Request/response logging with timing
- Error response tracking (4xx/5xx status codes)
- Database connection monitoring
- Session management logging

**Log Levels:**
- 🔴 ERROR - Critical errors requiring immediate attention
- 🟡 WARN - Warning conditions that should be monitored
- 🔵 INFO - General information about system operations
- 🟢 DEBUG - Detailed debugging information

**Example Output:**
```
2025-08-16T20:56:50.422Z 🔵 INFO [TIMEZONE] Set timezone to PST (America/Los_Angeles)
2025-08-16T20:56:50.727Z 🔵 INFO [SERVER] Server started successfully on port 5000
2025-08-16T20:57:01.443Z 🟢 DEBUG [REQUEST] Incoming GET /api/organizations
```

### 2. Authentication Logging
Location: `server/replitAuth.ts`

**Features:**
- OAuth flow tracking
- Token refresh monitoring
- Session validation logging
- User authentication state tracking
- Failed authentication attempts

**Key Functions:**
- `authLog()` - Structured authentication logging
- Session creation/destruction tracking
- Token expiration handling

**Example Output:**
```
2025-08-16T20:57:01.443Z 🟢 [AUTH] Authentication check for GET /api/organizations
2025-08-16T20:57:01.443Z 🔵 [AUTH] User authenticated
2025-08-16T20:57:01.443Z 🟡 [AUTH] Access token expired, attempting refresh
```

### 3. Database Storage Logging
Location: `server/storage.ts`

**Features:**
- User management operations
- Role change tracking
- Database operation success/failure
- Query execution monitoring

**Key Functions:**
- `storageLog()` - Database operation logging
- User creation/update tracking
- Role management logging

**Example Output:**
```
2025-08-16T20:57:01.443Z 🗄️ [STORAGE] UPDATE_USER_ROLE: Updating role for user test-admin-user to admin
2025-08-16T20:57:01.443Z 🗄️ [STORAGE] UPDATE_USER_ROLE: Successfully updated role for admin@test.com
2025-08-16T20:57:01.443Z 🗄️ [STORAGE] GET_USER: Fetching user with id: test-admin-user
```

## Frontend Logging

### 1. Enhanced API Request Logging
Location: `client/src/lib/queryClient.ts`

**Features:**
- Request/response tracking with timing
- Error categorization and details
- Request ID correlation
- Automatic error reporting to backend

**Key Functions:**
- `frontendLog()` - Structured frontend logging
- `apiRequest()` - Enhanced API request logging
- Automatic backend error reporting

**Example Output:**
```
2025-08-16T20:57:01.443Z 🟢 [FRONTEND-API] Request abc123: GET /api/organizations
2025-08-16T20:57:01.443Z 🔵 [FRONTEND-API] Response abc123: 200 (45ms)
2025-08-16T20:57:01.443Z 🔴 [FRONTEND-API] Request def456 failed
```

### 2. Centralized Error Reporting
Frontend errors are automatically sent to the backend via `/api/log/frontend-error` endpoint for centralized logging.

## Logging Categories

### Server Categories:
- **PROCESS** - Process-level events (startup, shutdown, crashes)
- **TIMEZONE** - Timezone configuration
- **SERVER** - Server startup and configuration
- **REQUEST** - HTTP request logging
- **RESPONSE** - HTTP response logging
- **EXPRESS** - Express.js middleware errors
- **AUTH** - Authentication and authorization
- **STORAGE** - Database operations

### Frontend Categories:
- **API** - API request/response logging
- **AUTH** - Frontend authentication state
- **NAVIGATION** - Route changes and navigation
- **USER** - User interaction tracking
- **ERROR** - Frontend error handling

## How to Use Logs for Troubleshooting

### 1. Authentication Issues
Look for logs with `[AUTH]` category:
```bash
grep "AUTH" server_logs.txt
```

Common patterns:
- Failed login attempts
- Token expiration issues
- Session problems
- Permission denied errors

### 2. API Issues
Look for logs with `[REQUEST]` and `[RESPONSE]` categories:
```bash
grep -E "\[REQUEST\]|\[RESPONSE\]" server_logs.txt
```

Common patterns:
- 4xx errors (client issues)
- 5xx errors (server issues)
- Slow response times
- Failed requests

### 3. Database Issues
Look for logs with `[STORAGE]` category:
```bash
grep "STORAGE" server_logs.txt
```

Common patterns:
- Database connection issues
- Query failures
- Data validation errors
- Role management issues

### 4. Frontend Issues
Look for logs in browser console or centralized backend logs:
```bash
grep "FRONTEND" server_logs.txt
```

Common patterns:
- Network request failures
- JavaScript errors
- React component errors
- State management issues

## Log Monitoring Recommendations

### Development Environment
1. Keep browser console open during testing
2. Monitor server console for real-time feedback
3. Pay attention to ERROR and WARN level messages
4. Use DEBUG level for detailed troubleshooting

### Production Environment
1. Set up log aggregation (ELK stack, Splunk, etc.)
2. Configure log rotation to prevent disk space issues
3. Set up alerts for ERROR level messages
4. Monitor authentication failures and security events
5. Track API response times and error rates

## Log Configuration

### Environment Variables
- `NODE_ENV` - Controls logging verbosity
- `LOG_LEVEL` - Set minimum log level (DEBUG, INFO, WARN, ERROR)

### Development vs Production
- Development: All log levels enabled, detailed stack traces
- Production: INFO and above, sanitized error messages

## Security Considerations

1. **PII Protection**: User passwords and sensitive data are never logged
2. **Error Sanitization**: Stack traces limited in production
3. **IP Logging**: User IP addresses logged for security monitoring
4. **Session Tracking**: Session IDs logged for debugging (not session data)

## Performance Impact

The logging system is designed for minimal performance impact:
- Asynchronous logging operations
- Conditional logging based on environment
- Efficient JSON serialization
- Log level filtering

## Troubleshooting Common Issues

### High Memory Usage
- Check for log rotation configuration
- Monitor log file sizes
- Consider reducing DEBUG level logging in production

### Authentication Failures
1. Check `[AUTH]` logs for specific error messages
2. Verify token expiration handling
3. Monitor session creation/destruction
4. Check OIDC configuration

### API Performance Issues
1. Monitor request/response times in logs
2. Check for database query performance
3. Look for error patterns in specific endpoints
4. Monitor resource usage during peak times

## Log Analysis Tools

### Recommended Tools:
1. **grep/awk** - Basic log analysis
2. **jq** - JSON log parsing
3. **ELK Stack** - Elasticsearch, Logstash, Kibana
4. **Splunk** - Enterprise log management
5. **Datadog** - Cloud-based monitoring

### Sample Commands:
```bash
# Find all authentication errors
grep "🔴.*AUTH" logs.txt

# Count errors by category
grep "🔴" logs.txt | awk '{print $4}' | sort | uniq -c

# Monitor real-time logs
tail -f logs.txt | grep -E "🔴|🟡"

# API performance analysis
grep "Response.*ms" logs.txt | awk '{print $NF}' | sort -n
```

This comprehensive logging system ensures your team can quickly identify and resolve issues in both development and production environments.