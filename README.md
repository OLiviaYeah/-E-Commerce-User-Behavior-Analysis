# E-Commerce-User-Behavior-Analysis


## 1.About Dataset

### About
I use the eCommerce Events History in Cosmetics Shop dataset from Kaggle. 
You can download it from here:
https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop.

This file contains behavior data for 5 months (Oct 2019 – Feb 2020) from a medium cosmetics online store. For this project I only use the dataset of Feb 2020.
Each row in the file represents an event. All events are related to products and users. Each event is like many-to-many relation between products and users.

### File Structure

<img width="737" alt="Structure" src="https://user-images.githubusercontent.com/101906347/169888445-d52b45d6-21be-4c44-95e5-0ce46a475f07.png">


## 2. Problem Statement

### How’s the overall Funnel looks like? 
What is the conversion rate?  It is healthy enough?
Through which step we lost the most customers?

### How’s customer behavior path?
What’s users most active date?
What’s users most active time?

### How each product contributes to business?
Which product has high conversion rate?
How many users will continue to repurchase?  Are they repurchasing the same product?

### Who are the most valuable customers? 
What are their purchase preferences?


## 3. Data Cleaning

### Duplicate elimination 
```
SELECT 
COUNT(DISTINCT user_id) AS unique_user_id,
COUNT(DISTINCT product_id) as unique_product_id,
COUNT(DISTINCT category_id) as unique_category,
COUNT(event_type) as unique_event_type
FROM ecommers.cosmetics;
```

### Find missing value
```
select *
from ecommers.cosmetics
where 
user_id is null
or product_id is null
or category_id is null
or event_type is null
or event_time is null;
```

## 4. Solving the problems

### A)	How’s the overall Funnel looks like? 
I used Python package plotly to generate the visualizations.

<img width="468" alt="funnel" src="https://user-images.githubusercontent.com/101906347/169897780-b623460d-db11-4570-9f35-2370b2eb2d28.png">

what is the conversion rate?

```
--create customer bahavior view to count the number of various behaviors(event_type) occur
SELECT user_id, 
count(event_type) as customer_behaviors,
sum(case when event_type='view' then 1 else 0 end) as view,
sum(case when event_type='cart' then 1 else 0 end) as cart,
sum(case when event_type='remove_from_cart' then 1 else 0 end) as remove_from_cart,
sum(case when event_type='purchase' then 1 else 0 end) as purchase
FROM `ecommerce-analysis-345602.ecommers.cosmetics`
group by user_id
order by customer_behaviors desc;
```

```
#users conversion rate
-- # of users who purchsed
SELECT count(user_id) as number_of_purchase
FROM `ecommerce-analysis-345602.ecommers.cosmetics`
where event_type='purchase';
```

```
-- # of users who visit the store
select count(user_id) as number_of_visit
FROM `ecommerce-analysis-345602.ecommers.cosmetics`;
```

<img width="369" alt="funnel result1" src="https://user-images.githubusercontent.com/101906347/169891155-a2d328be-fd69-4b20-8ab5-e6e97a1b561f.png">

<img width="369" alt="funnel result 2 " src="https://user-images.githubusercontent.com/101906347/169891205-71b70272-d383-4c7d-9a07-4a72a88933d8.png">

Conclusion: 
Only 5.82% of users purchased from website, and the funnel doesn’t look very healthy for me.

What's next:
I want to know which shopping step users leave?

### 1. Step: users view but not add to cart or not purchase directly

```
-- users churn rate on differante stages
--view but not add to cart and not purchase directly 
select sum(view) as total_view,
(select sum(view) 
from `ecommerce-analysis-345602.ecommers.customer_behaviours`
where view>0
and cart=0
and purchase=0
) as not_add_to_cart
from `ecommerce-analysis-345602.ecommers.customer_behaviours`
where view>0
and purchase=0;
```

<img width="452" alt="step 1 r" src="https://user-images.githubusercontent.com/101906347/169892137-89886730-ea16-4d06-8dd6-c6cfcf8f649d.png">

