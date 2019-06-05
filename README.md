# PL-SQL

## introduction

My task is to build a PL/SQL-based application to insert and update records in a video rental store database and generate some reports. 

The database consists of only the following essential tables.

•	CUSTOMER(CUSTOMER_ID, CUSTOMER_NAME, EMAIL, PHONE_NUMBER,
     REGISTRATION_DATE, EXPIRATION_DATE, LAST_UPDATE_DATE);

•	VIDEO(VIDEO_ID, VIDEO_NAME, FORMAT, PUBLISH_DATE,
  MAXIMUM_RENTAL_DAYS);

•	VIDEO_COPY(COPY_ID, VIDEO_ID*, COPY_STATUS, NOTE);

•	RENTAL_HISTORY (CUSTOMER_ID*, COPY_ID*, CHECKOUT_DATE,
   DUE_DATE, RETURN_DATE);


The primary keys are marked in red and the foreign keys are marked with asterisks.

VIDEO_COPY(COPY_STATUS): 	A – Available, R – Rented, D – Damaged

Each video in the VIDEO table has at least one video copy in the VIDEO_COPY table.

## part 1: Create table

Code for create 4 tables and import data to the tables
```
//(1)Customer table
CREATE TABLE customer 
( CUSTOMER_ID    		NUMBER 	 PRIMARY KEY,
  CUSTOMER_NAME      	VARCHAR2(30) NOT NULL,
  EMAIL   			VARCHAR2(50) NOT NULL,
  PHONE_NUMBER		VARCHAR2(15) NOT NULL,
  REGISTRATION_DATE	DATE NOT NULL,
  EXPIRATION_DATE  	DATE NOT NULL, 
  LAST_UPDATE_DATE	DATE NOT NULL);
/
INSERT INTO customer SELECT * FROM ####.customer; // #### is from other database and import to this database
COMMIT;
/
SELECT COUNT(*) FROM customer;


//(2) Video table
CREATE TABLE video 
( VIDEO_ID			NUMBER 		PRIMARY KEY,
  VIDEO_NAME     		VARCHAR2(100) 	NOT NULL,
  FORMAT			VARCHAR2(100) 	NOT NULL,
  PUBLISH_DATE  		DATE 			NOT NULL,
  MAXIMUM_RENTAL_DAYS	NUMBER(3) 		NOT NULL
); 
/
INSERT INTO video SELECT * FROM ####.video;
COMMIT;
/
SELECT COUNT(*) FROM video;

//(3) video copy
CREATE TABLE video_copy 
( COPY_ID       	NUMBER 	PRIMARY KEY,
  VIDEO_ID      	NUMBER 	NOT NULL REFERENCES VIDEO (VIDEO_ID),
  COPY_STATUS   	CHAR NOT NULL CONSTRAINT ck_item 
				CHECK (COPY_STATUS in ('A', 'R', 'D')),
  NOTE		VARCHAR2(200));
/
INSERT INTO video_copy SELECT * FROM ####.video_copy;
COMMIT;
/
SELECT COUNT(*) FROM video_copy;


// (4)rental history data
CREATE TABLE rental_history
( CUSTOMER_ID    	NUMBER 	REFERENCES CUSTOMER (CUSTOMER_ID),
  COPY_ID  		NUMBER 	REFERENCES VIDEO_COPY (COPY_ID),
  CHECKOUT_DATE   DATE NOT NULL,
  DUE_DATE  	DATE NOT NULL,
  RETURN_DATE  	DATE,
  NOTE		VARCHAR2(200),
  CONSTRAINT 	pk_rental PRIMARY KEY 
				(CUSTOMER_ID, COPY_ID, CHECKOUT_DATE));
/
INSERT INTO rental_history SELECT * FROM ####.rental_history;
COMMIT;
/
SELECT COUNT(*) FROM rental_history;
```
## Part 2
### (1)new_customer_registration()
* Create a procedure called new_customer_registration to add a new customer to the CUSTOMER table. 
```
date, 'yyyymmdd') < 
TO_CHAR(in_registration_date, 'yyyymmdd') THEN
          	DBMS_OUTPUT.PUT_LINE('Invalid expiration date!');
		RETURN;
	END IF;
  
	INSERT INTO customer 
       	VALUES(in_customer_id, UPPER(in_customer_name), in_email, 
			in_phone_number, in_registration_date, in_expiration_date,
sysdate);
	COMMIT;
	
      DBMS_OUTPUT.PUT_LINE
		(INITCAP(in_customer_name) || 
' has been added into the customer table.');
		
EXCEPTION
	WHEN OTHERS THEN
		DBMS_OUTPUT.PUT_LINE('My exception: ' || 
			TO_CHAR(SQLCODE) || '   ' || SQLERRM);
END;	

```
* Testing the procedure

