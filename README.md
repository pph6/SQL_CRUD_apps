# **SQL CRUD**

#Setup:
- Creating 2 fake app databases to query from. The details of these two apps are down below.
- To fake this data, I used the site [mockaroo.com](https://mockaroo.com) - a tool for generating mock data. 


# a) Restaurant App:

## _Code to create the tables:_
For the restaurant app: Designed a database table named `restaurants` that would allow an application that uses it to find restaurants and a table named `reviews` that would hold reviews for any restaurant.
The relevant fields:
- **Category** (i.e. genre of food)
- **Price tier** (i.e cheap, medium, or expensive)
- **Neighborhood** (a particular NYC neighborhood)
- **Opening hours** (for simplicity, you can assume each restaurant has the same opening hours every day)
- **Average rating** (out of 5 stars)
- **Good for kids** (true or false)
- The application also allows users to leave reviews associated with any restaurant - for this I created a second table.
- 
### 1. Restaurant
```
CREATE TABLE restaurant ( 
	...> id INTEGER PRIMARY KEY NOT NULL, 
	...> name TEXT NOT NULL, 
	...> price_tier INTEGER CHECK (price_tier > 0 AND price_tier <= 3), 
	...> category TEXT NOT NULL, neighborhood TEXT NOT NULL, 
	...> avg_rating NUMERIC CHECK (avg_rating > 0 AND avg_rating <= 5) NOT NULL, 
	...> good_for_kids BOOLEAN NOT NULL CHECK (good_for_kids in (0, 1)), 
	...> opening_hrs BLOB NOT NULL);
```
Notes for table: 
- price_tier: 1 is cheapest, 3 is most expensive
- avg_rating: from 1 (lowest) to 5 (highest)
- good_for_kids: 0 for no, 1 for yes

<br>

### 2. Reviews
```
CREATE TABLE reviews (rev_id INTEGER PRIMARY KEY NOT NULL, 
rest_id INTEGER NOT NULL CHECK (rest_id>=0 AND rest_id<=1000), 
review TEXT NOT NULL CHECK (length(review) <= 600));
```

## _Importing Data_

[Link to restaurant data](data/data_restaurants.csv)

Code to import: ` .import '/Users/poorvihosabettu/Desktop/DBDI/sql-crud-pph6/data/data_restaurants.csv' restaurant --skip 1`

<br>

##  _Queries_ 
1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example):
`SELECT name FROM restaurant WHERE neighborhood = 'Bronx' AND price_tier = 1;`

2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.
`SELECT name, avg_rating FROM restaurant WHERE category == 'China' AND avg_rating >= 3.0  ORDER BY avg_rating DESC;`

3. Find all restaurants that are open now (see hint below).
`SELECT name, opening_hrs FROM restaurant WHERE opening_hrs < strftime('%H:%M','now');`

4. Leave a review for a restaurant (pick any restaurant as an example).
`INSERT INTO reviews (rest_id, review) VALUES(78, 'Horrible restaurant. Head chef kicked my grandma in the shin and threw raw steak at us. Will not be coming back.')`

5. Delete all restaurants that are not good for kids.
`DELETE FROM restaurant WHERE good_for_kids = 0;`

6. Find the number of restaurants in each NYC neighborhood.
`SELECT neighborhood, COUNT(name) FROM restaurant GROUP BY neighborhood;`


<br>

# b) Social Media App
The social media app I designed a database for allows Users to share two kinds of content: **Messages** and **Stories**.

- **Messages:**
  - Messages consist of text only.
  - Messages are sent from one user to another specific user.
  - Messages become invisible immediately after view and don't show up in the app thereafter.
  - Messages are never actually deleted from the database table, even when invisible to the user (the social media company that produces the app keeps 'deleted' content in its database for future data harvesting, monetization purposes, and more.)

