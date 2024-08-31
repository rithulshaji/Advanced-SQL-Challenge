# Advanced SQL Challenges

This repository contains a collection of advanced SQL queries that tackle complex data challenges using multiple tables, joins, and aggregation techniques.

## Data Overview

### Employees Table
| Column        | Description                                    | Data Type   |
|---------------|------------------------------------------------|-------------|
| `EmployeeID`  | Unique identifier for employees                | INTEGER     |
| `FirstName`   | Employee's first name                          | VARCHAR     |
| `LastName`    | Employee's last name                           | VARCHAR     |
| `DepartmentID`| Identifier linking the employee to a department| INTEGER     |
| `Salary`      | Employee's salary                              | DECIMAL     |
| `HireDate`    | Date when the employee was hired               | DATE        |

### Departments Table
| Column          | Description                                      | Data Type  |
|-----------------|--------------------------------------------------|------------|
| `DepartmentID`  | Unique identifier for departments                | INTEGER    |
| `DepartmentName`| Name of the department                           | VARCHAR    |
| `ManagerID`     | Identifier linking the department to its manager | INTEGER    |

### Projects Table
| Column       | Description                              | Data Type  |
|--------------|------------------------------------------|------------|
| `ProjectID`  | Unique identifier for projects           | INTEGER    |
| `ProjectName`| Name of the project                      | VARCHAR    |
| `StartDate`  | Date when the project started            | DATE       |
| `EndDate`    | Date when the project ended (NULL if ongoing) | DATE   |

### Employee_Projects Table
| Column        | Description                                      | Data Type  |
|---------------|--------------------------------------------------|------------|
| `EmployeeID`  | Unique identifier for employees                  | INTEGER    |
| `ProjectID`   | Unique identifier for projects                   | INTEGER    |
| `HoursWorked` | Number of hours the employee worked on the project | INTEGER  |


## SQL Queries

### Query 1:
List all employees who have not worked on any project. Include their `FirstName`, `LastName`, and `DepartmentName`.

```sql
SELECT e.FirstName, e.LastName, d.DepartmentName
FROM Employees e 
INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
LEFT JOIN Employee_Projects ep ON e.EmployeeID = ep.EmployeeID
WHERE ep.ProjectID IS NULL;
```

### Query 2:
Find the total number of hours worked by each employee on all projects. Return `EmployeeID`, `FirstName`, `LastName`, and `TotalHoursWorked`. Exclude employees who haven't worked on any project.

```sql
SELECT e.EmployeeID, e.FirstName, e.LastName, SUM(ep.HoursWorked) AS TotalHoursWorked
FROM Employees e 
INNER JOIN Employee_Projects ep ON e.EmployeeID = ep.EmployeeID
GROUP BY e.EmployeeID, e.FirstName, e.LastName;
```

### Query 3:
Retrieve the names of employees and their respective department names who are working under the same department as the manager of their department.

```sql
WITH Employee_departments AS (
    SELECT e.FirstName AS employee_First_Name, e.LastName AS employee_Last_Name, 
           d.DepartmentName AS employee_department, d.ManagerID AS ManagerID
    FROM Employees e 
    INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
)
SELECT employee_First_Name, employee_Last_Name, employee_department, ManagerID,
       e.FirstName AS Manager_First_Name, e.LastName AS Manager_Last_Name, 
       e.DepartmentName AS Manager_department
FROM Employee_departments ed 
INNER JOIN Employees e ON ed.ManagerID = e.EmployeeID
WHERE ed.employee_department = e.DepartmentName;
```

### Query 4:
List all the projects that were active in 2023 (i.e., projects that either started or ended in 2023 or were ongoing throughout 2023). Include `ProjectID`, `ProjectName`, `StartDate`, and `EndDate`.

```sql
SELECT ProjectID, ProjectName, StartDate, EndDate
FROM Projects
WHERE YEAR(StartDate) = 2023 
   OR YEAR(EndDate) = 2023 
   OR (StartDate <= '2023-12-31' AND (EndDate >= '2023-01-01' OR EndDate IS NULL));
```

### Query 5:
Find the highest-paid employee(s) in each department. Return `DepartmentName`, `FirstName`, `LastName`, and `Salary`.

```sql
WITH Ranked_Salary AS (
    SELECT e.FirstName, e.LastName, d.DepartmentName, e.Salary,
           DENSE_RANK() OVER (PARTITION BY e.DepartmentID ORDER BY e.Salary DESC) AS d_rnk
    FROM Employees e 
    INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
)
SELECT FirstName, LastName, DepartmentName, Salary 
FROM Ranked_Salary
WHERE d_rnk = 1;
```

### Query 6:
Retrieve the list of employees who have worked on more than three projects. Include `EmployeeID`, `FirstName`, `LastName`, and `NumberOfProjects`.

```sql
SELECT e.EmployeeID, e.FirstName, e.LastName, COUNT(DISTINCT ep.ProjectID) AS NumberOfProjects
FROM Employees e 
INNER JOIN Employee_Projects ep ON e.EmployeeID = ep.EmployeeID
GROUP BY e.EmployeeID, e.FirstName, e.LastName
HAVING COUNT(DISTINCT ep.ProjectID) > 3;
```

### Query 7:
Find the department that has the highest total salary among all its employees. Return the `DepartmentName` and `TotalSalary`.

```sql
WITH Department_Salary AS (
    SELECT d.DepartmentName, SUM(e.Salary) AS TotalSalary
    FROM Employees e 
    INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
    GROUP BY d.DepartmentName
)
SELECT DepartmentName, TotalSalary
FROM Department_Salary
ORDER BY TotalSalary DESC
LIMIT 1;
```

