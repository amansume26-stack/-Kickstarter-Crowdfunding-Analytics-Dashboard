# 📊 Kickstarter Crowdfunding Analytics Dashboard

![Project Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Tools](https://img.shields.io/badge/Tools-SQL%20%7C%20Excel%20%7C%20Tableau%20%7C%20Power%20BI-blue)
![Data](https://img.shields.io/badge/Dataset-366K%2B%20Projects-orange)
![Domain](https://img.shields.io/badge/Domain-Crowdfunding%20Analytics-purple)

> A comprehensive, multi-tool data analytics project analyzing **366,000+ Kickstarter campaigns** to uncover what drives crowdfunding success — built with SQL, Excel, Tableau, and Power BI.

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Key Findings](#-key-findings)
- [Tech Stack](#-tech-stack)
- [Project Architecture](#-project-architecture)
- [SQL Analysis](#-sql-analysis)
- [Dashboard Previews](#-dashboard-previews)
- [Dataset](#-dataset)

---

## 🔍 Project Overview

This end-to-end analytics project investigates Kickstarter's crowdfunding ecosystem using a dataset of over **366,000 campaigns**. The goal was to identify the key drivers of campaign success, understand funding patterns across categories and geographies, and present insights through interactive dashboards across three BI platforms.

The project follows a full data analytics pipeline:
Raw Data → MySQL (SQL Queries) → Excel Dashboard → Tableau Dashboard → Power BI Dashboard → Insights & Recommendations
---

## 💡 Key Findings

| Metric | Value |
|---|---|
| 📁 Total Projects Analyzed | 366,000+ |
| 💰 Total Funds Raised (Successful) | $1.34 Billion+ |
| 👥 Total Backers | 39,970+ |
| 📅 Avg. Duration of Successful Projects | 80 Days |
| ✅ Overall Project Success Rate | ~38% |

### Strategic Insights

- **High Failure Rate** — Only ~38% of projects succeed, making data-driven strategy essential
- **Lower Goals Win** — Campaigns with realistic, lower funding targets show significantly higher success rates
- **Backer Momentum Matters** — Projects with high early backer volume have exponentially better odds of reaching their goal
- **Optimal Duration** — 60–90-day campaigns hit the sweet spot between urgency and audience fatigue
- **US Dominance** — The United States contributes the highest volume of projects and total funds raised
- **Pareto Principle** — Niche categories like **Games** and **Design** drive the majority of total funds raised

---

## 🛠 Tech Stack

| Tool | Purpose |
|---|---|
| **MySQL** | Data storage, cleaning, and multi-dimensional SQL analysis |
| **Microsoft Excel** | Pivot-based dashboard, high-level KPI tracking |
| **Tableau** | Interactive visual analytics, geographic and category deep-dives |
| **Power BI** | Advanced DAX measures, dynamic slicers, executive-level reporting |
| **PowerPoint** | Final presentation of findings for stakeholders |

---

## 🏗 Project Architecture
📦 Kickstarter-Crowdfunding-Analytics-Dashboard
┣ 📂 sql
┃ ┗ 📄 Crwdproject.sql
┣ 📂 dashboards
┃ ┣ 📊 CrowdFunding_Dashboard_Excel.xlsx
┃ ┣ 📉 CROWDFUNDING_DASHBOARD.twbx
┃ ┗ 📈 CrowdFunding_Dashboard_PowerBI.pbix
┣ 📂 presentation
┃ ┗ 📑 Crowdfunding_Analytics.pptx
┣ 📂 images
┃ ┣ 🖼 excel_dashboard.png
┃ ┣ 🖼 tableau_dashboard.png
┃ ┗ 🖼 powerbi_dashboard.png
┗ 📄 README.md
---

## 🗄 SQL Analysis

All analysis was performed in **MySQL** on a normalized schema with tables for `projects`, `creator`, `location`, `category`, and a `calender_table`.

### Key Queries Covered

<details>
<summary>📋 Click to expand all SQL analyses</summary>

#### 1. Total Projects by Outcome
```sql
SELECT state, COUNT(*) AS total_projects
FROM projects
GROUP BY state
ORDER BY total_projects DESC;
```

#### 2. Total Projects by Country
```sql
SELECT country, COUNT(*) AS total_projects
FROM projects
GROUP BY country
ORDER BY total_projects DESC;
```

#### 3. Total Projects by Category
```sql
SELECT category_id, COUNT(ProjectID) AS total_projects
FROM projects
GROUP BY category_id
ORDER BY category_id ASC;
```

#### 4. Projects by Year, Quarter & Month
```sql
SELECT 
    YEAR(FROM_UNIXTIME(created_at)) AS year,
    QUARTER(FROM_UNIXTIME(created_at)) AS quarter,
    MONTHNAME(FROM_UNIXTIME(created_at)) AS month,
    COUNT(*) AS total_projects
FROM projects
GROUP BY 1, 2, 3
ORDER BY 1 DESC, 2, 3;
```

#### 5. Total Amount Raised by Successful Projects
```sql
SELECT 
    CONCAT(ROUND(SUM(goal * static_usd_rate) / 1000000000, 2), 'B') AS total_in_billions
FROM projects
WHERE state = 'successful';
```

#### 6. Overall Success Rate
```sql
SELECT 
    CONCAT(
        ROUND((COUNT(CASE WHEN state = 'successful' THEN 1 END) * 100.0 / COUNT(*)), 2),
    '%') AS success_percentage
FROM projects;
```

#### 7. Success Rate by Goal Range
```sql
SELECT 
    CASE 
        WHEN (goal * static_usd_rate) < 5000 THEN 'Less than $5K'
        WHEN (goal * static_usd_rate) BETWEEN 5000 AND 20000 THEN '$5K – $20K'
        WHEN (goal * static_usd_rate) BETWEEN 20000 AND 50000 THEN '$20K – $50K'
        WHEN (goal * static_usd_rate) BETWEEN 50000 AND 100000 THEN '$50K – $100K'
        ELSE 'Greater than $100K'
    END AS goal_range,
    COUNT(ProjectID) AS total_projects,
    COUNT(CASE WHEN state = 'successful' THEN 1 END) AS successful_projects,
    CONCAT(ROUND(COUNT(CASE WHEN state = 'successful' THEN 1 END) * 100.0 / COUNT(ProjectID), 2), '%') AS success_rate
FROM projects
GROUP BY 1
ORDER BY 2;
```

#### 8. Top 10 Projects by Backers (Successful Only)
```sql
SELECT project_name, state, backers_count, rnk
FROM (
    SELECT 
        name AS project_name, state, backers_count,
        RANK() OVER (ORDER BY backers_count DESC) AS rnk
    FROM projects
    WHERE state = 'successful'
) ranked
WHERE rnk <= 10;
```

#### 9. Average Campaign Duration (Successful Projects)
```sql
SELECT 
    ROUND(AVG(DATEDIFF(FROM_UNIXTIME(successful_at), FROM_UNIXTIME(created_at))), 0) AS avg_days
FROM projects
WHERE state = 'successful'
  AND successful_at IS NOT NULL
  AND created_at IS NOT NULL;
```

#### 10. Success Rate by Category
```sql
SELECT 
    category_id,
    COUNT(*) AS total_projects,
    SUM(CASE WHEN state = 'successful' THEN 1 ELSE 0 END) AS successful_projects,
    CONCAT(ROUND(SUM(CASE WHEN state = 'successful' THEN 1 ELSE 0 END) / COUNT(*) * 100, 2), '%') AS success_rate
FROM projects
GROUP BY category_id
ORDER BY success_rate DESC;
```

</details>

---

## 📸 Dashboard Previews

### 📊 Excel Dashboard
> Pivot-based high-level tracking of KPIs, outcome segmentation, and geographic distribution.

![Excel Dashboard](https://github.com/amansume26-stack/-Kickstarter-Crowdfunding-Analytics-Dashboard/blob/main/Snapshot%20Of%20Excel%20Dashboard(Cf).png)

---

### 📉 Tableau Dashboard
> Interactive filters for region, category, and funding timelines. Pareto analysis highlights top-performing categories.

![Tableau Dashboard](https://raw.githubusercontent.com/amansume26-stack/-Kickstarter-Crowdfunding-Analytics-Dashboard/main/images/tableau_dashboard.png)

---

### 📈 Power BI Dashboard
> Advanced DAX-powered dashboard with dynamic slicers. Reveals that while *Exploding Kittens* attracted the highest backer volume, hardware projects like *Pebble Time* dominate total funds raised.

![Power BI Dashboard](https://github.com/amansume26-stack/-Kickstarter-Crowdfunding-Analytics-Dashboard/blob/main/Snapshot%20Of%20Power%20BI%20Dashboard(Cf).png)

---

## 📥 Download Dashboards

> These files exceed GitHub's 25MB limit and are hosted on Google Drive.

| Dashboard | Link |
|---|---|
| 📈 Power BI (.pbix) | [Download from Google Drive](YOUR_POWERBI_DRIVE_LINK) |
| 📉 Tableau (.twbx) | [Download from Google Drive](YOUR_TABLEAU_DRIVE_LINK) |

---

## 📂 Dataset

| Field | Description |
|---|---|
| `ProjectID` | Unique campaign identifier |
| `name` | Campaign title |
| `state` | Outcome: successful, failed, canceled, live |
| `goal` | Funding target (local currency) |
| `static_usd_rate` | Currency-to-USD conversion rate |
| `backers_count` | Number of backers |
| `created_at` | Campaign launch timestamp (Unix) |
| `successful_at` | Timestamp of goal achievement (Unix) |
| `category_id` | Campaign category reference |
| `country` | Country of origin |

---

## 📬 How to Use This Repository

1. **SQL** — Open `Crwdproject.sql` in MySQL Workbench and run against your crowdfunding database
2. **Excel** — Open the `.xlsx` file directly; all pivots and charts are pre-built
3. **Tableau** — Download `.twbx` from Google Drive and open in Tableau Desktop
4. **Power BI** — Download `.pbix` from Google Drive and open in Power BI Desktop

---

> *"Data is the new oil — but only if you know how to refine it."*
>
> 📊 Kickstarter Crowdfunding Analytics | End-to-End Data Analytics Project
