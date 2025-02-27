Below is a full script for the **DDL (Database Schema Creation)** and **DML (Data Insertion)** in one SQL script. The skeleton code for the **ASP.NET C# Web API** to implement the workflow application logic is also provided.


### **SQL Script (DDL + DML)**

```sql
-- -------------------------------
-- DDL: Schema Creation
-- -------------------------------

-- Create departments table
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create users table
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,  -- Employee, Manager, Department Head, Organization Head, Admin
    department_id INT REFERENCES departments(department_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create roles table
CREATE TABLE roles (
    role_id SERIAL PRIMARY KEY,
    role_name VARCHAR(50) UNIQUE NOT NULL,  -- e.g., Employee, Manager, Admin
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create forms table
CREATE TABLE forms (
    form_id SERIAL PRIMARY KEY,
    form_name VARCHAR(255) NOT NULL,
    form_type VARCHAR(100),  -- e.g., Leave Application, Expense Report
    created_by INT REFERENCES users(user_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create form fields table
CREATE TABLE form_fields (
    field_id SERIAL PRIMARY KEY,
    form_id INT REFERENCES forms(form_id),
    field_name VARCHAR(255) NOT NULL,
    field_type VARCHAR(50) NOT NULL,  -- e.g., text, date, dropdown
    field_options TEXT[],  -- Options for dropdown or radio button fields
    is_required BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create workflow states table
CREATE TABLE workflow_states (
    state_id SERIAL PRIMARY KEY,
    form_id INT REFERENCES forms(form_id),
    state_name VARCHAR(50) NOT NULL,  -- e.g., Draft, Submitted, Approved
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create workflow transitions table
CREATE TABLE workflow_transitions (
    transition_id SERIAL PRIMARY KEY,
    from_state_id INT REFERENCES workflow_states(state_id),
    to_state_id INT REFERENCES workflow_states(state_id),
    allowed_role_id INT REFERENCES roles(role_id),  -- Role that can perform the transition
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create permissions table for form actions
CREATE TABLE permissions (
    permission_id SERIAL PRIMARY KEY,
    form_id INT REFERENCES forms(form_id),
    role_id INT REFERENCES roles(role_id),
    action VARCHAR(50) NOT NULL,  -- e.g., view, create, edit, delete
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create form submissions table
CREATE TABLE form_submissions (
    submission_id SERIAL PRIMARY KEY,
    form_id INT REFERENCES forms(form_id),
    user_id INT REFERENCES users(user_id),
    current_state_id INT REFERENCES workflow_states(state_id),
    submission_data JSONB,  -- The form data, stored as JSON
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create user roles table
CREATE TABLE user_roles (
    user_id INT REFERENCES users(user_id),
    role_id INT REFERENCES roles(role_id),
    PRIMARY KEY (user_id, role_id)
);

-- -------------------------------
-- DML: Sample Data Insertion
-- -------------------------------

-- Insert roles
INSERT INTO roles (role_name) VALUES
('Employee'),
('Manager'),
('Department Head'),
('Organization Head'),
('Admin');

-- Insert departments
INSERT INTO departments (department_name) VALUES
('Human Resources'),
('Engineering'),
('Finance');

-- Insert users
INSERT INTO users (name, email, password_hash, role, department_id) VALUES
('John Doe', 'john.doe@example.com', 'hashed_password', 'Employee', 1),
('Jane Smith', 'jane.smith@example.com', 'hashed_password', 'Manager', 1),
('Alice Johnson', 'alice.johnson@example.com', 'hashed_password', 'Department Head', 1),
('Bob Brown', 'bob.brown@example.com', 'hashed_password', 'Organization Head', NULL),
('Admin User', 'admin@example.com', 'hashed_password', 'Admin', NULL);

-- Insert forms
INSERT INTO forms (form_name, form_type, created_by) VALUES
('Leave Application', 'Leave', 1),
('Expense Report', 'Expense', 1);

-- Insert form fields for Leave Application
INSERT INTO form_fields (form_id, field_name, field_type, is_required) VALUES
(1, 'Leave Start Date', 'date', TRUE),
(1, 'Leave End Date', 'date', TRUE),
(1, 'Leave Reason', 'text', TRUE);

-- Insert workflow states for Leave Application
INSERT INTO workflow_states (form_id, state_name) VALUES
(1, 'Draft'),
(1, 'Submitted'),
(1, 'Approved'),
(1, 'Rejected');

-- Insert workflow transitions for Leave Application
INSERT INTO workflow_transitions (from_state_id, to_state_id, allowed_role_id) VALUES
(1, 2, 1),  -- Employee can submit from Draft to Submitted
(2, 3, 2),  -- Manager can approve from Submitted to Approved
(2, 4, 2);  -- Manager can reject from Submitted to Rejected

-- Insert permissions for Leave Application form
INSERT INTO permissions (form_id, role_id, action) VALUES
(1, 1, 'create'),
(1, 2, 'approve'),
(1, 2, 'reject'),
(1, 3, 'view');

-- Insert form submissions (Example)
INSERT INTO form_submissions (form_id, user_id, current_state_id, submission_data) VALUES
(1, 1, 1, '{"leave_start_date": "2025-02-01", "leave_end_date": "2025-02-05", "leave_reason": "Vacation"}');

```

