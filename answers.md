# SQL Exercise 1 

In this exercise we will use the table `datasets.airbnb_search_details`. 

One row of this table is one search along with one result for that search.

The description of the dataset is in the following table.

| Column Name | Column Type | Has NULL or missing values | Short description |
|---|---|---|---|
|*id*|INTEGER|NO|The primary key for this table.
|*log_price*|NUMERIC|NO|Logarithm of the apartment price?
|*property_type*|TEXT|NO|Type of the property sought, examples: 'Villa', 'Timeshare'.  
|*room_type*|TEXT|NO|Type of room, example: 'Private room'
|*amenities*|TEXT|NO|Amenities of the apartment stored as TEXT but written in a mathematical set notation.
|*accommodates*|INTEGER|NO|Number of people the apartment/room can accommodate.
|*bathrooms*|INTEGER|**YES**|Number of bathrooms in the search.
|*bed_type*|TEXT|NO|Type of the bed, examples: 'Airbed', 'Real Bed'
|*cancellation_policy*|TEXT|NO| Cancellation policy, examples: 'flexible', 'moderate'
|*cleaning_fee*|TEXT|NO|The possible values are TRUE and FALSE which means this column tells us about the existence of a cleaning fee and should be of type BOOL.
|*city*|TEXT|NO|Name of the city either as acronym (SF, LA, NYC) or full name (Chicago, Boston)
|*description*|TEXT|NO|Textual description of the appartment. Can be of any number of characters.
|*first_review*|TEXT|**YES**|Date of the first review stored as TEXT. To use it we need to cast it to DATE first.
|*host_has_profile_pic*|TEXT|NO|A binary value stored as text where t denotes TRUE and f denotes FALSE.
|*host_identity_verified*|TEXT|**YES**|Similar to *host_has_profile_pic*
|*host_response_rate*|TEXT|**YES**|A number in the format 100%, 71%, 83%, etc. Sometimes missing. 
|*host_since*|TEXT|NO|The date of the registration of the host. Is of type text but should be DATE.
|*instant_bookable*|TEXT|NO|A binary value stored as text where t denotes TRUE and f denotes FALSE.
|*last_review*|TEXT|**YES**|The date of last review.
|*latitude*|NUMERIC|NO|The geographical latitude of the apartment.
|*longitude*|NUMERIC|NO|The geographical longitude of the apartment.
|*name*|TEXT|NO|The name of the apartment.
|*neighborhood*|TEXT|NO|The name of the neighborhood where the apartment is located.
|*number_of_reviews*|INTEGER|NO|The number of reviews for this apartment.
|*review_scores_rating*|INTEGER|**YES**|The average rating of this apartment|
|*thumbnail_url*|TEXT|**YES**|The URL for the thumbnail image. Not useful for us.
|*zipcode*|INTEGER|**YES**|Zip code for the municipality where the apartment is located.
|*bedrooms*|INTEGER|**YES**|The number of bedrooms.
|*beds*|INTEGER|**YES**|The total number of beds.


This is a humongous table but fear not, because we will use only small parts of it.

To make life easier, always do a select for all the columns needed in the question, look at the returned values, ponder a bit and then try to write the solution.

## Beginner exercise

1. Find out the searches made by the people who search for apartments where they will be the sole person staying. 

Columns: Use the `beds` and `accomodates` columns.

```sql
SELECT *
FROM datasets.airbnb_search_details
WHERE accommodates = 1 AND beds = 1
```

2. Find the first 50 apartments in New York City which are in the Harlem neighborhood.

Columns: `city` and `neighborhood`

Hint: New York City is present as NYC in this dataset.

```sql
SELECT *
FROM datasets.airbnb_search_details
WHERE city = 'NYC' AND neighbourhood = 'Harlem'
LIMIT 20
```

3. Find all searches where the number of bedrooms is equal to the number of bathrooms.

Columns: `bedrooms` and `bathrooms`.

```sql
SELECT *
FROM datasets.airbnb_search_details
WHERE bedrooms = bathrooms
```

4. Find Los Angeles neighborhoods.

Columns: `city` and `neighbourhood`

```sql
SELECT neighbourhood
FROM datasets.airbnb_search_details
WHERE city = 'LA'
```

## Intermediate exercises

1. Find all houses and villas which have internet access but no wireless internet access.

Columns: `amenities` and `property_type`

Hint: Use the ILIKE, IN and NOT operators.

```sql
SELECT *
FROM datasets.airbnb_search_details
WHERE
     amenities ILIKE '%Internet%' AND
     amenities NOT ILIKE '%Wireless Internet%' AND
     property_type IN ('House', 'Villa')
```

