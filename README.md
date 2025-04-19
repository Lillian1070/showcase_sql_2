# [SQL] Identifying the Top 3 Unique Salaries in Each Department

_This SQL practice problem is based on **[LeetCode SQL 50 - 185. Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/description/?envType=study-plan-v2&envId=top-sql-50)**, used here for personal learning and educational purposes._ 

- **Objective:** Calculate each department's top 3 unique salaries.  
- **Practice Purpose:** Self-learning and reinforcement of SQL joins, aggregation, and window functions.
- **Outline:**
  - [**Practice**](#section-1) (practice problem and query output)
  - [**Solution**](#section-2) (step-by-step explanation)
  - [**Query Optimization**](#section-3) (refinement for efficiency and readability)


## <a name="section-1"></a>🧪 Practice

Table: `Employee`

| Column Name  | Type    |
|--------------|---------|
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |


- `id` is the primary key (column with unique values) for this table.
- `departmentId` is a foreign key (reference column) of the ID from the Department table.
- Each row of this table indicates the ID, name, and salary of an employee. It also contains the ID of their department.
 

Table: `Department`

| Column Name | Type    |
|-------------|---------|
| id          | int     |
| name        | varchar |


- `id` is the primary key (column with unique values) for this table.
- Each row of this table indicates the ID of a department and its name.
 

A company's executives are interested in seeing who earns the most money in each of the company's departments. A high earner in a department is an employee who has a salary in the **top three unique** salaries for that department.

Write a solution to find the employees who are high earners in each of the departments.

Return the result table in any order.

The result format is in the following example.

 

### Example 

#### Input: 

`Employee` table:

| id | name  | salary | departmentId |
|----|-------|--------|--------------|
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |


`Department` table:

| id | name  |
|----|-------|
| 1  | IT    |
| 2  | Sales |


#### Output: 

| Department | Employee | Salary |
|------------|----------|--------|
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |



#####  Explanation: 

- In the IT department:
  - Max earns the highest unique salary
  - Both Randy and Joe earn the second-highest unique salary
  - Will earns the third-highest unique salary

- In the Sales department:
  - Henry earns the highest salary
  - Sam earns the second-highest salary
  - There is no third-highest salary as there are only two employees
 

#####  Constraints:

- There are no employees with the exact same name, salary and department.





## <a name="section-2"></a>🧠 Solution

*This section outlines my thought process for solving the problem.*

### Step 1: Identify the Fields Required

To create the result table with department name, employee name, and top 3 unique salaries in each department, I need to find out the following info: 

* Ranking of unique salaries in each group (see temp ranking table `T`)
* Merging department name into the temp ranking table by using department ID (see step 3)


### Step 2: Create a Temporary Table `T` to Rank Unique Salaries Using `DENSE_RANK()`

To find out the salary ranking in each department, I created a temporary table `t` that ranks salaries in each group using [`DENSE_RANK()`](https://www.geeksforgeeks.org/mysql-ranking-functions/) function. 

Please note that compared to `RANK()`, `DENSE_RANK()` ensures unique salaries are ranked without gaps.

```sql
WITH 
    T AS (
        SELECT 
            departmentId,
            name,
            salary, 
            DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rank_in_group
        FROM Employee
    )
```

The temporary table `T` should be similar to what we have below. 

| departmentId | name  | salary | rank_in_group |
| ------------ | ----- | ------ | ------------- |
| 1            | Max   | 90000  | 1             |
| 1            | Joe   | 85000  | 2             |
| 1            | Randy | 85000  | 2             |
| 1            | Will  | 70000  | 3             |
| 1            | Janet | 69000  | 4             |
| 2            | Henry | 80000  | 1             |
| 2            | Sam   | 60000  | 2             |


### Step 3: Retrieve the Final Table

Finally, I can merge the department names into the temp ranking table `T` using the department ID to include the department names, employee names, and salaries as required. Additionally, I filter the results to include only salaries with a ranking number of 3 or lower in each department.

```sql
SELECT 
    d.name AS Department, 
    t.name AS Employee,
    t.salary AS Salary 
FROM T t
LEFT JOIN Department d ON t.departmentId=d.id
WHERE rank_in_group <= 3
ORDER BY d.name, t.salary DESC;
```



#### Final Syntax and Output using MySQL

##### * Syntax

```sql
WITH 
    T AS (
        SELECT 
            departmentId,
            name,
            salary, 
            DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rank_in_group
        FROM Employee
    )
    
    
SELECT 
    d.name AS Department, 
    t.name AS Employee,
    t.salary AS Salary 
FROM T t
LEFT JOIN Department d ON t.departmentId=d.id
WHERE rank_in_group <= 3
ORDER BY d.name, t.salary DESC;
```

##### * Output

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |


## <a name="section-3"></a>🛠️ Query Optimization using MySQL

*Note: This section is updated on 02/17/2025.*

Upon review, I realized that the `DENSE_RANK()` function can be performed in a single statement without a `WITH` clause, making the query more concise.


```sql
SELECT 
    d.name AS Department, 
    e.name AS Employee,
    e.salary AS Salary 
FROM (
    SELECT 
        departmentId, 
        name, 
        salary, 
        DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rank_in_group
    FROM Employee
) e
LEFT JOIN Department d ON e.departmentId = d.id
WHERE rank_in_group <= 3
ORDER BY d.name, e.salary DESC;
```
