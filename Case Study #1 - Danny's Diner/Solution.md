


# [Case Study #1: Danny's Diner](https://8weeksqlchallenge.com/case-study-1/) 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

***

### 1. What is the total amount each customer spent at the restaurant?

    SELECT
      s.customer_id,
      SUM(price) AS total_sales
    FROM dannys_diner.sales AS s
      JOIN dannys_diner.menu AS me
      ON s.product_id = me.product_id
    GROUP BY s.customer_id
    ORDER BY s.customer_id;
####  Answer

    | customer_id | total_sales |
    | ----------- | ----------- |
    | A           | 76          |
    | B           | 74          |
    | C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.
***

### 2. How many days has each customer visited the restaurant?

    SELECT
      customer_id,
      COUNT(DISTINCT order_date) AS visit_count 
    FROM dannys_diner.sales AS s
    GROUP BY customer_id;
   #### Answer
  

    | customer_id | visit_count |
    | ----------- | ----------- |
    | A           | 4           |
    | B           | 6           |
    | C           | 2           |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.
***

### 3. What was the first item from the menu purchased by each customer?

     WITH sale_temp AS (
       SELECT customer_id, product_id,
        row_number() OVER (PARTITION BY customer_id) AS first_order
      FROM dannys_diner.sales
      )
      
    SELECT s.customer_id, 
    	me.product_name 
      FROM sale_temp s  
        INNER JOIN dannys_diner.menu me
      ON s.product_id = me.product_id
        WHERE s.first_order = 1;
    
  
  #### Answer   
 

    | customer_id | product_name |
    |-------------|--------------|
    |           A |        sushi |
    |           B |        curry |
    |           C |        ramen |
 - Customer A's first orders is sushi.
- Customer B's first order is curry.
- Customer C's first order is ramen.
***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

    SELECT 
    	me.product_name AS most_purchased, 
    	count(*) AS No_of_times_ordered
    FROM dannys_diner.sales AS s
    	JOIN dannys_diner.menu AS me
     	ON s.product_id = me.product_id
      GROUP BY s.product_id, product_name
      ORDER BY No_of_times_ordered DESC
      limit 1;
   #### Answer
   

    | most_purchased | no_of_times_ordered |
    |----------------|---------------------|
    |          ramen |                   8 |
   
- Most purchased item on the menu is ramen which is 8 times.

***

### 5. Which item was the most popular for each customer?

    WITH temp AS (
        SELECT customer_id, product_id,
        	COUNT(product_id) AS order_count,
        	RANK() OVER (PARTITION BY customer_id order by count(customer_id) desc) AS rank
        FROM dannys_diner.sales
        GROUP BY customer_id, product_id 
      )
      
      SELECT customer_id,
      	me.product_name, 
        order_count
       FROM temp JOIN dannys_diner.menu me
       ON temp.product_id = me.product_id
       WHERE rank = 1
       ORDER BY 1;
#### Answer

    | customer_id | product_name | order_count |
    |-------------|--------------|-------------|
    |           A |        ramen |           3 |
    |           B |        sushi |           2 |
    |           B |        curry |           2 |
    |           B |        ramen |           2 |
    |           C |        ramen |           3 |
- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu, equally. 
 ***

### 6. Which item was purchased first by the customer after they became a member?

    WITH temp AS 
    (
     SELECT s.customer_id, 
      m.join_date, s.order_date,   
      s.product_id,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
         FROM dannys_diner.sales AS s
     JOIN dannys_diner.members AS m
      ON s.customer_id = m.customer_id
     WHERE s.order_date >= m.join_date
    )
    
    SELECT s.customer_id, s.order_date, me.product_name 
    FROM temp AS s
    JOIN dannys_diner.menu AS me
     ON s.product_id = me.product_id
     WHERE rank = 1;
 
#### Answer

     | customer_id | order_date | product_name |
     |-------------|------------|--------------|
     |           B | 2021-01-11 |        sushi |
     |           A | 2021-01-07 |        curry |
- Customer A's first order as member is curry.
- Customer B's first order as member is sushi.
 ***

### 7. Which item was purchased just before the customer became a member?

    WITH temp AS 
    (
     SELECT s.customer_id,
        m.join_date, 
        s.order_date, 
        s.product_id,
        DENSE_RANK() OVER(PARTITION BY s.customer_id
        ORDER BY s.order_date DESC) AS rank
        FROM dannys_diner.sales AS s
        JOIN dannys_diner.members AS m
          ON s.customer_id = m.customer_id
        WHERE s.order_date < m.join_date
    )
    
    SELECT s.customer_id, 
    	s.order_date, 
        me.product_name 
    FROM temp AS s
    	JOIN dannys_diner.menu AS me
     	ON s.product_id = me.product_id
      WHERE rank = 1;
   #### Answer
  

    | customer_id | order_date | product_name |
    |-------------|------------|--------------|
    |           B | 2021-01-04 |        sushi |
    |           A | 2021-01-01 |        sushi |
    |           A | 2021-01-01 |        curry |
- Before becoming a member, Customer A’s orders sushi and curry.
- For Customer B, it's sushi.
***

### 8. What is the total items and amount spent for each member before they became a member?

    SELECT
        s.customer_id,
        COUNT(DISTINCT s.product_id) AS No_of_menu_items_tried,
        SUM(me.price) AS Amount_spent
      FROM dannys_diner.sales s
      INNER JOIN dannys_diner.members m
     	 ON s.customer_id = m.customer_id
      INNER JOIN dannys_diner.menu me
         ON s.product_id = me.product_id
      WHERE s.order_date < m.join_date
      GROUP BY s.customer_id;
