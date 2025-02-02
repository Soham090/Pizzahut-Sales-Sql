# Pizzahut Sales Analysis SQL Project 🍕.

## Project Overview

**Project Title**: Pizzahut Sales Analysis  
**Database**: `pizzahut`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze pizzahut sales data. The project involves setting up a pizzahut sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. 

1. **Set up a pizzahut sales database**: Create and populate a pizzahut sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `pizzahut`.
- **Table Creation**: A table named `orders` & `orders_details` is created to store the sales data. The table structure includes columns for order_id ,order_date ,order_time for table `orders` & order_details_id, order_id, pizza_id, quantity for table `orders_details`

```sql
CREATE DATABASE pizzahut;
create table orders(
order_id int not  null,
order_date date not null,
order_time time not null,
primary key(order_id)
);
 
create table orders_details (
order_details_id int not  null,
order_id int not null,
pizza_id text not null,
quantity int not null,
primary key(order_details_id)
);
```

### 2. Data Analysis & Findings.

The following SQL queries were developed to answer specific questions:

1.**Retrieve the total number of orders placed**:
```sql
select count(order_id) as total_id from orders;
```

2.**Calculate the total revenue generated from pizza sales**:
 ```sql
SELECT 
    ROUND(SUM(orders_details.quantity * pizzas.price),
            2) AS total_revenue
FROM
    orders_details
        JOIN
    pizzas ON orders_details.pizza_id = pizzas.pizza_id
```


3.**Identify the highest-priced pizza**:
```sql
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```

4.**Identify the most common pizza size ordered**:
```sql
SELECT 
    pizzas.size,
    COUNT(orders_details.order_details_id) AS order_count
FROM
    pizzas
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC ;
```

5.**List the top 5 most ordered pizza types along with their quantities**:
```sql
SELECT 
    pizza_types.name, SUM(orders_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
```
6.**Join the necessary tables to find the total quantity of each pizza category ordered**:
```sql
SELECT 
    pizza_types.category,
    SUM(orders_details.quantity) AS QUANTITY
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;
```
7.**Determine the distribution of orders by hour of the day**:
```sql
SELECT 
    HOUR(order_time) AS Hours, COUNT(order_id) AS Orders
FROM
    orders
GROUP BY HOUR(order_time)
ORDER BY COUNT(order_id);
```
8.**Join relevant tables to find the category-wise distribution of pizzas**:
```sql
SELECT 
    category, COUNT(name)
FROM
    pizza_types
GROUP BY category
```
9.**Group the orders by date and calculate the average number of pizzas ordered per day**:
```sql
SELECT 
    ROUND(AVG(quantity), 0) as average_pizza_orders_per_day
FROM
    (SELECT 
        orders.order_date, SUM(orders_details.quantity) AS quantity
    FROM
        orders
    JOIN orders_details ON orders.order_id = orders_details.order_id
    GROUP BY orders.order_date) AS order_quantity;
```
10.**Determine the top 3 most ordered pizza types based on revenue**:
```sql
SELECT 
    pizza_types.name,
    round(SUM(orders_details.quantity * pizzas.price), 2)
             AS Revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY Revenue DESC
LIMIT 3;
```
11.**Calculate the percentage contribution of each pizza type to total revenue**:
```sql
SELECT 
    pizza_types.category,
    ROUND(SUM(orders_details.quantity * pizzas.price) / (SELECT 
                    ROUND(SUM(orders_details.quantity * pizzas.price),
                                2) AS total_sales
                FROM
                    orders_details
                        JOIN
                    pizzas ON orders_details.pizza_id = pizzas.pizza_id) * 100,
            2) AS Revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY Revenue DESC;
```
11.**Analyze the cumulative revenue generated over time**:
```sql
select order_date,
round(sum(revenue) over(order by order_date), 2) as cum_revenue
from
(select orders.order_date,
sum(orders_details.quantity * pizzas.price) as revenue
from orders_details join pizzas
on orders_details.pizza_id = pizzas.pizza_id
join orders
on orders.order_id = orders_details.order_id
group by orders.order_date) as sales;
```
12.**Determine the top 3 most ordered pizza types based on revenue for each pizza category**:
```sql
select name, revenue from 
(select category, name, revenue,
rank() over(partition by category order by revenue desc) as rn
from
(select pizza_types.category,pizza_types.name,
sum((orders_details.quantity)* pizzas.price) as revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join orders_details
on orders_details.pizza_id = pizzas.pizza_id
group by pizza_types.category, pizza_types.name)as a) as b
where rn <=3;
```
## Findings

- **Orders Dataset:**  Tracks the date and time of each order.
                       Can be used to analyze peak order times, busiest days, and customer ordering trends.
- **Pizza Types Dataset:** Contains information about each pizza, categorized by type. Includes ingredients, which can be used to determine the popularity of specific categories (e.g., Chicken, Vegetarian, etc.).
- **Pizzas Dataset:** Provides detailed pricing for different pizza sizes (Small, Medium, Large). Allowing for revenue calculation by type and size.


## Reports

- **Sales by Date and Time:** A report identifying high-demand hours and days for pizza orders.
- **Revenue by Pizza Size:** Total revenue contribution from Small, Medium, and Large pizzas.
- **Top-Selling Pizza Types:** Analysis of which pizza types are most popular across all orders.
- **Category Popularity:** Breakdown of orders by category (e.g., Chicken, Vegetarian).


## Conclusion

The dataset allows for a clear analysis of sales performance, such as identifying the most profitable pizza size and type.
Peak order times and seasonal trends can be derived to optimize kitchen operations and staffing.
Ingredient analysis from the pizza_types table can help businesses identify customer preferences (e.g., which toppings or combinations are most preferred).
Combining these tables with SQL joins can provide insights into revenue drivers, menu optimization, and marketing strategies.
