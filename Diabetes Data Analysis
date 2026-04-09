--Display any 10 random DM patients.

Select Concat("Firstname",' ',"Lastname") as "FullName" from "Patients" as P
Inner join "Group" as Gr on P."Group_ID"=Gr."Group_ID"
where Gr."Group"='DM' 
order by Random() 
limit 10;

--Display DM patient names with highest day MAP and night MAP.

WITH map_cte AS 
	(
	SELECT "Firstname","Lastname",("24Hr_Day_DBP"*2 + "24Hr_Day_SBP")/3 AS DAY_MAP,
	("24Hr_Night_DBP"*2 +"24Hr_Night_SBP" )/3 AS NIGHT_MAP,public."Group"."Group" FROM public."Patients" 
	JOIN public."Blood_Pressure" ON public."Patients"."Patient_ID" =public."Blood_Pressure"."Patient_ID"
	JOIN public."Group" on public."Group"."Group_ID" = public."Patients"."Group_ID"
	where public."Group"."Group" = 'DM'
	)
SELECT "Firstname","Lastname","day_map","night_map" FROM map_cte 
where "day_map" = (select max("day_map") from map_cte)
and "night_map" = (select max("night_map") from map_cte);

--Create view on table Lab Test by selecting some columns and filter data using Where condition.

Create Or Replace View "Dibetes_Lab_Test"
as
Select "Lab_ID","Patient_ID" from public."Lab_Test"
where "Fasting_Glucose" >120 and "Insulin" not between 2.6 and 24.9;

Select * from "Dibetes_Lab_Test";

-- Count of patients by first letter of firstname.
select left("Firstname",1) as first_letter,count(left("Firstname",1)) as first_letter_count from public."Patients" 
group by first_letter;

--get the list of patients whose lipid test value is null.

Select P."Patient_ID" ,"Firstname","Lastname" from "Patients" as P
Left Join public."Lipid_Lab_Test" as LP on P."Patient_ID" = LP."Patient_ID"
where LP."Fasting_Cholestrol" is null
or LP."Fasting_Triglyc" is null
or LP."Fasting_LDL" is null;

-- Find the patients that have eye damage due to diabetes.
select "Firstname","Lastname","Diabetic_Retinopathy" from public."Patients"
join public."Opthalmology" on public."Patients"."Opthal_ID"=public."Opthalmology"."Opthal_ID"
where "Diabetic_Retinopathy">0;

--Query to classify Gait RPE End into 5 categories as per the intensity.

Select "Patient_ID","Gait_RPE_End ",
CASE
when "Gait_RPE_End "='0' then 'Rest'
when "Gait_RPE_End " between '1'and'3' then 'Easy Intensity'
when "Gait_RPE_End " between '4'and'6' then 'Moderate Intensity'
when "Gait_RPE_End " between '7'and'3' then 'Hard Intensity'
when "Gait_RPE_End " = '10' then 'Max Effort intensity'
end as Intensity
from public."Walking_Test"
order by "Gait_RPE_End ";

-- get DM patient names whose distance is greater than 400 and speed is greater than 1.

select "Firstname","Lastname","Gait_DT_Distance","Gait_DT_Speed" from public."Patients"
join public."Walking_Test" on public."Patients"."WalkTest_ID"=public."Walking_Test"."WalkTest_ID"
join public."Group" on public."Group"."Group_ID" = public."Patients"."Group_ID"
where public."Group"."Group" = 'DM' and "Gait_DT_Distance">400 and "Gait_DT_Speed">1;

--Create a trigger to raise notice and prevent the deletion of a record from a view.
	
-- Create view
Create Or Replace View Obese_patients As
Select "Patient_ID",CONCAT("Firstname",' ',"Lastname") AS "Full_name","Age", "Height", "BMI"
From public."Patients"
Where "BMI">25;
Select * From Obese_patients;
-- Create Function
Create Function public.prevent_delete()
Returns Trigger AS $$
Begin Raise EXCEPTION 'Record cannot be deleted';
end; $$
LANGUAGE plpgsql;
-- Create Trigger
CREATE OR REPLACE TRIGGER record_delete_trigger
 INSTEAD OF DELETE
 ON public.obese_patients
 FOR EACH ROW
 EXECUTE FUNCTION public.prevent_delete();
-- Deleting a row
 Delete from public.obese_patients where "Patient_ID" = 'S0033';
 
 -- Get the number of patients in the year 2005 in each of the Genesis and Cultivate labs.

