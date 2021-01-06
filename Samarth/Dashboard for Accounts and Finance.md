# DASHBOARD FOR ACCOUNTS AND FINANCE

This will detail the various modules in Samarth under Accounts and Finance such as:
- Payroll Management System

## PAYROLL MANAGEMENT SYSTEM

The Payroll Management System will display metrics related to the payroll such as nature of the payroll, details of salaries in the payroll, nature of employment and employee cadre.

The demo dashboard in samarth looks as follows:

![Demo Payroll Dashboard](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Accounts%20and%20Finance/Payroll%20Demo%20Portal.png)

The metrics being displayed in the Payroll Dashboard in Kibana are as follows:

| S No. | METRIC | DESCRIPTION | FIELDS AND TABLE | QUERY |
| :---: | :---: | :---: | :---: | :---: |
| 1 | Nature of Payroll | Shows the distribution of employees in payroll on the basis of their nature | payroll_budget_head.nature | select nature from payroll_budget_head |
| 2 | Salary Details | Shows the details of the amount paid in salary for every month of every year separately | payroll_final_processed_salary.year <br/><br/> payroll_final_processed_salary_breakup.net_pay <br/><br/> payroll_final_processed_salary.month <br/><br/> payroll_final_processed_salary_breakup.employee_id | select month, year from payroll_final_processed_salary <br/><br/> select net_pay, employee_id from payroll_final_processed_salary_breakup |
