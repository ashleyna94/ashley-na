  ```sql
  USE sakila;
  SELECT * FROM actor;
  ```

* 1a. Display the first and last names of all actors from the table `actor`. 

  ```sql
  SELECT first_name, last_name FROM actor;
  ```

* 1b. Display the first and last name of each actor in a single column in upper case letters. Name the column `Actor Name`. 

  ```sql
  SELECT UPPER(CONCAT(first_name, " ", last_name)) AS "Actor Name" FROM actor;
  ```

* 2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?

  ```sql
  SELECT actor_id, first_name, last_name FROM actor 
  WHERE first_name = "Joe"; 
  ```

* 2b. Find all actors whose last name contain the letters `GEN`:

  ```sql
  SELECT * FROM actor
  WHERE last_name LIKE "%GEN%";
  ```

* 2c. Find all actors whose last names contain the letters `LI`. This time, order the rows by last name and first name, in that order:

  ```sql
  SELECT last_name, first_name FROM actor
  WHERE last_name LIKE "%LI%"
  ORDER BY last_name, first_name; 
  ```

* 2d. Using `IN`, display the `country_id` and `country` columns of the following countries: Afghanistan, Bangladesh, and China:

  ```sql
  SELECT country_id, country
  FROM country
  WHERE country = "Afghanistan" 
  OR country = "Bangladesh" 
  OR country = "China";
  ```

* 3a. Add a `middle_name` column to the table `actor`. Position it between `first_name` and `last_name`. Hint: you will need to specify the data type.

  ```sql
  ALTER TABLE actor
  ADD COLUMN middle_name varchar(30) AFTER first_name;
  ```

* 3b. You realize that some of these actors have tremendously long last names. Change the data type of the `middle_name` column to `blobs`.

  ```sql
  ALTER TABLE actor
  MODIFY COLUMN middle_name blob;
  ```

* 3c. Now delete the `middle_name` column.

  ```sql
  ALTER TABLE actor
  DROP COLUMN middle_name;
  ```

* 4a. List the last names of actors, as well as how many actors have that last name.

  ```sql
  SELECT last_name, COUNT(last_name) AS "count"
  FROM actor
  GROUP BY last_name;
  ```

* 4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors

  ```sql
  SELECT last_name, COUNT(last_name) AS "count"
  FROM actor
  GROUP BY last_name 
  HAVING "count" >= 2;
  ```

* 4c. Oh, no! The actor `HARPO WILLIAMS` was accidentally entered in the `actor` table as `GROUCHO WILLIAMS`, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.

  ```sql
  UPDATE actor
  SET first_name = "HARPO"
  WHERE actor_id = 172;
  ```

* 4d. Perhaps we were too hasty in changing `GROUCHO` to `HARPO`. It turns out that `GROUCHO` was the correct name after all! In a single query, if the first name of the actor is currently `HARPO`, change it to `GROUCHO`. Otherwise, change the first name to `MUCHO GROUCHO`, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO `MUCHO GROUCHO`, HOWEVER! (Hint: update the record using a unique identifier.)

  ```sql
  UPDATE actor
  SET first_name =
  CASE WHEN first_name = 'HARPO'
    THEN 'GROUCHO'
    ELSE 'MUCHO GROUCHO'
  END
  WHERE actor_id = 172;
  ```

* 5a. You cannot locate the schema of the `address` table. Which query would you use to re-create it?

  ```sql
  SHOW CREATE TABLE address;
  ```

* 6a. Use `JOIN` to display the first and last names, as well as the address, of each staff member. Use the tables `staff` and `address`:

  ```sql
  -- SELECT * FROM staff;
  -- SELECT * FROM address;
  SELECT staff.first_name, staff.last_name, address.address
  FROM staff
  INNER JOIN address ON staff.address_id = address.address_id;
  ```

* 6b. Use `JOIN` to display the total amount rung up by each staff member in August of 2005. Use tables `staff` and `payment`. 

  ```sql
  -- SELECT * FROM staff;
  -- SELECT * FROM payment;
  SELECT staff.first_name,staff.last_name, SUM(payment.amount) AS "total_amount"
  FROM staff
  INNER JOIN payment ON staff.staff_id = payment.staff_id
  WHERE payment.payment_date LIKE "2005-08%"
  GROUP BY staff.first_name, staff.last_name;
  ```


* 6c. List each film and the number of actors who are listed for that film. Use tables `film_actor` and `film`. Use inner join.

  ```sql
  -- SELECT * FROM film_actor;
  -- SELECT * FROM film;
  SELECT title, COUNT(actor_id) AS "actor_count"
  FROM film
  INNER JOIN film_actor ON film.film_id = film_actor.film_id
  GROUP BY title;
  ```

* 6d. How many copies of the film `Hunchback Impossible` exist in the inventory system?

  ```sql
  -- SELECT * FROM inventory;
  -- SELECT * FROM film;
  SELECT COUNT(film_id) AS "count"
  FROM inventory 
  WHERE film_id IN
  (
  SELECT film_id
  FROM film
  WHERE title = "Hunchback Impossible"
  );
  ```

