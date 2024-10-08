# Attendance

## Attendance Clears
These queries can be used to check attendance on the selected days and delete the attendance records if necessary.

### Daily Attendance
```sql
SELECT
    ad.id,
    ad.student_id,
    sc.title AS school,
    sg.title AS grade,
    ad.school_date,
    ad.state_value,
    ad.daily_code,
    ad.last_updated_date,
    ad.last_updated_user
FROM attendance_day          ad
     JOIN student_enrollment se ON ad.student_id = se.student_id
     JOIN schools            sc ON se.school_id = sc.id
     JOIN school_gradelevels sg ON se.grade_id = sg.id
WHERE school_date IN ('2024-10-11')
  AND (ad.school_date BETWEEN se.start_date AND se.end_date OR se.end_date IS NULL)
  AND COALESCE(se.custom_9, 'N') = 'N'
  AND sg.title NOT IN ('30', '31')
ORDER BY sc.title, ad.school_date;

DELETE
FROM attendance_day
WHERE id IN (
    SELECT
        ad.id
    FROM attendance_day          ad
         JOIN student_enrollment se ON ad.student_id = se.student_id
         JOIN schools            sc ON se.school_id = sc.id
         JOIN school_gradelevels sg ON se.grade_id = sg.id
    WHERE school_date IN ('2024-10-09', '2024-10-10')
      AND (ad.school_date BETWEEN se.start_date AND se.end_date OR se.end_date IS NULL)
      AND sg.title NOT IN ('30', '31')
      AND COALESCE(se.custom_9, 'N') = 'N')
RETURNING *;
```

### Period Attendance
```sql
SELECT
    ap.id,
    sc.title      AS school,
    s.student_id,
    ap.school_date,
    ac.title      AS attendance_code,
    ac.short_name AS code,
    sp.title      AS period,
    c.title       AS course,
    c.short_name  AS course_code,
    cp.title      AS section
FROM students               s
     JOIN attendance_period ap ON s.student_id = ap.student_id
     JOIN school_periods    sp ON ap.period_id = sp.period_id
     JOIN course_periods    cp ON ap.course_period_id = cp.course_period_id
     JOIN attendance_codes  ac ON ap.attendance_code = ac.id
     JOIN courses           c ON cp.course_id = c.course_id
     JOIN schools           sc ON cp.school_id = sc.id
WHERE 1 = 1
  AND ac.short_name NOT IN ('E') --Adult Ed
  AND ap.school_date IN ('2024-10-11')
ORDER BY sc.title, ap.school_date;

DELETE
FROM attendance_period
WHERE id IN (
    SELECT
        ap.id
    FROM students               s
         JOIN attendance_period ap ON s.student_id = ap.student_id
         JOIN school_periods    sp ON ap.period_id = sp.period_id
         JOIN course_periods    cp ON ap.course_period_id = cp.course_period_id
         JOIN attendance_codes  ac ON ap.attendance_code = ac.id
         JOIN courses           c ON cp.course_id = c.course_id
    WHERE 1 = 1
      AND ac.short_name NOT IN ('E') --Adult Ed
      AND ap.school_date IN ('2024-10-09', '2024-10-10'))
RETURNING *;
```