- **Stories:**
  - Stories consist of text only.
  - Stories are public and every user can see them.
  - Stories become invisible 24 hours after posting and don't show up in the app thereafter.
  - Stories are never deleted from the database table, even when invisible to the user.


## _Code to create the tables:_

### 1. Users table
```
CREATE TABLE users (
   ...> user_id INTEGER PRIMARY KEY NOT NULL, 
   ...> email VARCHAR(500) NOT NULL, 
   ...> password varchar(500) NOT NULL, 
   ...> username varchar(500) UNIQUE NOT NULL);
```
### 2. Posts table
```
CREATE TABLE posts(
...> id INTEGER PRIMARY KEY NOT NULL, 
...> switch INTEGER CHECK (switch IN (0,1)) NOT NULL,
...> sender INTEGER CHECK NOT NULL,
...> receiver INTEGER CHECK NOT NULL,
...> story_or_message TEXT NOT NULL,
...> time_sent DATETIME DEFAULT CURRENT_TIMESTAMP NOT NULL
...> visibility BOOLEAN NOT NULL);
```
Notes: "sender" also refers to poster in case of stories.
If the "reciever" and "switch" columns are 0, then it is a story. 
If "switch" == 1 then it is a message and the receiver number is the user_id of the person it was sent to.

## _Importing data_

Links to data:
- [User table](data/data_users.csv)
- [Posts table1](data/data_posts1.csv)
- [Posts table2](data/data_posts2.csv)
<br>
Note: there are 2 posts tables since I had to download 2000 columns of data and Mockaroo only supports 1000. 

Code to import:
`.import '/Users/poorvihosabettu/Desktop/DBDI/sql-crud-pph6/data/data_user.csv' users --skip 1;`
<br>

`.import '/Users/poorvihosabettu/Desktop/DBDI/sql-crud-pph6/data/data_posts1.csv' posts --skip 1;`
<br>

`.import '/Users/poorvihosabettu/Desktop/DBDI/sql-crud-pph6/data/data_posts2.csv' posts --skip 1;`


## _Queries_ 

1. Register a new User.
`INSERT INTO users (email, password, username) VALUES ('happy@gmail.com','dt67u8o.,uy','happboi'); `

2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).
`INSERT INTO posts(sender, receiver, story_or_message) VALUES(32, 320, "I got fired. Let's get tacos.");`

3. Create a new Story by a particular User (pick any User for example).
`INSERT INTO posts (user_id, story_or_message) VALUES (876, "My pug is the best dog in the world.");`

4. Show the 10 most recent visible Messages and Stories, in order of recency.
`SELECT story_or_message FROM posts WHERE visiblity = 1 
   ...> ORDER BY time_sent DESC
   ...> LIMIT 10;`

5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.
`SELECT story_or_message FROM posts WHERE
   ...> (sender = 550 AND receiver = 685) 
   ...> AND visibility = 1
   ...> ORDER BY time_sent DESC
   ...> LIMIT 10;`

6. Make all Stories that are more than 24 hours old invisible.
`UPDATE posts SET visibility=0 
	WHERE switch=0 AND 
	(((JULIANDAY(CURRENT_TIMESTAMP)-JULIANDAY(time_sent))*24)<24 = 0);`

7. Show all invisible Messages and Stories, in order of recency
`SELECT story_or_message FROM posts 
   ...> WHERE visibility = 0
   ...> ORDER BY time_sent DESC;`

8. Show the number of posts by each User.
`SELECT sender, count(*) FROM posts WHERE switch = 0 GROUP BY sender;`

9. Show the post text and email address of all posts and the User who made them within the last 24 hours.
`SELECT users.email, posts.story_or_message FROM users INNER JOIN posts ON users.user_id = posts.sender WHERE visibility = 1;`

10.  Show the email addresses of all Users who have not posted anything yet.
`SELECT users.email FROM users LEFT JOIN posts ON users.user_id = posts.sender WHERE 
   ...> posts.sender IS NULL;`