Churn rate=702712/1314918=53.44%

### 2. Step: Add to the cart but not purchase

```
--add to cart but not purchase
select sum(cart) as total_add_to_cart,
(select sum(cart)
from `ecommerce-analysis-345602.ecommers.customer_behaviours`
where cart>0
and purchase=0) as total_not_purchase
from `ecommerce-analysis-345602.ecommers.customer_behaviours`;
```
<img width="468" alt="step2" src="https://user-images.githubusercontent.com/101906347/169898551-e7be89e2-ba1f-4ff1-bbd3-a1a007f6113d.png">

Conclusion: Churn rate= 52.71%

### 3.Step: remove from cart
```
--remove from cart
select sum(cart) as total_add_to_cart,
(select sum(remove_from_cart)
from `ecommerce-analysis-345602.ecommers.customer_behaviours`
where cart>0
and purchase=0) as total_remove_from_cart
from `ecommerce-analysis-345602.ecommers.customer_behaviours`;
```
<img width="468" alt="step 3" src="https://user-images.githubusercontent.com/101906347/169898766-606b4b14-6b6f-4f77-a293-dd839d5253c8.png">

Conclusion: churn rate=34%

What's next:
What factors are causing the high churn rate of 1 & 2 stpes?
I will use hypothesis testing to find out the reasons.

### Hypothesis testing

Hypothesis 1:  products which website recommend do not meet user’s preference
```
# hy1,website recommend products do not meet user pereference
--create view top 100 most viewed products
select product_id, sum(view) as total_view
from `ecommerce-analysis-345602.ecommers.customer`
group by product_id
order by sum(view) desc
limit 100;
```
```
--create viwe,top 100 most purchased products  
select product_id, sum(purchase) as total_purchase
from `ecommerce-analysis-345602.ecommers.customer`
group by product_id
order by sum(purchase) desc
limit 100;
```
```
-- the common product_id of top 100 view & purchased
select count(a.product_id) as common_product_id
from `ecommerce-analysis-345602.ecommers.most view product` as a
inner join `ecommerce-analysis-345602.ecommers.most purchase product` as b
on a.product_id=b.product_id;
```

<img width="462" alt="hy1" src="https://user-images.githubusercontent.com/101906347/169899326-7c2f1961-17b3-4906-9fe8-0967b0c75271.png">

Conclusion: there are 31 products are both appear on Top100 view and Top100 purchase list, so hypothesis 1 accept. We need to recommend products that are suitable for users according to their preferences, so as to retain customers


Hypothesis 2:  customers are not satisfied with product/service they purchased, that’s why they don’t want to purchase again

```
# hy2: customers are not satisfied with product/service they purchased, that's why the churn rate is high

-- total amount of products which be purchaed 2 times or more, 20482(total products is 48579)
select product_id, count(event_type) as event_purchase
from `ecommerce-analysis-345602.ecommers.cosmetics`
where event_type='purchase'
group by product_id
having count(event_type)>=2
order by count(event_type) desc;

-- total amount of users who purchased 2 times or more, 23190 (total products is 391055) 
select count(user_id)
from (select user_id, sum(purchase) as total_purchase
from `ecommerce-analysis-345602.ecommers.customer_behaviours`
group by user_id) as total_purchase_count
where total_purchase>=2;
```

<img width="425" alt="hy2,1" src="https://user-images.githubusercontent.com/101906347/169899578-afefe81b-ee3e-4a75-a212-b0c6687d8228.png">
<img width="295" alt="hy22" src="https://user-images.githubusercontent.com/101906347/169899583-1a3f5924-50fb-4d80-b436-fda20a537559.png">

Conclusion: 
42% of products was repurchased by customers, but only 6% of customer purchased again, so Hypothesis 2 was accepted.

What's next: 
I want to further understand the user behavior patterns, so that we can better recommend products and services for users, and better improve the conversion rate