select count("Patient_ID"), 'Genesis' as lab_name from public."Lab_Visit" 
join public."Link_Reference" on public."Link_Reference"."Lab_visit_ID" = public."Lab_Visit"."Lab_visit_ID"
join public."Patients" on public."Patients"."Link_Reference_ID" = public."Link_Reference"."Link_Reference_ID"
where "Lab_names" = 'Genesis Lab' and extract(year from "Lab_Visit_Date")='2005' group by "Lab_names"
union
select count("Patient_ID"), 'Cultivate' as lab_name from public."Lab_Visit" 
join public."Link_Reference" on public."Link_Reference"."Lab_visit_ID" = public."Lab_Visit"."Lab_visit_ID"
join public."Patients" on public."Patients"."Link_Reference_ID" = public."Link_Reference"."Link_Reference_ID"
where "Lab_names" = 'Cultivate Lab' and extract(year from "Lab_Visit_Date")='2005' group by "Lab_names";

--get a list of patient IDs' and their Fasting Cholesterol in February 2006.

SELECT  P."Patient_ID",L."Fasting_Cholestrol","Visit_Date" 
FROM public."Patients" AS P
LEFT JOIN public."Lipid_Lab_Test" AS L ON P."Patient_ID"=L."Patient_ID" 		 
WHERE 
P."Visit_Date"::DATE BETWEEN '2006-02-01':: DATE AND '2006-02-28':: DATE;
 
-- get the number of patients who visited the Lab between 9 am to 12 am.

select count("patient_count") as total_patients from (select count("Patient_ID") as patient_count from public."Patients"
join public."Link_Reference" on public."Link_Reference"."Link_Reference_ID" = public."Patients"."Link_Reference_ID"
join public."Lab_Visit" on public."Lab_Visit"."Lab_visit_ID" = public."Link_Reference"."Lab_visit_ID"
where extract(hour from public."Lab_Visit"."Lab_Visit_Date") between '9' and '11'
group by public."Lab_Visit"."Lab_Visit_Date") as patient_number ;

-- find the number of patients visited each month. (Display with month Name)

select to_char("Visit_Date", 'Month') as Month_Name, count("Patient_ID") as Num_patients_visited from public."Patients"
group by Month_Name;

--get a number of visual/motor dementia patients who have any 2 abnormal conditions.

SELECT (CASE
		When "RCFT_IR"<=71 And "TM">=42 Then 'RCFT & TM'
        When "RCFT_IR"<=71 And "Clock"<=2 Then 'RCFT & Clock'
        When "TM">=42 And "Clock"<=2 Then 'TM & Clock'
Else ''
End) As "Condition_Name", COUNT(*)
From "Visual/Motor_Cog"
Where ("RCFT_IR"<=71 And "TM">=42)
 Or ("RCFT_IR"<=71 And "Clock"<=2)
 Or ("TM">=42 And "Clock"<=2)
Group By "Condition_Name";

--calculate Creatinine ALbumin Ratio (uCAR) For DM Patients.

select "Firstname", "Lastname", round(("Creatinine"/"Albumin") :: numeric,2) as uCAR from public."Patients"
join public."Group" on public."Group"."Group_ID" = public."Patients"."Group_ID"
join public."Link_Reference" on public."Link_Reference"."Link_Reference_ID"=public."Patients"."Link_Reference_ID"
join public."Urine_Test" on public."Urine_Test"."Urine_ID" = public."Link_Reference"."Urine_ID"
where public."Group"."Group" = 'DM';

-- get the number of patients whose urine creatinine is in a normal range (Gender wise).

Select G."Gender", COUNT(P."Patient_ID") as "Patients_With_Normal_Creatinine"
from "Urine_Test" AS UT
     Inner Join "Link_Reference" AS LR
     on UT."Urine_ID" = LR."Urine_ID"
	 Inner join "Patients" AS P
	 on LR."Link_Reference_ID" = P."Link_Reference_ID"
	 Inner Join "Gender" as G
	 on P."Gender_ID" = G."Gender_ID" 
Where (G."Gender" ='Male' and UT."Creatinine" BETWEEN 65.4 and 119.3) 
or (G."Gender" = 'Female' and UT."Creatinine" BETWEEN 52.2 and 91.9)
Group by  G."Gender";


--Number of Diabetic Male and Female patients who are Anemic.

Select "Gender", Count(P."Patient_ID") AS "Diabetic_And_Anemic_Patient"
FROM "Lab_Test" AS L 
	Inner Join  "Patients" As P 
	On P."Patient_ID" = L."Patient_ID"
	Inner Join  "Gender" As G 
	On P."Gender_ID" = G."Gender_ID"
