# Challenge 2
# Detective SQL

A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a murder that occurred sometime on Jan.15, 2018 and that it took place in SQL City. Start by retrieving the corresponding crime scene report from the police department’s database.

Follow these steps to solve this challenge:

<li> Download the sql-murder-mystery.db file here https://drive.google.com/drive/folders/1SLlSSzIqhu9m4p8HmoJYjn5X_GTYdDsf?usp=share_link
Visit www.sqliteonline.com </li>
<li> Click on file, then open db and load in the database file you downloaded above </li>
<li> Write your SQL queries to see the different tables and the content </li>
<li> Use the Schema Diagram from the Google Drive Folder above to navigate the different tables </li>
<li> Figure out who committed the crime with the details you remembered above </li>
<li> Create a word or txt document that contains your thought process, narrative and SQLcodes written to arrive at the solution to the murder mystery </li>
<li> Submit a Google Drive or OneDrive link to the word or text document </li>

# Answer

### 1. Load sql-murder-mistery.db

### 2. The starting information we have are:

The date: Jan.15, 2018 <br>
The type of the crime: murder <br>
The place: SQL City <br>

Based on the schema given, we see that the crime_scene_report table could provide more information if we filter it with the date, type and place. But we do not know the format of the date, so first we will run the following query:

```
SELECT * FROM crime_scene_report
```

### 3. Based on the previous step result we know that date must be yyyymmdd format, so we run the following query to find out more information about the crime we are investigating:

```
SELECT * FROM crime_scene_report
WHERE date = 20180115 AND type = "murder" AND city = "SQL City"
```

Description reads: Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

### 4. Based on the new information we need to know who the witnesses are and which is each person_id in order to access the transcript of their interview. For that we will now explore the person table.

The first witness lives at the last house on "Northwestern Dr". So we run the following query to see who lives in the last house of that given address.

```
SELECT * FROM person
WHERE address_street_name = "Northwestern Dr"
ORDER BY address_number DESC
```

Based on the output, Morty Schapiro is the person and the id is 14887.

The second witness, named Annabel, lives somewhere on "Franklin Ave". So we run the following query:

```
SELECT * FROM person
WHERE address_street_name = "Franklin Ave"
AND name LIKE "%Annabel%" 
```

Based on the output, Annabel Miller is the person and the id is 16371.

### 5. Knowing our witnesses id we can explore the interview table.

For the first witness we run the following query: 

```
SELECT * FROM interview
WHERE person_id = "14887"
```

First witness transcript reads: I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

For the second witness we run the following query:

```
SELECT * FROM interview
WHERE person_id = "16371"
```

Second witness transcript reads: I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

### 6. First witness lead:

With the information form the first witness let’s try to figure out who is the hitman by exploring the drivers_license table and using the partial plate number.

```
SELECT * FROM drivers_license
WHERE plate_number LIKE "%H42W%"
```

We have obtained 3 different results, but from the witness transcript we can try to filter those 3 entries by membership number and membership status. So first we use the persons table to get the id of our 3 suspects.

```
SELECT * FROM person
WHERE license_id = 183779 OR license_id = 423327 OR license_id = 664760
```

So now we will turn to the get_fit_now_member table to check which of the 3 suspects has a gold membership. 

```
SELECT * FROM get_fit_now_member
WHERE name LIKE "%Jeremy%"
```

By running the following query and replacing the name for each of the suspects we see that Jeremy Bowers is the only suspect that matches the description of the first witness: the membership (id) contains 48Z, the plate number matches the information given and the membership is gold.

### 7. Second witness lead:

The finding from the first witness leads us to Jeremy Bowers, now we can use the second witness information to check if Jeremy Bowers was working out on January 9th 2018. So we run the following query to verify this information (keeping in mind that dates are in yyyymmdd):

```
SELECT * FROM get_fit_now_check_in
WHERE membership_id = "48Z55" AND date = 20180109
```

And yes! Jeremy Bowers was working out between 15h30 and 17h00 on January 09th 2018. We can even check if our witness attended the gym in the same time frame to ensure they were at the gym at the same time and that the witness could have seen the hitman:

```
SELECT * FROM get_fit_now_check_in
WHERE check_in_date = 20180109 AND check_in_time >= 1530 AND check_out_time <= 1700
```

Our only match is the person with membership id 90081, that if we run the following query will reveal our second witness Annabel Miller:

```
SELECT * from get_fit_now_member
WHERE id = 90081
```

### 8. What did Jeremy Bowers has to say?

Based on the process described in this document, the person who committed the crime is Jeremy Bowers, but why? Was he working alone or did he have a partner in crime? Let’s verify his interview statement to see if he had anything to say, by running the following query:

```
SELECT * FROM interview
WHERE person_id = "67318"
```

Bingo! The transcript says: I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

Let’s use the drivers_license table to filter possible suspects with the following query:

```
SELECT * FROM drivers_license
WHERE car_make = "Tesla" AND car_model = "Model S" AND hair_color = "red" AND height <= 67
```

And we have 3 suspects! Next step we will filter to see which of the suspects attended SQL Symphony Concert 3 times in December 2017. 

```
SELECT * FROM facebook_event_checkin
WHERE person_id = 99716 OR person_id = 90800 OR person_id = 78881
```

And we verify that Miranda Priestly is the only suspect that attended SQL Symphony Concert 3 times in December 2017. So we add Miranda Priestly in the solution table and check the result:

```
INSERT INTO solution
VALUES ('1', 'Miranda Priestly')
```

There is no statement from Miranda Priestly in the records, but by running the following query we can confirm that we have found the criminal mastermind

```
SELECT * FROM solution
```

Output: Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!

**Jeremy Bowers** and **Miranda Priestly** are the criminals behind the murder in SQL City!

#### Witness 1: **Morty Schapiro**
#### Witness 2: **Annabel Miller**
#### Hitman: **Jeremy Bowers**
#### Crime mastermind: **Miranda Priestly**