2. Find all searches where the `host_response_rate` column is missing data.

Hint: Not all missing values are NULL. Some can be also empty strings.

```sql
SELECT *
FROM datasets.airbnb_search_details
WHERE
     host_response_rate <> ''
```

3. Find all searches for San Francisco with flexible cancellation policies and which have a review score present. Present them ordered by the review score from highest to lowest.

Columns: `city`, `cancellation_policy` and `review_scores_rating`

Hint: San Francisco is stored as SF in this dataset.

```sql
SELECT *
FROM datasets.airbnb_search_details
WHERE
    city='SF' 
    AND cancellation_policy='flexible' 
    AND review_scores_rating IS NOT NULL
ORDER BY review_scores_rating DESC    
```

4. Find the average number of bathrooms and bedrooms per city per property type.

Columns: `city`, `property_type`, `bathrooms`, `bedrooms`

```sql
SELECT 
    city, property_type,
    AVG(bathrooms) AS avg_bathrooms,
    AVG(bedrooms) AS avg_bedrooms
FROM datasets.airbnb_search_details
GROUP BY city, property_type
```

5. Find the total number of houses which have TVs in the Westlake neighborhood.

Columns: `neighbourhood`, `property_type`, `amenities`

```sql 
SELECT 
   COUNT(*)
FROM datasets.airbnb_search_details
WHERE 
    neighbourhood = 'Westlake' AND 
    property_type = 'House'    AND 
    amenities ILIKE '%TV%'
```

6. It is time for your vacation. You need to chose which city to visit. You don't have much money so you decide to stay in a shared room, but you want to share the room with as little people as possible. You devise a score which tells you the average number of persons accomodated by the shared room over the average number of beds available for each city and order the choices by that score. 

Columns: `city`, `accomodates` and `beds`

```sql
SELECT 
   city,    
   AVG(accommodates) AS avg_accomodates,
   AVG(beds) AS avg_beds,
   AVG(accommodates) / AVG(beds) AS crowdness_ratio
FROM datasets.airbnb_search_details
WHERE 
    room_type='Shared room'
GROUP BY city
ORDER BY crowdness_ratio ASC
```

7. Find the cheapest property in every city based on the `log_price` column.

```sql
SELECT 
   city,
   MIN(log_price) AS min_log_price
FROM datasets.airbnb_search_details
GROUP BY city
```

8. Find all neighbourhoods present in this dataset.

```sql
SELECT DISTINCT
    neighbourhood
FROM datasets.airbnb_search_details
```

9. Find the average number of beds in neighbourhoods in which no property has less than 3 beds.

Columns: `neighbourhood` and `beds`

Hint: Use a HAVING clause.

```sql
SELECT
    neighbourhood,
    AVG(beds) AS avg_beds
FROM datasets.airbnb_search_details
GROUP BY neighbourhood
HAVING MIN(beds) > 3
```

10. To better understand the effects of the number of reviews on the price you decide to bin the reviews into the following groups: NO, FEW, SOME, MANY, A LOT.
The rules you use are:
- 0 reviews is NO
- 1 to 5 is FEW
- 5 to 15 is SOME
- 15 to 40 is MANY
- more than 40 is A LOT

Hint: Use CASE and BETWEEN

Columns: `number_of_reviews`

```sql
SELECT
    CASE 
        WHEN number_of_reviews = 0 THEN 'NO'
        WHEN number_of_reviews BETWEEN 1 AND 5 THEN 'FEW'
        WHEN number_of_reviews BETWEEN 5 AND 15 THEN 'SOME'
        WHEN number_of_reviews BETWEEN 15 AND 40 THEN 'MANY'
        WHEN number_of_reviews > 40 THEN 'A LOT'
    END AS 
    reviews_qualificiation,
    log_price
FROM datasets.airbnb_search_details
```

11. How many hosts are verified by AirBnB staff? How many aren't?

Columns: `host_identity_verified`

Hint: Use GROUP BY

```sql
SELECT
    host_identity_verified,
    COUNT(*)
FROM datasets.airbnb_search_details
GROUP BY host_identity_verified
```

12. Find the price of the most expensive beach properties for each city.

```sql
SELECT
    city,
    MAX(log_price)
FROM datasets.airbnb_search_details
WHERE description ILIKE '%beach%'
GROUP BY city
```

13. Find all neighbourhoods where there are properties which have no cleaning fees and have parking space.

Hint: Use DISTINCT

Columns: `neighbourhood`, `description`, `cleaning_fee`

