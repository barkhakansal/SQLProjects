--get a list of patients with event type of EGV and  glucose (mgdl) greater than 155 .

SELECT DISTINCT DEMOGRAPHICS.PATIENTID
FROM DEMOGRAPHICS
JOIN DEXCOM ON DEMOGRAPHICS.PATIENTID = DEXCOM.PATIENTID
JOIN EVENTTYPE ON EVENTTYPE.ID = DEXCOM.EVENTID
WHERE 
	EVENTTYPE.EVENT_TYPE = 'EGV'
	AND DEXCOM.GLUCOSE_VALUE_MGDL > 155;

--patients consumed meals with at least 20 grams of protein in it?

SELECT COUNT(DISTINCT(PATIENTID)) FROM FOODLOG 
WHERE PROTEIN >=20;

--Who consumed maximum calories during dinner? (assuming the dinner time is after 6pm-8pm)

SELECT CONCAT(DEMOGRAPHICS.FIRSTNAME,' ',DEMOGRAPHICS.LASTNAME) AS FULLNAME, CALORIE FROM DEMOGRAPHICS
JOIN FOODLOG ON DEMOGRAPHICS.PATIENTID = FOODLOG.PATIENTID
WHERE 
	CAST (DATETIME::TIMESTAMP AS TIME) BETWEEN '18:00:00' AND '20:00:00'
ORDER BY 
	CALORIE DESC
LIMIT 1;

--patient who showed a high level of stress on most days recorded for him/her?

SELECT CONCAT(FIRSTNAME,' ',LASTNAME) AS FULLNAME
FROM
	(SELECT IBI.PATIENTID,
			EXTRACT (DAY FROM IBI.DATESTAMP) AS DAY,
			AVG(RMSSD_MS) * 600 AS HRV,
			EDA.MEAN_EDA,
			HR.MEAN_HR,
			DEMOGRAPHICS.FIRSTNAME,
			DEMOGRAPHICS.LASTNAME
		FROM IBI
		JOIN EDA ON EDA.PATIENTID = IBI.PATIENTID
		JOIN HR ON HR.PATIENTID = EDA.PATIENTID
		JOIN DEMOGRAPHICS ON DEMOGRAPHICS.PATIENTID = HR.PATIENTID
		WHERE 
	 		EXTRACT (DAY FROM IBI.DATESTAMP) = EXTRACT (DAY FROM EDA.DATESTAMP)
			AND EXTRACT (DAY FROM IBI.DATESTAMP) = EXTRACT (DAY FROM HR.DATESTAMP)
		GROUP BY 
	 		IBI.PATIENTID,EXTRACT (DAY FROM IBI.DATESTAMP), MEAN_EDA,
			MEAN_HR,
			DEMOGRAPHICS.FIRSTNAME,
			DEMOGRAPHICS.LASTNAME
		HAVING 
	 		MEAN_HR > 100
			OR MEAN_EDA > 40
			OR (AVG(RMSSD_MS) * 600) < 20) AS STRESS_LEVEL
GROUP BY 
	PATIENTID,
	FIRSTNAME,
	LASTNAME
ORDER BY 
	COUNT(PATIENTID) DESC
	LIMIT 1;

-- least healthy patient Based on mean HR and HRV 

select concat(firstname,' ',lastname) as fullname from demographics
join hr on hr.patientid=demographics.patientid
join ibi on demographics.patientid=ibi.patientid
group by 
	demographics.firstname,
	demographics.lastname,
	hr.mean_hr
having 
	hr.mean_hr>100 
	or (avg(rmssd_ms)*600) <20
order by 
	count(demographics.patientid)  
limit 1;

-- percentage of the male and female 
SELECT
ROUND(((COUNT(1) FILTER (WHERE gender = 'MALE'))/CAST(COUNT(*) AS DECIMAL))*100,2) AS MALE_PERCENT,
ROUND(((COUNT(1) FILTER (WHERE gender = 'FEMALE'))/CAST(COUNT(*) AS DECIMAL))*100,2) AS FEMALE_PERCENT
FROM DEMOGRAPHICS;

--patient with the highest max eda?

SELECT CONCAT(DEMOGRAPHICS.FIRSTNAME,' ',DEMOGRAPHICS.LASTNAME) AS FULLNAME FROM DEMOGRAPHICS
JOIN EDA ON EDA.PATIENTID=DEMOGRAPHICS.PATIENTID
WHERE 
	MAX_EDA = (SELECT MAX(MAX_EDA) FROM EDA);

--list prediabetic patients.

SELECT * FROM demographics
WHERE hba1c BETWEEN 5.7 AND 6.4;

-- patients that fall into the highest EDA category by name, gender and age

SELECT DISTINCT(d.firstname), d.gender, EXTRACT(YEAR FROM age(current_date, dob)) AS age
FROM demographics d
LEFT JOIN eda e
ON d.patientid = e.patientid
WHERE e.max_eda > 40;

--patients have names starting with 'A'?

SELECT COUNT(*) FROM demographics
WHERE firstname LIKE 'A%' ;