---

### **Skeleton Code for ASP.NET C# Web API**

Following is the skeleton of an ASP.NET C# Web API project that implements the logic for managing forms, users, workflows, and roles:

#### **1. Project Setup**
- Create a new **ASP.NET Core Web API** project.
- Add necessary NuGet packages for **Entity Framework Core** and **JWT Authentication**.

```bash
dotnet new webapi -n WorkflowApp
cd WorkflowApp
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

#### **2. Models (Entities)**

**User.cs**

```csharp
public class User
{
    public int UserId { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string PasswordHash { get; set; }
    public string Role { get; set; }
    public int? DepartmentId { get; set; }
    public Department Department { get; set; }
}
```

**Department.cs**

```csharp
public class Department
{
    public int DepartmentId { get; set; }
    public string DepartmentName { get; set; }
    public ICollection<User> Users { get; set; }
}
```

**Form.cs**

```csharp
public class Form
{
    public int FormId { get; set; }
    public string FormName { get; set; }
    public string FormType { get; set; }
    public int CreatedBy { get; set; }
    public User CreatedByUser { get; set; }
    public ICollection<FormField> FormFields { get; set; }
    public ICollection<WorkflowState> WorkflowStates { get; set; }
    public ICollection<Permission> Permissions { get; set; }
}
```

**FormField.cs**

```csharp
public class FormField
{
    public int FieldId { get; set; }
    public int FormId { get; set; }
    public string FieldName { get; set; }
    public string FieldType { get; set; }
    public bool IsRequired { get; set; }
    public Form Form { get; set; }
}
```

#### **3. DbContext**

**ApplicationDbContext.cs**

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }

    public DbSet<User> Users { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Form> Forms { get; set; }
    public DbSet<FormField> FormFields { get; set; }
    public DbSet<WorkflowState> WorkflowStates { get; set; }
    public DbSet<Permission> Permissions { get; set; }
    public DbSet<WorkflowTransition> WorkflowTransitions { get; set; }
}
```

#### **4. Controllers**

**FormsController.cs**

```csharp
[Route("api/[controller]")]
[ApiController]
public class FormsController : ControllerBase
{
    private readonly ApplicationDbContext _context;

    public FormsController(ApplicationDbContext context)
    {
        _context = context;
    }

    // Get all forms
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Form>>> GetForms()
    {
        return await _context.Forms.Include(f => f.FormFields).ToListAsync();
    }

    // Create a form (Admin only)
    [HttpPost]
    public async Task<ActionResult<Form>> CreateForm(Form form)
    {
        _context.Forms.Add(form);
        await _context.SaveChangesAsync();
        return CreatedAtAction("GetForms", new { id = form.FormId }, form);
    }

    // Workflow transition logic (Admin/Manager)
    [HttpPost("transition/{submissionId}")]
    public async Task<IActionResult> TransitionForm(int submissionId, [FromBody] WorkflowTransitionRequest request)
    {
        // Check user permissions and transition rules
        // Update form state based on the transition
        return Ok();
    }
}
```

#### **5. Workflow Transition Logic**

You can implement the workflow logic (state transitions, permissions check, etc.) in a service class or directly within the controller action. For simplicity, I’ve left placeholders in the code.

---

This is a high-level design for your application with the database schema, sample data, and the skeleton code for the ASP.NET C# Web API. The Web API includes essential routes for managing forms and handling workflow transitions based on user roles. 
