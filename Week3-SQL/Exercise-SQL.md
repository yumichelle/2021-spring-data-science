
# SQL:  Structured Query Language  Exercise

### Getting Started
1. Go to BigQuery UI https://console.cloud.google.com/bigquery
2. Add in the public data sets. 
	3. Click the Add Data icon
	4. Add any dataset
	5. `bigquery-public-data` should become visible and populate in the BigQuery UI. 
3. Add your queries where it says [YOUR QUERY HERE].
4. Make sure you add your query in between the triple tick marks. 
---

For this section of the exercise we will be using the `bigquery-public-data.austin_311.311_service_requests`  table. 

5. Write a query that tells us how many rows are in the table. 
	```
	select count(*)
    from bigquery-public-data.austin_311.311_service_requests
	```

7. Write a query that tells us how many _distinct_ values there are in the complaint_description column.
	``` 
	select count(distinct complaint_description)
    from bigquery-public-data.austin_311.311_service_requests
	```
  
8. Write a query that counts how many times each owning_department appears in the table and orders them from highest to lowest. 
	``` 
	select owning_department, count(owning_department) as counter
    from bigquery-public-data.austin_311.311_service_requests
    group by owning_department
    order by count(owning_department) desc 
	```

9. Write a query that lists the top 5 complaint_description that appear most and the amount of times they appear in this table. (hint... limit)
	```
	select complaint_description, count(complaint_description) as counter
    from bigquery-public-data.austin_311.311_service_requests
    group by complaint_description
    order by count(complaint_description) desc 
    limit 5
	  ```
10. Write a query that lists and counts all the complaint_description, just for the where the owning_department is 'Animal Services Office'.
	```
	select complaint_description, count(complaint_description) as counter
    from bigquery-public-data.austin_311.311_service_requests
    where owning_department = 'Animal Services Office'
    group by complaint_description
	```

11. Write a query to check if there are any duplicate values in the unique_key column (hint.. There are two was to do this, one is to use a temporary table for the groupby, then filter for values that have more than one count, or, using just one table but including the  `having` function). 
	```
	select unique_key, count(unique_key) as counter
    from bigquery-public-data.austin_311.311_service_requests
    group by unique_key
    having count(unique_key)>1
	```
    ```    
    with temptbl as (
        select unique_key, count(unique_key) as counter
        from bigquery-public-data.austin_311.311_service_requests
        group by unique_key
    )
    select *
    from temptbl
    where counter>1
	```


### For the next question, use the `census_bureau_usa` tables.

1. Write a query that returns each zipcode and their population for 2000 and 2010. 
	```
	SELECT population_by_zip_2000.zipcode as `zip2000`, population_by_zip_2010.zipcode as `zip2010`, population_by_zip_2000.population as `pop2000`,population_by_zip_2010.population as `pop2010`
    FROM `bigquery-public-data.census_bureau_usa.population_by_zip_2000` as population_by_zip_2000
    inner join `bigquery-public-data.census_bureau_usa.population_by_zip_2010` as population_by_zip_2010
    on population_by_zip_2000.zipcode = population_by_zip_2010.zipcode
	```

### For the next section, use the  `bigquery-public-data.google_political_ads.advertiser_weekly_spend` table.
1. Using the `advertiser_weekly_spend` table, write a query that finds the advertiser_name that spent the most in usd. 
	```
	select advertiser_name, sum(spend_usd) as usdsum
    from bigquery-public-data.google_political_ads.advertiser_weekly_spend
    group by advertiser_name
    order by sum(spend_usd) desc
    limit 1
	```
2. Who was the 6th highest spender? (No need to insert query here, just type in the answer.)
	```
	SENATE LEADERSHIP FUND
	```

3. What week_start_date had the highest spend? (No need to insert query here, just type in the answer.)
	```
	2020-10-18
	```

4. Using the `advertiser_weekly_spend` table, write a query that returns the sum of spend by week (using week_start_date) in usd for the month of August only. 
	```
	select sum(spend_usd) as usdsum, week_start_date
    from bigquery-public-data.google_political_ads.advertiser_weekly_spend
    where extract (month from week_start_date) = 8
    group by week_start_date
    order by week_start_date
	```
6.  How many ads did the 'TOM STEYER 2020' campaign run? (No need to insert query here, just type in the answer.)
	```
	50
	```
