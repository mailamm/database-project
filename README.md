# Database Management Project - Homer Consulting

## 📌 Project Overview
This project is designed for Homer Consulting as part of the Database Management course. It implements a relational database system for managing employees, projects, clients, skills, and training records using Oracle SQL.

The database is designed to:
- Maintain employee records, including their departments and supervisors.
- Track client projects and their estimated costs.
- Record employee training and skill acquisition.
- Manage project assignments and work hours.

---

## 📂 Project Structure
```plaintext
DBM_Project/
│── queries/
│   ├── create_tables.txt    # SQL script for creating tables with constraints
│   ├── insert_data.txt      # SQL script to populate tables with sample data
│   ├── queries.md         # SQL queries for retrieving and analyzing data
│── data/
│   ├── sample_data.pdf      # Sample dataset for testing queries
│── docs/
│   ├── project_overview.md  # Detailed database documentation
│── README.md                # Project documentation
│── .gitignore               # Ignores unnecessary files
```

---

## 🛠 Database Schema
### **Entities & Relationships**
The database consists of multiple tables that define the structure of Homer Consulting's operations:

- **`SKILL`**: Stores different skills employees can acquire.
- **`TRAINING`**: Tracks training programs completed by employees.
- **`DEPARTMENT`**: Stores department information, including managers.
- **`EMPLOYEE`**: Records employee details and hierarchy.
- **`CLIENT`**: Stores client information.
- **`PROJECT`**: Maintains records of client projects.
- **`ASSIGNMENT`**: Tracks employees assigned to projects.

---

## 📊 SQL Queries
This project includes various SQL queries for data retrieval and analysis, stored in **`queries.txt`**.

### **1️⃣ Employee Training Summary by Skill**
```sql
SELECT NVL(TO_CHAR(e.emp_num), '---') AS "Employee ID", 
       NVL(e.fname || ' ' || e.lname, '# of Trainings:') AS "Employee Name", 
       SUM(CASE WHEN s.name = 'Data Analysis' THEN 1 ELSE 0 END) AS "Data Analysis", 
       SUM(CASE WHEN s.name = 'Project Management' THEN 1 ELSE 0 END) AS "Project Management", 
       SUM(CASE WHEN s.name = 'Software Engineering' THEN 1 ELSE 0 END) AS "Software Engineering", 
       SUM(CASE WHEN s.name = 'Financial Analysis' THEN 1 ELSE 0 END) AS "Financial Analysis", 
       SUM(CASE WHEN s.name = 'Marketing' THEN 1 ELSE 0 END) AS "Marketing", 
       SUM(CASE WHEN s.name = 'Cybersecurity' THEN 1 ELSE 0 END) AS "Cybersecurity", 
       COUNT(DISTINCT t.train_num) AS "# of Skills" 
FROM employee e 
     LEFT JOIN training t ON e.emp_num = t.emp_num 
     LEFT JOIN skill s ON t.code = s.code 
GROUP BY GROUPING SETS ((TO_CHAR(e.emp_num), e.fname || ' ' || e.lname), ())
ORDER BY CASE WHEN "Employee ID" = '---' THEN 1 ELSE 0 END, "Employee ID";
```

### **2️⃣ Employee Hierarchy**
```sql
SELECT
   LPAD(' ', 2*(LEVEL-1)) || e.FName || ' ' || e.LName AS Employee_Name,
   LEVEL AS Hierarchy_Level,
   d.Name AS Department_Name
FROM Employee e
JOIN Department d ON e.Dept_Code = d.Dept_Code
START WITH
   e.Super_ID IS NULL
CONNECT BY NOCYCLE PRIOR e.Emp_Num = e.Super_ID
ORDER SIBLINGS BY e.LName;
```

### **3️⃣ Employee Training-Based Bonus Ranking**
```sql
SELECT d.name AS "Department",
       s.category AS "Skill Category",
       NVL(COUNT(t.train_num), 0) AS "Training Count",
       DENSE_RANK() OVER (PARTITION BY d.name ORDER BY COUNT(t.train_num) DESC) AS "Rank"
FROM department d
CROSS JOIN skill s
LEFT JOIN employee e ON d.dept_code = e.dept_code
LEFT JOIN training t ON e.emp_num = t.emp_num AND t.code = s.code
GROUP BY d.name, s.category
ORDER BY d.name, "Rank";
);
```

📌 **For more queries, check `queries.sql`.**

---

## 🎯 Project Goals
- Build an efficient relational database with normalized tables.
- Enforce data integrity using primary & foreign keys, constraints, and triggers.
- Provide meaningful reports through complex SQL queries.
- Optimize query performance for large-scale datasets.

---

## 🚀 Future Enhancements
- Implement stored procedures for business logic automation.
- Develop triggers to update records automatically.
- Use PL/SQL scripts for batch operations.
- Extend the project with a web-based dashboard for data visualization.
