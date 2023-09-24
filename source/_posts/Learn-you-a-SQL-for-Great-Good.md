---
title: Learn you a SQL for Great Good
tags:
  - misc
date: 2021-11-07 12:53:22
---


Lately I have been spending some time on [Reddit's /r/SQL](https://www.reddit.com/r/SQL/) and a question that comes up frequently is where to find good resources to learn SQL. This post traces a learning path. I will list down courses, books, and websites which you can use to learn SQL. 

## Getting Started  

The first step is to get comfortable with the relational model. Depending on whether you prefer books or courses, you can pick one of the following.  

### Course - [Databases: Relational Databases and SQL](https://www.edx.org/course/databases-5-sql)

The course, offered by Stanford on edX, provides an introduction to relational databases and SQL. It assumes no prior knowledge of either relational databases or SQL. It is the first in a series of 5 courses with each course covering a more advanced aspect of SQL and relational model. As of writing, the course costs USD 50 for the certificate track.  

### Book - [Learning SQL, 3rd Edition](https://www.oreilly.com/library/view/learning-sql-3rd/9781492057604/)  

If you are a reader, Learning SQL by Alan Beaulieu is the book for you. It begins with a quick introduction of the relational model and then dives head-first into writing SQL queries. It explains everything by examples and also provides exercises at the end of each chapter to solidify learning. The book is intended to be read cover to cover as each chapter builds upon the previous one. You will find a comprehensive coverage of SQL from the basics of creating tables, to inserting, retrieving, and deleting data, to the more advanced analytical functions.  

### Website - [SQLBolt](https://sqlbolt.com/)  

If you're looking for a free alternative to learn SQL then you can use SQLBolt. It begins with a quick one-page introduction of relational databases and then takes you straight into writing SQL queries. The good part about SQLBolt is that you do not need to set up any database on your local machine. You can do all the exercises right in the browser.   

SQLBolt is a good website to learn the basics of SQL. However, there are a few topics which are yet to be covered. These are topics of intermediate difficulty and you will have to find alternative resources to learn them.

## Practice, Practice, Practice

Once you have a firm grasp of the basics, the next step is to get some practice.  
### Website - [PgExercises](https://pgexercises.com/)  

PgExercises is a good place to start practicing your new SQL skills. You are presented with the database of a fictitious country club on which the problems are based. The problems begin with simple retrieval of data and gradually become more involved by introducing conditional statements, and so on. The nice thing about PgExercises is that you can solve all the problems on the website itself and get immediate feedback on whether your answer is correct or not. There is an option to get a hint if you are stuck and to view the answer if you are really stuck.  

You can also use PgExercises to prepare for an interview. Set a timer when solving the exercises and don't look at the hints. This will give you an indication of how prepared you are.  

### Website - [Window Functions](https://www.windowfunctions.com/)  

If you're asked to find out the maximum or minimum value among a group of rows, you'd probably come up with a combination of GROUP BY clause along with MIN or MAX functions. What if, instead, you were asked to rank your results in decreasing order? SQL allows you to do that using the RANK function.   

MIN, MAX, and AVG are aggregate functions whereas RANK is a window function. If you'd like to learn how to use window functions, click on the link in the title. The Window Functions website begins with a refresher on GROUP BY and HAVING clauses and then proceeds to introduce you to window functions. Like PgExercieses, you'll be solving problems in an interactive editor using Postgres. All this on a cute dataset of cats.  

Use Window Functions and PgExercises to prepare for your upcoming interviews.


### Website - [AdvancedSQLPuzzles](https://advancedsqlpuzzles.com/)  

If you are looking for something more mind-bending then head over to AdvancedSQLPuzzles. It is a collection of 40 puzzles that will help you determine your level of expertise with SQL. The only caveat to using the website is that you will have to put in some effort to set up a local database before you can start solving the puzzles. Additionally, the solutions are written in T-SQL so you might need to translate the solutions to your favorite dialect of SQL.  

Finito.