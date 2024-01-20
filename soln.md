## Q1 
a) 
```shell 
[postgres@hsm ~]$ psql
psql (16.1)
Type "help" for help.

postgres=# CREATE DATABASE MOVIES
postgres-# \l
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 movies    | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 test      | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
(5 rows)
```
c) <-- this is the C part here
```shell 
postgres-# \c movies
You are now connected to database "movies" as user "postgres".

movies=# CREATE TABLE Review (
    Rew_ID SERIAL PRIMARY KEY,
    Rating FLOAT,
    Text TEXT
);
CREATE TABLE

movies=# CREATE TABLE movie (
    movie_id SERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    year DATE NOT NULL,
    genre INT NOT NULL );
CREATE TABLE

movies=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | movie  | table | postgres
 public | review | table | postgres
(2 rows)

movies=# \d
                 List of relations
 Schema |        Name        |   Type   |  Owner   
--------+--------------------+----------+----------
 public | movie              | table    | postgres
 public | movie_movie_id_seq | sequence | postgres
 public | review             | table    | postgres
 public | review_rew_id_seq  | sequence | postgres
(4 rows)
```

b ) 
```shell 
movies=# ALTER TABLE Review
ADD COLUMN movie_id INT,
ADD CONSTRAINT FK_Review_Movie FOREIGN KEY (movie_id) REFERENCES movie(movie_id);
ALTER TABLE
```

## Q2
```shell 
movies=# INSERT INTO movie (name, year, genre)
VALUES
    ('Movie 1', '2022-01-01', 1),
    ('Movie 2', '2023-02-15', 2),
    ('Movie 3', '2021-11-30', 3);
INSERT 0 3

movies=# SELECT * FROM movie ;
 movie_id |  name   |    year    | genre 
----------+---------+------------+-------
        1 | Movie 1 | 2022-01-01 |     1
        2 | Movie 2 | 2023-02-15 |     2
        3 | Movie 3 | 2021-11-30 |     3
(3 rows)

movies=# INSERT INTO Review (Rating, Text, movie_id)
VALUES
    (4.5, 'Great movie! Loved the plot and the acting.', 1),
    (3.8, 'Interesting storyline, but the pacing could be better.', 2),
    (5.0, 'Hilarious comedy with an amazing cast.', 3);
INSERT 0 3
movies=# SELECT * FROM Review ;
 rew_id | rating |                          text                          | movie_id 
--------+--------+--------------------------------------------------------+----------
      4 |    4.5 | Great movie! Loved the plot and the acting.            |        1
      5 |    3.8 | Interesting storyline, but the pacing could be better. |        2
      6 |      5 | Hilarious comedy with an amazing cast.                 |        3
(3 rows)
```



## Q3
```shell 
movies=# CREATE TABLE genre (
    genre_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
CREATE TABLE
movies=# INSERT INTO genre (name) VALUES
    ('Action'),
    ('Drama'),
    ('Comedy');
INSERT 0 3
movies=# CREATE VIEW vMovie_detailed AS
SELECT                          
    M.name AS Name,
    G.name AS Genre,
    M.year AS Year,
    R.rating AS Rating
FROM
    movie M
    LEFT JOIN genre G ON M.genre = G.genre_id
    LEFT JOIN review R ON M.movie_id = R.movie_id
ORDER BY
    M.year DESC;
CREATE VIEW

movies=# SELECT * FROM vMovie_detailed;
  name   | genre  |    year    | rating 
---------+--------+------------+--------
 Movie 2 | Drama  | 2023-02-15 |    3.8
 Movie 1 | Action | 2022-01-01 |    4.5
 Movie 3 | Comedy | 2021-11-30 |      5
(3 rows)
```

## Q4
```shell 
movies=# SELECT
    movie_id,
    name,
    year,
    genre,
    AGE(CURRENT_DATE, year) AS DATEDIFF
FROM
    movie;
 movie_id |  name   |    year    | genre |       datediff        
----------+---------+------------+-------+-----------------------
        1 | Movie 1 | 2022-01-01 |     1 | 2 years 19 days
        2 | Movie 2 | 2023-02-15 |     2 | 11 mons 5 days
        3 | Movie 3 | 2021-11-30 |     3 | 2 years 1 mon 20 days
(3 rows)
```

## Q5

```shell 
movies=# CREATE TABLE email (
    email_id SERIAL PRIMARY KEY,
    email_address VARCHAR(200) NOT NULL
);
CREATE TABLE

movies=# INSERT INTO email (email_address) VALUES
    ('abhihere@example.com'),
    ('user@mail.com'),
    ('my.abhi.mail.@com.ru'),
    ('inv-email');
INSERT 0 4

movies=# CREATE OR REPLACE FUNCTION check_email(email_address VARCHAR(200))
RETURNS VOID AS $$
DECLARE
    index_atsign INT;
    index_dot INT;
BEGIN
    index_atsign := POSITION('@' IN email_address);
    index_dot := POSITION('.' IN email_address);

    IF index_atsign > 0 THEN
        IF index_dot > index_atsign THEN
            RAISE NOTICE 'Email is correct';
        ELSE
            RAISE NOTICE 'Email is not correct';
        END IF;
    ELSE
        RAISE NOTICE 'Email is not correct';
    END IF;
END;
$$ LANGUAGE plpgsql;
CREATE FUNCTION

movies=# DO $$ 
DECLARE
    email_to_check VARCHAR(200);
BEGIN 
    -- Check each email in the table
    FOR email_to_check IN (SELECT email_address FROM email)
    LOOP
        PERFORM check_email(email_to_check);
    END LOOP;
END $$;
NOTICE:  Email is correct
NOTICE:  Email is correct
NOTICE:  Email is not correct
NOTICE:  Email is not correct
DO
```
