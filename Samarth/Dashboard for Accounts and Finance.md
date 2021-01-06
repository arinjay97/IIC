# DASHBOARD FOR ACCOUNTS AND FINANCE

This will detail the various modules in Samarth under Accounts and Finance such as:
- Payroll Management System

## PAYROLL MANAGEMENT SYSTEM

The Payroll Management System will display metrics related to the payroll such as nature of the payroll, details of salaries in the payroll, nature of employment and employee cadre.

The demo dashboard in samarth looks as follows:

![Demo Payroll Dashboard](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Accounts%20and%20Finance/Payroll%20Demo%20Portal.png)

The metrics being displayed in the Payroll Dashboard in Kibana are as follows:

| S No. | METRIC | DESCRIPTION | FIELDS AND TABLE | QUERY |
| :---: | :----: | :----: | :---: | :---: |
| 1 | Nature of Payroll | Shows the distribution of employees in payroll on the basis of their nature | payroll_budget_head.nature | select nature from payroll_budget_head |
| 2 | Salary Details | Shows the details of the amount paid in salary for every month of every year separately | payroll_final_processed_salary.year <br/><br/> payroll_final_processed_salary_breakup.net_pay <br/><br/> payroll_final_processed_salary.month <br/><br/> payroll_final_processed_salary_breakup.employee_id | select month, year from payroll_final_processed_salary <br/><br/> select net_pay, employee_id from payroll_final_processed_salary_breakup |
| 3 | Nature of Employment - Payroll Salary | Shows the Net Payment made to employees on the basis of their nature of employees | payroll_final_processed_salary.nature_of_employment <br/><br/> payroll_final_processed_salary_breakup.net_pay <br/><br/> payroll_final_processed_salary_breakup.employee_id | select nature_of_employment from payroll_final_processed_salary <br/><br/> select net_pay, employee_id from payroll_final_processed_salary_breakup
| 4 | Employee Cadre Salary Details | Shows the Net Payment made to employees on payroll based on their cadre along with their count | payroll_final_processed_salary.employee_cadre <br/><br/> payroll_final_processed_salary_breakup.net_pay <br/><br/> payroll_final_processed_salary_breakup.employee_id | select employee_cadre from payroll_final_processed_salary <br/><br/> select net_pay, employee_id from payroll_final_processed_salary_breakup |

#### The visualizations are created with the following metrics and aggregations:

1. Nature of Payroll: The metric used is count and the aggregation is done on the term nature from the table payroll_budget_head. The visualization looks like below:
 ![Nature of Payroll](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Accounts%20and%20Finance/Nature%20of%20Payroll.png)

2. Salary Details: The metrics used here are the Sum of the term net_pay and Unique Count of the term employee_id and 2 aggregations are done to create 2 columns in the table. The first aggregation is on the term year from the table payroll_final_processed_salary while the second aggregation is on the term month from the table payroll_final_processed_salary.
 ![Salary Details](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Accounts%20and%20Finance/Salary%20Details.png)

3. Nature of Employment: The metrics used here are the Sum of the term net_pay and Unique Count of the term employee_id on the aggregation of the term nature_of_employment from the table payroll_final_processed_salary. The visualization looks as seen below:
 ![Nature of Employment](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Accounts%20and%20Finance/Nature%20of%20Employment.png)

4. Employee Cadre: The metrics used here are the Sum of the term net_pay and Unique Count of the term employee_id on the aggregation of the term employee_cadre from the table payroll_final_processed_salary. The visualization looks as seen below:
 ![Employee Cadre](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Accounts%20and%20Finance/Employee%20Cadre%20Salary.png)