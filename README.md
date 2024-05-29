# Datamart-case-study


-- DATA CLEANSING
create table clean_weekly_sales as
select week_date,week(week_date) as week_number,
month(week_date) as month_number,
year(week_date) as calender_year,
region, platform,
case 
when segment=null then "Unknown"
else segment
end as segment,
case
when right(segment,1) ='1' then "Young Adults"
when right (segment,1)='2' then "Middle Aged"
when right (segment,1)in ('3','4') then "Retirees"
else "Unknown"
end as age_band,
case
when left(segment,1)='c' then "couples"
when left(segment,1)='f' then "Families"
else 'Unknown'
end as demographic,
customer_type,transactions,sales,
round(sales/transactions,2) as "Avg_transactions"
from weekly_sales;

select * from clean_weekly_sales limit 10;


-- DATA EXPLORATION
-- 1) which week number are missing from the dataset?
create table senquence100 (x int auto_increment primary key);
insert into senquence100 values (),(),(),(),(),(),(),(),(),();
insert into senquence100 values (),(),(),(),(),(),(),(),(),();
insert into senquence100 values (),(),(),(),(),(),(),(),(),();
insert into senquence100 values (),(),(),(),(),(),(),(),(),();
insert into senquence100 values (),(),(),(),(),(),(),(),(),();
insert into senquence100 select x + 50 from senquence100; 
select * from senquence100;

create table seq52 as (select x from senquence100 limit 52);
select distinct x as week_day from seq52 where x not in(select distinct week_number from clean_weekly_sales); 


-- 2) How many total transactions were there for each year in the dataset?
SELECT
  calender_year,
  SUM(transactions) AS total_transactions
FROM clean_weekly_sales group by calender_year;

-- 3)  3.What are the total sales for each region for each month?
SELECT
  month_number,
  region,
  SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY month_number, region
ORDER BY month_number, region;

-- 4) What is the total sum of transactions for each platform
select platform,count(transactions)as transactions from clean_weekly_sales
group by platform;

-- 5) What is the percentage of sales for Retail vs Shopify for each month?
WITH cte_monthly_platform_sales AS (
  SELECT
    month_number,calender_year,
    platform,
    SUM(sales) AS monthly_sales
  FROM clean_weekly_sales
  GROUP BY month_number,calender_year, platform
)
SELECT
  month_number,calender_year,
  ROUND(
    100 * MAX(
    CASE
    WHEN platform = 'Retail' THEN monthly_sales ELSE NULL END) /
      SUM(monthly_sales),2
  ) AS retail_percentage,
  ROUND(
    100 * MAX(
    CASE WHEN platform = 'Shopify' THEN monthly_sales ELSE NULL END) /
      SUM(monthly_sales),
    2
  ) AS shopify_percentage
FROM cte_monthly_platform_sales
GROUP BY month_number,calender_year
ORDER BY month_number,calender_year;

-- 6)What is the percentage of sales by demographic for each year in the dataset?
SELECT calender_year, demographic,SUM(SALES) AS yearly_sales,
  ROUND(
    (100 * SUM(sales) / SUM(SUM(SALES)) OVER (PARTITION BY demographic)),2) AS percentage
FROM clean_weekly_sales
GROUP BY calender_year, demographic ORDER BY calender_year,demographic;

-- 7) Which age_band and demographic values contribute the most to Retail sales?
SELECT  age_band,  demographic,SUM(sales) AS total_sales FROM clean_weekly_sales
WHERE platform = 'Retail' GROUP BY age_band, demographic ORDER BY total_sales DESC;

