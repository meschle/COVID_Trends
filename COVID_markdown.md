{
    "metadata": {
        "kernelspec": {
            "name": "SQL",
            "display_name": "SQL",
            "language": "sql"
        },
        "language_info": {
            "name": "sql",
            "version": ""
        }
    },
    "nbformat_minor": 2,
    "nbformat": 4,
    "cells": [
        {
            "cell_type": "markdown",
            "source": [
                "The purpose of this project is to explore data gathered on patterns and trends related to infection of, death of, and vaccination from COVID-19"
            ],
            "metadata": {
                "azdata_cell_guid": "282372cb-c15f-4653-a0b8-e8f2901581bb"
            }
        },
        {
            "cell_type": "markdown",
            "source": [
                "Data is pulled from OurWorldInData.org (https://ourworldindata.org/covid-deaths)"
            ],
            "metadata": {
                "azdata_cell_guid": "c66aba1f-3d79-4d3f-906d-242d9b809d56"
            }
        },
        {
            "cell_type": "markdown",
            "source": [
                "Data was split into two seperate tables using Excel (COVID deaths and subsequent information and COVID vaccinations and subsequent information). Within this exploration, I used multiple different SQL skills beyond `select`, `from`, and `where`, including `max`, `order by`, `group by`, `JOIN`, a CTE, `CREATE TABLE`, `OVER`, and `PARTITION BY.`"
            ],
            "metadata": {
                "azdata_cell_guid": "729b5b2e-15ba-4191-9357-282a9d3ca541"
            },
            "attachments": {}
        },
        {
            "cell_type": "markdown",
            "source": [
                "I began by checking that data was uploaded correctly, first with the Covid Deaths table."
            ],
            "metadata": {
                "azdata_cell_guid": "804696fd-8212-4484-b90b-24c75875bb3e"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "Select * \n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "where continent is not NULL\n",
                "order by 3,4"
            ],
            "metadata": {
                "azdata_cell_guid": "ea658ce1-c262-432c-a820-bfdf11ae1dfe",
                "language": "sql"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Next, I checked that the Covid Vaccinations table was uploaded correctly."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "d46d0a51-a5c4-4538-a4ed-66ec37be1e6e"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "select *\n",
                "from PortfolioProject.dbo.CovidVaccinations\n",
                "order by 3,4"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "104c5dea-ff04-44ab-93d3-d3bcea9a1480"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Then, I wanted to create a table with only the information that I would be looking closely at in the upcoming analysis. I used the `select` statment to narrow down for the location, date, total cases, new cases, total deaths, and population colums. I also included a `is not null` statement due to some inconsistencies in the data."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "7ab97ccd-ae8c-4611-a086-4459759635e8"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "-- select data that we are going to be using\n",
                "\n",
                "select Location, date, total_cases, new_cases, total_deaths, population\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "where continent is not NULL\n",
                "order by 1,2"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "b30d35ee-e70f-4b18-a903-e1f25891b3de"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Next, I created a statement that would show the total cases compared to the total deaths of each country, as well as using a computational function to show the percentage of people who have died from COVID. While this statement can be used to show it with an individual country, in this example I used the United States, you could comment the `where location =` statement to show all countries."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "628d4a62-6538-4ffa-92e0-7c8cd071bb1d"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "\n",
                "-- looking at total cases vs total deaths\n",
                "-- shows likelihood of dying of COVID contraction in your country (United Statesa as example)\n",
                "\n",
                "select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Death_Percentage\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "where location = 'United States'\n",
                "order by 1,2"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "90eaf7bf-c66a-4757-819c-c888eb9e6743",
                "tags": []
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Following, I created a simlar statement as to above, however, I wanted it to display the percentage of the population that contracted COVID. So, I selected for location, date, total cases, and population, then I created another computational function to show the percentage of the population that has reported COVID infection."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "ca006197-4c2d-4188-9897-c1d0ea666e87"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "-- looking at total cases vs population\n",
                "-- shows what percentage of population got COVID\n",
                "\n",
                "select Location, date, total_cases, population, (total_cases/population)*100 as Percent_COVID_Population\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "--where location = 'United States'\n",
                "order by 1,2"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "73b0109d-f348-4e50-9379-a2eb09e28bad",
                "tags": []
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "The next statment I created compared countries with the highest infection rate, their population, and the percentage of their population that has reported to be infected with COVID. Then I oredered it to show the the countries with the highest percentage of their population reporting to be infected."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "ad65bd66-71d5-4795-8970-21d73dc58dc2"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "--looking at countries with highest infection rate compared to population\n",
                "\n",
                "select Location, MAX(total_cases) as Highest_Infection_Count, population, (MAX(total_cases)/population)*100 as Percent_COVID_Population\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "--where location = 'United States'\n",
                "group by Location, population\n",
                "order by Percent_COVID_Population desc"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "a5e0213b-271a-47b6-becc-b72f3839c9d0"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Then, I wanted to see the same information, but I wanted it to be grouped by date as well to see if there were major differences on specific dates."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "0b5c68c8-d8a8-4a22-9d8b-29f5ca09d65c"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "\n",
                "--same query, grouped by date as well\n",
                "select Location, date, MAX(total_cases) as Highest_Infection_Count, population, (MAX(total_cases)/population)*100 as Percent_COVID_Population\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "--where location = 'United States'\n",
                "group by Location, population, date\n",
                "order by Percent_COVID_Population desc"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "8aaf5007-f7bd-4704-9631-a80cd7de4588"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "I was also interested to see which countries experienced the most death related to COVID. This information could be helpful to see where there should be targeted efforts for future pandemics. Then, I decided to do the same function but I wanted to see it from a greater macro level to see which contients experienced the most deaths related to COVID."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "5917962b-8bfd-47d0-aa60-7a0f408da252"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "\n",
                "-- showing countries with highest death count per population\n",
                "\n",
                "select Location, MAX(total_deaths) as Total_Death_Count\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "where continent is not NULL\n",
                "group by Location\n",
                "order by Total_Death_Count desc\n",
                "\n",
                "-- let's break things down by continent \n",
                "\n",
                "select continent, MAX(total_deaths) as Total_Death_Count\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "where continent is not NULL\n",
                "group by continent\n",
                "order by Total_Death_Count desc"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "61d3600f-7c32-470b-a846-98104e29be81"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Continuing at the macro, global level, I wanted to understand how many cases and deaths there were, and I also made another computational function to find the the percentage of deaths from all cases accross the globe. The first query finds the cases, death, and death percentage per date; however, the next query finds the toal global numbers from all reported dates."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "b6c0d989-5bd5-4441-a0f6-581d986e814d"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "\n",
                "-- Global Death Rate numbers by date and by total \n",
                "-- date grouped first \n",
                "select date, sum(new_cases) as Total_New_Cases_Globably, sum(new_deaths) as Total_New_Deaths_Globaly, (sum(new_deaths)/sum(new_cases))*100 as Total_Death_Percentage\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "where continent is not null \n",
                "group by date\n",
                "order by 1,2\n",
                "\n",
                "-- total of all time \n",
                "\n",
                "select sum(new_cases) as Total_New_Cases_Globably, sum(new_deaths) as Total_New_Deaths_Globaly, (sum(new_deaths)/sum(new_cases))*100 as Total_Death_Percentage\n",
                "from PortfolioProject.dbo.CovidDeaths\n",
                "where continent is not null \n",
                "order by 1,2"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "95e72f14-b5d4-4c21-bf05-2e6fccf249cc"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Next, I knew I wanted to join both tables intially uploaded seperately together using a `JOIN` function. I combined these tables on the primary keys of location and date that were the same in each. "
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "bf2299da-dfb6-4f7b-9b09-8ca0f5855343"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "-- Moving to the Covid Vaccinations table \n",
                "\n",
                "select *\n",
                "from PortfolioProject.dbo.CovidVaccinations\n",
                "\n",
                "-- joining both tables\n",
                "\n",
                "select *\n",
                "from PortfolioProject.dbo.CovidDeaths dea\n",
                "JOIN PortfolioProject.dbo.CovidVaccinations vac\n",
                "    on dea.location= vac.location\n",
                "    and dea.date=vac.date"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "c5d7e9b8-cd30-4079-b2c3-3ab29aaac9ec"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "I wanted to understand how much of the population were vaccinated. Since the COVID deaths table contained population information and the COVID vaccinations table contained vaccination information, I expanded on the previous `JOIN` query from above. In this function, I selected the continent, location, date, population and new vaccination column. "
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "e7d281cf-7be4-4d60-a472-6e475bfd7b26"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations\n",
                "from PortfolioProject.dbo.CovidDeaths dea\n",
                "JOIN PortfolioProject.dbo.CovidVaccinations vac\n",
                "    on dea.location= vac.location\n",
                "    and dea.date=vac.date\n",
                "where dea.continent is not null \n",
                "order by 1,2,3"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "d3fa5521-d8c5-4111-ab4c-f83e5d72cc23",
                "tags": []
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "After successfuly getting the table above to work, I then attempted to create a rolling count of vaccinations per country per date and a rolling count of the percent of people who became vaccinated. Now, there were two ways to about doing this since I would need to essentially have to create a seperate storage of the information in some way in order to create this rolling count as it would need to used in a computational function in order to get a percentage, so I could either use a `CTE` or `Create Table` to create a temporary table. So I decided to do both. \n",
                "\n",
                "Below is the creation of the `CTE` in order to create the rolling count, including the usage of the `OVER(PARTITION BY)` function which created a rolling count of vaccinations day to day for each country. \n",
                "\n",
                "Please note there are many counties that did not report vaccinations, so many columns were left blank or `NULL`, so I added a `where` clause for new.vaccinations to eliminate null values, if desired, it is only commented it out. "
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "55648ab8-ae3b-4883-8bc5-70120b702418"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "-- making a rolling count of vaccinations per country and using that to compare to population\n",
                "\n",
                "-- using a CTE\n",
                "\n",
                "With Pop_vs_Vac (Continent, location, date, population, new_vaccinations, Rolling_Vaccination_Count)\n",
                "AS\n",
                "(\n",
                "select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations\n",
                ", SUM(vac.new_vaccinations) OVER (partition by dea.location order by dea.location, dea.date) as Rolling_Vaccination_Count\n",
                "--(Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated\n",
                "from PortfolioProject.dbo.CovidDeaths dea\n",
                "JOIN PortfolioProject.dbo.CovidVaccinations vac\n",
                "    on dea.location= vac.location\n",
                "    and dea.date=vac.date\n",
                "where dea.continent is not null --AND vac.new_vaccinations is not null\n",
                "--order by 2,3\n",
                ")\n",
                "\n",
                "select *, (Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated\n",
                "from Pop_vs_Vac"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "631d905b-cc3c-4b97-8e71-a1edb2aec367"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "To test myself, I did the same thing using a temporary table. I used the `DROP TABLE IF EXISTS` function in order to prevent mistakes when rerunning the code so I could run it multiple times when fixing errors. After the `DROP` function, I used the `CREATE TABLE` function to create a new table and assigned data types. After, I took information from the COVID deaths and COVID vaccination databases to add to the new table, which was named \"#Percent_Poopulation_Vaccinated\" using the `INSERT INTO` and `SELECT` functions. I also used the `OVER(PARTITION BY )` function in order to find the rolling vaccination count for each country on each date. "
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "821ea920-1542-4270-8172-4bf83d00d6e0"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "\n",
                "-- using a temp table\n",
                "\n",
                "DROP TABLE if exists #Percent_Population_Vaccinated -- included to prevent errors\n",
                "create table #Percent_Population_Vaccinated\n",
                "(\n",
                "Continent nvarchar(255),\n",
                "Location nvarchar(255),\n",
                "Date datetime,\n",
                "Population NUMERIC,\n",
                "new_vaccinations NUMERIC, \n",
                "Rolling_Vaccination_Count numeric    \n",
                ")\n",
                "\n",
                "Insert into #Percent_Population_Vaccinated\n",
                "select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations\n",
                ", SUM(vac.new_vaccinations) OVER (partition by dea.location order by dea.location, dea.date) as Rolling_Vaccination_Count \n",
                "--(Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated\n",
                "from PortfolioProject.dbo.CovidDeaths dea\n",
                "JOIN PortfolioProject.dbo.CovidVaccinations vac\n",
                "    on dea.location= vac.location\n",
                "    and dea.date=vac.date\n",
                "where dea.continent is not null  AND vac.new_vaccinations is not null\n",
                "--order by 2,3\n",
                "\n",
                "select *, (Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated\n",
                "from #Percent_Population_Vaccinated"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "396d3f22-19a8-460d-8f71-3f5ce398b60a"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                "Finally, I wanted to save all this so I created a \\`view\\` to save for visualizations later on."
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "d4bf0bba-fe23-42c9-b4c3-53cc00cb1a59"
            },
            "attachments": {}
        },
        {
            "cell_type": "code",
            "source": [
                "\n",
                "-- creating view to store data for later visualizations\n",
                "\n",
                "--create View Percent_Population_Vac as\n",
                "select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations\n",
                ", SUM(vac.new_vaccinations) OVER (partition by dea.location order by dea.location, dea.date) as Rolling_Vaccination_Count\n",
                "-- (Rolling_Vaccination_Count/population)*100 as Percent_Vaccinated\n",
                "from PortfolioProject.dbo.CovidDeaths dea\n",
                "JOIN PortfolioProject.dbo.CovidVaccinations vac\n",
                "    on dea.location= vac.location\n",
                "    and dea.date=vac.date\n",
                "where dea.continent is not null \n",
                "--order by 2,3\n",
                "\n",
                "select * \n",
                "from Percent_Population_Vac"
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "8259e8b8-56d4-49f0-b0a4-1c719aeca6c4"
            },
            "outputs": [],
            "execution_count": null
        },
        {
            "cell_type": "markdown",
            "source": [
                ""
            ],
            "metadata": {
                "language": "sql",
                "azdata_cell_guid": "fa834e44-5347-44e6-a241-4549dea3a31f"
            }
        }
    ]
}