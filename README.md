# Project Instructions

This repo is your starter code for the project. It is the same as the starter code we began with in lesson 2. Install and configure Webpack just as we did in the course. Feel free to refer to the course repo as you build this one, and remember to make frequent commits and to create and merge branches as necessary!

The goal of this project is to give you practice with:
- Setting up Webpack
- Sass styles
- Webpack Loaders and Plugins
- Creating layouts and page design
- Service workers
- Using APIs and creating requests to external urls

On top of that, I want to introduce you to the topic of Natural Language Processing. NLPs leverage machine learning and deep learning create a program that can interpret natural human speech. Systems like Alexa, Google Assistant, and many voice interaction programs are well known to us, but understanding human speech is an incredibly difficult task and requires a lot of resources to achieve. Full disclosure, this is the Wikipedia definition, but I found it to be a clear one:
```
Natural language processing (NLP) is a subfield of computer science, information engineering, and artificial intelligence
concerned with the interactions between computers and human (natural) languages, in particular how to program computers to
process and analyze large amounts of natural language data.
```
You could spend years and get a masters degree focusing on the details of creating NLP systems and algorithms. Typically, NLP programs require far more resources than individuals have access to, but a fairly new API called Aylien has put a public facing API in front of their NLP system. We will use it in this project to determine various attributes of an article or blog post.

## Getting started

It would probably be good to first get your basic project setup and functioning. Follow the steps from the course up to Lesson 4 but don't add Service Workers just yet. We won't need the service workers during development and having extra caches floating around just means there's more potential for confusion. So, fork this repo and begin your project setup.

Remember that once you clone, you will still need to install everything:

`cd` into your new folder and run:
- ```npm install```


## Setting up the API

The Aylien API is perhaps different than what you've used before. It has you install a node module to run certain commands through, it will simplify the requests we need to make from our node/express backend.

### Step 1: Signup for an API key
First, you will need to go here: https://developer.aylien.com/signup. Signing up will get you an API key. Don't worry, at the time of this course, the API is free to use up to 1000 requests per day or 333 intensive requests. It is free to check how many requests you have remaining for the day.

### Step 2: Install the SDK
Next you'll need to get the SDK. SDK stands for Software Development Kit, and SDKs are usually a program that brings together various tools to help you work with a specific technology. SDKs will be available for all the major languages and platforms, for instance the Aylien SDK brings together a bunch of tools and functions that will make it possible to interface with their API from our server and is available for Node, Python, PHP, Go, Ruby and many others. We are going to use the Node one, the page is available here: https://docs.aylien.com/textapi/sdks/#sdks.

### Step 3: Require the SDK package
Install the SDK in your project and then we'll be ready to set up your server/index.js file.

Your server index.js file must have these things:

-[ ] Require the Aylien npm package
```
var aylien = require("aylien_textapi");
```

### Step 4: Environment Variables
Next we need to declare our API keys, which will look something like this:
```
// set aylien API credentias
var textapi = new aylien({
  application_id: "your-api-id",
  application_key: "your-key"
});
```

...but there's a problem with this. We are about to put our personal API keys into a file, but when we push, this file is going to be available PUBLICLY on Github. Private keys, visible publicly are never a good thing. So, we have to figure out a way to make that not happen. The way we will do that is with environment variables. Environment variables are pretty much like normal variables in that they have a name and hold a value, but these variables only belong to your system and won't be visible when you push to a different environment like Github.

-[ ] Use npm or yarn to install the dotenv package ```npm install dotenv```. This will allow us to use environment variables we set in a new file
-[ ] Create a new ```.env``` file in the root of your project
-[ ] Go to your .gitignore file and add ```.env``` - this will make sure that we don't push our environment variables to Github! If you forget this step, all of the work we did to protect our API keys was pointless.
-[ ] Fill the .env file with your API keys like this:
```
API_ID=**************************
API_KEY=**************************
```
-[ ] Add this code to the very top of your server/index.js file:
```
const dotenv = require('dotenv');
dotenv.config();
```
-[ ] Reference variables you created in the .env file by putting ```process.env``` in front of it, an example might look like this:
```
console.log(`Your API key is ${process.env.API_KEY}`);
```
... Not that you would want to do that. This means that our updated API credential settings will look like this:
```javascript
// set aylien API credentials
// NOTICE that textapi is the name I used, but it is arbitrary. You could call it aylienapi, nlp, or anything else, just make sure to make that change universally!
var textapi = new aylien({
  application_id: process.env.API_ID,
  application_key: process.env.API_KEY
});
```

### Step 5: Using the API
We're ready to go! The API has a lot of different endpoints you can take a look at here: https://docs.aylien.com/textapi/endpoints/#api-endpoints

And you can see how using the SDK simplifies the requests we need to make. Here is an example of a sentiment analysis request made through the SDK:
```javascript
// the express router takes a GET type request for the url [domain]/sentiment-analysis
app.get('/sentiment-analysis', function(req, res) {
    // uses the sentiment function in the SDK to make a request for the analysis of the given text
    textapi.sentiment({'text': 'John is a very good football player!'}, function(error, response) {
        if (error === null) {
            console.log(response)
            // send the json response to the page
            res.send(response)
        }
    })
})
```

Notice though, that in the code above we are hard coding the text to be evaluated by Aylien. That of course isn't very helpful, we would want to instead take in a piece of information from the request. Let's say you sent a fetch request with some text as a query parameter.

Your fetch request in the Client:
```javascript
//get the innerHTML text (doesn't have to be done this way, getting info from a form for example, be done via the event)
var text = getElementById(...your element).innerHTML
// create the fetch request to your backend server
// make sure to encode the text into the correct format for a url. The encodeURI js function will do this for you automatically
fetch(`http://localhost:8081/sentiment-analysis?text=${encodeURI(text)}`)
```

Your server code in server/index.js:
```javascript
// the express router takes a GET type request for the url [domain]/sentiment-analysis
app.get('/sentiment-analysis', function(req, res) {
    // get uri encoded parameter sent with the request
    var text = req.query.text
    // uses the sentiment function in the SDK to make a request for the analysis of the given text
    // notice the use of decodeURI - this will undo the url encoding we did in the client so that we send Aylien our original string
    textapi.sentiment({'text': decodeURI(text)}, function(error, response) {
        if (error === null) {
            console.log(response)
            // send the json response to the page
            res.send(response)
        }
    })
})
```

## Deploying

A great step to take with your finished project would be to deploy it! Unfortunately its a bit out of scope for me to explain too much about how to do that here, but checkout Netlify or Heroku for some really intuitive free hosting options.
