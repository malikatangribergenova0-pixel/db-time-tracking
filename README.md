# db-time-tracking

### 4. ER-схема базы данных
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
WITH project_hours AS (
    -- 1. Считаем сумму часов для каждого проекта
    SELECT 
        p.project_id,
        p.project_name,
        ROUND(SUM(EXTRACT(EPOCH FROM (tl.end_time - tl.start_time)) / 3600)::numeric, 2) as total_hours
    FROM projects p
    JOIN tasks t ON p.project_id = t.project_id
    JOIN time_logs tl ON t.task_id = tl.task_id
    WHERE tl.end_time IS NOT NULL
    GROUP BY p.project_id, p.project_name
),
project_percentages AS (
    -- 2. Считаем процент времени каждого проекта и накопительную сумму часов
    SELECT 
        project_id,
        project_name,
        total_hours,
        SUM(total_hours) OVER() as total_all_hours,
        ROUND((total_hours / SUM(total_hours) OVER() * 100), 2) as duration_percent,
        SUM(total_hours) OVER(ORDER BY total_hours DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative_hours
    FROM project_hours
),
abc_calculation AS (
    -- 3. Вычисляем итоговый накопительный процент для деления на классы
    SELECT 
        project_id,
        project_name,
        total_hours,
        duration_percent,
        ROUND((cumulative_hours / total_all_hours * 100), 2) as cumulative_percent
    FROM project_percentages
)
-- 4. Присваиваем категории A, B, C на основе накопительного процента
SELECT 
    project_id,
    project_name,
    total_hours,
    duration_percent,
    cumulative_percent,
    CASE 
        WHEN cumulative_percent <= 80 THEN 'A (Высокая приоритетность)'
        WHEN cumulative_percent <= 95 THEN 'B (Средняя приоритетность)'
        ELSE 'C (Низкая приоритетность)'
    END as abc_class
FROM abc_calculation
ORDER BY total_hours DESC;
