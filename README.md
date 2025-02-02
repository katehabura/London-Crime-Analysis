# London Crime Analysis - Katherine Habura

## London Crime Analysis Introduction
The following is an analysis of crimes committed in the 32 London boroughs + the City of London from 2008 to 2016. The crimes reported are split up into major and minor categories. Crimes included in the "major category" include sexual offences, violence against the person, drugs, theft and handling, robbery, fraud or forgery, criminal damage, burglary, and other notable offenses. There are 32 crimes included in the "minor category", and they can be considered as a more specific categorization of the major category crimes. 

For this analysis, I will be taking on the role of somebody who is considering purchasing property in London, and I only want to purchase a property in a borough that is considered safe. The following questions will be used to guide the analysis: 
* How can crime in London be summarized in order to get a general idea of the safety of the city?
* What boroughs are considered to be the most dangerous and safest to live based on violent crime?
* What boroughs are considered to be the most dangerous and safest to live based on non-violent crime?

## A Summary of London Crime
Figure 1 reveals the total number of crimes within each major category among all London buroughs and the City of London, with Theft and Handling, Violence Against the Person, and Criminal Damange are the top 3 most common crimes. Figure 2 reveals the top 10 buroughs (including the City of London) with the most amount of crimes. Figure 3 reveals that the most common crime within the top 10 most crime-laden buroughs is Theft and Handling.

```sql

SELECT
major_category,
count(*) as NumberOfCrimes
FROM `bigquery-public-data.london_crime.crime_by_lsoa`
group by major_category
order by NumberOfCrimes desc
```
<div align="center">
  <img src="https://i.imgur.com/GLq6t3l.png" width="250">
</div>
<p align="center">Figure 1</p>

```sql
SELECT
borough,
count(*) as NumberOfCrimes
FROM `bigquery-public-data.london_crime.crime_by_lsoa`
group by borough
order by NumberOfCrimes desc
limit 5
```
<div align="center">
  <img src="https://i.imgur.com/KCQ5cix.png" width="250">
</div>
<p align="center">Figure 2</p>

```sql
SELECT
  borough,
  major_category,
  CrimeCount
FROM (
  SELECT
      borough,
      major_category,
      COUNT(*) AS CrimeCount,
      ROW_NUMBER() OVER (PARTITION BY borough ORDER BY COUNT(*) DESC) AS rn
  FROM `bigquery-public-data.london_crime.crime_by_lsoa`
  WHERE borough IN ('Croydon', 'Barnet', 'Ealing', 'Bromley', 'Lambeth', 'Enfield', 'Wandsworth', 'Brent', 'Lewisham', 'Southwark')
  GROUP BY
      borough,
      major_category
)
WHERE rn = 1
ORDER BY borough;
```
<div align="center">
  <img src="https://i.imgur.com/4B1kmJt.png" width="250">
</div>
<p align="center">Figure 3</p>

## What boroughs are considered to be the most dangerous and safest to live based on violent crime?
As Figure 3 revealed, the most common crime is Theft and Handling. To me, Theft and Handling is not a concern to be, as there are precautions one can take (installing a camera, fence, etc...) Instead, I want to focus on violent crimes which include robbery, sexual offences, and and violence against the person. Based strictly on violent crimes, Figure 4 reveals the top 10 most dangerous boroughs in London, and Figure 5 reveals the top 10 safest boroughs in London. 

```sql
SELECT
   borough,
   COUNTIF(major_category IN ('Sexual Offences', 'Robbery', 'Violence Against the Person')) AS TotalViolentCrimes,
FROM
   `bigquery-public-data.london_crime.crime_by_lsoa`
GROUP BY
   borough
order by TotalViolentCrimes desc
Limit 10
```
<div align="center">
  <img src="https://i.imgur.com/dco1udQ.png" width="250">
</div>
<p align="center">Figure 4</p>

```sql
SELECT
   borough,
   COUNTIF(major_category IN ('Sexual Offences', 'Robbery', 'Violence Against the Person')) AS TotalViolentCrimes,
FROM
   `bigquery-public-data.london_crime.crime_by_lsoa`
GROUP BY
   borough
order by TotalViolentCrimes asc
Limit 10
```
<div align="center">
  <img src="https://i.imgur.com/43pQi9c.png" width="250">
</div>
<p align="center">Figure 5</p>

## What boroughs are considered to be the most dangerous and safest to live based on non-violent crime?
As Figure 6 reveals, violent crime makes up "only" about 1/3 of all London crime. So, what boroughs are considered to be the most dangerous and safest to live based on non-violent crime? Figure 7 reveals those results, while figure 8 shows the results for the safest neighborhoods based on non-violent crimes.

```sql
SELECT
   borough,
   count(*) as CrimeCount,
   COUNTIF(major_category IN ('Sexual Offences', 'Robbery', 'Violence Against the Person')) AS TotalViolentCrimes,
   FORMAT("%'.2f%%", SAFE_DIVIDE(COUNTIF(major_category IN ('Sexual Offences', 'Robbery', 'Violence Against the Person')), COUNT(*)) * 100) AS PercentViolentCrimes
FROM
  `bigquery-public-data.london_crime.crime_by_lsoa`
GROUP BY
  borough
order by TotalViolentCrimes desc
limit 10
```
<div align="center">
  <img src="https://i.imgur.com/T6h2vCm.png" width="250">
</div>
<p align="center">Figure 6</p>

```sql
SELECT
  borough,
  COUNTIF(major_category IN ('Theft and Handling', 'Fraud or Forgery', 'Criminal Damage', 'Burglary', 'Drugs')) AS TotalNonViolentCrimes,
FROM
  `bigquery-public-data.london_crime.crime_by_lsoa`
GROUP BY
  borough
order by TotalNonViolentCrimes desc
limit 10
```
<div align="center">
  <img src="https://i.imgur.com/ULODeME.png" width="250">
</div>
<p align="center">Figure 7</p>

```sql
SELECT
  borough,
  COUNTIF(major_category IN ('Theft and Handling', 'Fraud or Forgery', 'Criminal Damage', 'Burglary', 'Drugs')) AS TotalNonViolentCrimes,
FROM
  `bigquery-public-data.london_crime.crime_by_lsoa`
GROUP BY
  borough
order by TotalNonViolentCrimes asc
limit 10
```
<div align="center">
  <img src="https://i.imgur.com/WFCQv2H.png" width="250">
</div>
<p align="center">Figure 8</p>

## So, where will purchase a property?
The City of London appears to be the best place to purchase property based strictly on crime. If I wanted to further this analysis, I could consider the following:
* The average price of property per burough 
* Crime trends throughout the years. For example, is there a burough that isn't currently rated the safest, but is trending towards safe? That might make it possible to purchase a cheaper property at the present time. Unfortunately, this data set is not showing any change in crime from 2008-2016, which makes me question the validity of this dataset. 


