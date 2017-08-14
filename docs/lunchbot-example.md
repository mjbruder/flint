#### Example Template to build a Lunch Bot Using Express
```js
var Flint = require('node-flint');
var webhook = require('node-flint/webhook');
var express = require('express');
var bodyParser = require('body-parser');
var yelp = require('yelp-fusion');
var oauth = require('./oauth.js'); //see lunch-bot-oauth-example.md
var spark = require('./spark.js'); //see lunch-bot-spark-example.md
var clientId = oauth.client_id;
var clientSecret = oauth.client_secret;
var sparkKey = spark.api_key;
var app = express();
app.use(bodyParser.json());
app.set('port', (process.env.PORT || 5000)); //required to run the bot on Heroku's dynamic port

// flint options
var config = {
  webhookUrl: 'https://myappurl.com/flint', //put your application URL here for whatever hosting service you are using
  token: sparkKey,
  port: process.env.PORT, //port 5000 for local or dynamic Heroku port
  removeWebhookOnStart: false,
  maxConcurrent: 5,
  minTime: 50
};

// init flint
var flint = new Flint(config);
flint.start();

// say hello
flint.hears('/hello', function(bot, trigger) {
  bot.say('Hello %s! Send me an @ followed by /help to see what I can do. For questions, reach out to mbruder@cisco.com', trigger.personDisplayName);
});

// request lunch and ask for zip code
flint.hears('/help', function(bot, trigger) {
  bot.say('Where are you located, %s? Send me an @ followed by /zip <zipcode> or /city <city, state> to get the top 5 lunch spots.', trigger.personDisplayName);
});

// set zip code and respond with lunch options
flint.hears('/zip', function(bot, trigger) {
  //bot.store('zip', trigger.args[1]);
  //bot.store('lunch', 'lunch');
  // get the first word after /zip and set it as the yelp location
  bot.say('Thanks! Here are the top lunch spots in the %s zipcode:', trigger.args[1]);
  yelp.accessToken(clientId, clientSecret).then(response => {
    var client = yelp.client(response.jsonBody.access_token);

    client.search({
      term: 'lunch',
      location: trigger.args[1],
      sort_by: 'rating', //sort by the top rated lunch spots
      radius: '8046', //set the search radius to 5 miles
      limit: 10 // limit the results to 10
    }).then(response => {
      // parse through the yelp results and output the name, phone, and yelp URL to Spark
      for (var i = 0; i < 5; i++) {
        bot.say(response.jsonBody.businesses[i].name + " - phone: " + response.jsonBody.businesses[i].phone + " - " + response.jsonBody.businesses[i].url);
      }
    });
  }).catch(e => {
    console.log(e);
  });
});

// set city/state and respond with lunch options
flint.hears('/city', function(bot, trigger) {
  //bot.store('zip', trigger.args[1]);
  //bot.store('lunch', 'lunch');
  // get the first word after /city and set it as the yelp location
  bot.say('Thanks! Here are the top lunch spots in %s', trigger.args.join(" ").replace('/city ',''));
  yelp.accessToken(clientId, clientSecret).then(response => {
    var client = yelp.client(response.jsonBody.access_token);

    client.search({
      term: 'lunch',
      location: trigger.args.join(" ").replace('/city ',''),
      sort_by: 'rating', //sort by the top rated lunch spots
      radius: '8046', //set the search radius to 5 miles
      limit: 10 // limit the results to 10
    }).then(response => {
      // parse through the yelp results and output the name, phone, and yelp URL to Spark
      for (var i = 0; i < 5; i++) {
        bot.say(response.jsonBody.businesses[i].name + " - phone: " + response.jsonBody.businesses[i].phone + " - " + response.jsonBody.businesses[i].url);
      }
    });
  }).catch(e => {
    console.log(e);
  });
});

// add flint event listeners
flint.on('message', function(bot, trigger, id) {
  flint.debug('"%s" said "%s" in room "%s"', trigger.personEmail, trigger.text, trigger.roomTitle);
});

flint.on('initialized', function() {
  flint.debug('initialized %s rooms', flint.bots.length);
});

// define express path for incoming webhooks
app.post('/flint', webhook(flint));

// start express server
var server = app.listen(config.port, function() {
  flint.debug('Flint listening on port %s', config.port);
});

// gracefully shutdown (ctrl-c)
process.on('SIGINT', function() {
  flint.debug('stoppping...');
  server.close();
  flint.stop().then(function() {
    process.exit();
  });
});
```
