# SQL-Data-Exploartion-Covid-19







### Q1: Is there a relationship between COVID spread and vaccination rate? (Aggregated by country and month).
 
 
```SQL
SELECT     v.location 'Country',
	   CONVERT(NVARCHAR(7),v.date,111) 'Date',
	   SUM(cd.new_cases) 'New Cases',
	   SUM(v.daily_vaccinations) 'Vaccinations',
FROM  vaccinations v INNER JOIN coviddeaths  cd 
ON cd.date = v.date AND cd.location = v.location
GROUP BY v.location, CONVERT(NVARCHAR(7),v.date,111)
ORDER BY 1,2
```
<br>
</br>







### Q2: Calculating For each country the fully vaccinated citizens as a percentage of population. 
(Note: The calculation was executed for countries with population over 1 million)

```SQL
WITH temp_table
AS(
SELECT  v.location 'Country',
 	v.people_fully_vaccinated,
	cd.population,
	(v.people_fully_vaccinated/cd.population)* 100 'Percent Population Fully Vacinated'
FROM  vaccinations v INNER JOIN coviddeaths  cd 
ON cd.date = v.date AND cd.location = v.location
GROUP BY v.location, v.people_fully_vaccinated ,cd.population
HAVING cd.population > 1000000)


SELECT Country, MAX([Percent Population Fully Vacinated])
FROM temp_table
GROUP BY Country
HAVING MAX([Percent Population Fully Vacinated]) <= 100 /*Note: due to vaccination of non-residentce, some contries
	                                            	exceded vacinating 100% of their population, thus removed from output*/
ORDER BY 2
```



<br>
</br>




### Q3: Listing the countries that were the first to reach herd immunity.
(Note: Although a controversial topic, for the sake of practice I choose to define herd immunity as fully vacinating atleast 70% of the population)

```SQL
WITH temp_table
AS(
SELECT  v.location 'Country',
	v.people_fully_vaccinated,
	cd.date 'date_time',
	cd.population,
	(v.people_fully_vaccinated/cd.population)* 100 'Percent Population Fully Vacinated'
FROM  vaccinations v INNER JOIN coviddeaths  cd 
ON cd.date = v.date AND cd.location = v.location
GROUP BY v.location, cd.date, v.people_fully_vaccinated ,cd.population
HAVING cd.population > 1000000 
)


SELECT Country, MIN([date_time]), MAX([Percent Population Fully Vacinated])
FROM temp_table
WHERE [Percent Population Fully Vacinated] >= 70 
GROUP BY Country
HAVING MAX([Percent Population Fully Vacinated]) <= 100
ORDER BY 2
```
