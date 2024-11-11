# Data Modeling Assignment

## Introduction

Data modeling is a crucial step in the process of creating a database. It involves defining and structuring the data that will be stored, ensuring that it is organized in a way that supports efficient access and manipulation. By creating a clear and logical data model, developers can ensure that the database will meet the needs of the application and its users.

## Why Data Modeling is Important

1. **Improved Data Quality**: A well-designed data model helps to ensure that the data stored in the database is accurate, consistent, and reliable. This reduces the risk of data anomalies and errors.

2. **Enhanced Performance**: Proper data modeling can optimize the performance of the database by organizing data in a way that supports efficient querying and indexing.

3. **Better Communication**: Data models provide a visual representation of the data and its relationships, making it easier for developers, analysts, and stakeholders to understand and communicate about the database structure.

4. **Scalability and Maintenance**: A good data model makes it easier to scale the database as the application grows and to maintain it over time by providing a clear blueprint for adding new data and making changes.

## What Type of Data Modeling are We Doing?

At devslopes we teach you to model data in a way that emulates proper data modeling in SQL. The way to think about how data is modeled in SQL is that it is a collection of tables that are related to each other. For example if my only table is a `User` table, and a `User` has an `id` and a `username`, then my table would look like this:

| id  | username |
| --- | -------- |
| 1   | johndoe  |
| 2   | janedoe  |
| 3   | janedoe  |

Although ultimately the data structures that javascript uses fundementally look different, they both can represent the same data.

```js
const users = [
  { id: 1, username: "johndoe" },
  { id: 2, username: "janedoe" },
  { id: 3, username: "janedoe" },
];
```

By itself, this table isn't very useful, after all who wants to look at a list of users without any information about what they are doing? This is where relationships come into play.

## What are Data Relationships?

Let's say that we want to build an app that allows users to look at and favorite different dogs. For example, take a look at our very own [ Pup-E-Picker app ](https://optimitistic-pup-e-picker-deployed.vercel.app/functional)

Let's say that our app is preseeded with the following data:

```js
const dogs = [
  { id: 1, name: "Doomslayer", description: "A very good boy" },
  { id: 2, name: "Zoey", description: "A very good girl" },
  { id: 3, name: "Boye", description: "Also a very good boy" },
];

const users = [
  { id: 1, username: "Jon" },
  { id: 2, username: "Rachel" },
  { id: 3, username: "Hannah" },
];
```

Then let's say that we want to represent the following relationship:

- Jon likes Doomslayer and Zoey
- Rachel likes Zoey, Doomslayer, and Boye
- Hannah likes Doomslayer and Boye

As Javascript devs, we can probably think of a few different ways to represent this data...

```js
// Here's one way to represent this data
const users = [
  { id: 1, username: "Jon", favoriteDogs: [1, 2] },
  { id: 2, username: "Rachel", favoriteDogs: [2, 1, 3] },
  { id: 3, username: "Hannah", favoriteDogs: [1, 3] },
];

// Here's another way to represent this data
const dogs = [
  {
    id: 1,
    name: "Doomslayer",
    description: "A very good boy",
    usersWhoLike: [1, 3],
  },
  {
    id: 2,
    name: "Zoey",
    description: "A very good girl",
    usersWhoLike: [2, 1],
  },
  {
    id: 3,
    name: "Boye",
    description: "Also a very good boy",
    usersWhoLike: [2, 3],
  },
];

// Here's another way to represent this data
const usersWhoLikeDogs = [
  { userId: 1, dogId: [1, 2] },
  { userId: 2, dogId: [2, 1, 3] },
  { userId: 3, dogId: [1, 3] },
];

// Here's another way to represent this data
const dogsWhoUsersLike = [
  { dogId: 1, usersWhoLike: [1, 3] },
  { dogId: 2, usersWhoLike: [2, 1] },
  { dogId: 3, usersWhoLike: [2, 3] },
];
```

