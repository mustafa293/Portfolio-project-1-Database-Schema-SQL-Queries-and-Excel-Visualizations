# Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations

For this project I was presented with the following scenario:

The client 'Ben' is opening a new pizzeria and would like us to design & build a bespoke relational database for him, he would also like us to query the data in ways which he can use in the future to glean insights and observe key metrics for his business. To end the project off, I will create a simple dashoboard in Excel to wrap everything up. This project will serve primarily to showcase my aptitude with SQL & Excel rather than a thorough analysis of the dataset.

# Part I: DB Schema design with quickDBD 

My approach for the DB design is as follows:
- Design the database by first speccing out all the data fields that we want to collect
- Normalise the data tables
- Define the table relationships including primary & foreign keys

We will also start with some data fields that Ben already knows that he wants to collect, these
are:
- Item name
- Item price
- Quantity
- Customer name
- Delivery address

I also know that we should add some additional data fields, these are:
- An 'order id' field
- Multiple fields in place of the 'delivery address' field to make querying them more
efficient later on
- Food categories & the size of the food (e.g. 'Mains/Desserts' & 'Large/Small')
- We will also need to make create a 'row id' field to be a primary key as one order could
span multiple rows as an order has multiple food items

Here is the revised list of data fields that we need:
- **Row ID**
- **Order ID**
- Item name
- **Item category**
- **Item Size**
- Item Price
- Quantity
- **Customer First name**
- **Customer Last name**
- **Delivery Address 1**
- **Delivery Address 2**
- **Delivery City**
- **Delivery zip code**

*Bold denotes fields added by me after revision

Here is the first design of how our database will looklike within quickDBD:

![image](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/assets/56410464/2e8a3789-e289-4805-b96f-03e8163314c5)

On reflection, there is a lot of redundancy to have the customer and address information on
every instance of order. Normalising the data here would mean we can:
- Reduce redundancy
- Improve efficiency

We can achieve this by replacing the redundant data fields with relevant identifiers and instead
store those pieces of information in a new table

I will make another table for ‘items’, which will allow for greater data flexibility for Ben in
the future. These table will include some further variables:
- Item ID
- SKU

Additionally, Ben wants to be able to know when it’s time to order new stock. I will need the following
variables in order to help with this, including:
- What ingredients go into each pizza
- Quantity of ingredients based on the size of the pizza
- Existing stock level

I will create 2 new tables for this; ingredient & recipe
I will match ‘recipe id’ with the ‘sku’ found in the item table. I will also create a variable
‘ingredient id’ that will sit in both the recipe & ingredient tables

Here is the schema after all the changes denoted above:

![image](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/assets/56410464/687bc725-ce46-42c5-9d20-3568930b0957)

Lastly for this section, Ben wants to know:
- Which staff members are working when
- How much each pizza costs based on staff salary information (ingredients + chefs +
delivery)

We will have to create the following tables:

Staff table
- staff id
- staff first name
- last name
- position
- hourly rate

Shift table:
- shift id
- day of the week
- start time
- end time

rota table:
- row id
- rota id
- date
- shift id
- staff id

Final database schema after these final additions:

![image](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/assets/56410464/3dce6944-aab7-48d4-a43e-23eed9de1855)

Finally I can export this out of quickDBD into MySQL - After accessing my instance of MySQL, i’ve created a database called Pizza DB and uploaded
the aforementioned file to set up the structure. Now I can upload some dummy data into the database and start writing some SQL queries to explore the data.

[Download Dataset Folder](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/blob/main/pizza_proj_new.zip)

# Part II: Writing SQL Queries

In this section, I will run SQL queries using techniques like JOINS, sub-queries, aliases aswell as general functions in order get some interesting results from the dataset.

Some interesting metrics we could visualise later include:
- Total orders
- Total sales
- Total items
- Average order value
- Sales by category
- Orders & sales by hour
- Orders by address
- Orders by delivery/pickup
- Plus much more

We will use various SQL queries to get the right data sets to help us with the aforementioned metrics

## Query 1:
```
SELECT
  orders.order_id,
  item.item_price,
  orders.quantity,
  item.item_cat,
  item.item_name,
  orders.created_at,
  address.delivery_address1,
  address.delivery_address2,
  address.delivery_city,
  address.delivery_zipcode,
  orders.delivery
FROM orders
LEFT JOIN item on orders.item_id = item.item_id
LEFT JOIN address on orders.add_id = address.add_id;
```
This resulted in the following table: [View Table Data](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/blob/main/query1.csv)

