# Mongo 101
Getting started with the MongoDB Shell to query data at doopoll.

## What is MongoDB
[MongoDB](https://www.mongodb.com/) is the name of the database we use to store our data. It is a 'document orientated' database, which means that instead of storing our data in tables and rows (like in Excel, or MySQL) we store documents with dynamic schemas. That means it is possible for one document to have a field, and another not.

Within the database we store 'collections' that our documents live in. For example, at doopoll we have a users collection, as well as one for themes, polls, questions etc. [A list of all the collection names](collection-names) we use can be found further on in this document.

## Getting Setup
Before you do anything you'll need to install Mongo onto your Mac. The recommended way to do this is via a package manager called Homebrew.

Open up your terminal and paste in the following commands:
```bash
// Install Homebrew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
// Update the list of brew packages
brew update
// Install mongoDB
brew install mongodb
```

Once you're at this point you can paste in your personal login shell script. If you don't have one yet you can request one. Once you have it, keep it secret, keep it safe.

It should look something like this:
```bash
mongo candidate.5.mongolayer.com:10832/doopoll -u <username> -p<password>
```

Once it loads, you should be in the shell!

## Writing count() queries

Here is the simplest query we can use to get back some data. Copy and paste this link into the Mongo shell and you'll get back the current number of polls users have created.

`db.Polls.find().count()`

The shell runs the command and returns a number. We can get more meaningful data by adding a query. A query allows us to filter down documents based on values. This next query will return the number of polls with public results.

`db.Polls.find({ publicResults: true }).count()`

We may want to know if a value is greater than or less than a value. There are a number of special operators that MongoDB supports. Here we use the `$gte` operator which will let us see how many polls have greater than or equal to 100 responses. 

`db.Polls.find({ responses: { $gte: 100 } }).count()`

This also works for dates. This query would count all the polls since the beginning of 2016.

`db.Polls.find({ createdAt: { $gte: ISODate('2016-01-01T00:00:01.552Z') } }).count()`

We can use a combination of the `$gte` and `$lte` operators to find polls created in a data range. Here we'll count the number of polls created in June 2016.

`db.Polls.find({ createdAt: { $gte: ISODate('2016-06-01T00:00:01.552Z'),  $lte: ISODate('2016-06-30T11:59:59.552Z')  } }).count()`

Finally, we can chain together different parts of a query to be really specific. This query will return the number of Welsh language polls created before the June 2016 that have had at least one response. For readability this query has been formatted over multiple lines and indented. You can do this too, and copy and paste it in, or you can write it on one line!.

```bash
db.Polls.find({ 
  baseLanguage: 'cy',
  responses: { $gte: 1 },
  createdAt: { $lt: ISODate('2016-06-01T00:00:01.552Z') }
}).count()
```

## Collection Names
There are many collections stored on the database, here are the useful ones. If you run the `findOne()` you can see an example of a document from that collection. This helps when trying to work out what to query! All the collections are PascalCase appart from `users`.

```bash
db.AccountData.findOne();
db.Invoices.findOne();
db.Options.findOne();
db.Polls.findOne();
db.Questions.findOne();
db.Themes.findOne();
db.users.findOne();
```
