# SQL-Data-Exploartion-Covid-19

In this project I explored some of the COVID-19 data from https://ourworldindata.org/.






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







### Q2:  Investigating what are the top 10 countries with the highest percentage of their population  fully vaccinated.
Note: The calculation was executed for large countries defined as countries with population over 3 million citizens.
```SQL
DROP TABLE IF EXISTS my_fully_vac_table
SELECT v.location 'Country',
		v.people_fully_vaccinated,
		v.date,
		cd.population,
	       (v.people_fully_vaccinated/cd.population)* 100 'Percent Population Fully Vacinated'
	   INTO my_fully_vac_table -- Creating a temporarly table 
FROM  vaccinations v INNER JOIN coviddeaths  cd 
ON cd.date = v.date AND cd.location = v.location
WHERE cd.location != cd.continent
GROUP BY v.location, v.people_fully_vaccinated ,cd.population, v.date
HAVING cd.population > 3000000


DROP TABLE IF EXISTS top_10_vac
SELECT TOP 10 Country, MAX(ROUND([Percent Population Fully Vacinated],2)) AS 'Percent Vaccinated' 
INTO top_10_vac
FROM my_fully_vac_table
GROUP BY Country
HAVING MAX([Percent Population Fully Vacinated]) <= 100 /*Note: due to vaccination of non-residentce, some contries
														exceded vacinating 100% of their population, thus removed from output*/
ORDER BY 2 DESC



SELECT * 
FROM top_10_vac

```
<div class='tableauPlaceholder' id='viz1677071221641' style='position: relative'><noscript><a href='#'><img alt='Dashboard 1 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Q2&#47;Q2_WorkBook&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'                     vizElement.style.width='300px';
                    vizElement.style.height='450px'; style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Q2_WorkBook&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Q2&#47;Q2_WorkBook&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /></object></div> 


<br>



</br>











### Here I'm creating (and pre-processing) data i'll be using in the next questions.
```SQL
DROP TABLE IF EXISTS percent_table
CREATE TABLE percent_table (Country NVARCHAR(20), fully_vaccinated BIGINT, population_size BIGINT ,date_time DATE, percent_fully_vaccinated FLOAT)


INSERT INTO percent_table(Country, fully_vaccinated, population_size, date_time, percent_fully_vaccinated)

SELECT v.location,
		v.people_fully_vaccinated,
		cd.population,
		cd.date,
	   (v.people_fully_vaccinated/cd.population)* 100 
FROM  vaccinations v INNER JOIN coviddeaths  cd ON cd.date = v.date AND cd.location = v.location
WHERE ((v.people_fully_vaccinated/cd.population)* 100) >= 70  AND -- keeping only those who crossed the "herd immunity" threshold.
	   cd.location NOT LIKE '%income%' AND cd.Location NOT LIKE '%world%' -- removing non-relevent observations.


GROUP BY v.location, cd.date, v.people_fully_vaccinated ,cd.population
HAVING cd.population > 3000000 


DELETE FROM percent_table
WHERE Country IN (
    SELECT Country
    FROM percent_table
    WHERE percent_fully_vaccinated > 100)

```

<br>
</br>









### Q3: Listing the first 10 countries that were the first to reach herd immunity.
##### Note: Although a controversial topic, for the sake of practice I choose to define herd immunity as conuntries that atleast 70% of their population is fully vaccinated.
```SQL
WITH temp_table
AS(
SELECT *, ROW_NUMBER() OVER(PARTITION BY Country ORDER BY percent_fully_vaccinated) 'ranked'
FROM percent_table)



SELECT  TOP 10 Country,  date_time, percent_fully_vaccinated
FROM temp_table
WHERE ranked = 1
ORDER BY date_time

```


| Country | Date |
|---------|------|
|Uruguay | 2021-08-12|
|Singapore | 2021-08-14|
|Portugal | 2021-08-27|
|Belgium | 2021-08-30|
|Denmark | 2021-08-31|
|Spain	| 2021-08-31|
|Chile | 2021-09-02|
|Ireland | 2021-09-06|
|China | 2021-09-15|
|Wales |2021-09-24|
|          |              |







 It appears that not all those who were to first to reach herd immunity also ended being the most (fully) vaccinated countries.
 Perhaps the rate of vaccinations in former countries reached a platoo when these countries achieved herd immunity. 
 In the next question I will test this hypothesis.
 
 <br>
 
 </br>
 
 
 
 ### Q4: Investegating how the vacination rates have changed across time for countries top countries who were first to reach herd immunity but weren't also th etop counttries who vaccinated most of their population.

```SQL
WITH ranked_countries
AS(
SELECT *, ROW_NUMBER() OVER(PARTITION BY Country ORDER BY percent_fully_vaccinated) 'ranked'
FROM percent_table),

herd_table
AS (
	SELECT  TOP 10 Country,  date_time, percent_fully_vaccinated
	FROM ranked_countries
	WHERE ranked = 1
	ORDER BY date_time),
	
top_vac
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





SELECT *
FROM my_fully_vac_table
WHERE Country IN 
				(SELECT Country
				FROM herd_table 
				WHERE Country NOT IN (SELECT Country FROM top_10_vac))

ORDER BY Country, date
```

<div class='tableauPlaceholder' id='viz1677685620037' style='position: relative'><noscript><a href='#'><img alt='Dashboard 1 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Co&#47;Covid-19Question4&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Covid-19Question4&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Co&#47;Covid-19Question4&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /><param name='filter' value='publish=yes' /></object></div>                

<br>

</br>


It appears my hypothesize is correct as the slope of the vaccination rate is reaching asymptote around august, which is the month that most countries in Q2 have reached herd immunity.
