# TimeTracker Pro - Database Schema Documentation

## Overview
This document describes the PostgreSQL database schema for the TimeTracker Pro application, designed for comprehensive employee time tracking with organizational hierarchy support.

## Database Tables

### Core Authentication Tables

#### `sessions`
Session storage for Replit Auth (required for authentication)
- `sid` (VARCHAR) - Primary key, session identifier  
- `sess` (JSONB) - Session data
- `expire` (TIMESTAMP) - Session expiration time
- Index: `IDX_session_expire` on expire column

#### `users`
User profiles synchronized with Replit Auth
- `id` (VARCHAR) - Primary key, UUID from Replit Auth
- `email` (VARCHAR) - User email (unique)
- `first_name` (VARCHAR) - First name from Replit profile
- `last_name` (VARCHAR) - Last name from Replit profile  
- `profile_image_url` (VARCHAR) - Profile picture URL
- `role` (VARCHAR) - User role: admin, manager, project_manager, employee, viewer
- `is_active` (BOOLEAN) - Account active status (default: true)
- `last_login_at` (TIMESTAMP) - Last login timestamp
- `created_at` (TIMESTAMP) - Account creation time
- `updated_at` (TIMESTAMP) - Last profile update

### Organizational Structure

#### `organizations`
Top-level organizational entities
- `id` (VARCHAR) - Primary key, UUID
- `name` (VARCHAR) - Organization name
- `description` (TEXT) - Organization description
- `logo_url` (VARCHAR) - Organization logo URL
- `website` (VARCHAR) - Organization website
- `user_id` (VARCHAR) - Foreign key to users (creator)
- `created_at` (TIMESTAMP) - Creation time
- `updated_at` (TIMESTAMP) - Last update time

#### `departments`
Departments within organizations
- `id` (VARCHAR) - Primary key, UUID
- `organization_id` (VARCHAR) - Foreign key to organizations
- `name` (VARCHAR) - Department name
- `description` (TEXT) - Department description
- `manager_id` (VARCHAR) - Foreign key to employees (department manager)
- `user_id` (VARCHAR) - Foreign key to users (creator)
- `created_at` (TIMESTAMP) - Creation time
- `updated_at` (TIMESTAMP) - Last update time

#### `employees`
Employee profiles linked to user accounts
- `id` (VARCHAR) - Primary key, UUID
- `employee_id` (VARCHAR) - Human-readable employee ID
- `first_name` (VARCHAR) - Employee first name
- `last_name` (VARCHAR) - Employee last name
- `email` (VARCHAR) - Employee email
- `phone` (VARCHAR) - Employee phone number
- `department_id` (VARCHAR) - Foreign key to departments
- `organization_id` (VARCHAR) - Foreign key to organizations
- `position` (VARCHAR) - Job position/title
- `hire_date` (DATE) - Employment start date
- `hourly_rate` (DECIMAL) - Hourly compensation rate
- `user_id` (VARCHAR) - Foreign key to users (linked account)
- `created_at` (TIMESTAMP) - Record creation time
- `updated_at` (TIMESTAMP) - Last update time

### Project Management

#### `projects`
Project definitions and configuration
- `id` (VARCHAR) - Primary key, UUID
- `name` (VARCHAR) - Project name
- `description` (TEXT) - Project description
- `color` (VARCHAR) - Hex color code for UI (default: #1976D2)
- `start_date` (TIMESTAMP) - Project start date
- `end_date` (TIMESTAMP) - Project end date
- `is_enterprise_wide` (BOOLEAN) - Access scope: true=all employees, false=restricted
- `user_id` (VARCHAR) - Foreign key to users (project owner)
- `created_at` (TIMESTAMP) - Creation time
- `updated_at` (TIMESTAMP) - Last update time

#### `tasks`
Tasks within projects
- `id` (VARCHAR) - Primary key, UUID
- `project_id` (VARCHAR) - Foreign key to projects
- `name` (VARCHAR) - Task name
- `description` (TEXT) - Task description
- `status` (VARCHAR) - Task status: active, completed, archived (default: active)
- `created_at` (TIMESTAMP) - Creation time
- `updated_at` (TIMESTAMP) - Last update time

#### `project_employees`
Project access control (for restricted projects)
- `id` (VARCHAR) - Primary key, UUID
- `project_id` (VARCHAR) - Foreign key to projects
- `employee_id` (VARCHAR) - Foreign key to employees
- `user_id` (VARCHAR) - Foreign key to users (who assigned)
- `created_at` (TIMESTAMP) - Assignment time

### Time Tracking

#### `time_entries`
Individual time log entries
- `id` (VARCHAR) - Primary key, UUID
- `user_id` (VARCHAR) - Foreign key to users
- `project_id` (VARCHAR) - Foreign key to projects
- `task_id` (VARCHAR) - Foreign key to tasks (nullable)
- `description` (TEXT) - Work description
- `date` (DATE) - Work date
- `start_time` (VARCHAR) - Start time in HH:MM format
- `end_time` (VARCHAR) - End time in HH:MM format
- `duration` (DECIMAL) - Duration in decimal hours
- `created_at` (TIMESTAMP) - Entry creation time
- `updated_at` (TIMESTAMP) - Last update time

## Key Relationships

### User Management Flow
1. User authenticates via Replit Auth → `users` table
2. Admin creates employee profile → `employees` table
3. Employee profile linked to user account → `employees.user_id`

### Organizational Hierarchy
- Organizations → Departments → Employees
- Department managers assigned via `departments.manager_id`

### Project Access Control
- Enterprise-wide projects: accessible to all employees
- Restricted projects: access controlled via `project_employees` table

### Time Tracking Flow
1. Employee selects project → validates access
2. Optional task selection → validates task belongs to project
3. Time entry creation → stores in `time_entries` with PST timezone

## Timezone Handling

All time-related operations use Pacific Standard Time (PST):
- Server configured with `TZ=America/Los_Angeles`
- Date calculations use PST timezone functions
- Time entries stored with PST context

## Security Considerations

- Session management via PostgreSQL store in production
- Role-based access control enforced at API level
- Project access validated for restricted projects
- User authentication required for all operations

## Database Constraints

- Cascading deletes maintain referential integrity
- Foreign key constraints enforce valid relationships
- Unique constraints prevent duplicate records
- Default values ensure consistent data state

This schema supports a full-featured enterprise time tracking system with organizational hierarchy, role-based permissions, and comprehensive audit trails.