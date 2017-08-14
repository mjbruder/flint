## Installation

#### Via Git
```bash
mkdir myproj
cd myproj
git clone https://github.com/nmarus/flint
npm install ./flint
npm install ./yelp-fusion
```

#### Via NPM
```bash
mkdir myproj
cd myproj
npm install node-flint
npm install yelp-fusion
```
#### Notes
Lunch Bot uses the yelp-fusion Node.js module from npm, and requires you to sign up for and use a Yelp Fusion client ID and password. Yelp Fusion requires oauth authentication, so you need to fill in the credentials in an oauth.js file as well as put your Spark bot API key into a spark.js file. There's also an example package.json file to use for your app. Finally, Lunch Bot is prepped to run on Heroku to make it easy to host your bot.
