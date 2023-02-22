# SQL-Data-Exploartion-Covid-19







### Q1: Exploring infections the death rates globally and by continents. 

```SQL

WITH continents_average 
AS (
	SELECT  'Continent Average' as 'Location', 
	       CONVERT(DATE,v.date,103) 'Date',
		   ROUND(SUM(cd.new_cases)/COUNT(DISTINCT cd.continent),0) 'New Infections',
		   ROUND(SUM(v.daily_vaccinations)/COUNT(DISTINCT cd.continent),0) 'Vaccinations',
		   SUM(CAST(cd.new_deaths AS INT))/COUNT(DISTINCT cd.continent) 'New Deaths'
	FROM vaccinations v INNER JOIN coviddeaths cd 
	ON cd.date = v.date AND cd.iso_code = v.iso_code
	WHERE cd.Location NOT LIKE '%income%' AND cd.Location NOT LIKE '%world%' AND cd.Location != cd.continent
	GROUP BY CONVERT(DATE,v.date,103)),






continents_cases
AS (
SELECT cd.continent 'Location',
	   CONVERT(DATE,v.date,103) 'Date',
	   SUM(cd.new_cases) 'New Infections',
	   SUM(v.daily_vaccinations) 'Vaccinations',
	   SUM(CAST (cd.new_deaths AS INT)) 'New Deaths'
		FROM vaccinations v INNER JOIN coviddeaths  cd 
		ON cd.date = v.date AND cd.location = v.location
	WHERE cd.Location NOT LIKE '%income%' AND cd.Location NOT LIKE '%world%' AND cd.Location != cd.continent
		GROUP BY cd.continent, CONVERT(DATE,v.date,103))





SELECT *  FROM continents_average
UNION ALL
SELECT *  FROM continents_cases
```

<div class='tableauPlaceholder' id='viz1677070976883' style='position: relative'><noscript><a href='#'><img alt='Dashboard 1 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Q1&#47;Q1_WorkBook&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Q1_WorkBook&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Q1&#47;Q1_WorkBook&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /></object></div>               

<br>




</br>







### Q2: Calculating For each country the fully vacitaed citizens as a percentage of population. 

```SQL
WITH temp_table
AS(
SELECT v.location 'Country',
		v.people_fully_vaccinated,
		cd.population,
	   (v.people_fully_vaccinated/cd.population)* 100 'Percent Population Fully Vacinated'
FROM  vaccinations v INNER JOIN coviddeaths  cd 
ON cd.date = v.date AND cd.location = v.location
WHERE cd.location != cd.continent
GROUP BY v.location, v.people_fully_vaccinated ,cd.population
HAVING cd.population > 3000000)



SELECT TOP 20 Country, MAX(ROUND([Percent Population Fully Vacinated],2)) AS 'Percent Vaccinated'
FROM temp_table
GROUP BY Country
HAVING MAX([Percent Population Fully Vacinated]) <= 100 /*Note: due to vaccination of non-residentce, some contries
														exceded vacinating 100% of their population, thus removed from output*/
ORDER BY 2 DESC

```
<div class='tableauPlaceholder' id='viz1677071221641' style='position: relative'><noscript><a href='#'><img alt='Dashboard 1 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Q2&#47;Q2_WorkBook&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'                     vizElement.style.width='300px';
                    vizElement.style.height='450px'; style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Q2_WorkBook&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Q2&#47;Q2_WorkBook&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /></object></div> 


<br>



</br>




### Q3: Listing the countries that were the first to reach herd immunity.
##### Note: Although a controversial topic, for the sake of practice I choose to define herd immunity as fully vacinating atleast 70% of the population.
```SQL
WITH temp_table
AS(
SELECT v.location 'Country',
		v.people_fully_vaccinated,
		cd.date 'date_time',
		cd.population,
	   (v.people_fully_vaccinated/cd.population)* 100 'Percent Population Fully Vacinated'
FROM  vaccinations v INNER JOIN coviddeaths  cd 
ON cd.date = v.date AND cd.location = v.location
GROUP BY v.location, cd.date, v.people_fully_vaccinated ,cd.population
HAVING cd.population > 3000000 
)



SELECT TOP 10 Country, MIN([date_time]) 'Date'
FROM temp_table
WHERE [Percent Population Fully Vacinated] >= 70 
GROUP BY Country
HAVING MAX([Percent Population Fully Vacinated]) <= 100
ORDER BY 2 DESC
```


| Country | Date |
|---------|------|
|Rwanda	| 2023-01-01|
|Liberia |	2022-12-11|
|Nepal	| 2022-08-29|
|Colombia |	2022-07-29|
|Bangladesh |	2022-07-16|
|Asia	| 2022-06-25|
|Nicaragua |	2022-06-03|
|Panama	| 2022-05-13|
|Thailand |	2022-03-30|
|South America |2022-03-05|