## Query 2:
```
SELECT 
	s1.item_name,
	s1.ing_id,
	s1.ing_name,
	s1.ing_weight,
	s1.ing_price,
	s1.order_quantity,
	s1.recipe_quantity,
	s1.order_quantity*s1.recipe_quantity as ordered_weight,
	s1.ing_price*s1.ing_weight as unit_cost,
	(s1.order_quantity*s1.recipe_quantity)*(s1.ing_price/s1.ing_weight) as ingredient_cost
FROM (SELECT
	orders.item_id,
	item.sku,
	item.item_name,
	recipe.ing_id,
	ingredient.ing_name,
	recipe.quantity as recipe_quantity,
sum(orders.quantity) as order_quantity,
	ingredient.ing_weight,
	ingredient.ing_price
from 
	orders
	left join item on orders.item_id=item.item_id
	left join recipe on item.sku=recipe.recipe_id
	left join ingredient on ingredient.ing_id=recipe.ing_id
group by 
	orders.item_id,
	item.sku,
	item_name,
	recipe.ing_id,
	recipe.quantity,
	ingredient.ing_name,
	ingredient.ing_weight,
	ingredient.ing_price) s1
```
Table result: [View Table Data](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/blob/main/query2.csv)

## Query 3:
```
SELECT
	s2.ing_name,
	s2.ordered_weight,
	ingredient.ing_weight*inventory.quantity as total_inv_weight,
	(ingredient.ing_weight*inventory.quantity)-s2.ordered_weight as remaining_weight
FROM
(SELECT 
	ing_id,
	ing_name,
SUM(ordered_weight) as ordered_weight 
FROM 
	stock1
GROUP BY ing_name, ing_id) s2

LEFT JOIN inventory ON inventory.item_id=s2.ing_id
LEFT JOIN ingredient ON ingredient.ing_id=s2.ing_id
```
Table result: [View Table Data](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/blob/main/query3.csv)

## Query 4:
```
SELECT
	rota.date,
	staff.first_name,
	staff.last_name,
	staff.hourly_rate,
	shift.start_time,
	shift.end_time,
	((hour(timediff(shift.end_time, shift.start_time))*60) + (minute(timediff(shift.end_time,shift.start_time))))/60 as hours_in_shift,
	((hour(timediff(shift.end_time, shift.start_time))*60) + (minute(timediff(shift.end_time,shift.start_time))))/60 * staff.hourly_rate as staff_cost
FROM
	rota
LEFT JOIN staff on staff.staff_id=rota.staff_id
LEFT JOIN shift on shift.shift_id=rota.shift_id
```
Table result: [View Table Data](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/blob/main/query4.csv)

# Part III: Excel Dashboard

In this final section, I will be using query 1 from the previous section to build out a simple 'orders dashborad' for Ben's pizzeria within Excel.

First I saved the table result of query1 as a CSV and uploaded it into Excel.

I then did a quick checkup of the data, making sure that each data type was represented correctly and then formatted the raw data as a table.

Before moving onto pivot tables & charts, I included one more column named 'sales value' which is the multiplication of item price & quantity - as I forgot to do this within SQL.

![image](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/assets/56410464/01076162-05f9-437f-9071-6220170eb690)

Using the table - I created pivot tables in order to make some basic visualisations, mainly using my newly created 'sales value' field. I then made the following charts from the pivot tables:
- A donut chart showing the % of sales between the different item categories
- A bar chart showing the total sales value per individual food item
- A line chart showing the amount or orders and the sum of sales per hour

Then I done some formatting & included some other KPIs such as: Total orders, Total Sales, Total Items and Avergae Order Value. Lastly, I included a slicer for the line chart so that stakeholders could visualise how indidual food items were doing in terms of orders & sales.

Here is the final dashboard to cap off this project:

![image](https://github.com/mustafa293/Portfolio-project-1-Database-Schema-SQL-Queries-and-Excel-Visualizations/assets/56410464/dfdd3d55-af38-463f-aaf3-b68ae7a160c7)



