# Global COVID Trends from 2020-2022

### The purpose of this project is to explore data gathered on patterns and trends related to infection of, death of, and vaccination from COVID-19

Data was split into two seperate tables using Excel (COVID deaths and subsequent information and COVID vaccinations and subsequent information). Within this exploration, I used multiple different SQL skills beyond `select`, `from`, and `where`, including `max`, `order by`, `group by`, `JOIN`, a CTE, `CREATE TABLE`, `OVER`, and `PARTITION BY.`

Data is pulled from [OurWorldInData.org](https://ourworldindata.org/covid-deaths), however, raw data can be found in this repository as well as Excel files of the COVID Deaths and COVID Vaccination tables, if you would like to work along. 
______

I first began by checking that data was uploaded correctly, first with the Covid Deaths table.
```SQL
Select * 
from PortfolioProject.dbo.CovidDeaths
where continent is not NULL
order by 3,4
```
Next, I checked that the Covid Vaccinations table was uploaded correctly.

``` SQL
select *
from PortfolioProject.dbo.CovidVaccinations
order by 3,4
```
Then, I wanted to create a table with only the information that I would be looking closely at in the upcoming analysis. I used the `select` statment to narrow down for the location, date, total cases, new cases, total deaths, and population colums. I also included a `is not null` statement due to some inconsistencies in the data.

``` SQL
-- select data that we are going to be using

select Location, date, total_cases, new_cases, total_deaths, population
from PortfolioProject.dbo.CovidDeaths
where continent is not NULL
order by 1,2
```
Next, I created a statement that would show the total cases compared to the total deaths of each country, as well as using a computational function to show the percentage of people who have died from COVID. While this statement can be used to show it with an individual country, in this example I used the United States, you could comment the `where location =` statement to show all countries.
``` SQL

-- looking at total cases vs total deaths
-- shows likelihood of dying of COVID contraction in your country (United Statesa as example)

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Death_Percentage
from PortfolioProject.dbo.CovidDeaths
where location = 'United States'
order by 1,2
```
Following, I created a simlar statement as to above, however, I wanted it to display the percentage of the population that contracted COVID. So, I selected for location, date, total cases, and population, then I created another computational function to show the percentage of the population that has reported COVID infection.
```SQL
--looking at countries with highest infection rate compared to population

select Location, MAX(total_cases) as Highest_Infection_Count, population, (MAX(total_cases)/population)*100 as Percent_COVID_Population
from PortfolioProject.dbo.CovidDeaths
--where location = 'United States'
group by Location, population
order by Percent_COVID_Population desc
```
Then, I wanted to see the same information, but I wanted it to be grouped by date as well to see if there were major differences on specific dates.
```SQL
--same query, grouped by date as well
select Location, date, MAX(total_cases) as Highest_Infection_Count, population, (MAX(total_cases)/population)*100 as Percent_COVID_Population
from PortfolioProject.dbo.CovidDeaths
--where location = 'United States'
group by Location, population, date
order by Percent_COVID_Population desc
```
I was also interested to see which countries experienced the most death related to COVID. This information could be helpful to see where there should be targeted efforts for future pandemics. Then, I decided to do the same function but I wanted to see it from a greater macro level to see which contients experienced the most deaths related to COVID.
```SQL
-- showing countries with highest death count per population

select Location, MAX(total_deaths) as Total_Death_Count
from PortfolioProject.dbo.CovidDeaths
where continent is not NULL
group by Location
order by Total_Death_Count desc

-- let's break things down by continent 

select continent, MAX(total_deaths) as Total_Death_Count
from PortfolioProject.dbo.CovidDeaths
where continent is not NULL
group by continent
order by Total_Death_Count desc
```
Continuing at the macro, global level, I wanted to understand how many cases and deaths there were, and I also made another computational function to find the the percentage of deaths from all cases accross the globe. The first query finds the cases, death, and death percentage per date; however, the next query finds the toal global numbers from all reported dates.
```SQL
-- Global Death Rate numbers by date and by total 
-- date grouped first 
select date, sum(new_cases) as Total_New_Cases_Globably, sum(new_deaths) as Total_New_Deaths_Globaly, (sum(new_deaths)/sum(new_cases))*100 as Total_Death_Percentage
from PortfolioProject.dbo.CovidDeaths
where continent is not null 
group by date
order by 1,2

-- total of all time 

select sum(new_cases) as Total_New_Cases_Globably, sum(new_deaths) as Total_New_Deaths_Globaly, (sum(new_deaths)/sum(new_cases))*100 as Total_Death_Percentage
from PortfolioProject.dbo.CovidDeaths
where continent is not null 
order by 1,2
```
Next, I knew I wanted to join both tables intially uploaded seperately together using a `JOIN` function. I combined these tables on the primary keys of location and date that were the same in each. 
```SQL
-- Moving to the Covid Vaccinations table 

select *
from PortfolioProject.dbo.CovidVaccinations

-- joining both tables

select *
from PortfolioProject.dbo.CovidDeaths dea
JOIN PortfolioProject.dbo.CovidVaccinations vac
    on dea.location= vac.location
    and dea.date=vac.date
```
I wanted to understand how much of the population were vaccinated. Since the COVID deaths table contained population information and the COVID vaccinations table contained vaccination information, I expanded on the previous `JOIN` query from above. In this function, I selected the continent, location, date, population and new vaccination column. 
```SQL
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
from PortfolioProject.dbo.CovidDeaths dea
JOIN PortfolioProject.dbo.CovidVaccinations vac
    on dea.location= vac.location
    and dea.date=vac.date
where dea.continent is not null 
order by 1,2,3
```
After successfuly getting the table above to work, I then attempted to create a rolling count of vaccinations per country per date and a rolling count of the percent of people who became vaccinated. Now, there were two ways to about doing this since I would need to essentially have to create a seperate storage of the information in some way in order to create this rolling count as it would need to used in a computational function in order to get a percentage, so I could either use a `CTE` or `Create Table` to create a temporary table. So I decided to do both. 

Below is the creation of the `CTE` in order to create the rolling count, including the usage of the `OVER(PARTITION BY)` function which created a rolling count of vaccinations day to day for each country. 

Please note there are many counties that did not report vaccinations, so many columns were left blank or `NULL`, so I added a `where` clause for new.vaccinations to eliminate null values, if desired, it is only commented it out. 
```SQL
-- making a rolling count of vaccinations per country and using that to compare to population

-- using a CTE

With Pop_vs_Vac (Continent, location, date, population, new_vaccinations, Rolling_Vaccination_Count)
AS
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (partition by dea.location order by dea.location, dea.date) as Rolling_Vaccination_Count
--(Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated
from PortfolioProject.dbo.CovidDeaths dea
JOIN PortfolioProject.dbo.CovidVaccinations vac
    on dea.location= vac.location
    and dea.date=vac.date
where dea.continent is not null --AND vac.new_vaccinations is not null
--order by 2,3
)

select *, (Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated
from Pop_vs_Vac
```
To test myself, I did the same thing using a temporary table. I used the `DROP TABLE IF EXISTS` function in order to prevent mistakes when rerunning the code so I could run it multiple times when fixing errors. After the `DROP` function, I used the `CREATE TABLE` function to create a new table and assigned data types. After, I took information from the COVID deaths and COVID vaccination databases to add to the new table, which was named "#Percent_Poopulation_Vaccinated" using the `INSERT INTO` and `SELECT` functions. I also used the `OVER(PARTITION BY )` function in order to find the rolling vaccination count for each country on each date. 
```SQL
-- using a temp table

DROP TABLE if exists #Percent_Population_Vaccinated -- included to prevent errors
create table #Percent_Population_Vaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population NUMERIC,
new_vaccinations NUMERIC, 
Rolling_Vaccination_Count numeric    
)

Insert into #Percent_Population_Vaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (partition by dea.location order by dea.location, dea.date) as Rolling_Vaccination_Count 
--(Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated
from PortfolioProject.dbo.CovidDeaths dea
JOIN PortfolioProject.dbo.CovidVaccinations vac
    on dea.location= vac.location
    and dea.date=vac.date
where dea.continent is not null  AND vac.new_vaccinations is not null
--order by 2,3

select *, (Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated
from #Percent_Population_Vaccinated
```
Finally, I wanted to save all this so I created a `view` to save for visualizations later on.
```SQL
-- creating view to store data for later visualizations

--create View Percent_Population_Vac as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (partition by dea.location order by dea.location, dea.date) as Rolling_Vaccination_Count
-- (Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated
from PortfolioProject.dbo.CovidDeaths dea
JOIN PortfolioProject.dbo.CovidVaccinations vac
    on dea.location= vac.location
    and dea.date=vac.date
where dea.continent is not null 
--order by 2,3

select * 
from Percent_Population_Vac
```


