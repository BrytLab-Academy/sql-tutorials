A database may contain some data in a rows which might be presented in a better way in a form of culumns that rows.
Database like Oracle and SQL Server have the pivot function to transform rows of data into culumns.
You can implent this functionality inn MySQL using the CASE statement.



To show this, lets create a database with three tables: users, courses, and user_courses.
The `users` table will contain a list of users, `courses` table a list of courses and te `user_courses` 
the list of students and the courses they have enrolled.



Create `tutorials` database.
```
create database tutorials;
```

Create  `users` table. 
```
create table users (
  id smallint(3)  unsigned  primary key auto_increment,
  name varchar(150) not null
);
```


Create `courses` table
```
create table courses(
  id  smallint(3)  unsigned  primary key auto_increment,
  name varchar(255) not null 
);
```


Create `user_courses` table.
```
CREATE TABLE user_courses (
  user_id smallint(3) unsigned NOT NULL,
  course_id smallint(3) unsigned NOT NULL,
  score int(3) unsigned default 0  NOT NULL,
  FOREIGN KEY (course_id) REFERENCES courses (id),
  FOREIGN KEY (user_id) REFERENCES users (id),
  PRIMARY KEY (user_id, course_id)
);
```



Let's insert some data into the tables to work with.

Insert three users into `users` table.
```
insert into users(name) 
values  
  ('Kofi'), 
  ('Ama'), 
  ('Abena');
```

Insert three courses into courses table.
```
insert into courses(name) values
  ("Geography"),
  ("History"),
  ("Law");
```

Insert assign three courses to each of the three users.
```
insert into user_courses(user_id, course_id, score) 
values
    (1,1,55),
    (1,2,87),
    (1,3,78),
    (2,1,62),
    (2,2,92),
    (2,3,59),
    (3,1,85),
    (3,2,88),
    (3,3,79);
```


Lets see the content of each table 


Users table
```
mysql> select * from users;
+----+-------+
| id | name  |
+----+-------+
|  1 | Kofi  |
|  2 | Ama   |
|  3 | Abena |
+----+-------+
3 rows in set (0.00 sec)
```


Courses table
```
mysql> select * from courses;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | Geography |
|  2 | History   |
|  3 | Law       |
+----+-----------+
3 rows in set (0.00 sec)
```


user_courses table
```
mysql> select * from user_courses;
+---------+-----------+-------+
| user_id | course_id | score |
+---------+-----------+-------+
|       1 |         1 |    55 |
|       1 |         2 |    87 |
|       1 |         3 |    78 |
|       2 |         1 |    62 |
|       2 |         2 |    92 |
|       2 |         3 |    59 |
|       3 |         1 |    85 |
|       3 |         2 |    88 |
|       3 |         3 |    79 |
+---------+-----------+-------+
9 rows in set (0.00 sec)
```

The three columns in `user_courses` table represent the score on each user in a particlular course.
For instance, the first row the user_id 1 represent usr Kofi, and course_id 1 represents Geography.
Hence Kofi has a score of 55 in Geography.

To make this table easy to understand, let's replace the `user_id` and the `course_id` with the user name and the course name respectively.
We can do this using the inner join.


Replace user_id and course_id with user name and course name respectively using inner join.
```
mysql>  select 
          users.name as user_name, 
          courses.name as course_name,
          user_courses.score
        from 
          users 
          inner join user_courses on users.id = user_courses.user_id
          inner join courses on courses.id = user_courses.course_id
        order by user_name;
+-----------+-------------+-------+
| user_name | course_name | score |
+-----------+-------------+-------+
| Abena     | Law         |    79 |
| Abena     | History     |    88 |
| Abena     | Geography   |    85 |
| Ama       | Geography   |    62 |
| Ama       | Law         |    59 |
| Ama       | History     |    92 |
| Kofi      | Law         |    78 |
| Kofi      | History     |    87 |
| Kofi      | Geography   |    55 |
+-----------+-------------+-------+
9 rows in set (0.12 sec)
```

This view looks more presentable. But there is one problem here.
There are only three users but we have 9 rows with each row representing a course enrolled by a user.
We can transform this row into colums so that  we would have only three rows: each row representing a user.
And five columns : one for the user names and the rest to the four courses. 
This will create a grid-like table with each intersectin representing a course enrolled by a user.

