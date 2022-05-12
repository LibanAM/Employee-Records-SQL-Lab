# Task - Employee Records

The major corporation BusinessCorp&#8482; wants to do some analysis of varioius metrics around its employees and have contracted the job out to you. They have provided you with two SQL files, you will need to write the queries which will extract the data they are looking for.

## Setup

- Create a database to run the files against
- Use the `psql -d database -f file` command to run the SQL files
- Before starting to write the queries take some time to look at the data and figure out the relationships between the tables - maybe even draw an ERD to help.

## Tasks

### CTEs

1) Calculate the average salary of all employees

```sql
<!--Copy solution here-->
WITH avgSalary (id, avg_salary) AS 
    (SELECT 
        employees.id,
		AVG(salary) OVER(ORDER BY employees.id ASC)
    FROM employees
	GROUP BY employees.id)
SELECT id, avg_salary
FROM avgSalary;
```

2) Calculate the average salary of the employees in each team (hint: you'll need to `JOIN` and `GROUP BY` here)

```sql
WITH average_department_salary (id, department, average_salary) AS
(SELECT DISTINCT department_id, 
		departments.name,
		AVG(salary) OVER (PARTITION BY departments.name)
	FROM employees
	INNER JOIN departments
	ON departments.id = employees.department_id)
SELECT id, department, average_salary FROM average_department_salary;
```

3) Using a CTE find the ratio of each employees salary to their team average, eg. an employee earning £55000 in a team where the average is £50000 has a ratio of 1.1

```sql
WITH average_department_salary (id, name, department, department_average_salary) AS
(SELECT employees.id,
        CONCAT(employees.first_name,' ', employees.last_name),
		departments.name,
		AVG(salary) OVER (PARTITION BY departments.name)
	FROM employees
	INNER JOIN departments
	ON departments.id = employees.department_id)
SELECT average_department_salary.id, name, department, department_average_salary, employees.salary, (employees.salary / department_average_salary) AS employee_department_ratio FROM average_department_salary
INNER JOIN employees
ON employees.id = average_department_salary.id;
```

4) Find the employee with the highest ratio in Argentina
```sql
WITH average_department_salary (id, name, department, department_average_salary) AS
(SELECT employees.id, CONCAT(employees.first_name,' ', employees.last_name), departments.name, AVG(salary) OVER (PARTITION BY departments.name)
FROM employees
INNER JOIN departments
ON departments.id = employees.department_id)
SELECT average_department_salary.id, name, department, department_average_salary, employees.salary, (employees.salary / department_average_salary) AS salary_ratio, employees.country FROM average_department_salary
INNER JOIN employees
ON employees.id = average_department_salary.id
WHERE country = 'Argentina'
ORDER BY (employees.salary / department_average_salary) DESC
LIMIT 1;
```

5) **Extension:** Add a second CTE calculating the average salary for each country and add a column showing the difference between each employee's salary and their country average

```sql
WITH average_department_salary (id, name, department, department_average_salary) AS
(SELECT employees.id, CONCAT(employees.first_name,' ', employees.last_name), departments.name, AVG(salary) OVER (PARTITION BY departments.name)
FROM employees
INNER JOIN departments
ON departments.id = employees.department_id), 
average_country_salary (id, country, country_average_salary) AS
(SELECT employees.id, employees.country, AVG(employees.salary) OVER (PARTITION BY employees.country)
FROM employees
INNER JOIN departments
ON departments.id = employees.department_id)
SELECT employees.id, average_department_salary.name, average_department_salary.department, employees.salary,department_average_salary, (employees.salary / department_average_salary) AS salary_department_ratio, employees.country, country_average_salary, (employees.salary - country_average_salary) AS country_salary_difference FROM average_department_salary
INNER JOIN employees
ON employees.id = average_department_salary.id
INNER JOIN average_country_salary
ON average_country_salary.id = average_department_salary.id;
```

---

### Window Functions

1) Find the running total of salary costs as the business has grown and hired more people
```sql
SELECT id, CONCAT(first_name, ' ', last_name) AS full_name, SUM(salary) OVER(ORDER BY id ASC) AS running_salary_total FROM employees; 
```


2) Determine if any employees started on the same day (hint: some sort of ranking may be useful here)

```sql
SELECT DISTINCT id, CONCAT(first_name, ' ', last_name) AS full_name, start_date, RANK() OVER (ORDER BY start_date ASC) AS start_order FROM employees;
```

3) Find how many employees there are from each country
```sql
SELECT DISTINCT country, COUNT(id) OVER(PARTITION BY country) AS employees_country_count FROM employees;
```

4) Show how the average salary cost for each department has changed as the number of employees has increased
```sql
SELECT employees.id, CONCAT(first_name, ' ', last_name) AS full_name, departments.name , AVG(salary) OVER(PARTITION BY departments.name ORDER BY employees.id DESC) AS employees_country_count FROM employees
FULL OUTER JOIN departments
ON employees.department_id = departments.id;
```

5) **Extension:** Research the `EXTRACT` function and use it in conjunction with `PARTITION` and `COUNT` to show how many employees started working for BusinessCorp&#8482; each year. If you're feeling adventurous you could further partition by month...

---

### Combining the two

1) Find the maximum and minimum salaries
```sql

WITH max_salary (id, name, max_employee_salary) AS 
(SELECT 
	employees.id, 
	CONCAT(first_name, ' ', last_name), 
	MAX(employees.salary) OVER(ORDER BY salary DESC)  
FROM employees)
SELECT DISTINCT employees.id, name, max_employee_salary FROM max_salary
INNER JOIN employees
ON employees.id = max_salary.id
LIMIT 1;

WITH min_salary (id, name, min_employee_salary) AS 
(SELECT 
	employees.id, 
	CONCAT(first_name, ' ', last_name), 
	MIN(employees.salary) OVER(ORDER BY salary ASC)  
FROM employees)
SELECT DISTINCT employees.id, name, min_employee_salary FROM min_salary
INNER JOIN employees
ON employees.id = min_salary.id
LIMIT 1;

```

2) Find the difference between the maximum and minimum salaries and each employee's own salary

3) Order the employees by start date. Research how to calculate the **median** salary value and the **standard deviation** in salary values and show how these change as more employees join the company

4) Limit this query to only Research & Development team members and show a rolling value for only the 5 most recent employees.

