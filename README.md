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
    -- 2. Считаем процент времени каждого проекта и накопительную сумму часов (добавлен project_id в ORDER BY)
    SELECT 
        project_id,
        project_name,
        total_hours,
        SUM(total_hours) OVER() as total_all_hours,
        ROUND((total_hours / NULLIF(SUM(total_hours) OVER(), 0) * 100), 2) as duration_percent,
        SUM(total_hours) OVER(ORDER BY total_hours DESC, project_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative_hours
    FROM project_hours
),
abc_calculation AS (
    -- 3. Вычисляем итоговый накопительный процент (добавлен NULLIF для защиты от деления на 0)
    SELECT 
        project_id,
        project_name,
        total_hours,
        duration_percent,
        ROUND((cumulative_hours / NULLIF(total_all_hours, 0) * 100), 2) as cumulative_percent
    FROM project_percentages
)
-- 4. Присваиваем категории A, B, C
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