The Pivot table
```
mysql>  select user_name,
            max( case when course_name='Geography' then score  end)  Geography,
            max( case when course_name='Law' then score end)  Law,
            max( case when course_name='History' then score end)  History
        from
          (
            select 
              users.name as user_name, 
              courses.name as course_name,
              user_courses.score
            from 
              users 
              inner join user_courses on users.id = user_courses.user_id
              inner join courses on courses.id = user_courses.course_id
          ) as inner_table
        group by user_name
        order by user_name;
+-----------+-----------+------+---------+
| user_name | Geography | Law  | History |
+-----------+-----------+------+---------+
| Abena     |        85 |   79 |      88 |
| Ama       |        62 |   59 |      92 |
| Kofi      |        55 |   78 |      87 |
+-----------+-----------+------+---------+
3 rows in set (0.00 sec)
```


We can also apply aggregate functions on the result of a pivot table. 
In this scenario, we can calculate the total score of each user accross all courses.
```
mysql>  select user_name,
          max( case when course_name='Geography' then score  end)  Geography,
          max( case when course_name='Law' then score end)  Law,
          max( case when course_name='History' then score end)  History,
          sum(score) as total_score
        from 
          (
            select 
          users.name as user_name, 
          courses.name as course_name,
          user_courses.score
        from 
          users 
          inner join user_courses on users.id = user_courses.user_id
          inner join courses on courses.id = user_courses.course_id
        ) as inner_table
        group by user_name
        order by user_name;
+-----------+-----------+------+---------+-------------+
| user_name | Geography | Law  | History | total_score |
+-----------+-----------+------+---------+-------------+
| Abena     |        85 |   79 |      88 |         252 |
| Ama       |        62 |   59 |      92 |         213 |
| Kofi      |        55 |   78 |      87 |         220 |
+-----------+-----------+------+---------+-------------+
3 rows in set (0.01 sec)
```

This structure looks perfect at the moment. 
Let's insert a new user into the `users` table and assign only one course to this user and check the output of the pivot query again.


Add one more user.
```
mysql> insert into users(name) value ('Musa');
Query OK, 1 row affected (0.12 sec)
```

Assign only one course to the user Musa
```
mysql>  insert into user_courses(user_id, course_id, score) 
        values 
          (4, 3, 96);
Query OK, 1 row affected (0.07 sec)
```


This is the out put from the pivot query above.
```
+-----------+-----------+------+---------+-------------+
| user_name | Geography | Law  | History | total_score |
+-----------+-----------+------+---------+-------------+
| Abena     |        85 |   79 |      88 |         252 |
| Ama       |        62 |   59 |      92 |         213 |
| Kofi      |        55 |   78 |      87 |         220 |
| Musa      |      NULL |   96 |    NULL |          96 |
+-----------+-----------+------+---------+-------------+
4 rows in set (0.03 sec)
```

We can see `NULL` values at the Geography and History columns for Musa because he is not enrolled in these courses.
We can use the  coalesce function to replace each  `NULL` value with a meaningfull value.
In this case, we will use 'Not enrolled'
```
mysql>  select user_name,
          coalesce( max( case when course_name='Geography' then score end), 'Not enrolled')  Geography,
          coalesce( max( case when course_name='Law' then score end), 'Not enrolled')  Law,
          coalesce( max( case when course_name='History' then score end), 'Not enrolled')  History,
          sum(score) as total_score
        from 
          (
          select 
            users.name as user_name, 
            courses.name as course_name,
            user_courses.score
          from 
            users 
            inner join user_courses on users.id = user_courses.user_id
            inner join courses on courses.id = user_courses.course_id
        ) as inner_table
        group by user_name
        order by user_name;
+-----------+--------------+------+--------------+-------------+
| user_name | Geography    | Law  | History      | total_score |
+-----------+--------------+------+--------------+-------------+
| Abena     | 85           | 79   | 88           |         252 |
| Ama       | 62           | 59   | 92           |         213 |
| Kofi      | 55           | 78   | 87           |         220 |
| Musa      | Not enrolled | 96   | Not enrolled |          96 |
+-----------+--------------+------+--------------+-------------+
4 rows in set (0.00 sec)
```

Conclusion