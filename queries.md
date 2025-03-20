# **SQL Queries for Employee & Project Management**

---

## **1⃣ Employee Training Summary**
```sql
SELECT
    e.Emp_Num,
    e.FName || ' ' || e.LName AS Employee_Name,
    s.Code AS Skill_Code,
    s.Name AS Skill_Name,
    COUNT(t.Train_Num) AS Num_Trainings,
    MIN(t.Date_Acquired) AS Earliest_Training_Date,
    TRUNC(MONTHS_BETWEEN(TO_DATE('2024-11-11', 'YYYY-MM-DD'), MAX(t.Date_Acquired))) AS Months_Since_Most_Recent_Training
FROM Employee e
LEFT JOIN Training t ON e.Emp_Num = t.Emp_Num
LEFT JOIN Skill s ON t.Code = s.Code
GROUP BY
    e.Emp_Num,
    e.FName || ' ' || e.LName,
    s.Code,
    s.Name
ORDER BY
    e.Emp_Num,
    s.Code;
```

---

## **2⃣ Employee Hierarchy**
```sql
SELECT
    LPAD(' ', 2*(LEVEL-1)) || e.FName || ' ' || e.LName AS Employee_Name,
    LEVEL AS Hierarchy_Level,
    d.Name AS Department_Name
FROM Employee e
JOIN Department d ON e.Dept_Code = d.Dept_Code
START WITH e.Super_ID IS NULL
CONNECT BY NOCYCLE PRIOR e.Emp_Num = e.Super_ID
ORDER SIBLINGS BY e.LName;
```

---

## **3⃣ Employee Assignments by Project**
```sql
SELECT
    p.Proj_Number,
    p.Name AS Project_Name,
    p.Start_Date,
    NVL(TO_CHAR(a.Date_Assigned, 'YYYY-MM'), 'Total') AS Assignment_Month,
    COUNT(DISTINCT a.Emp_Num) AS Num_Employees_Assigned,
    SUM(a.Hours_Used) AS Total_Hours_Spent
FROM Project p
JOIN Assignment a ON p.Proj_Number = a.Proj_Number
WHERE p.Total_Cost IS NULL
GROUP BY p.Proj_Number, p.Name, p.Start_Date, ROLLUP(TO_CHAR(a.Date_Assigned, 'YYYY-MM'))
HAVING p.Proj_Number = 107 -- Replace with specific ongoing project IDs if needed
ORDER BY TO_CHAR(a.Date_Assigned, 'YYYY-MM') NULLS LAST;
```

---

## **4⃣ Employee Bonus Calculation**
```sql
-- Add the new column
ALTER TABLE Employee ADD (BONUS_AMT NUMBER(10,2));

-- Update the BONUS_AMT for each employee
UPDATE Employee e
SET BONUS_AMT = NVL((
    SELECT COUNT(DISTINCT a.Proj_Number) * 200
    FROM Assignment a
    JOIN Project p ON a.Proj_Number = p.Proj_Number
    WHERE a.Emp_Num = e.Emp_Num
    AND p.Start_Date BETWEEN TO_DATE('2024-07-01', 'YYYY-MM-DD')
                         AND TO_DATE('2024-09-30', 'YYYY-MM-DD')
    AND a.Hours_Used >= 100
), 0);
```

---

## **5⃣ Training and Project Participation**
```sql
SELECT
    e.Emp_Num,
    e.FName || ' ' || e.LName AS Employee_Name,
    e.Hire_Date,
    t.Name AS Training_Name,
    t.Date_Acquired AS Training_Date,
    t.Date_Acquired - e.Hire_Date AS Days_Between_Hire_And_Training,
    (SELECT COUNT(DISTINCT a.Proj_Number)
     FROM Assignment a
     WHERE a.Emp_Num = e.Emp_Num) AS Num_Projects_Worked_On
FROM Employee e
LEFT JOIN Training t ON e.Emp_Num = t.Emp_Num
WHERE e.Hire_Date BETWEEN TO_DATE('2024-04-01', 'YYYY-MM-DD')
                     AND TO_DATE('2024-06-30', 'YYYY-MM-DD');
```

---

## **6⃣ Project Status Classification**
```sql
SELECT DISTINCT
    p.proj_number AS "Project Number",
    p.name AS "Project Name",
    MIN(a1.date_assigned) AS "Start Date",
    CASE
        WHEN p.total_cost IS NULL THEN 'On-going'
        ELSE 'Completed'
    END AS "Project Status"
FROM project p
JOIN assignment a1 ON p.proj_number = a1.proj_number
LEFT JOIN assignment a2
    ON a1.proj_number = a2.proj_number
    AND a1.date_ended < a2.date_assigned
    AND a2.date_assigned = (
        SELECT MIN(a3.date_assigned)
        FROM assignment a3
        WHERE a3.proj_number = a1.proj_number
        AND a3.date_assigned > a1.date_ended
    )
WHERE a2.date_assigned - a1.date_ended > 30
GROUP BY p.proj_number, p.name, p.total_cost
ORDER BY p.proj_number;
```

---

## **7⃣ Quarterly Project Analysis**
```sql
SELECT
    q.Quarter,
    COUNT(DISTINCT p.Proj_Number) AS Num_Projects_Started,
    COUNT(DISTINCT a.Emp_Num) AS Num_Employees_Working,
    ROUND(NVL(SUM(a.Hours_Used), 0) / NULLIF(COUNT(DISTINCT p.Proj_Number), 0), 2) AS Avg_Hours_Per_Project
FROM (
    SELECT 'Q1' AS Quarter, TO_DATE('01-JAN-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY') AS Start_Date,
           TO_DATE('31-MAR-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY') AS End_Date FROM DUAL
    UNION ALL
    SELECT 'Q2', TO_DATE('01-APR-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY'),
                 TO_DATE('30-JUN-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY') FROM DUAL
    UNION ALL
    SELECT 'Q3', TO_DATE('01-JUL-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY'),
                 TO_DATE('30-SEP-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY') FROM DUAL
    UNION ALL
    SELECT 'Q4', TO_DATE('01-OCT-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY'),
                 TO_DATE('31-DEC-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MON-YYYY') FROM DUAL
) q
LEFT JOIN Project p ON p.Start_Date BETWEEN q.Start_Date AND q.End_Date
LEFT JOIN Assignment a ON a.Proj_Number = p.Proj_Number AND a.Date_Assigned BETWEEN q.Start_Date AND q.End_Date
GROUP BY q.Quarter
ORDER BY q.Quarter;
```

