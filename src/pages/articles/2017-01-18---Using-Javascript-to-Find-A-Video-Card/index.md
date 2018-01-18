---
title: Using Javascript to Find a Video Card
layout: post
category: "Programming"
path: "/posts/finding-a-videocard-with-javascript/"
tags: 
- programming
date: 2018-01-17 20:53:18
description: "I wanted to buy a video card, but Bitcoin miners were buying them up way faster than I could. So I used a programmatic approach using the Reddit API and Twilio to immediately get notified when one was selling."
---

# Buying a Video Card with Javascript

I made a resolution this year to build 1 project from start to finish every month, no matter how big or how small, just 12 projects. The first project I met a very specific need and was really fun to boot.

## The Reason

I'd been wanting to finish my computer build for a while. I had a handful of parts collecting dust in my bedroom, and I decided that now was the time to finish it. The video card was the last component I needed to buy and finding one was hard.

## Bitcoin argh...

Because of the recent popularity of Bitcoin people deem it feasible to mine their own coin to make some money. This isn't a bad idea, but as a result of this video cards that are "great values" for the price get bought up within seconds of stock releasing or they are just marked up like crazy from other 3rd party retailers. I wanted a GTX 1070 or 1060 as they seemed to suit my needs. Finding a new one was impossible. So finally, I went ahead and decided to buy a used one of [r/hardwareswap](https://reddit.com/r/hardwareswap). Unfortunatley, they sell fast here too so I thought, "Well reddit has an API, and [Twilio](https://twilio.com) is a thing... cue the project.

## The project

I wanted to build just a simple program to query Reddit's API and everytime it found a match, it would text me a link so I could respond to the post and buy it. Fairly simple. So lets walk through it step by step.

### Setup

You'll need Node and NPM (or Yarn) to start as well as some understanding of Javascript. You'll also need a Reddit account, a Twilio Account, and develoepr apps/keys created on both of these sites. They both have great guides on these on their website.

#### Packages
- Snoocore (Reddit API library with easy as hell Authentication)
- Twilio (Official Twilio API library)
- Dotenv 

### The code

First we have to require all the things we need and do some boiler plate:

```javascript
// Load dotenv for our environment variables sent in a 'git ignored' .env file
require('dotenv').config();

const Snoocore = require('snoocore');
const Twilio = require('twilio');

const accountSid = process.env.TWILIO_SID; // Obtained through creating an app on Twilio
const authToken = process.env.TWILIO_AUTH_TOKEN; // Obtained through creating an app on Twilio

const client = new Twilio(accountSid, authToken);
const reddit = new Snoocore({
    userAgent: process.env.APP_USER_AGENT, // Obtained through creating an app on Reddit
    oauth: {
        type: 'script',
        key: process.env.APP_KEY,
        secret: process.env.APP_SECRET, // Obtained through creating an app on Reddit
        username: process.env.APP_USERNAME,
        password: process.env.APP_PASSWORD,
        scope: ['identity', 'read']
    }
});

sentArray = [];
```

Setting these variables is pretty straight forward and just loads everything we need into local scope for us to have our way with. Next we need the logic to query the subreddit r/hardwareswap. 

```javascript
function getListings() {
    // This calls this pulls the last 5 posts on the subredit, sets the returned data to variables inside of a for loop.
    reddit('/r/hardwareswap/new').listing({ limit: 5 }).then(function(slice) {
        for(i = 0; i <= 4; i++) {
            var post_id = slice.children[i].data.id;
            var post_title = slice.children[i].data.title;
            var post_url = slice.children[i].data.url;

            if (checkProduct(post_title) && !sentArray.includes(post_id)) {
                textMe(post_id, post_title, post_url);
            };
        };
    });
}
```

I used a `for loop` cause I was to lazy to think of a better way. A hash of data is returned with an array as the value of `slice`. The posts are in there, so I iterate over each post and set the appropriate variables, then run two methods: `checkProduct()` and `!sentArray.includes(post_id)`. Lets write the first one.

```javascript
function checkProduct(string) {
    if (string.includes('1070') || string.includes('1060')) {
        console.log('FOUND MATCH ==>> ' + string);
        return true
    } else {
        console.log(string);
        return false
    }
};
```

This simply uses `.includes()` to do some string matching. If the string (in the title) contains '1070' or '1060' console log some things and return true. Else just log it and return false. Pretty simple. The second method was the `!sentArray.includes(post_id)` bit. Lets talk about that.

