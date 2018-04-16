---
layout: post
title:      "Relational Databases - How Are We Related Again?"
date:       2018-04-16 00:33:30 +0000
permalink:  relational_databases_-_how_are_we_related_again
---


Today, I wanted to briefly touch on relational databases and their importance in modern programming. We'll take a quick look at what makes database relations so meaningful, and also very powerful.

**It's All Relative, Really**

So what's a relational database? A relational database, at it's most basic, is a group of tables that contain data which relate to one another. Think of a car dealership. The dealership itself is a database, full of different makes of car - Ford, Toyota, Nissan. These are one table. Each make of car has a number of models - Ford Focus, Toyota Corolla, Nissan Titan, etc. This would be another table. In this simple model, we have two tables, a "makes" table, which contains information about each brand of car, and a "models" table, which contains the information about each unique car.  Each make *has many* models, and each model *belongs to* a particular make.

It would be logical, then, to find a way to associate, or relate, this data. We, as lazy programmers, want to be able to say things like, "Show me all of the Ford models" or, "Who makes Corollas" or, "Show me all of the cars in the dealership".

Relational databases make this possible. We can have separate tables containing unique information, all interacting with each other.

So how do we go about doing this?

**Have Foreign Key, Will Travel**

We do this through the use of keys! When we create a table for our database (using SQL), the table usually looks something like this:

```
CREATE TABLE makes (
  id INTEGER PRIMARY KEY,
	name TEXT,
	location TEXT
	);
```

The line ```id INTEGER PRIMARY KEY,``` is what's important there. That line gives each row (or "make" in this example), it's own unique identifier. So the "Ford" make in our case may have an id of 1. *Only* the row of the table containing the Ford data will have the id 1. This allows us to refer to a single row of a table anywhere else.

What good is that data? Well, if we have another table, "models":

```
CREATE TABLE models (
  id INTEGER PRIMARY KEY,
	name TEXT,
	number_of_wheels INTEGER,
	horsepower INTEGER,
	mpg INTEGER,
	foreign_key INTEGER,
	);
```

We can use the ```foreign_key INTEGER``` to associate a specific model to its make! Say, for example, we were making a row to store data about a Ford Focus. It would look like this:

```
INSERT INTO models VALUES ("Focus", 4, 130, 25, 1);
```

You'll notice we gave the Focus a foreign_key of 1. That relates back to the row in our "makes" table with an id of 1, which is Ford! Now we have a way of letting our database know that a Focus is made by Ford. The benefit of this becomes very clear when we scale up our tables.

Ford, Toyota, and Nissan produce MANY different models of car, and new models are released every year! Without this relational database structure, not only would we have to store all of the data in one place, but for every model Ford makes, we would have to *REPEAT* all of the Ford make data in every model row! Same with every Toyota, same with every Nissan. That's way, way too wet to adhere to the DRY principle! No way!

(DRY = Don't Repeat Yourself)

Paradoxically, it's worth repeating.

>DON'T REPEAT YOURSELF.

**Cod? .... The Fish? No. Codd the Programmer**

Why would we store all this data, repeated on tons of lines, in one gigantic table, when we could abstract it away into smaller, easier to read, more direct tables? Answer: We wouldn't, that's silly. And we have a cool programming guy named [Edgar F. Codd](https://en.wikipedia.org/wiki/Edgar_F._Codd) to thank for it. Somewhere between flying for the RAF in WW2 and working for IBM in their San Jose Research Lab, he came up with this idea of related databases.

Oh, and he was also awarded the Turing Award in 1981.

He's totally awesome, well worth reading about.

**It's All Connected, Really**

There you have it! Though this was a very, very basic look at how tables in databases can be related, the concept is a powerful one. As a matter of fact, nearly all the sites we use on the web, and most computing programs, use some form of relational database to store and access information.

Thanks for taking a look at relations with me, and remember, everyone is related somehow!

![We're probably related.](https://media.giphy.com/media/3og0IELgMsujKccJgc/giphy.gif)
