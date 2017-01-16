# Mongo 101
Getting started with the MongoDB Shell to query data at doopoll. 🤓

## What is MongoDB 🤔
[MongoDB](https://www.mongodb.com/) is the name of the database we use to store our data. It is a 'document orientated' database, which means that instead of storing our data in tables and rows (like in Excel, or MySQL) we store documents with dynamic schemas. That means it is possible for one document to have a field, and another not.

Within the database we store 'collections' that our documents live in. For example, at doopoll we have a users collection, as well as one for themes, polls, questions etc. [A list of all the collection names](#collection-names) we use can be found further on in this document.

## Getting Setup 🤖
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

Note: If you make a mistake at any point in the shell don't worry! Your database user is set to readyOnly 🙃
If the shell hangs you can press `CTRL + C` and stop the command.

## Writing count() queries 💯

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

## Collection Names 🌟
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

## Writing cursor queries with find() 🕵

Sometimes we want to know what the values are rather than just count them. For this we can use `find()` without `count()`. We'll see the documents that match that query. The cursor will return the first X and then you can type `it` and hit enter to see more, otherwise `db.Options.find();` would flood your terminal window for a minutes!

We also don't have to return the whole document. We can add a 'projection' which specifies the fields to show for these documents. We can also sort the documents by a field, and limit the amount of documents returned but chaining the commands.

`db.CollectionName.find({ query }, { projection }).sort({ sort }).limit(10)`

#### Here is a practical example:

If we wanted to know the title and response count for every Welsh language poll:
`db.Polls.find({ baseLanguage: 'cy' }, { name: true, responses: true })`

We could make that query slightly more useful.
- The id of the poll is returned by default unless we set it to false.
- We can use `1` and `0` instead of `true` and `false`.

`db.Polls.find({ baseLanguage: 'cy' }, { _id: 0, name: 1, responses: 1 })`

That's cleaner! Now lets find the response count for the top 10 Welsh language polls. First we add apply a sort (`1` for ascending, `-1` for descending), then we limit our returned documents to 10.

`db.Polls.find({ baseLanguage: 'cy' }, { _id: 0, name: 1, responses: 1 }).sort({ responses: -1 }).limit(10)`

Much more useful!

Finally we can chain together everything we've learnt to create a complex query. This final query will return the email address and poll count of the top 5 most active users who signed up in June 2016.

Note: When a field is nested in another field, we use dot syntax to target it. We also need to add quotes around the field name. For example `'profile.pollCount'`.

```
db.users.find({ 
  createdAt: { $gte: ISODate('2016-06-01T00:00:01.552Z'),  $lte: ISODate('2016-06-30T11:59:59.552Z')  },
  'profile.pollCount': { $gt: 1 }
}, {
  _id: 0,
  'emails.address': 1,
  'profile.pollCount': 1,
}).sort({ 'profile.pollCount': -1 }).limit(5)
```

## Regex 🍉
This is a super useful tool for gaining insight. This query would count every minLabel of a question that includes a '£' sign, which would allow us to work out whether people are using sliders for currency based questions:

```
db.Questions.find({
  minLabel: { $regex: /£/ }
}).count();
```

## Using $or 👫
Sometimes you want to find out whether a document meets one or more criteria. You need $or! Here's how you use it:

`db.Collection.find( { $or: [ { query A }, { query B } ] } ).count()`

So if you wanted to find out the number of accounts with more than one users, or over 100 polls:

`db.AccountData.find( { $or: [ { 'usage.users': { $gt: 1 } }, { 'usage.polls': { $gte: 100 } } ] } ).count()`

## Writing a function 💡
If you find yourself needed the same set of numbers all the time you can write a javascript function that will generate a report. Here's a quick example:

```js
var starterPollReport = function() {
 var starterPollCount = db.Polls.find({isStarterPoll: true}).count();
 var duplicatedStarterPollCount = db.Polls.find({isStarterPoll: true, duplicatedCount: {$gte: 1}}).count();
 print('Starter Polls: ' + starterPollCount);
 print('Duplicated Starter Polls: ' + duplicatedStarterPollCount);
}

starterPollReport();
```

Functions can be really powerful, and help you interact between collections, however the aggregate framework is probably better for this.

This example will print the name of every poll name created by users who signed up in June 2016:

```js
var juneSignups = db.users.find({
  createdAt: { $gte: ISODate('2016-06-01T00:00:01.552Z'), $lte: ISODate('2016-06-30T11:59:59.552Z') }
});

juneSignups.forEach(function(user) {
  var polls = db.Polls.find({_id: { $in: user.profile.polls }});
  polls.forEach(function(poll) {
    print(poll.name + " https://app.doopoll.co/poll/" + poll._id + "/live-results");
  });
});
```

## Aggregation and Cohort Analysis 🍱
MongoDB includes a framework called 'aggregate' that can be used for complex metrics. When we pair this with some native javascript, data gets really useful. If you want more information, [check out the docs](https://docs.mongodb.com/manual/reference/operator/aggregation/)

Here's a simple example that uses the `$sum` operator to count the total number of responses for all polls.

```js
db.Polls.aggregate([{ 
  $group: { 
    _id: null, 
    total: { 
      $sum: "$responses" 
    } 
  } 
}]);
```

We can extend this query to count the number of polls too to provide some context:

```js
db.Polls.aggregate([{ 
  $group: { 
    _id: null, 
    pollCount: { 
      $sum: 1 
    },
    responseCount: { 
      $sum: "$responses" 
    } 
  } 
}]);
```

Finally, we can add a date range to get the count of polls created in a time period, and their total response count:

```js
var dateRange = {
  $gte: ISODate('2016-06-01T00:00:01.552Z'), $lte: ISODate('2016-06-30T11:59:59.552Z')
}

db.Polls.aggregate([
  { $match: {
    createdAt: dateRange
  }},{ 
  $group: { 
    _id: null, 
    pollCount: { 
      $sum: 1 
    },
    responseCount: { 
      $sum: "$responses" 
    } 
  }}
]);
```

#### Cohort analysis
Here we define a cohort of users who signed up in June 2016 that created a poll in June 2016. With this we can track the cohort across various actions, and compare them with other cohorts.

```js
var cohortDateRange = {
  $gte: ISODate('2016-06-01T00:00:01.552Z'), $lte: ISODate('2016-06-30T11:59:59.552Z')
}

var createdPollDateRange = {
  $gte: ISODate('2016-06-01T00:00:01.552Z'), $lte: ISODate('2016-06-30T11:59:59.552Z')
}

var cohort = db.users.aggregate(
  [
    { $project : { createdAt: 1 , _id: 1 } },
    { $match: {
      createdAt: cohortDateRange
    }},
    { $sort : { createdAt : 1 } },
    { $group: { _id: '$_id' }}
  ]
).toArray().map(function(user) {
  return user._id
});

var usersWhoCreatedPoll = db.Polls.aggregate(
  [
    { $project : { createdAt: 1 , _id: 1, ownerId: 1 } },
    { $match: {
      ownerId: { $in: cohort },
      createdAt: createdPollDateRange
    }},
    { $sort : { createdAt : 1 } },
    { $group: { _id: { ownerId: '$ownerId' } }}
  ]
).toArray().map(function(poll) {
  return poll._id.ownerId;
});;

print('--------------------\n' +
'Size of Cohort: ' + cohort.length + '\n' +
'Created poll in time period: ' + usersWhoCreatedPoll.length  + '\n' +
'Percentage: ' + (usersWhoCreatedPoll.length/ (cohort.length / 100)).toFixed(2) + '%');
```

