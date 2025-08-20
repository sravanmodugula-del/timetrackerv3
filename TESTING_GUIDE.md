# TimeTracker Pro - Testing Guide

## Overview

This guide covers the comprehensive unit testing infrastructure implemented for TimeTracker Pro, including automated testing pipelines that run after every code change.

## Testing Stack

- **Test Framework**: Vitest (fast, modern testing framework)
- **UI Testing**: React Testing Library
- **Mocking**: Vitest built-in mocking capabilities
- **Coverage**: Built-in coverage reporting
- **CI/CD**: GitHub Actions automated pipeline

## Test Structure

```
tests/
├── setup.ts              # Test environment configuration
├── utils/
│   └── test-utils.tsx     # Custom render helpers and mock data
├── components/            # Component unit tests
│   ├── auth.test.tsx
│   ├── project-form.test.tsx
│   └── time-entry.test.tsx
├── api/                   # API endpoint tests
│   ├── auth.test.ts
│   ├── projects.test.ts
│   └── time-entries.test.ts
└── integration/           # Integration tests
    └── dashboard.test.ts
```

## Running Tests

### Local Development

```bash
# Run all tests in watch mode
npm run test:watch

# Run tests once
npm run test

# Run specific test file
npm run test auth.test.tsx

# Generate coverage report
npm run test:coverage

# Run tests for CI (with JUnit output)
npm run test:ci
```

### Available Test Commands

- `npm run test` - Run tests in interactive watch mode
- `npm run test:run` - Run tests once and exit
- `npm run test:coverage` - Generate coverage report
- `npm run test:watch` - Run tests in watch mode
- `npm run test:ci` - Run tests with CI-specific output format

## Test Categories

### 1. Authentication Tests (`tests/components/auth.test.tsx`)

Tests the authentication system including:
- Loading states
- User authentication status
- Role-based permissions
- Admin, Manager, Employee, and Viewer role access

**Example Test:**
```typescript
it('should display user information when authenticated as admin', () => {
  mockUseAuth.mockReturnValue({
    user: mockAdminUser,
    isAuthenticated: true,
    isLoading: false,
  });

  render(<TestAuthComponent />);
  
  expect(screen.getByTestId('user-email')).toHaveTextContent('admin@test.com');
  expect(screen.getByTestId('can-create-project')).toHaveTextContent('yes');
});
```

### 2. Component Tests

#### Project Form Tests (`tests/components/project-form.test.tsx`)
- Form rendering and field interactions
- Input validation
- Form submission handling
- Error state management

#### Time Entry Tests (`tests/components/time-entry.test.tsx`)
- Time calculation accuracy
- Form validation
- Duration calculations (including overnight entries)
- API integration error handling

### 3. API Tests

#### Authentication API (`tests/api/auth.test.ts`)
- User authentication endpoints
- Role management API
- Permission validation
- Session management
- Token refresh handling

#### Projects API (`tests/api/projects.test.ts`)
- CRUD operations for projects
- Employee assignment functionality
- Project filtering and search
- Data validation
- Permission-based access control

#### Time Entries API (`tests/api/time-entries.test.ts`)
- Time entry CRUD operations
- Duration calculations
- Date/time validation
- Aggregation functions
- Role-based data filtering

### 4. Integration Tests (`tests/integration/dashboard.test.ts`)

Tests complete dashboard functionality:
- Statistics aggregation
- Project breakdown calculations
- Recent activity displays
- Department-level reporting
- Data consistency across endpoints
- PST timezone handling

## Mock Data and Utilities

### Test Utilities (`tests/utils/test-utils.tsx`)

Provides helper functions and mock data:
- Custom render function with providers
- Mock user data for different roles
- Mock project and time entry data
- API response creation helpers

**Mock Data Examples:**
```typescript
export const mockUser = {
  id: 'test-user-1',
  email: 'test@example.com',
  role: 'admin',
  // ... other properties
};

export const mockProject = {
  id: 'project-1',
  name: 'Test Project',
  projectNumber: 'P001',
  // ... other properties
};
```

### Setup Configuration (`tests/setup.ts`)

Configures the test environment:
- Browser API mocks (matchMedia, IntersectionObserver, ResizeObserver)
- Global fetch mocking
- Window location mocking
- DOM testing library setup

## Automated Testing Pipeline

### GitHub Actions Workflow (`.github/workflows/test.yml`)

The automated pipeline runs on:
- Every push to `main` or `develop` branches
- Pull requests to `main`
- Daily scheduled runs at 2 AM UTC