```sql
SELECT 
    DISTINCT
    neighbourhood
FROM datasets.airbnb_search_details
WHERE description ILIKE '%parking%' AND cleaning_fee = 'FALSE'
```

## Advanced exercises

1. Complete the statistical analysis started in question 10 of the Intermediate exercises. Find the min, avg and max log price per review qualification.

Hint: Use a subquery. Remember: subqueries must be named.

Columns: `number_of_reviews`, `log_price`

```sql
SELECT
    tmp.reviews_qualificiation,
    MIN(log_price) AS min_log_price,
    AVG(log_price) AS average_log_price,
    MAX(log_price) AS max_log_price
FROM
(SELECT
    CASE 
        WHEN number_of_reviews = 0 THEN 'NO'
        WHEN number_of_reviews BETWEEN 1 AND 5 THEN 'FEW'
        WHEN number_of_reviews BETWEEN 5 AND 15 THEN 'SOME'
        WHEN number_of_reviews BETWEEN 15 AND 40 THEN 'MANY'
        WHEN number_of_reviews > 40 THEN 'A LOT'
    END AS 
    reviews_qualificiation,
    log_price
FROM datasets.airbnb_search_details) tmp
GROUP BY tmp.reviews_qualificiation
```

2. Find the search for each city which has the highest number of amenities. Estimate the number of amenities as the number of characters in the `amenities` column.

Columns: `city` and `amenities`

Hint: Use the LENGTH function and an INNER join with the result of the subquery.

```sql
SELECT
    main.*
FROM
    datasets.airbnb_search_details AS main
INNER JOIN
    (SELECT 
        city,
        MAX(LENGTH(amenities)) AS max_amen
    FROM datasets.airbnb_search_details
    GROUP BY city) AS tmp
ON
    main.city = tmp.city AND LENGTH(main.amenities) = tmp.max_amen
```

3. Convert the following two columns to their true types using type casts and if necessary string processing:
- `cleaning_fee` from TEXT to BOOL
- `host_response_rate` from TEXT to NUMERIC. Missing values should be NULL.

Find the average host_response_rate per zip code after type casting. This will tell you which municipality has the most talkative people.

Hint: String processing is necessary for `host_response_rate`.

```sql
SELECT
    zipcode,
    AVG(tmp.clean_host_response_rate) AS avg_host_response_rate
FROM
(SELECT
    zipcode,
    cleaning_fee :: BOOL AS clean_cleaning_fee,
    (CASE 
        WHEN host_response_rate = '' THEN NULL
        ELSE SUBSTR(host_response_rate, 0, POSITION ('%' IN host_response_rate))    
    END) :: NUMERIC AS clean_host_response_rate
FROM 
    datasets.airbnb_search_details) tmp
GROUP BY zipcode
HAVING AVG(tmp.clean_host_response_rate) IS NOT NULL
ORDER BY avg_host_response_rate
```

4. Fix the `host_since` column using string processing to be a valid DATE.

