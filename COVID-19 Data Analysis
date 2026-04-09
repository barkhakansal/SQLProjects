/*
Covid 19 Data Exploration 

Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types

*/

SELECT * FROM Covid..CovidDeaths
ORDER BY 3,4 


select location,date,total_cases,new_cases, total_deaths, population
FROM Covid..CovidDeaths
Where continent is not null
order by 1,2

-- looking at total cases vs total deaths

select location,date,total_cases, round((total_deaths/total_cases)*100,2) as DeathPercentage
FROM Covid..CovidDeaths
Where continent is not null
order by 1,2

-- Total Cases vs Population
-- Shows what percentage of population infected with Covid

select location,date,population,total_cases, round((total_cases/population)*100,2) as CovidPercentage
FROM Covid..CovidDeaths
--where location like '%states%'
order by 1,2

-- looking at countries with highest infection rate compared to population

select location,population,max(total_cases) as HighestInfectionCount,max((total_cases/population))*100 as PercentPopulationInfected
FROM Covid..CovidDeaths
group by location,population
order by PercentPopulationInfected desc

-- showing countries with highest death count per population

select location,max(cast(total_deaths as int)) as TotalDeathCount
FROM Covid..CovidDeaths
where continent is not null
group by location
order by TotalDeathCount desc

-- showing continents with highest death count per population

select continent,max(cast(total_deaths as int)) as TotalDeathCount
FROM Covid..CovidDeaths
where continent is not null
group by continent
order by TotalDeathCount desc

-- Global numbers

select date,sum(new_cases) as Total_Cases,sum(cast(new_deaths as int)) as Total_Deaths, 
sum(cast(new_deaths as int))/sum(new_cases)*100 as DeathPercentage
FROM Covid..CovidDeaths
where continent is not null
group by date
order by 1,2 desc

select sum(new_cases) as Total_Cases,sum(cast(new_deaths as int)) as Total_Deaths, 
sum(cast(new_deaths as int))/sum(new_cases)*100 as DeathPercentage
FROM Covid..CovidDeaths
where continent is not null
order by 1,2 desc


-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From Covid..CovidDeaths dea
Join Covid..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3

-- Using CTE to perform Calculation on Partition By in previous query

with popvsvac as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From Covid..CovidDeaths dea
Join Covid..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
)
select continent,location, date, population, new_vaccinations,RollingPeopleVaccinated, 
(RollingPeopleVaccinated/population)*100 as RollingPercentage
from popvsvac

-- Using Temp Table to perform Calculation on Partition By in previous query

DROP Table if exists #PercentPopulationVaccinated
create table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_Vaccinations numeric,
RollingPeopleVaccinated numeric
)
Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From Covid..CovidDeaths dea
Join Covid..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

select *, (RollingPeopleVaccinated/population)*100 as RollingPercentage
from #PercentPopulationVaccinated

-- Creating View to store data for later visualizations

create view PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From Covid..CovidDeaths dea
Join Covid..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

select * from PercentPopulationVaccinated

