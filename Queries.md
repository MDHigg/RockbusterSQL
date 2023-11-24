## **Exemplars of Queries Used**

### Initial Analysis Statistics
#### Query to find the key Quantative overview data
```
SELECT
-- Finding min max and average of the main quantative variables
	MIN(rental_duration) AS min_rental_duration,
	MAX(rental_duration) AS max_rental_duration,
	AVG(rental_duration) AS avg_rental_duration,
	MIN(rental_rate) AS min_rental_rate,
	MAX(rental_rate) AS max_rental_rate,
	AVG(rental_rate) AS avg_rental_rate,
	MIN(length) AS min_length,
	MAX(length) AS max_length,
	AVG(length) AS avg_length
FROM film
```

### Using Joins
#### Script to find various income statitics by country:
```
SELECT 	D.country,
		-- number of customers
		count(DISTINCT A.customer_id) AS customers,
		-- total rentals made
		count(E.rental_id) AS rental_count,
		-- average number of rental per customer
		round((cast(count(E.rental_id) as DECIMAL))/count(DISTINCT A.customer_id),2) AS rentals_per_customer,
		-- average spent on each rental
		round(avg(G.rental_rate),2) AS avg_rental_rate,
		-- Total (net) income
		SUM(H.amount) AS total_income,
		-- Average spent by each customer
		round(SUM(H.amount)/count(DISTINCT A.customer_id),2) AS income_per_customer

-- Joins link the customer (table A) to their country (D) via Address (B) and city (C).
FROM customer A
INNER JOIN address B ON A.address_id = B.address_id
INNER JOIN city C ON B.city_id = C.city_id
INNER JOIN country D on C.country_id = D.country_id
-- Joins link the customer (A) to the film (G) made via rentals (E) and inventory (F))
INNER JOIN rental E ON A.customer_id = E.customer_id
INNER JOIN inventory F ON E.inventory_id = F.inventory_id
INNER JOIN film G ON F.film_id = G.film_id
-- Joins link the customer (A) to the payments (H)
INNER JOIN payment H ON E.rental_id = H.rental_id

-- Output by country, ordered by decreasing net income
GROUP BY D.country
ORDER BY total_income DESC
```

###Subquery example:
#### Script to find the top five highest spending customers in each of the five highest spending countries:
```
SELECT * FROM (
-- Start of sub-query
-- Finding the customers total spend
SELECT	A.customer_id,
		A.first_name AS first_name,
		A.last_name AS last_name,
		D.country,
		SUM(E.amount) AS total_paid,
-- the following ranks the customer's total payments made with others in the same country
		row_number() OVER(PARTITION BY D.country ORDER BY SUM(E.amount) DESC) AS country_rank

-- Join the customer information (Table A) to the payment information (E)
FROM customer A
INNER JOIN address B ON A.address_id = B.address_id
INNER JOIN city C ON B.city_id = C.city_id
INNER JOIN country D ON C.country_id = D.country_id
INNER JOIN payment E ON A.customer_id = E.customer_id

-- Clause for selection to top five countries
WHERE D.country IN(
-- Join the customer (Table A) to the country (D) , via address (B), city (C)
	SELECT D.country
	FROM customer A
	INNER JOIN address B ON A.address_id = B.address_id
	INNER JOIN city C ON B.city_id = C.city_id
	INNER JOIN country D ON C.country_id = D.country_id
-- Join the customer (A) to the payment (E)
	INNER JOIN payment E ON A.customer_id = E.customer_id

-- Order the coutries by total income generated and output the top 5
	GROUP BY D.country
	ORDER BY SUM(E.amount) DESC
	LIMIT 5
	)

-- Output the customers from the selected countries
-- Limited to those that are ranked in the top five for spending in each country
-- Order the customers by country (alphabetical), then by decreasing customer spending	
GROUP BY	D.country, A.customer_id, A.first_name, A.last_name
ORDER BY 	D.country, SUM(E.amount) DESC) ranks
WHERE country_rank <= 5
```
