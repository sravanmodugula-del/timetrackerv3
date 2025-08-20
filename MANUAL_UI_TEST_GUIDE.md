# Manual UI Testing Guide - Employee Assignment Functionality

**Current Status**: You are logged in as **ADMIN** role  
**Browser URL**: Check your current page and navigate as needed

---

## 🎯 TEST 1: Admin Employee Assignment (CREATE PROJECT)

### Step-by-Step Instructions:

1. **Navigate to Projects Page**
   - Click on "Projects" in the sidebar navigation
   - **VERIFY**: You should see a "New Project" button (blue button with + icon)

2. **Open Project Creation Dialog**
   - Click the "New Project" button  
   - **VERIFY**: Dialog opens with "Create New Project" title

3. **Fill Project Details**
   - Enter Project Name: "Manual Test Admin Project"
   - Enter Description: "Testing employee assignment UI"
   - **VERIFY**: Form fields accept input

4. **Access Employee Assignment Tab**
   - Click the "Assigned Employees" tab (should be the second tab)
   - **VERIFY**: Tab is clickable and switches content

5. **Test Employee Selection**
   - **VERIFY**: You see a list of employees with checkboxes
   - **VERIFY**: Each employee shows: Name, Department, Employee ID
   - **VERIFY**: You can check/uncheck employee boxes
   - **VERIFY**: Selection counter updates (e.g., "2 employee(s) selected")

6. **Submit Project**
   - Select 2-3 employees by checking their boxes
   - Click "Create Project" button
   - **VERIFY**: Dialog closes and project appears in project list
   - **VERIFY**: No error messages appear

### Expected Results:
- ✅ Admin sees New Project button
- ✅ Employee assignment tab is accessible  
- ✅ Employee checkboxes are functional
- ✅ Project creation succeeds with employee assignments

---

## 🎯 TEST 2: Admin Employee Assignment (EDIT PROJECT)

### Step-by-Step Instructions:

1. **Find Existing Project**
   - Look for any project card in the projects list
   - **VERIFY**: You see "Edit" buttons on project cards

2. **Open Project Edit Dialog**
   - Click "Edit" button on any project
   - **VERIFY**: Dialog opens with "Edit Project" title
   - **VERIFY**: Project details are pre-filled

3. **Check Current Assignments**
   - Click "Assigned Employees" tab
   - **VERIFY**: Some employees may already be selected
   - **VERIFY**: Selection counter shows current count

4. **Modify Assignments**
   - Check/uncheck some employee boxes
   - **VERIFY**: Counter updates as you change selections

5. **Save Changes**
   - Click "Save Changes" or "Update Project" button
   - **VERIFY**: Dialog closes successfully
   - **VERIFY**: No error messages

### Expected Results:
- ✅ Admin can edit ANY project (even from other users)
- ✅ Current employee assignments are visible
- ✅ Can modify employee assignments
- ✅ Changes save successfully

---

## 🎯 TEST 3: Project Manager Role Testing

### Switch to Project Manager Role:

1. **Use Role Testing Page**
   - Navigate to `/role-testing` in browser OR
   - Use API: `curl -X POST http://localhost:5000/api/users/change-role -H "Content-Type: application/json" -d '{"role":"project_manager"}'`

2. **Verify Role Switch**
   - Check top-right corner shows "Project Manager" role
   - Page should refresh/update

### Test Project Manager Capabilities:

1. **Navigate to Projects Page**
   - **VERIFY**: "New Project" button is visible

2. **Create New Project**
   - Click "New Project" 
   - Fill in: "Manual Test PM Project"
   - Click "Assigned Employees" tab
   - **VERIFY**: Employee checkboxes are visible
   - Select employees and create project
   - **VERIFY**: Project creation succeeds

3. **Edit Project**
   - **VERIFY**: Can edit projects (but may be limited to own projects)

### Expected Results:
- ✅ Project Manager sees New Project button
- ✅ Can create projects with employee assignments
- ✅ Employee assignment interface works identically to admin

---

## 🎯 TEST 4: Manager Role Restrictions

### Switch to Manager Role:
```bash
curl -X POST http://localhost:5000/api/users/change-role -H "Content-Type: application/json" -d '{"role":"manager"}'
```

### Test Manager Limitations:

1. **Navigate to Projects Page**
   - **VERIFY**: NO "New Project" button visible
   - **VERIFY**: Can see project list (view-only)

2. **Check Project Cards**
   - **VERIFY**: NO "Edit" buttons on project cards
   - **VERIFY**: NO "Delete" buttons on project cards

### Expected Results:
- ❌ Manager does NOT see New Project button
- ❌ Manager does NOT see Edit/Delete buttons
- ✅ Manager can view projects (oversight only)

---

## 🎯 TEST 5: Employee Role Restrictions

### Switch to Employee Role:
```bash
curl -X POST http://localhost:5000/api/users/change-role -H "Content-Type: application/json" -d '{"role":"employee"}'
```

### Test Employee Limitations:

1. **Navigate to Projects Page**
   - **VERIFY**: NO "New Project" button visible
   - **VERIFY**: NO "Edit" buttons visible
   - **VERIFY**: May see fewer projects (only assigned ones)

### Expected Results:
- ❌ Employee does NOT see New Project button
- ❌ Employee does NOT see Edit/Delete buttons
- ⚠️ Employee sees limited project list (assigned projects only)

---

## 🚨 Common Issues to Look For:

### UI Problems:
- Checkboxes don't respond to clicks
- Employee list is empty when it should have employees
- Selection counter doesn't update
- Form submission fails or shows errors
- Dialog doesn't close after successful submission

### Permission Problems:
- Wrong buttons visible for role restrictions
- Roles can access features they shouldn't have
- Error messages for unauthorized actions

### Data Problems:
- Selected employees not saved to database
- Employee assignments not visible after creation
- Changes not persisting after page refresh

---

## 📊 Report Your Results:

Please test each section and report:

1. **Which tests PASSED** (worked as expected)
2. **Which tests FAILED** (specific errors or issues)
3. **Any unexpected behavior** you observed

**Format your results like:**
- ✅ TEST 1 PASSED - Admin can create projects with employee assignments
- ❌ TEST 2 FAILED - Employee checkboxes don't respond to clicks
- ⚠️ TEST 3 PARTIAL - Project Manager can create but edit dialog crashes

This will help me identify and fix any actual UI issues before deployment.