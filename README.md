# db-time-tracking
### ER-схема базы данных
```mermaid
erDiagram
    EMPLOYEES ||--o{ TASKS : "выполняет"
    EMPLOYEES ||--o{ TIME_LOGS : "фиксирует время"
    EMPLOYEES ||--o{ REPORTS : "создает"
    PROJECTS ||--o{ TASKS : "содержит"
    PROJECTS ||--o{ REPORTS : "включает"
    TASKS ||--o{ TIME_LOGS : "отслеживается"

    EMPLOYEES {
        int employee_id PK
        string first_name
        string last_name
        string position
        string email
    }
    PROJECTS {
        int project_id PK
        string project_name
        string description
        date start_date
    }
    TASKS {
        int task_id PK
        int project_id FK
        int employee_id FK
        string task_name
        string status
    }
    TIME_LOGS {
        int log_id PK
        int employee_id FK
        int task_id FK
        timestamp start_time
        timestamp end_time
    }
    REPORTS {
        int report_id PK
        int project_id FK
        int created_by FK
        date report_date
        text content
    }