--Display the Date and Time in 2 seperate columns for the patient who consumed only Egg

SELECT DATE(datetime) AS date_only,
datetime::timestamp::time
FROM foodlog
WHERE logged_food ILIKE 'Egg';

--Display list of patients along with the gender and hba1c for whom the glucose value is null.

SELECT distinct(d.patientid), d.gender, d.hba1c, dx.glucose_value_mgdl
FROM dexcom dx
LEFT JOIN demographics d
ON dx.patientid = d.patientid
WHERE dx.glucose_value_mgdl IS null;

--Patient ID's whose sugar consumption exceeded 30 grams on a meal from FoodLog table. 

SELECT DISTINCT PATIENTID FROM FOODLOG 
WHERE 
	SUGAR>30 
ORDER BY 
	PATIENTID;

--patients who are celebrating their birthday this month?

SELECT COUNT(PATIENTID) FROM DEMOGRAPHICS
WHERE 
	EXTRACT(MONTH FROM DOB) = EXTRACT(MONTH FROM CURRENT_TIMESTAMP);

--count of types of events  recorded in the Dexcom tables. 

SELECT EVENTTYPE.EVENT_TYPE,COUNT(EVENTID) AS EVENT_COUNT FROM DEXCOM
JOIN EVENTTYPE ON DEXCOM.EVENTID=EVENTTYPE.ID
GROUP BY 
	EVENT_TYPE;

--prediabetic/diabetic patients also had a high level of stress? 

SELECT COUNT(PATIENTID) AS patient_count
FROM (
    SELECT DEMOGRAPHICS.PATIENTID
    FROM DEMOGRAPHICS
    JOIN HR ON DEMOGRAPHICS.PATIENTID = HR.PATIENTID
    JOIN IBI ON HR.PATIENTID = IBI.PATIENTID
    JOIN EDA ON IBI.PATIENTID = EDA.PATIENTID
    WHERE 
		HBA1C BETWEEN 5.7 AND 6.4 
		OR HBA1C >= 6.5
    GROUP BY 
		DEMOGRAPHICS.PATIENTID
    HAVING 
		MAX(mean_hr) > 100 OR 
		(AVG(rmssd_ms) * 600) < 20 OR 
		MAX(mean_eda) > 40
) AS high_stress;


--List the food that coincided with the time of highest blood sugar for every patient

select * from dexcom where patientid = '1' and glucose_value_mgdl = '155'
WITH max_glucose_cte AS (
    SELECT 
        d1.patientid,
        d1.glucose_value_mgdl as highest_glucose_value,
        d1.DATESTAMP as highest_glucose_time
    FROM dexcom d1
    WHERE 
        d1.glucose_value_mgdl = (
            SELECT MAX(d2.glucose_value_mgdl)
            FROM dexcom d2
            WHERE d1.patientid = d2.patientid
        )
)
SELECT f.patientid,f.logged_food,highest_glucose_time,highest_glucose_value
FROM max_glucose_cte m
JOIN foodlog f ON m.patientid = f.patientid
              where m.highest_glucose_time::timestamp::date = f.datetime::timestamp::date
			  and extract(hour from highest_glucose_time) = extract(hour from datetime);


--patients have first names with length >7 letters?

SELECT COUNT(PATIENTID) FROM DEMOGRAPHICS 
WHERE LENGTH(FIRSTNAME)>7;

--List all foods logged that end with 'se'. Ensure that the output is in Title Case

SELECT DISTINCT INITCAP(LOGGED_FOOD) FROM FOODLOG WHERE LOGGED_FOOD LIKE '%se';

--List the patients who had a birthday the same week as their glucose or IBI readings

SELECT DISTINCT D.PATIENTID
FROM DEMOGRAPHICS D
JOIN IBI I ON D.PATIENTID = I.PATIENTID AND EXTRACT(WEEK FROM I.DATESTAMP) = EXTRACT(WEEK FROM D.DOB)
LEFT JOIN DEXCOM DX ON D.PATIENTID = DX.PATIENTID AND EXTRACT(WEEK FROM DX.DATESTAMP) = EXTRACT(WEEK FROM D.DOB) - 1
WHERE I.PATIENTID IS NOT NULL OR DX.PATIENTID IS NOT NULL;


--Assuming breakfast is between 8 am and 11 am. How many patients ate a meal with bananas in it?

SELECT COUNT(DISTINCT(patientid))
FROM foodlog
WHERE logged_food ILIKE '%banana%'
AND EXTRACT(HOUR FROM datetime::timestamp) BETWEEN 8 AND 10;

--list of patients whose daily max_eda lies between 40 and 50.

select patientid , max_eda
from eda
where max_eda between 40 and 50;

--Count the number of hyper and hypoglycemic incidents per patient

select patientid, 
	SUM(CASE WHEN glucose_value_mgdl < 70 THEN 1 ELSE 0 END) AS hypoglycemic_incidents,
    SUM(CASE WHEN glucose_value_mgdl >= 126 THEN 1 ELSE 0 END) AS hyperglycemic_incidents
from dexcom
group by patientid;

















