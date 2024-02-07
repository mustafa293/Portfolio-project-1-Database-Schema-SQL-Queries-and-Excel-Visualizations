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

# Part II: Writing SQL Queries
