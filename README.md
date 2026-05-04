# taskflow — Team Task Manager

A full-stack web application for managing team projects, assigning tasks, and tracking progress with role-based access control.

---

## Features

### Authentication
- Email/password login system
- Session-based user context
- Quick-login demo accounts for evaluation

### Role-Based Access Control (RBAC)
| Feature | Admin | Member |
|---|---|---|
| View all projects | ✅ | ❌ (own projects only) |
| Create projects | ✅ | ❌ |
| Manage project members | ✅ | ❌ |
| Create tasks | ✅ | ✅ |
| Edit any task | ✅ | ❌ (own tasks only) |
| Delete tasks | ✅ | ❌ (created by self only) |
| View team management | ✅ | ❌ |
| Promote/demote members | ✅ | ❌ |

### Projects
- Create projects with name, description, color label, and team members
- Per-project Kanban board (To Do → In Progress → In Review → Done)
- Progress bar showing completion percentage
- Member management (add/remove team members)

### Tasks
- Create tasks with title, description, project, assignee, priority (Low/Medium/High), and due date
- Edit status, priority, assignee, and due date
- Overdue detection with visual warnings
- Delete tasks (Admin or task creator)

### Dashboard
- Summary stats: total tasks, in-progress, completed, overdue
- Per-project progress cards (clickable)
- Recent tasks feed
- Overdue tasks panel (Admin only)

### My Tasks
- Personal task view filtered to the logged-in user
- Filter by status: All, To Do, In Progress, In Review, Done
- Search tasks by title

---


## Database Schema

```sql
-- Users
CREATE TABLE users (
  id         SERIAL PRIMARY KEY,
  name       VARCHAR(100) NOT NULL,
  email      VARCHAR(150) UNIQUE NOT NULL,
  password   TEXT NOT NULL,
  role       VARCHAR(20) DEFAULT 'member',  -- 'admin' | 'member'
  created_at TIMESTAMP DEFAULT NOW()
);

-- Projects
CREATE TABLE projects (
  id          SERIAL PRIMARY KEY,
  name        VARCHAR(150) NOT NULL,
  description TEXT,
  color       VARCHAR(10),
  owner_id    INT REFERENCES users(id),
  created_at  TIMESTAMP DEFAULT NOW()
);

-- Project Members (join table)
CREATE TABLE project_members (
  project_id INT REFERENCES projects(id) ON DELETE CASCADE,
  user_id    INT REFERENCES users(id) ON DELETE CASCADE,
  PRIMARY KEY (project_id, user_id)
);

-- Tasks
CREATE TABLE tasks (
  id          SERIAL PRIMARY KEY,
  title       VARCHAR(200) NOT NULL,
  description TEXT,
  project_id  INT REFERENCES projects(id) ON DELETE CASCADE,
  assignee_id INT REFERENCES users(id),
  created_by  INT REFERENCES users(id),
  priority    VARCHAR(10) DEFAULT 'medium',  -- 'low' | 'medium' | 'high'
  status      VARCHAR(20) DEFAULT 'todo',    -- 'todo' | 'in_progress' | 'review' | 'done'
  due_date    DATE,
  created_at  TIMESTAMP DEFAULT NOW()
);
```

---

## Demo Accounts

| Name | Email | Password | Role |
|---|---|---|---|
| Alex Rivera | admin@demo.com | demo123 | Admin |
| Sam Chen | member@demo.com | demo123 | Member |
| Jordan Lee | jordan@demo.com | demo123 | Member |
| Maya Patel | maya@demo.com | demo123 | Member |

---



## Validations

- All required fields (title, email, password) are validated server-side
- Email must be unique; passwords are hashed with bcrypt (salt rounds: 10)
- Due dates are validated to be proper date strings
- Role changes are restricted to Admin users only
- Task edits by Members are restricted to their own assigned tasks
- Project membership is enforced — Members cannot view projects they're not part of

---

