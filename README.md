To install you need latest `node.js` and `npm`.

Node is only used to organize the code locally and push it to your couch. It's not needed on your Couch instance.

First clone this repo.

Then install the `couchapp` dependency with `npm`:
  
    npm install couchapp

And then simply push this source code to your couchdb instance:

    couchapp push app.js http://yourcouch.com/dbname

## Deploying to IrisCouch

If you want to install this app yourself, I strongly advise using the great [IrisCouch](http://www.iriscouch.com/) hosting service. It's free to start with and comes with a great pricing plan. This is where [hckr.it](http://www.hckr.it/) is hosted.

Simply click on the big "Sign Up Now" button on their homepage and create your own Couch instance. You should get a http://foo.iriscouch.com domain. So to put hckr.it on it you can do:

    couchapp push app.js http://username:password@foo.iriscouch.com

## You should probably host attachments/assets on a CDN

CouchDB is not very good in serving static content because it doesn't allow you to modify the Cache-Control property
to tell the browser to not fetch the content over and over. So an easy fix is to simply host the assets 
(things like images, .css, .js files etc.) on a [CDN](http://en.wikipedia.org/wiki/Content_delivery_network). A good one
is [CloudFare](http://www.cloudflare.com/).

The only asset that you need to keep an eye on is `site.js`. It has all the logic for the frontend of the app. So be sure to
update it from this repo, whenever an update is available.

## Beware! I was drinking lots of coffee when I wrote this

This is basically a Hacker News clone for CouchDB. As a Couchapp. 
I figured it would be awesome to have a Couchapp act as an entire news application such as HN.
This would enable anyone to easily host their own HN site, with their own topic.

### Each `item` and all its `comments` are stored inside a single document 

Why? Fuck it, that's why!

Here's what an `item` looks like:

    {
       "_id": "784289f1ac926d7d9ab85b3a22005546",
       "_rev": "5-7d72e6583ddd567d700a242618c3ce6f",
       "created_at": "2012-05-20T21:09:52.045Z",
       "author": "test",
       "title": "Nottingam",
       "url": "http://dfsf.com",
       "type": "item",
       "voted": [ 
           "test"
       ],
       "comments": [
           {
               "comment_id": "2012-05-21T07:43:01.410Z",
               "parent_id": 0,
               "text": "Ciao Bella",
               "voted": [
               ],
               "author": "test"
           },
           {
               "comment_id": "2012-05-21T07:48:50.963Z",
               "parent_id": "2012-05-21T07:43:01.410Z",
               "text": "My born super",
               "voted": [
               ],
               "author": "test"
           }
       ]
    } 

But what about document conflicts? Easy, I deal
with document conflicts on the Browser. Resend the request if it failed. Simple and relaxed approach.

### Front page order

Each `item` has a `voted` property which is an array of all the users
that have "upvoted" that item. In my `all` view I calculate the `score` based on the number of items in the voted array (which I call `points`) and the `created_at` property.

Then I simply `emit()` this score as the key of my view, which is then naturally ordered. This is the heart of the ranking algorithm and the order of the front-page, as well as comments.

### Designed for efficiency

Couch does only the very necessary. The rest is left to the browser to do. The hierarchy of the comments for example is done by the browser. However, everything is returned nicely by couch so that search engines can crawl it easily. 

The HTML returned is also the same as the one on Hacker News. This enables the re-use of already built crawlers and add-ons.
