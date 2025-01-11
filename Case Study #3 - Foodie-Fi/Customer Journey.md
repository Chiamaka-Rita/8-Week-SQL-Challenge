## Q1. Based off the 8 sample customers provided in the sample subscription table, write a brief description about each customer's onboarding journey.
```SQL
SELECT customer_id, fp.plan_id, plan_name, start_date, ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date)
FROM foodie_fi.subscriptions fs
JOIN foodie_fi.plans fp
ON fs.plan_id = fp.plan_id
WHERE customer_id = '1' 
OR customer_id = '2'
OR customer_id = '11'
OR customer_id = '13'
OR customer_id = '15'
OR customer_id = '16'
OR customer_id = '18'
OR customer_id = '19'
```
a. Customer_id 1: This customer signed up for the initial 7 days trial on 2020-08-01 and then subscribed for the basic plan on 2020-08-08.

b. Customer_id 2: This customer signed up for the initial 7 days trial on 2020-09-20 and then upgraded to the pro annual on 2020-09-27.

c. Customer_id 11: This customer signed up for the initial 7 days trial on 2020-11-19 and then churned their account after a week on 2020-11-26.

d. Customer_id 13: This customer signed up for the initial 7 days trial on 2020-12-15 and then subscribed for the basic plan on 2020-12-15 and subsequently upgraded to pro monthly on 2021-03-29
e. Customer_id 15: This customer signed up for the initial 7 days trial and was automatically subscribed to the pro monthly plan. The customer churned their account after a month and five days on 2020-04-29.

f. Customer_id 16: This customer signed up for the initial 7 days trial on 2020-05-31, then subscribed for the basic plan on 2020-06-07
and subsequently upgraded to pro annual on 2020-10-21. 

g. Customer_id 18: This customer signed up for the initial 7 days trial on 2020-07-06 and then the cx was automatically subscribed for the pro monthly plan on 2020-07-13 after the trial ended.

h. Customer_id 19: This customer signed up for the initial 7 free days trial ln 2020-06-22 and then the cx was automatically subscribed for the pro monthly plan on 2020-06-29 and subsequently upgraded to the pro annual plan after 2 months on 2020-08-29.