* 6e. Using the tables `payment` and `customer` and the `JOIN` command, list the total paid by each customer. List the customers alphabetically by last name:

  ```sql
  -- SELECT * FROM payment;
  -- SELECT * FROM customer;
  SELECT first_name, last_name, SUM(amount) AS "total_amount"
  FROM payment
  INNER JOIN customer ON payment.customer_id = customer.customer_id
  GROUP BY first_name, last_name
  ORDER BY 
    CASE WHEN last_name IS NULL THEN 1 ELSE 0 END, last_name;
  ```

* 7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters `K` and `Q` have also soared in popularity. Use subqueries to display the titles of movies starting with the letters `K` and `Q` whose language is English. 

  ```sql
  -- SELECT * FROM film;
  -- SELECT * FROM language;
  SELECT title
  FROM film
  WHERE title LIKE "K%"
  OR title LIKE "Q%"
  AND language_id IN 
  (
  SELECT language_id
  FROM language
  WHERE name = "English"
  );
  ```

* 7b. Use subqueries to display all actors who appear in the film `Alone Trip`.

  ```sql
  -- SELECT * FROM film;
  -- SELECT * FROM actor;
  -- SELECT * FROM film_actor;
  SELECT first_name, last_name
  FROM actor
  WHERE actor_id IN
  (
  SELECT actor_id
  FROM film_actor
  WHERE film_id IN
  (
  SELECT film_id
  FROM film
  WHERE title = "Alone Trip"
  )
  );
  ```

* 7c. You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.

  ```sql
  -- SELECT * FROM customer;
  -- SELECT * FROM address;
  -- SELECT * FROM city;
  -- SELECT * FROM country;
  SELECT first_name, last_name, email
  FROM customer
  JOIN address
  ON (customer.address_id = address.address_id)
  JOIN city
  ON (city.city_id = address.city_id)
  JOIN country
  ON (country.country_id = city.country_id)
  WHERE country = "Canada";
  ```

* 7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as _family_ films.

  ```sql
  -- SELECT * FROM category;
  -- SELECT * FROM film_category;
  -- SELECT * FROM film;
  SELECT title
  FROM film
  WHERE film_id IN
  (
  SELECT film_id
  FROM film_category
  WHERE category_id IN
  (
  SELECT category_id
  FROM category
  WHERE name = "Family"
  )
  );
  ```

* 7e. Display the most frequently rented movies in descending order.

  ```sql
  -- SELECT * FROM rental;
  -- SELECT * FROM inventory;
  -- SELECT * FROM film;
  SELECT title, COUNT(title) as "count"
  FROM film
  JOIN inventory
  ON (film.film_id = inventory.film_id)
  JOIN rental
  ON (inventory.inventory_id = rental.inventory_id)
  GROUP BY title
  ORDER BY count DESC;
  ```

* 7f. Write a query to display how much business, in dollars, each store brought in.

  ```sql
  -- SELECT * FROM store;
  -- SELECT * FROM payment;
  SELECT store_id, SUM(amount) as "total_sum"
  FROM payment
  JOIN rental 
  ON (payment.rental_id = rental.rental_id)
  JOIN inventory 
  ON (rental.inventory_id = inventory.inventory_id)
  JOIN store
  ON (inventory.store_id = store.store_id)
  GROUP BY store.store_id;
  ```


* 7g. Write a query to display for each store its store ID, city, and country.

  ```sql
  -- SELECT * FROM store;
  -- SELECT * FROM address;
  -- SELECT * FROM city;
  -- SELECT * FROM country;
  SELECT store.store_id, city.city, country.country
  FROM city
  JOIN country
  ON (city.country_id = country.country_id)
  JOIN address
  ON (address.city_id = city.city_id)
  JOIN store
  ON (store.address_id = address.address_id)
  WHERE store_id = 1
  OR store_id = 2;
  ```

* 7h. List the top five genres in gross revenue in descending order. (**Hint**: you may need to use the following tables: category, film_category, inventory, payment, and rental.)

  ```sql
  -- SELECT * FROM category;
  -- SELECT * FROM film_category;
  -- SELECT * FROM inventory;
  -- SELECT * FROM rental;
  -- SELECT * FROM payment;
  SELECT category.name, SUM(amount) as "gross_revenue"
  FROM payment
  JOIN rental
  ON (rental.rental_id = payment.rental_id)
  JOIN inventory
  ON (inventory.inventory_id = rental.inventory_id)
  JOIN film_category
  ON (film_category.film_id = inventory.film_id)
  JOIN category
  ON (category.category_id = film_category.category_id)
  GROUP BY category.name
  ORDER BY gross_revenue DESC LIMIT 5;
  ```

* 8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.

  ```sql
  CREATE VIEW top_five_genres AS 
  SELECT category.name, SUM(amount) as "gross_revenue"
  FROM payment
  JOIN rental
  ON (rental.rental_id = payment.rental_id)
  JOIN inventory
  ON (inventory.inventory_id = rental.inventory_id)
  JOIN film_category
  ON (film_category.film_id = inventory.film_id)
  JOIN category
  ON (category.category_id = film_category.category_id)
  GROUP BY category.name
  ORDER BY gross_revenue DESC LIMIT 5;
  ```

* 8b. How would you display the view that you created in 8a?

  ```sql
  SELECT * FROM top_five_genres;
  ```

* 8c. You find that you no longer need the view `top_five_genres`. Write a query to delete it.

  ```sql
  DROP VIEW top_five_genres;
  ```