#### Answer

    | customer_id | no_of_menu_items_tried | amount_spent |
    |-------------|------------------------|--------------|
    |           A |                      2 |           25 |
    |           B |                      2 |           40 |
Before becoming members,
- Customer A spent $ 25 on 2 items.
- Customer B spent $40 on 2 items.
***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

    SELECT
      s.customer_id,
      SUM(  CASE
        WHEN s.product_id = '1' THEN 20 * me.price
        ELSE 10 * me.price
        END) AS total_points
    FROM dannys_diner.sales AS s
    JOIN dannys_diner.menu AS me
      ON s.product_id = me.product_id
    LEFT JOIN dannys_diner.members m
      ON s.customer_id = m.customer_id
    GROUP BY s.customer_id
    ORDER BY s.customer_id;
#### Answer

    | customer_id | total_points       |
    |-------------|--------------------|
    |           A |                860 |
    |           B |                940 |
    |           C |                360 |
- Total points for Customer A is 860.
- Total points for Customer B is 940.
- Total points for Customer C is 360.

***
### 10. 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

    SELECT
      s.customer_id,
      SUM(  CASE
        WHEN s.product_id = '1' OR
          	(s.order_date - m.join_date < 7 AND
          	 s.order_date - m.join_date > -1) THEN 20 * me.price
        ELSE 10 * me.price
        END)  AS total_points
      FROM dannys_diner.sales AS s
      LEFT JOIN dannys_diner.menu AS me
     	 ON s.product_id = me.product_id
      INNER JOIN dannys_diner.members m
         ON s.customer_id = m.customer_id
      GROUP BY s.customer_id
      ORDER BY s.customer_id;
#### Answer

    | customer_id | total_points       |
    |-------------|--------------------|
    |           A |               1370 |
    |           B |                940 |
- Total points for Customer A is 1,370.
- Total points for Customer B is 940.
***

## BONUS QUESTIONS

### Join All The Things - Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)

    SELECT
      s.customer_id,
      s.order_date,
      me.product_name,
      me.price,
      CASE  WHEN s.order_date >= m.join_date THEN 'Y'
            ELSE 'N'
            END AS member
      FROM dannys_diner.sales s
    	LEFT JOIN dannys_diner.menu me
     	  ON s.product_id = me.product_id
      LEFT JOIN dannys_diner.members m
          ON s.customer_id = m.customer_id
      ORDER BY s.customer_id,s.order_date;
#### Answer

    | customer_id | order_date | product_name | price | member |
    |-------------|------------|--------------|-------|--------|
    |           A | 2021-01-01 |        sushi |    10 |      N |
    |           A | 2021-01-01 |        curry |    15 |      N |
    |           A | 2021-01-07 |        curry |    15 |      Y |
    |           A | 2021-01-10 |        ramen |    12 |      Y |
    |           A | 2021-01-11 |        ramen |    12 |      Y |
    |           A | 2021-01-11 |        ramen |    12 |      Y |
    |           B | 2021-01-01 |        curry |    15 |      N |
    |           B | 2021-01-02 |        curry |    15 |      N |
    |           B | 2021-01-04 |        sushi |    10 |      N |
    |           B | 2021-01-11 |        sushi |    10 |      Y |
    |           B | 2021-01-16 |        ramen |    12 |      Y |
    |           B | 2021-02-01 |        ramen |    12 |      Y |
    |           C | 2021-01-01 |        ramen |    12 |      N |
    |           C | 2021-01-01 |        ramen |    12 |      N |
    |           C | 2021-01-07 |        ramen |    12 |      N |
***

### Rank All The Things - Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

    WITH temp AS  (
      SELECT
     	s.customer_id,
      	s.order_date,
      	me.product_name,
     	 me.price,
     	 CASE  WHEN s.order_date >= m.join_date THEN 'Y'
         ELSE 'N'
         END AS member
       FROM dannys_diner.sales s
       LEFT JOIN dannys_diner.menu me
          ON s.product_id = me.product_id
       LEFT JOIN dannys_diner.members m
          ON s.customer_id = m.customer_id
       ORDER BY s.customer_id,s.order_date)
       
    SELECT  *,
      CASE WHEN member = 'Y' THEN RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
        ELSE NULL  END AS RANKING
    FROM temp;
   #### Answer
   

    | customer_id | order_date | product_name | price | member | ranking |
    |-------------|------------|--------------|-------|--------|---------|
    |           A | 2021-01-01 |        sushi |    10 |      N |  (null) |
    |           A | 2021-01-01 |        curry |    15 |      N |  (null) |
    |           A | 2021-01-07 |        curry |    15 |      Y |       1 |
    |           A | 2021-01-10 |        ramen |    12 |      Y |       2 |
    |           A | 2021-01-11 |        ramen |    12 |      Y |       3 |
    |           A | 2021-01-11 |        ramen |    12 |      Y |       3 |
    |           B | 2021-01-01 |        curry |    15 |      N |  (null) |
    |           B | 2021-01-02 |        curry |    15 |      N |  (null) |
    |           B | 2021-01-04 |        sushi |    10 |      N |  (null) |
    |           B | 2021-01-11 |        sushi |    10 |      Y |       1 |
    |           B | 2021-01-16 |        ramen |    12 |      Y |       2 |
    |           B | 2021-02-01 |        ramen |    12 |      Y |       3 |
    |           C | 2021-01-01 |        ramen |    12 |      N |  (null) |
    |           C | 2021-01-01 |        ramen |    12 |      N |  (null) |
    |           C | 2021-01-07 |        ramen |    12 |      N |  (null) |

***
