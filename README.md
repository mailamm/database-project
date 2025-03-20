# 📊 Database Management Project - Homer Consulting

## 📌 Project Overview
This project is designed for **Homer Consulting** as part of the **Database Management ** course. It implements a relational database system for managing **employees, projects, clients, skills, and training records** using **Oracle SQL**.

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
│   ├── create_tables.sql    # SQL script for creating tables with constraints
│   ├── insert_data.sql      # SQL script to populate tables with sample data
│   ├── queries.sql          # SQL queries for retrieving and analyzing data
│── data/
│   ├── sample_data.sql      # Sample dataset for testing queries
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

📌 **A detailed ER diagram can be found in the `docs/project_overview.md` file.**

---

## 💾 Setup Instructions
### **Step 1: Setup Oracle Database**
1. Install **Oracle SQL Developer** or connect to an Oracle Database instance.
2. Ensure you have the necessary permissions to create tables and insert data.

### **Step 2: Create the Database**
1. Run the `create_tables.sql` script:
   ```sql
   @queries/create_tables.sql
   ```
2. Verify that all tables have been successfully created.

### **Step 3: Insert Sample Data**
1. Run the `insert_data.sql` script:
   ```sql
   @queries/insert_data.sql
   ```
2. Confirm the records exist:
   ```sql
   SELECT * FROM EMPLOYEE;
   SELECT * FROM PROJECT;
   ```

---

## 📊 SQL Queries
This project includes various SQL queries for data retrieval and analysis, stored in **`queries.sql`**.

### **1️⃣ Employee Training Records**
```sql
SELECT e.Emp_Num, e.LName, e.FName, t.Code, COUNT(t.Train_Num) AS Training_Count,
       MIN(t.Date_Acquired) AS First_Training, 
       MONTHS_BETWEEN(SYSDATE, MAX(t.Date_Acquired)) AS Months_Since_Last_Training
FROM EMPLOYEE e
LEFT JOIN TRAINING t ON e.Emp_Num = t.Emp_Num
GROUP BY e.Emp_Num, e.LName, e.FName, t.Code;
```

### **2️⃣ Employee Hierarchy**
```sql
SELECT e1.Emp_Num AS Employee_ID, e1.LName AS Employee_Name,
       e2.Emp_Num AS Supervisor_ID, e2.LName AS Supervisor_Name,
       d.Name AS Department_Name
FROM EMPLOYEE e1
LEFT JOIN EMPLOYEE e2 ON e1.Super_ID = e2.Emp_Num
LEFT JOIN DEPARTMENT d ON e1.Dept_Code = d.Dept_Code
ORDER BY d.Name, e1.Emp_Num;
```

### **3️⃣ Employee Bonus Calculation**
```sql
UPDATE EMPLOYEE
SET BONUS_AMT = 200
WHERE Emp_Num IN (
    SELECT a.Emp_Num
    FROM ASSIGNMENT a
    JOIN PROJECT p ON a.Proj_Number = p.Proj_Number
    WHERE EXTRACT(QUARTER FROM p.Start_Date) = 3
    GROUP BY a.Emp_Num
    HAVING SUM(a.Hours_Used) >= 100
);
```

📌 **For more queries, check `queries.sql`.**

---

## 🎯 Project Goals
- **Build an efficient relational database** with **normalized tables**.
- **Enforce data integrity** using **primary & foreign keys, constraints, and triggers**.
- **Provide meaningful reports** through complex SQL queries.
- **Optimize query performance** for large-scale datasets.

---

## 🚀 Future Enhancements
- Implement **stored procedures** for business logic automation.
- Develop **triggers** to update records automatically.
- Use **PL/SQL scripts** for batch operations.
- Extend the project with a **web-based dashboard** for data visualization.

---

## 📜 License
This project is licensed under the **MIT License**.

---

### 🔗 **How to Get Started**
1. Clone this repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/dbm-project.git
   ```
2. Follow the **Setup Instructions** above.
3. Run SQL queries and analyze the results.

📧 **For any questions, contact [Your Email].**

---