7. Write a query that has, in the US region only, the total spend in usd for each advertiser_name and how many ads they ran. (Hint, you're going to have to join tables for this one). 
	```
	with tempspent as (
        select advertiser_name, sum(spend_usd) as usdsum
        from bigquery-public-data.google_political_ads.advertiser_weekly_spend
        where election_cycle like 'US%'
        group by advertiser_name
    ),
    tempadcount as (
        select advertiser_name, count(spend_usd) as counter
        from bigquery-public-data.google_political_ads.advertiser_weekly_spend
        where election_cycle like 'US%'
        group by advertiser_name
    )
    select tempspent.advertiser_name, tempspent.usdsum, tempadcount.counter 
    from tempspent inner join tempadcount
        on tempspent.advertiser_name = tempadcount.advertiser_name
	```
8. For each advertiser_name, find the average spend per ad. 
	```
	with tempspent as (
        select advertiser_name, sum(spend_usd) as usdsum
        from bigquery-public-data.google_political_ads.advertiser_weekly_spend
        where election_cycle like 'US%'
        group by advertiser_name
    ),
    tempadcount as (
        select advertiser_name, count(spend_usd) as counter
        from bigquery-public-data.google_political_ads.advertiser_weekly_spend
        where election_cycle like 'US%'
        group by advertiser_name
    )
    select tempspent.advertiser_name,tempspent.usdsum,tempadcount.counter, tempspent.usdsum/tempadcount.counter as avgspend
    from tempspent inner join tempadcount
        on tempspent.advertiser_name = tempadcount.advertiser_name
	```
10. Which advertiser_name had the lowest average spend per ad that was at least above 0. 
	``` 
	with tempspent as (
        select advertiser_name, sum(spend_usd) as usdsum
        from bigquery-public-data.google_political_ads.advertiser_weekly_spend
        where election_cycle like 'US%'
        group by advertiser_name
    ),
    tempadcount as (
        select advertiser_name, count(spend_usd) as counter
        from bigquery-public-data.google_political_ads.advertiser_weekly_spend
        where election_cycle like 'US%'
        group by advertiser_name
    )
    select tempspent.advertiser_name,tempspent.usdsum,tempadcount.counter, tempspent.usdsum/tempadcount.counter as avgspend
    from tempspent inner join tempadcount
        on tempspent.advertiser_name = tempadcount.advertiser_name
    where  tempspent.usdsum/tempadcount.counter != 0
    order by avgspend
    limit 1
	```
## For this next section, use the `new_york_citibike` datasets.

1. Who went on more bike trips, Males or Females?
	```
	SELECT gender, count(gender)
    FROM `bigquery-public-data.new_york_citibike.citibike_trips`
    where gender = 'female' or gender = 'male'
    group by gender
	```
2. What was the average, shortest, and longest bike trip taken in minutes?
	```
	SELECT avg(tripduration)/60 as average, max(tripduration)/60 as longest, min(tripduration)/60 as shortest
    FROM `bigquery-public-data.new_york_citibike.citibike_trips`
	```

3. Write a query that, for every station_name, has the amount of trips that started there and the amount of trips that ended there. (Hint, use two temporary tables, one that counts the amount of starts, the other that counts the number of ends, and then join the two.) 
	```
	with tempstarted as (
        select start_station_name, count(start_station_name) as startedcounter
        FROM `bigquery-public-data.new_york_citibike.citibike_trips`
        group by start_station_name
    ),
    tempended as (
        select end_station_name, count(end_station_name) as endedcounter
        FROM `bigquery-public-data.new_york_citibike.citibike_trips`
        group by end_station_name
    )
    select tempstarted.start_station_name, tempstarted.startedcounter, tempended.endedcounter
    from tempstarted inner join tempended
    on tempstarted.start_station_name = tempended.end_station_name
	```
# The next section is the Google Colab section.  
1. Open up this [this Colab notebook](https://colab.research.google.com/drive/1kHdTtuHTPEaMH32GotVum41YVdeyzQ74?usp=sharing).
2. Save a copy of it in your drive. 
3. Rename your saved version with your initials. 
4. Click the 'Share' button on the top right.  
5. Change the permissions so anyone with link can view. 
6. Copy the link and paste it right below this line. 
	* YOUR LINK:  https://colab.research.google.com/drive/1ILPHY3FS3iTC-Cvxt5Dbes865_TILiKf?usp=sharing
9. Complete the two questions in the colab notebook file. 
