
my heroku project https://guarded-oasis-24198.herokuapp.com
## Objectives

* Be able to create tables using migrations
* Be able to create tables in your Heroku DB using migrations
* Be able to explain why migrations are important
* Be able to explain why migrations have unique identifying numbers

## Why you should care

Migrations are a convenient way to alter your database schema over time in a consistent and easy way. Migrations reduce the opportunity for human error and allow you to automate schema creation in both development and production.

You can think of each migration as being a new 'version' of the database. A schema starts off with nothing in it, and each migration modifies it to add or remove tables, columns, or entries.

These migrations also provide further documentation about your database schema for future you, your team, and _future you!_

## EXERCISE SUMMARY

### SET UP

First, doublecheck that you don't already have a database called `library_development`. If you do, drop the database.

```
psql postgres
\l
(if you already have a library_development db, then drop it)
drop database library_development
```

### STEP 1 OVERVIEW (detailed istructions to follow):
Included in this repo is a Library CRUD app. Your mission is to add `Books` and `Readers` to this application using `knex` and `migrations`.

__Books__

* id
* author
* title
* rating
* description

__Readers__

* id
* first_name
* last_name

### STEP 2 OVERVIEW (detailed istructions to follow):

Deploy this CRUD app to Heroku and use your migrations to add your `Readers` and `Books` tables to your Heroku database.

# Let's Get Started!

## Installing and setting up knex

```sh
$ npm install --save pg knex  #install knex locally
$ npm install knex -g         #install knex cli globally if you haven't before
```

---

## knexfile.js

`touch knexfile.js` and add the following.

```js
require('dotenv').config({silent: true});

module.exports = {

  development: {
    client: 'pg',
    connection: 'postgres://localhost/library_development'
  },

  production: {
    client: 'pg',
    connection: process.env.DATABASE_URL + '?ssl=true'
  }

};
```

Notice that we've imported the `dotenv` module by adding the `require('dotenv').config({silent: true});` line.

This is because our `knex` migrations access the `knexfile.js` directly to
run, rather then hitting the connection we feed to our routes through `db/knex.js`. By adding this line to our file, our environment variables can be read by knex.

__NOTE:__ What npm module do you need to install to be able to use environment variables (i.e. `process.env.SOME_VARIABLE`)? You should go install that dependency now.
---dotenv?

## Migrations

Migrations are stored as files in the migrations directory, one for each migration. The name of the file is of the form `CURRENTDATETIME_create_books.js`, that is to say a __UTC timestamp__ identifying the migration followed by an __underscore__ followed by the __name of the migration__.  Knex uses this timestamp to determine which migration should be run and in what order.

Of course, calculating timestamps is no fun, so Knex provides a generator to handle making it for you:

__The migration cli is bundled with the knex global install.__

---

## Using the Knex migration tool

__CREATE the BOOKS schema:__

Create a new migration with the name create_books

```sh
knex migrate:make create_books
```
__You probably got an error about not having a database? Your migration file was still created, but go ahead and use knex to create your `library_development` database__

```sh
createdb library_development
```

Update the new books migration file `migrations/CURRENTDATETIME_create_books.js` accordingly:

```js
exports.up = function(knex, Promise) {
  return knex.schema.createTable('books', function(table){
    table.increments();
    table.string('author');
    table.string('title');
    table.integer('rating');
    table.text('description');
    table.timestamps();
  })
};

exports.down = function(knex, Promise) {
  return knex.schema.dropTableIfExists('books');
};
```
__What SQL statements is this migration running for you?__
---

## Run the latest migrations using the development connection string

__The `--env` flag tells knex to run the migrations on your local database only__

```sh
$ knex migrate:latest --env development
```

__confirm successful migration and proper schema__

```sh
psql library_development
select * from books;
```
---

## Establishing a connection

* Create a folder called `db`

* Inside the db folder create a new file `knex.js` with the following contents:

```js
var environment = process.env.NODE_ENV || 'development';
var config = require('../knexfile.js')[environment];
module.exports = require('knex')(config);
```

* This initializes knex with the connection information obtained from the configuration in `knexfile.js` for the current environment.

---

## Use the connection in your routes file

__STEP ONE: BOOKS__

The Books CRUD routes have all been written for you. All you have to do is import
your `db/knex.js` connection and assign it to a variable called `Books`.

_Once you've got Books CRUD all wired up, click through the local app and confirm
everything is working._ THEN, skip down to `Get it working on Heroku`. Once you've got it working on Heroku, come back and do `READERS`

__STEP TWO: READERS__

Ok. Take what you've learned so far and now apply it to create CRUD functionality for _Readers_. Scroll back up to see what your schema should look like.

Using knex and migrations, get the `Readers` schema wired up and confirm that your app is running as it should _locally___.

__You'll also have to create the necessary views and routes for Readers.__

---

## Get it working on Heroku

Run the following commands to create a new Heroku app and database.

```
heroku create
git push heroku master
heroku addons:create heroku-postgresql
```
From the command line, run the following command:

```sh
knex migrate:latest --env production
```

If successful, you should see an output similar to this:
`Batch 1 run: 1 migrations
/Users/marthaberner/workspace/g29/knex-migrations-and-deployment/migrations/20160711152436_create_books.js`

__Go to Heroku and click through the app to ensure everything is working.__

__THEN go back up to `STEP TWO: READERS` and complete the application.__

## Next Steps

__Once you've got this app up and running on Heroku, add your Heroku url for this app to the top of this `README` and submit a `pull request`.__

## Helpful Notes

__Using the `dotenv` module to config environment variables__

You'll need some help getting your app to talk to your environment variables, both locally as well as deployed.

Google `npm dotenv` and read the docs to help you get up and running with a `.env` file in your Node.js app.

__HINT:__ You only need the first two parts of the documentation to get
your app wired up.
