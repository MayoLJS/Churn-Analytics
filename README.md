
# Customer Analytics: Telephone subscribers churn analytics
![dash-image](images/main%20dashboard.png)

This project involves creating an ETL pipeline and Power BI dashboard to analyze customer data and identify areas for marketing campaigns and predict future churners.

## Project Goals
- **Analyze Customer Data** at the following levels:
  - Demographic
  - Geographic
  - Payment and Account Information
  - Services
- **Study Churner Profile** and identify areas for implementing marketing campaigns
- **Predict Future Churners**

## Metrics Required
- Total Customer
- Total Churn and Churn Rate
- New Joiners

## SQL - Database Creation and Data Exploration

### 1. Database Creation:
```sql
CREATE DATABASE db_Churn;
```

### 2. Data Exploration:

- Check distinct values:
```sql
SELECT * FROM stg_Churn;
```

- Gender distribution:
```sql
SELECT 
    Gender,
    COUNT(Gender) AS TotalCount,
    COUNT(Gender) * 1.0 / (SELECT COUNT(*) FROM stg_Churn) AS Percentage
FROM stg_Churn
GROUP BY Gender;
```

- Contract type distribution:
```sql
SELECT 
    Contract, 
    COUNT(Contract) AS TotalCount,
    COUNT(Contract) * 1.0 / (SELECT COUNT(*) FROM stg_Churn) AS Percentage
FROM stg_Churn
GROUP BY Contract;
```

- Customer status and revenue impact:
```sql
SELECT 
    Customer_Status,
    COUNT(Customer_Status) AS TotalCount, 
    SUM(Total_Revenue) AS TotalRev,
    SUM(Total_Revenue) / (SELECT SUM(Total_Revenue) FROM stg_Churn) * 100 AS RevPercentage
FROM stg_Churn
GROUP BY Customer_Status;
```

- State distribution:
```sql
SELECT 
    State, 
    COUNT(State) AS TotalCount,
    COUNT(State) * 100.0 / (SELECT COUNT(*) FROM stg_Churn) AS Percentage
FROM stg_Churn
GROUP BY State
ORDER BY Percentage DESC;
```

### 3. Checking for NULL values:
```sql
SELECT 
    SUM(CASE WHEN Customer_ID IS NULL THEN 1 ELSE 0 END) AS Customer_ID_Null_Count,
    ...
FROM stg_Churn;
```

### 4. Data Transformation and Insertion into Production Table:
```sql
SELECT 
    Customer_ID, Gender, Age, Married, State, 
    ISNULL(Value_Deal, 'None') AS Value_Deal, 
    ...
INTO [db_Churn].[dbo].[prod_Churn]
FROM [db_Churn].[dbo].[stg_Churn];
```

### 5. Creating Views for Power BI:
```sql
CREATE VIEW vw_ChurnData AS
    SELECT * 
    FROM prod_Churn 
    WHERE Customer_Status IN ('Churned', 'Stayed');
```

```sql
CREATE VIEW vw_JoinData AS
    SELECT * 
    FROM prod_Churn 
    WHERE Customer_Status IN ('Joined');
```
link to full query >> [SQL](scripts/SQLQuery.sql)

## Power BI

### 1. Data Transformation in Power BI:
![sqlload-image](images/Load%20Data%20in%20Power%20BI%20from%20SSMS.png)
- Import the data using Power Query from the `prod_Churn` table.
- Create additional columns: 
  - Churn Status
  - Monthly Charge Range

### 2. Measures:
- Total customers:
```DAX
Total customers = COUNT(prod_Churn[Customer_ID]);
```

- Total churn:
```DAX
Total Churn = SUM(prod_Churn[Churn Status]);
```

- New Joiners:
```DAX
New Joiners = CALCULATE(COUNT(prod_Churn[Customer_ID]), prod_Churn[Customer_Status] = "Joined");
```

- Churn Rate:
```DAX
Churn Rate = [Total Churn] / [Total customers];
```

### 3. Visualizations:
- **Card Visuals** for Total customers, Total churn, New joiners, and Churn rate.
- **Donut Chart** for Gender distribution.
- **Line/Stacked Column Chart** for churn rate by Age group.
- **Bar Chart** for churn rate by Contract type and Payment method.
- **Matrix Visual** for churn by service.

### 4. Enhancing the Report:
- Adding drop-downs for interaction and tooltips for churn reasons.
- Customizing the background using PowerPoint for a polished look. link to ppt >> [PowerPoint](other%20assets/Background%20Mockup.pptx)
- Using **Narrative Visual** for AI-based report summary.
  ![narrative-image](images/Narrative%20Visual.png)

## Analysis
- **Key Insights**:
  - Most churners are **female** (64%).
  - Services with the highest churn rates include **Device Protection Plan**, **Online Backup**, and **Premium Support**.
  - **Fiber Optic** users have the highest churn rate.
  - **Month-to-Month contracts** have the highest churn rate.

- **Recommended Actions**:
  - Improve **Device Protection Plan**, **Online Backup**, and **Premium Support** offerings.
  - Address pain points in **internet, phone, and unlimited data services**.
[Interactive Report](https://app.powerbi.com/view?r=eyJrIjoiOTc1N2VlNGQtOGMwOC00MDM2LWE0ZWMtZjNkNzBmZWU2NDIwIiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9)  



## Predicting Future Churners
- We can predict future churners using the **Random Forest** algorithm, which can be integrated into the Power BI report. link to notebook >> [Jupyter Notebook](scripts/Churn%20Prediction.ipynb)

## Conclusion
This project provides a comprehensive analysis of customer churn, allowing businesses to better understand customer behavior and implement strategies to reduce churn and improve customer retention.
Project thought process can be seen here [Word Document](docs/Project%20documentation%20and%20Thought%20process.docx)