I needed a way to "store" what values I had already texted myself to make sure I wasn't spamming myself with the same posts over and over again. The best thing I thought to do, was to store the `post_id` in an array each time a text got sent. If `checkProduct()` returns true, and if `sentArray.includes(post_id)` returns `false` it then runs `textMe()`. If it returns `true` it'll fail that conditional and not text me cause of the prepended `!` on that. So now lets look at `textMe()`.

```javascript
function textMe(id, title, url) {
    sentArray.push(id);
    console.log(sentArray);

    let message = 'I found this ' + title + ' ' + url;

    console.log('Sending Message: ' + message);
    createAndSendMessage(message);
};

function createAndSendMessage(message) {
    client.messages.create({
        body: message,
        to: process.env.APP_PERSONAL_PHONE_NUMBER,
        from: 'process.env.APP_TWILIO_PHONE_NUMBER
    });
}
```

This is pretty straight forward. If `textMe()` sucessfully gets called, it throws the `id` of whatever post it's running on into the `sentArray` so it's stored to be referenced later to prevent duplicates. Then I log the array just so I see it each time. I then construct a message which includes the title and the url (so I can just click it from the message) to send myself, log it and then pass it to `createAndSendMessage()`. This actually performs the logic of sending the message, which is pretty straight forward. 

Finally I just execute all this using `setInterval()` so I can do it every 10 seconds. Due to Reddits restrictions I didn't dare try to make it faster than 5 seconds. I thought 10 was a good conservative choice.

```javascript
setInterval(function(){ return getListings(); }, 10000);
```

That's all the code!

Then we can use Nodemon to just spin up a server. All this code hung out in my `index.js` file. I added this little piece to my `package.json` file.

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "nodemon index.js"
},
```

I then just typed `npm start` into my terminal and bam!

![](https://i.imgur.com/a23Fxji.png)

### The whole thing looks like this
```javascript
require('dotenv').config();

const Snoocore = require('snoocore');
const Twilio = require('twilio');

const accountSid = process.env.TWILIO_SID;
const authToken = process.env.TWILIO_AUTH_TOKEN;

const client = new Twilio(accountSid, authToken);
const reddit = new Snoocore({
    userAgent: process.env.APP_USER_AGENT,
    oauth: {
        type: 'script',
        key: process.env.APP_KEY,
        secret: process.env.APP_SECRET,
        username: process.env.APP_USERNAME,
        password: process.env.APP_PASSWORD,
        scope: ['identity', 'read']
    }
});

var sentArray = [];

function checkProduct(string) {
    if (string.includes('1070') || string.includes('1060')) {
        console.log('FOUND MATCH ==>> ' + string);
        return true
    } else {
        console.log(string);
        return false
    }
};

function createAndSendMessage(message) {
    client.messages.create({
        body: message,
        to: process.env.APP_PERSONAL_PHONE_NUMBER,
        from: 'process.env.APP_TWILIO_PHONE_NUMBER
    });
}

function textMe(id, title, url) {
    sentArray.push(id);
    console.log(sentArray);

    let message = 'I found this ' + title + ' ' + url;

    console.log('Sending Message: ' + message);
    createAndSendMessage(message);
};

function getListings() {
    reddit('/r/hardwareswap/new').listing({ limit: 5 }).then(function(slice) {
        for(i = 0; i <= 4; i++) {
            var post_id = slice.children[i].data.id;
            var post_title = slice.children[i].data.title;
            var post_url = slice.children[i].data.url;

            if (checkProduct(post_title) && !sentArray.includes(post_id)) {
                textMe(post_id, post_title, post_url);
            };
        };
    });
}

setInterval(function(){ return getListings(); }, 10000);
```

I then started to get text messages from people selling their 1060s and 1070s. I ended up getting a 1060 shortly after letting this run for a few days. Pretty snazzy if you ask me.

## What I Would Do Differently

If I were to do this again, I would definitely make `checkProduct` way more robust. Because it matches anything in the title, I ended up getting texted a few posts from people who wanted to _buy_ a 1070 or 1060 instead of sell theirs. But this worked fine, especially with the $15 Twilio gives you in credit as a trail offer.

I'd probably also automate a response to the user of the post. So I could respond to a message and it automatically send that Reddit user a message saying I'm interested. I felt like this probabbly violated some service agreement with Reddit and I was too lazy to build that out. Tapping on a link, opening my reddit app, and tapping on the user was fine for me. 


There you have it! Use responsibly.

Cheers