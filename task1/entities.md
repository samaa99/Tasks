## 📊 Entities (Data Model)

---

### 1. Employee

Represents all users of the system (both regular employees and managers).

**Attributes:**
- `id` (Primary Key) - Unique identifier for the employee
- `name` - Full name of the employee
- `email` - Email address for notifications
- `title` - Job title (e.g., "Software Engineer", "HR Manager")
- `isManager` - Boolean flag indicating if this employee has manager privileges
- `managerId` (Foreign Key) - References Employee (who is their manager)
- `createdDate` - When the employee record was created
- `isActive` - Boolean indicating if employee is currently active

**Example:**
```
id: 101
name: "John Smith"
email: "john.smith@company.com"
title: "Software Engineer"
isManager: false
managerId: 205
createdDate: "2023-01-15"
isActive: true
```

---

### 2. VacationCategory

Represents the different types of vacation/leave available in the company (e.g., PTO, Sick Leave, Personal Time).

**Attributes:**
- `id` (Primary Key) - Unique identifier for the category
- `name` - Name of the category (e.g., "PTO", "Sick Leave", "Personal Time")
- `description` - Detailed description of this category
- `requiresManagerApproval` - Boolean indicating if requests need manager approval
- `isActive` - Boolean indicating if this category is currently available for new requests (used for soft delete - keeps historical data while preventing new usage)

**Example:**
```
id: 1
name: "PTO"
description: "Paid Time Off for vacation and personal matters"
requiresManagerApproval: true
isActive: true
```
```
id: 2
name: "Sick Leave"
description: "Time off due to illness or medical appointments"
requiresManagerApproval: false
isActive: true
```

---

### 3. EmployeeVacationBalance

Represents how many days/hours an employee has available for each vacation category. This is the connecting entity between Employee and VacationCategory that tracks individual balances.

**Attributes:**
- `id` (Primary Key) - Unique identifier for this balance record
- `employeeId` (Foreign Key) - References Employee
- `categoryId` (Foreign Key) - References VacationCategory
- `balance` - Current available balance (in hours or days)
- `year` - The year this balance applies to (balances often reset yearly)
- `lastUpdated` - When this balance was last modified

**Example:**
```
id: 1001
employeeId: 101 (John Smith)
categoryId: 1 (PTO)
balance: 80 hours (10 days)
year: 2025
lastUpdated: "2025-11-01"
```
```
id: 1002
employeeId: 101 (John Smith)
categoryId: 2 (Sick Leave)
balance: 40 hours (5 days)
year: 2025
lastUpdated: "2025-11-01"
```

**Note:** When a request is approved, the balance is reduced. When a request is rejected or canceled, the balance may be restored.

---

### 4. Request

Represents a vacation/leave request submitted by an employee.

**Attributes:**
- `id` (Primary Key) - Unique identifier for the request
- `submitterId` (Foreign Key) - References Employee who submitted the request
- `approverId` (Foreign Key) - References Employee (manager) who approved/denied (nullable until processed)
- `categoryId` (Foreign Key) - References VacationCategory
- `title` - Short title for the request
- `description` - Detailed description (helps manager understand context)
- `startDate` - First day of the vacation
- `endDate` - Last day of the vacation
- `hoursPerDay` - Hours requested per day (e.g., 4 hours = half day, 8 hours = full day)
- `totalHours` - Total hours for entire request (calculated)
- `status` - Current state: "Pending", "Approved", "Rejected", "Cancelled"
- `submittedDate` - When the request was submitted
- `processedDate` - When the request was approved/rejected (nullable)
- `rejectionReason` - Explanation if rejected (nullable)

**Example:**
```
id: 5001
submitterId: 101 (John Smith)
approverId: 205 (Sarah Johnson - his manager)
categoryId: 1 (PTO)
title: "Family Vacation"
description: "Beach trip with family"
startDate: "2025-12-20"
endDate: "2025-12-27"
hoursPerDay: 8
totalHours: 64 (8 days × 8 hours)
status: "Approved"
submittedDate: "2025-11-03 09:30:00"
processedDate: "2025-11-03 14:15:00"
rejectionReason: null
```

**Business Rules:**
- System validates that submitter has sufficient balance in the category before allowing submission
- If requiresManagerApproval is true for the category, status starts as "Pending"
- Email notification sent to manager when status is "Pending"
- Email notification sent to submitter when status changes to "Approved" or "Rejected"

---

### Relationships:

- **Employee manages other Employees** (manager-subordinate relationship via managerId)
- **Employee has many EmployeeVacationBalances** (one employee can have balances in multiple categories)
- **Employee submits many Requests** (one employee can create multiple vacation requests)
- **Employee (Manager) approves many Requests** (one manager can approve requests from multiple subordinates)
- **VacationCategory has many EmployeeVacationBalances** (one category like "PTO" has balances for all employees)
- **VacationCategory has many Requests** (one category can be used in multiple requests)
- **Request belongs to one Employee (submitter)** (each request is submitted by exactly one employee)
- **Request belongs to one Employee (approver)** (each request is approved by exactly one manager)
- **Request belongs to one VacationCategory** (each request is for one specific type of leave)
- **EmployeeVacationBalance belongs to one Employee** (each balance record is for one employee)
- **EmployeeVacationBalance belongs to one VacationCategory** (each balance record is for one category)

---