```
•	EXEC new_customer_registration(2009, 'Adams', 'adams_1@yahoo.com', '3123621111', '02-OCT-2016', '01-OCT-2018')
Dbms Output: 	Adams has been added into the customer table.

•	EXEC new_customer_registration(2010, '', 'ford1@yahoo.com', 
'3123622222', '02-OCT-2016', '01-OCT-2018')
	Dbms Output:	Invalid customer name!
•	……
```
### (2)reset_expiration_date()
* Create a procedure called reset_expiration_date to reset an existing customer’s expiration date. 
### (3)video_search()
* Create a procedure called video_search to search a video and display the name, copy ID, format, and status of the video’s copies. In addition, the checkout dates and due dates are also displayed for unreturned copies. The damaged copies (COPY_STATUS = 'D') are excluded in your output. Sort your output by the video name (NAME) and then the copy ID (COPY_ID).

* The video name is not case-sensitive (e.g., Ocean = ocean). The format of a video is not case-sensitive (e.g., DVD = dvd).
* Testing the procedure

(If your output does not match mine EXACTLY, you will lose some points.)
```
•	EXEC video_search('ocean')
	Dbms Output: 

	***** 0 results found for ocean. *****

•	EXEC video_search('PRETTY WOMAN', 'Blu-Ray')
	Dbms Output:

	***** 0 results found for PRETTY WOMAN (Blu-Ray). *****

•	EXEC video_search('Pretty Woman')
	Dbms Output:

	***** 3 results found for Pretty Woman. (Available copies: 3) *****

 
	VIDEO NAME           	 COPY ID    FORMAT      COPY STATUS    CHECKOUT DATE       DUE DATE
	---------------------------------------------------------------------------------------------
	PRETTY WOMAN                  6000    VHS TAPE    Available
	PRETTY WOMAN                  6001    VHS TAPE    Available
	PRETTY WOMAN                  6015    DVD         Available

•	EXEC video_search('Another')
	Dbms Output:

       ***** 4 results found for Another. (Available copies: 2) *****
 
	VIDEO NAME           	 COPY ID    FORMAT      COPY STATUS    CHECKOUT DATE       DUE DATE
	---------------------------------------------------------------------------------------------
	DIE ANOTHER DAY               6010    VHS TAPE    Available
DIE ANOTHER DAY               6011    VHS TAPE    Rented           20-APR-2016    04-MAY-2016
DIE ANOTHER DAY               6014    DVD         Available
DIE ANOTHER DAY               6016    BLU-RAY     Rented           01-OCT-2016    04-OCT-2016

•	EXEC video_search('ANOTHER', 'Dvd')
	Dbms Output:

	***** 1 result found for ANOTHER (Dvd). (Available copies: 1) *****
 
	VIDEO NAME           	 COPY ID    FORMAT      COPY STATUS    CHECKOUT DATE       DUE DATE
	---------------------------------------------------------------------------------------------
	DIE ANOTHER DAY               6014    DVD         Available

•	EXEC video_search('Story')
	Dbms Output:

	***** 7 results found for Story. (Available copies: 4) *****
 
	VIDEO NAME           	 COPY ID    FORMAT      COPY STATUS    CHECKOUT DATE       DUE DATE
	---------------------------------------------------------------------------------------------
	TOY STORY                     6002    VHS TAPE    Rented           09-APR-2016    23-APR-2016
TOY STORY                     6003    VHS TAPE    Available
TOY STORY                     6017    DVD         Rented           01-OCT-2016    08-OCT-2016
TOY STORY 2                   6009    VHS TAPE    Available
TOY STORY 2                   6018    DVD         Rented           28-APR-2016    05-MAY-2016
TOY STORY 2                   6019    DVD         Available
TOY STORY 2                   6020    BLU-RAY     Available
```

### (4)video_checkout()
* Create a procedure called video_checkout to record a new rental. When the video is successfully checked out, you need to insert a new row (record) into the RENTAL_HISTORY table and update the corresponding row (record) in the VIDEO_COPY table. Otherwise, the action is denied. 

### (5)video_return()
### (6)print_unreturned_video()
### (7)Package video_rental_pkg