**Pipeline Steps:**
1. **Environment Setup**: Node.js 20, PostgreSQL 15
2. **Dependencies**: Install npm packages
3. **Database**: Set up test database with migrations
4. **Type Checking**: Run TypeScript compiler
5. **Unit Tests**: Execute all test suites
6. **Coverage**: Generate test coverage report
7. **Build Test**: Verify production build works
8. **Artifact Upload**: Store test results and coverage

### Pipeline Configuration

```yaml
# Runs on multiple events
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
```

## Test Coverage Goals

### Target Coverage Metrics
- **Statements**: >90%
- **Branches**: >85%
- **Functions**: >90%
- **Lines**: >90%

### Critical Areas (100% Coverage Required)
- Authentication logic
- Permission checking
- Data validation functions
- API endpoint handlers
- Time calculation utilities

## Writing New Tests

### Component Test Template

```typescript
import React from 'react';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen, fireEvent } from '../utils/test-utils';

describe('YourComponent', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should render correctly', () => {
    render(<YourComponent />);
    expect(screen.getByTestId('your-component')).toBeInTheDocument();
  });

  it('should handle user interactions', async () => {
    render(<YourComponent />);
    
    const button = screen.getByTestId('action-button');
    fireEvent.click(button);
    
    await waitFor(() => {
      expect(screen.getByText('Expected Result')).toBeInTheDocument();
    });
  });
});
```

### API Test Template

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { createMockResponse } from '../utils/test-utils';

const mockFetch = vi.fn();
global.fetch = mockFetch;

describe('Your API', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should handle successful requests', async () => {
    const mockData = { success: true };
    mockFetch.mockResolvedValueOnce(createMockResponse(mockData));

    const result = await apiRequest('/api/endpoint', 'GET');
    
    expect(result).toEqual(mockData);
  });

  it('should handle errors gracefully', async () => {
    mockFetch.mockResolvedValueOnce(createMockResponse({ error: 'Bad request' }, 400));

    await expect(apiRequest('/api/endpoint', 'GET'))
      .rejects.toThrow('400:');
  });
});
```

## Best Practices

### 1. Test Organization
- Group related tests in `describe` blocks
- Use descriptive test names
- Follow the AAA pattern (Arrange, Act, Assert)

### 2. Mocking Strategy
- Mock external dependencies (APIs, databases)
- Use realistic mock data
- Reset mocks between tests

### 3. Assertions
- Use specific assertions (`toHaveTextContent` vs `toContain`)
- Test user-visible behavior, not implementation details
- Include both positive and negative test cases

### 4. Data Testing
- Test edge cases (empty data, maximum values)
- Validate input sanitization
- Test error states and recovery

### 5. Async Testing
- Use `waitFor` for async operations
- Test loading states
- Handle promise rejections

## Continuous Integration Benefits

### Automated Quality Assurance
- **Prevents Regressions**: Catches breaking changes before deployment
- **Code Quality**: Maintains consistent code standards
- **Documentation**: Tests serve as executable documentation
- **Confidence**: Enables safe refactoring and feature additions

### Team Collaboration
- **Pull Request Validation**: Ensures new code doesn't break existing functionality
- **Consistent Environment**: Same tests run locally and in CI
- **Fast Feedback**: Immediate notification of test failures

### Deployment Safety
- **Pre-deployment Validation**: All tests must pass before deployment
- **Rollback Confidence**: Easy to identify and fix issues
- **Feature Stability**: Comprehensive test coverage ensures reliability

## Troubleshooting Common Issues

### Test Failures
1. **Check mock setup**: Ensure mocks are properly configured
2. **Verify test data**: Check that mock data matches expected format
3. **Review async handling**: Ensure promises are properly awaited
4. **Check environment**: Verify test environment matches expectations

### Coverage Issues
1. **Add missing test cases**: Focus on untested branches and functions
2. **Remove dead code**: Eliminate unused code paths
3. **Test error paths**: Include negative test cases
4. **Integration testing**: Add higher-level integration tests

### CI Pipeline Issues
1. **Check dependencies**: Ensure all packages are properly installed
2. **Database setup**: Verify test database configuration
3. **Environment variables**: Check required environment variables
4. **Timeout issues**: Increase timeout limits if needed

## Monitoring and Reporting

### Test Results
- JUnit XML reports for CI integration
- Coverage reports in HTML and LCOV formats
- Artifact storage for 30 days
- Daily automated test runs

### Metrics Tracking
- Test execution time trends
- Coverage percentage changes
- Failure rate monitoring
- Performance regression detection

This comprehensive testing infrastructure ensures TimeTracker Pro maintains high quality and reliability through automated validation of every code change.