In Javascript we have a TON of flexibility which can be nice but also means:

- It can be much harder to reason about the data
- It can be much harder to query the data
- It can be much harder to update the data

Fortunately, SQL doesn't have this flexibility, and instead forces us to represent data in a more rigid way. This rigidity helps us to reason about the data, query the data, and update the data in a more predictable way.

Let's take a look at the rules that SQL forces us to follow

## Rules For Data Modeling (SQL)

1. **Unique Identifier**: Every table must have a unique identifier.

This will be the unique identifier for that table, in this case we will use the `id` column. The reason we will need this later on is that we need a way to uniquely identify each row in the table.

For example I may have two dogs with the name "Doomslayer", but they are different dogs because in SQL we can still identify each individually because they have different `id`s.

2. **You Cannot Nest Data in a CELL in a Table**

Let's take our first example of JS representing our users favoriting dogs.

```js
const users = [{ id: 1, username: "Jon", favoriteDogs: [1, 2] }];
```

If we were to represent this data in a table, it would look like this:

| id  | username | favoriteDogs |
| --- | -------- | ------------ |
| 1   | Jon      | [1, 2]       |

Since the value of `favoriteDogs` is an array, this means that this is NOT A VALID SQL TABLE. Because we cannot nest data in a cell in a table.

That's pretty much it for our rules of what you CANT do in your tables.

But wait, if we can't nest data in a cell, how do we represent the relationship between users and dogs?

If users can have many dogs, and dogs can have many users, but we can't create an array like thing in a cell, how do we represent this?

## Join Tables

In order to represent the relationship between users and dogs, we need to create a join table. The answer in this case is to create a new table that will hold the relationship between users and dogs. We will call it `Favorites. So you don't have to scroll back up, here are our constraints although this time you can read their id's in parenthesis:

- Jon(1) likes Doomslayer(1) and Zoey(2)
- Rachel(2) likes Zoey(2), Doomslayer(1), and Boye(3)
- Hannah(3) likes Doomslayer(1) and Boye(3)

Now we can create our `Favorites` table:

| id  | userId | dogId |
| --- | ------ | ----- |
| 1   | 1      | 1     |
| 2   | 1      | 2     |
| 3   | 2      | 2     |
| 4   | 2      | 1     |
| 5   | 2      | 3     |
| 6   | 3      | 1     |
| 7   | 3      | 3     |

Now we have a table that represents the relationship between users and dogs! Overall the state of our entire system can be represented by the following tables:

- Users

  | id  | username |
  | --- | -------- |
  | 1   | Jon      |
  | 2   | Rachel   |
  | 3   | Hannah   |

- Dogs

  | id  | name       | description          |
  | --- | ---------- | -------------------- |
  | 1   | Doomslayer | A very good boy      |
  | 2   | Zoey       | A very good girl     |
  | 3   | Boye       | Also a very good boy |

- Favorites

  | id  | userId | dogId |
  | --- | ------ | ----- |
  | 1   | 1      | 1     |
  | 2   | 1      | 2     |
  | 3   | 2      | 2     |
  | 4   | 2      | 1     |
  | 5   | 2      | 3     |
  | 6   | 3      | 1     |
  | 7   | 3      | 3     |

## Why is this better?

1. **Normalization**: By separating the data into different tables, we can avoid redundancy and ensure that our data is consistent and accurate.

2. **Querying**: SQL is very good at querying data that is related to one another in this way.

3. **Simplicity**: It's much simpler to reason about data that is related to one another in this way because while in javascript we could represent the same data many different ways, in SQL there is only one "correct" way to represent the data. You will often hear this concept called "Single Source of Truth".

_Note: These rules specifically apply to when we model data in SQL, in other databases or systems the rules may work a little different, but don't worry you can be a senior engineer and just be really good at the SQL philosophy for data modeling. In your career you may encounter some different rules, but even when you do this should give you a good foundation for understanding how data modeling works._