Hint: Use the string [split_part](https://www.postgresql.org/docs/9.1/static/functions-string.html) function. The standard format is **yyyy-mm-dd**. You also need to take care of cases like 1/6/15 which should be 01/06/2018 


```sql
SELECT
    id,
    (tmp.year_fixed || tmp.month_fixed || tmp.day_fixed) :: DATE AS clean_host_since
FROM
(SELECT
    id,
    (CASE 
        WHEN LENGTH(split_part(host_since, '/' :: TEXT, 1)) = 1 
        THEN '0' || split_part(host_since, '/' :: TEXT, 1) 
        ELSE split_part(host_since, '/' :: TEXT, 1) END) AS day_fixed,
        
    (CASE 
        WHEN LENGTH(split_part(host_since, '/' :: TEXT, 2)) = 1 
        THEN '0' || split_part(host_since, '/' :: TEXT, 2) 
        ELSE split_part(host_since, '/' :: TEXT, 2) END) AS month_fixed,    
    
    (CASE
        WHEN split_part(host_since, '/' :: TEXT, 3) :: INTEGER < 2000 
        THEN 2000 + split_part(host_since, '/' :: TEXT, 3) :: INTEGER
        ELSE split_part(host_since, '/' :: TEXT, 3) :: INTEGER
    END) AS year_fixed
FROM datasets.airbnb_search_details
WHERE host_since <> '' AND host_since IS NOT NULL) tmp
```

5. What are the neighbourhoods where you can sleep on a real bed in a villa with a beach access while paying the minimal price possible.

Try to combine the main table and the subquery using IN.

```sql
SELECT
    DISTINCT neighbourhood
FROM 
    datasets.airbnb_search_details
WHERE
    log_price IN (
        SELECT
            MIN(log_price)
        FROM 
            datasets.airbnb_search_details
        WHERE
            bed_type = 'Real Bed' AND 
            property_type = 'Villa' AND
            description ILIKE '%beach%' AND
            -- there is a bug row with log_price = 0
            log_price > 0
    )
```

6. Estimate the growth of AirBnB year over year by looking at the count of hosts registered for each year. This is an estimate because we have searches here not hosts table but we assume that more searches means more hosts and less searches mean less hosts. We also don't have all data and must filter away all NULL or empty `host_since` values.

Hint: Use the cleaned column for year from question 4. Use the LAG function to find percent growth as 100 * ((year_current / year_previous) - 1.0).

Finding the number of hosts registered per year:

```sql
SELECT
    year_fixed AS year,
    COUNT(*) :: NUMERIC AS total_people_registered
FROM
(SELECT
    id,
    (CASE
        WHEN split_part(host_since, '/' :: TEXT, 3) :: INTEGER < 2000 
        THEN 2000 + split_part(host_since, '/' :: TEXT, 3) :: INTEGER
        ELSE split_part(host_since, '/' :: TEXT, 3) :: INTEGER
    END) AS year_fixed
FROM datasets.airbnb_search_details
WHERE host_since <> '' AND host_since IS NOT NULL) tmp
GROUP BY year_fixed
ORDER BY year_fixed
```

Using LAG to estimate growth: 

```sql
SELECT
    year,
    total_people_registered,
    COALESCE(LAG (total_people_registered, 1)
             OVER (ORDER BY year), 
             1) AS prev_total_people_registered,
    
    100 * ((total_people_registered / 
            COALESCE(LAG (total_people_registered, 1) OVER (ORDER BY year), 1)) 
            - 1.0) AS percent_growth
FROM
    (SELECT
        year_fixed AS year,
        COUNT(*) :: NUMERIC AS total_people_registered
    FROM
        (SELECT
            id,
            (CASE
                WHEN split_part(host_since, '/' :: TEXT, 3) :: INTEGER < 2000 
                THEN 2000 + split_part(host_since, '/' :: TEXT, 3) :: INTEGER
                ELSE split_part(host_since, '/' :: TEXT, 3) :: INTEGER
            END) AS year_fixed
        FROM datasets.airbnb_search_details
        WHERE host_since <> '' AND host_since IS NOT NULL) tmp
    GROUP BY year_fixed
    ORDER BY year_fixed) tmp2
```

7. Make a pivot table which shows the number of searches per city and room type. Rows are different cities while columns are different room types:

Hint: Find the room types using SELECT DISTINCT before making the pivot table.

```sql
SELECT
    tmp2.city,
    SUM(tmp2.apt_count) AS apt_count,
    SUM(tmp2.private_count) AS private_count,
    SUM(tmp2.shared_count) AS shared_count
FROM
(SELECT
    DISTINCT
    city,
    (CASE WHEN room_type = 'Entire home/apt' THEN cnt ELSE 0 END) AS apt_count,
    (CASE WHEN room_type = 'Private room' THEN cnt ELSE 0 END) AS private_count,
    (CASE WHEN room_type = 'Shared room' THEN cnt ELSE 0 END) AS shared_count
FROM
    (SELECT
        city,
        room_type,
        COUNT(*) AS cnt
    FROM datasets.airbnb_search_details
    GROUP BY city, room_type) tmp) tmp2
GROUP BY tmp2.city
```

8. Find properties which are close in price and location and choose the one with more amenities. Close in location means that the euclidean distance between their corresponding longitude, latitude pairs is small (e.g. 0.0005) and close in price means that the absolute difference in log price is small (e.g. 0.0001)

Hint: Use a self join.

```sql
SELECT
    air1.id AS id1,
    air2.id AS id2,
    
    LENGTH(air1.amenities) AS score1,
    LENGTH(air2.amenities) AS score2,
    
    (CASE 
        WHEN LENGTH(air1.amenities) > LENGTH(air2.amenities)
        THEN air1.id 
        ELSE air2.id 
     END) AS better_id
FROM 
    datasets.airbnb_search_details air1
CROSS JOIN
    datasets.airbnb_search_details air2
WHERE
    (air1.latitude  - air2.latitude)  * (air1.latitude  - air2.latitude) +
    (air1.longitude - air2.longitude) * (air1.longitude - air2.longitude) <= 0.0005 AND
    ABS(air1.log_price - air2.log_price) < 0.0001
```
 