Where (("Hb_A1C" >= 6.5 ) 
	   Or ("Fasting_Glucose"  >= 120)) 
	   And ((("Gender" Like '%Male%')
			 And ("Hgb" < 13.2 ))			
			Or (("Gender" Like 'Female%')
			 And ("Hgb" < 11.6 )))
Group By "Gender";

--list of unique lab names whose names is starting with G and end with b

Select distinct ("Lab_names")
 	From "Lab_Visit"
	Where "Lab_names" like 'G%b'
	
-- Find the patient who has the most damage in the eyes with the use of a max function.

select "Firstname","Lastname" from public."Patients"
join public."Opthalmology" on public."Patients"."Opthal_ID"=public."Opthalmology"."Opthal_ID"
where "Diabetic_Retinopathy" = (select max("Diabetic_Retinopathy") from public."Opthalmology");

--Create a procedure for checking if Race exists using an if else statement.

--To create procedure
Create or Replace Procedure race_exists(
	inputRace varchar)
Language plpgsql
AS $$
Begin
    --checking if race exists 
	If (Select Count(*) From "Race" Where "Race" = inputRace)>0
	Then Raise Notice 'RACE EXISTS';
	Else Raise Notice 'RACE DOES NOT EXISTS';
    End If;
End $$;

--To call procedure:
CALL race_exists('');

--  using the trigger after insert on the lab test table.
--If the patient has abnormal HbA1C and fasting glucose values.

create or replace function insert_lab_test()
returns trigger
language plpgsql
as
$$
begin
if NEW."Hb_A1C" >=5.7 and (NEW."Fasting_Glucose" =< 70 or NEW."Fasting_Glucose">=100) then
RAISE NOTICE 'Patient information having Abnormal HbA1C and fasting glucose values inserted';
end if;
 RETURN NEW;
END; $$

create trigger insert_lab_test_trigger
after insert
on public."Lab_Test"
for each row
execute function insert_lab_test();

insert into public."Lab_Test" values('LB080', 'SO616', 5.4,4.56, 13.2, 43.2, 213, 8, 1, 15.34);
select * from public."Lab_Test";

	
--get the number of patients for each age bin without using the CASE statement.(Bin size - 5)

Select patient_age_bin, Count(*) AS patients_for_each_bin from
(Select ("Firstname"||' '||"Lastname") As "Patientname", "Age",
WIDTH_BUCKET("Age",0,120,24) As patient_age_bin 
From "Patients"
Order By patient_age_bin
) As new_table
Group By patient_age_bin;

--Display patients names who have the same last name.

SELECT "Firstname","Lastname" from "Patients"
where "Lastname" IN (
select  "Lastname" from "Patients"
GROUP BY "Lastname"
HAVING COUNT("Lastname")>1)
order by "Lastname";

-- find out the percentage of Lab visits by Lab names.

select "Lab_names", 
round((count("Lab_visit_ID"):: decimal/(select count(*) from  public."Lab_Visit")::decimal) * 100::numeric,2) 
as percentage_lab_visit
from public."Lab_Visit" group by "Lab_names"

--get Patient IDs for verbally cognitively impaired who satisfy any 2 conditions. (HINT: dementia/cognitive impaired: any patient who has any two abnormal test results).

Select p."Patient_ID",p."Firstname"|| ' '||p."Lastname" fullname From "Patients" p
Inner Join "Verbal_Cognitive" vc
On p."Patient_ID" = vc."Patient_ID"
Where (vc."DS" < 13 And vc."HVLT" < 14) OR
(vc."DS" < 13 And vc."VF" < 42) OR
(vc."DS" < 13 And vc."WTAR" <= 20) OR
(vc."HVLT" < 14 And vc."VF" < 42) OR
(vc."VF" <42 And vc."WTAR" <=20) O
(vc."HVLT" < 14 And vc."WTAR" <= 20);

--Find the number of control and DM patients who visited each lab.

select "Group","Lab_names",count("Patient_ID") from public."Patients"
join public."Group" on public."Group"."Group_ID"=public."Patients"."Group_ID"
join public."Link_Reference" on public."Patients"."Link_Reference_ID"=public."Link_Reference"."Link_Reference_ID"
join public."Lab_Visit" on public."Lab_Visit"."Lab_visit_ID"=public."Link_Reference"."Lab_visit_ID"
group by "Lab_names","Group" order by "Group";

