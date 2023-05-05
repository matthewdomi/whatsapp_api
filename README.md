# whatsapp_api : https://developers.facebook.com/blog/post/2022/10/31/sending-messages-with-whatsapp-in-your-nodejs-application/?d=%7B%22u%22%3A100024373305763%2C%22f%22%3A207799259245384%2C%22t%22%3A1683303827%2C%22ed%22%3A[]%7D&s=AWXi-NFZEj_Z1uOw5KQ

Back to News for Developers
Search News

TOPICS
EVENTS
YEAR
Sending Messages via WhatsApp in Your Node.js Application
October 31, 2022
ByDmitry Vinnik

Subscribe to Developer news

WhatsApp is one of the world’s most popular mobile messenger apps, with nearly two billion monthly active users. Businesses can leverage the WhatsApp Business Platform to communicate closely with their audiences, help boost sales, and transform the customer experience.

This article describes how the Cloud API, hosted by Meta, Meta’s integration of the WhatsApp Business Platform, can be used in a Node.js application to provide the capability to send and manage WhatsApp messages sent and received via the Cloud API.

Let’s dive in and explore how to create a Node.js web app powered with WhatsApp messaging from scratch to send a simple text-based message and then using message templates for more detailed messages. If you’d like a preview of where you’ll end up, you can download the complete application code.

Prerequisites
To send and receive messages using a test phone number, follow the Set up Developer Assets and Platform Access tutorial, making sure you complete the steps below.

First, you’ll need to download and install Node.js and npm on your machine, if you don’t have it installed already.

Next, Register for a free account as a developer with Meta for Developers.

Enable two-factor authentication for your account.

Create a Meta App. The App ID and the App Secret will be used later in this tutorial.

Connect your Meta App with the WhatsApp product.

Then, associate your app with a Business Manager account.

On the App Dashboard, open the WhatsApp > Get Started menu and configure a recipient phone number. Your app will need it as a recipient for the WhatsApp messages. This number will be used later on.

Create a system user for your Business Account. For a more detailed walkthrough, see our Business Manager documentation.

On the System Users page, generate a new token for your new system user. Assign your app all the available permissions. This token will be used later in this article.

On the System Users page, configure the assets to your System User, assigning your app with full control. Don’t forget to click the Save Changes button.

The App We’re Building
This small sample application will work as an online movie ticket purchase and booking service. The application will use the API to provide the user with an engaging and more personalized experience than email communication. When the users log in, they’re greeted by a WhatsApp message. Then, when they buy a movie ticket, they receive a message confirming the purchase.

Creating a Minimal App with Node.js and Express
To get started, you need to get a new Node.js project up and running. We'll use EJS as a lightweight JavaScript templating engine and Express, the minimalist web framework for Node.js.

Open terminal/command prompt and create a folder for your project to live in. Then execute the npm init command from the root of your project folder to initialize your project to use npm packages:

npm init
Create a starter application using express-generator:

npx express-generator -v ejs
Next, run the following command to install the packages and resolve their dependencies:

npm install
Finally, execute the following command to start the server and host your app locally:

npm start
Now visit http://localhost:3000, and you’ll see the homepage of your Node + Express starter application:

Creating the Sample Login Page
To start your movie ticket application, you’ll create a sample login form that will work as your homepage. You’ll need to call the render function of the Response object to render a view from a separate HTML file.

Open the routes\index.js file and replace its code with the following contents:

var express = require('express');
var router = express.Router();

/_ GET home page. _/
router.get('/', function(req, res, next) {
res.render('index', { title: 'Login' });
});

module.exports = router;
Next, open the views\index.ejs file and add the HTML content below. Here, you’re creating a sample form login that comes with a stand-in login and password, so that you don’t need to provide those to use the application.

For your web app front end, use Bootstrap. This popular library will help you build a consistent, lightweight UI that comes with responsive styling, allowing you to easily run your app across devices with simplified CSS rules.

<!DOCTYPE html>
<html>
  <head>
    <title>Movie Ticket Demo for Node.js</title>
    <h1 class="text-center">Movie Ticket Demo for Node.js</h1>
    <link
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC"
      crossorigin="anonymous"
    />
  </head>
  <body>
    <div class="d-flex flex-row justify-content-center align-items-center">
      <div class="border px-3">
        <div class="row">
          <div class="col-sm-6 d-none d-sm-block">
            <img
              src="https://cdn.pixabay.com/photo/2015/12/09/17/12/popcorn-1085072__340.jpg"
              alt="Login image"
              class="w-100 vh-50 pt-3 pb-3"
              style="object-fit: cover; object-position: left"
            />
          </div>
          <div class="col-sm-6 text-black">
            <div class="px-5 ms-xl-4">
              <i
                class="fas fa-crow fa-2x me-3 pt-5 mt-xl-4"
                style="color: #709085"
              ></i>
            </div>

            <div class="d-flex align-items-center h-custom-2">
              <form class="w-100" method="post" action="/welcome">
                <div class="form-outline mb-4">
                  <input
                    type="text"
                    value="this_is_a_demo@email.com"
                    disabled
                    class="form-control form-control-md text-muted"
                  />
                </div>

                <div class="form-outline mb-4">
                  <input
                    type="text"
                    value="••••••••••••••••"
                    disabled
                    id="form2Example28"
                    class="form-control form-control-md text-muted"
                  />
                </div>

                <div class="pt-1 mb-4">
                  <input type="submit" class="btn btn-info btn-lg btn-block" value="Login"/>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>

    </div>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

  </body>
</html>
Then, restart the app again to see the new login page, by using CTRL+C and the command:

npm start
Sending Text Messages with Node.js and WhatsApp Business
Your Node.js application will need to use specific data from your Meta developer account created in the beginning of this article. For the convenience of having all of your configuration in one place, and not scattered throughout code during development, place it in a file.

Create a .env file at the project root with the following configurations:

APP_ID=<<YOUR-WHATSAPP-BUSINESS-APP_ID>>
APP_SECRET=<<YOUR-WHATSAPP-BUSINESS-APP_SECRET>>
RECIPIENT_WAID=<<YOUR-RECIPIENT-TEST-PHONE-NUMBER>>
VERSION=v13.0
PHONE_NUMBER_ID=<<YOUR-WHATSAPP-BUSINESS-PHONE-NUMBER-ID>>
ACCESS_TOKEN=<<YOUR-SYSTEM-USER-ACCESS-TOKEN>>
Note: Replace each of the above settings with data from your WhatsApp Business account Dashboard.

Your login form action tells the app to POST to the /welcome route. So, you’ll need a new router to:

Handle the “welcome” HTTP POST request.
Obtain the configuration needed for the welcome message.
Send a welcome message via the API.
Redirect the app to the homepage once the message is sent
Next, create a new routes\welcome.js file with the following code:

var express = require('express');
var router = express.Router();
var bodyParser = require('body-parser');
require('dotenv').config()
const { sendMessage, getTextMessageInput } = require("../messageHelper");

router.use(bodyParser.json());

router.post('/', function(req, res, next) {
var data = getTextMessageInput(process.env.RECIPIENT_WAID, 'Welcome to the Movie Ticket Demo App for Node.js!');

sendMessage(data)
.then(function (response) {
res.redirect('/);
res.sendStatus(200);
return;
})
.catch(function (error) {
console.log(error);
console.log(error.response.data);
res.sendStatus(500);
return;
});
});

module.exports = router;
Next, you’ll need the function to encapsulate the code that sends basic text messages via the API. Create a new messageHelper.js file at the project root with the following code:

var axios = require('axios');

function sendMessage(data) {
var config = {
method: 'post',
url: `https://graph.facebook.com/${process.env.VERSION}/${process.env.PHONE_NUMBER_ID}/messages`,
headers: {
'Authorization': `Bearer ${process.env.ACCESS_TOKEN}`,
'Content-Type': 'application/json'
},
data: data
};

return axios(config)
}

function getTextMessageInput(recipient, text) {
return JSON.stringify({
"messaging_product": "whatsapp",
"preview_url": false,
"recipient_type": "individual",
"to": recipient,
"type": "text",
"text": {
"body": text
}
});
}

module.exports = {
sendMessage: sendMessage,
getTextMessageInput: getTextMessageInput
};
The code above makes an HTTP POST request to the /messages endpoint on the Meta Graph API at graph.facebook.com, passing:

The Cloud API version you’re working with
The test phone number that will receive the message (you’ve already configured that)
The access token you generated for your System User
Also, note that the getTextMessageInput function returns a specific data structure required for sending basic text messages.

Moving on, open the app.js file and create a router variable for the /welcome route:

var welcomeRouter = require('./routes/welcome');
Then enable the app to use the new welcomeRouter variable:

app.use('/welcome', welcomeRouter);
Finally, restart the app again to see the new login page, by using CTRL+C and the command:

> npm start
> When the login screen loads, click the Login button. You’ll see the WhatsApp notification popping up on your screen:

Click the notification that appears to open WhatsApp and view the basic text message sent by the Node.js application:

So far, you’ve sent simple messages using WhatsApp. Next, you’ll send more complex messages using templates.

Creating the Movie Ticket Catalog Page
The next step is to create a catalog of available movies and their details so that online customers can buy tickets. This data will be stored in a separate file.

Create a new file at public\javascripts\movies.js with the following content:

var movies = [{
id: 1,
title: 'Airborne 404',
thumbnail: 'https://cdn.pixabay.com/photo/2014/10/02/06/34/war-469503__480.jpg',
license: 'https://pixabay.com/photos/war-soldiers-parachutes-469503/',
licenseDetails: 'Pixabay License / Free for commercial use / No attribution required',
time: 'October 23, 2022 - 8:00 PM',
venue: 'Cinema Daily'
},
{
id: 2,
title: 'Storm in the Pacific',
thumbnail: 'https://cdn.pixabay.com/photo/2017/04/04/19/47/ship-2202910__340.jpg',
license: 'https://pixabay.com/photos/ship-strom-sea-night-fantasy-red-2202910/',
licenseDetails: 'Pixabay License / Free for commercial use / No attribution required',
time: 'October 24, 2022 - 7:30 PM',
venue: 'Famous Studios'
},
{
id: 3,
title: 'Alienzz Game - The Movie',
thumbnail: 'https://cdn.pixabay.com/photo/2016/10/13/00/22/illustration-1736462__340.png',
license: 'https://pixabay.com/vectors/illustration-videogame-graphics-1736462/',
licenseDetails: 'Pixabay License / Free for commercial use / No attribution required',
time: 'October 24, 2022 - 10:30 PM',
venue: 'FilmStreamNow'
},
{
id: 4,
title: 'Captain Doctor Angel II',
thumbnail: 'https://cdn.pixabay.com/photo/2018/05/18/11/12/doctor-3410941__340.jpg',
license: 'https://pixabay.com/photos/doctor-physician-angel-care-3410941/',
licenseDetails: 'Pixabay License / Free for commercial use / No attribution required',
time: 'October 25, 2022 - 9:30 PM',
venue: 'Uptown Studio'
}];

if (exports ) {
exports.movies = movies;
}
You now need a new route for users to access the movie catalog page.

Create a new file at \routes\catalog.js with the following content to render the catalog page with the movies data:

var express = require('express');
const { movies } = require("../public/javascripts/movies");
var router = express.Router();

/_ GET home page. _/
router.post('/', function(req, res, next) {
res.render('catalog', { title: 'Movie Ticket Demo for Node.js', movies: movies });
});

router.get('/', function(req, res, next) {
res.render('catalog', { title: 'Movie Ticket Demo for Node.js', movies: movies });
});

module.exports = router;
Now create a new file at \views\catalog.ejs with the following content, which renders the movie data using the EJS template syntax:

<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <h1 class="text-center"><%= title %></h1>
    <p class="text-center">Welcome to <%= title %></p>

    <div class="container">
      <div class="row">

        <% movies.forEach(function(movie,index) { %>
          <div class="col col-3">
            <div class="card">
              <img src="<%=movie.thumbnail%>" class="card-img-top" alt="<%=movie.title %>"/>
              <div class="card-body">
                <h5 class="card-title"><%= movie.title %></h5>
                <p class="card-text"><%= movie.time %></p>
                <p class="card-text"><%= movie.venue %></p>
                <p>
                  <form method="POST" action="/buyTicket">
                    <div class="container">
                      <input type="hidden" name="id" value="<%=movie.id%>"/>
                      <div class="row gx-5">
                        <select name="seats" id="selSeats" onchange="javascript:updateSeats(<%=movie.id%>, this.value);" class="col col-7">
                          <%
                          var options = [1,2,3,4,5];
                          for (i in options)
                          {
                              var selected = ( movie.seats == options[i] ) ? "selected" : "";
                              %><option value="<%=options[i]%>" <%=selected %>><%=options[i] %> seats</option><%
                          }
                          %>
                        </select>
                        <input type="submit" value="&#127915; Buy" class="col col-5"/>
                      </div>
                    </div>
                  </form>
                </p>
              </div>
            </div>
          </div>
        <% }) %>
      </div>
    </div>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
    <script src="javascripts/movies.js"></script>

  </body>
</html>
Now you have to make the welcome endpoint redirect to the catalog page once the welcome message is sent to the user. Open the routes\welcome.js file and modify the redirect to the /catalog route :

sendMessage(data)
.then(function (response) {
res.redirect('/catalog');
Next, open the app.js file and create a router variable for the \catalog route:

var catalogRouter = require('./routes/catalog');
Then enable the app to use the new catalogRouter variable:

app.use('/catalog', catalogRouter);
Finally, restart the app again to see the new login page, by using CTRL+C and the command:

> npm start
> Now click the Login button. This will send your WhatsApp number a welcome message and redirect you to the catalog view:

Note that there’s a button to buy the ticket for each movie displayed on the screen above. Next, you need to configure the application to process the ticket purchase.

Sending Templated Messages with Node.js and WhatsApp Business
A message template is required to start a business-initiated conversation. These conversations can be customer care messages or appointment reminders, payment or shipping updates, alerts, and more.

Create new router code at routes\buyTicket.js:

var express = require('express');
var router = express.Router();
var bodyParser = require('body-parser');
require('dotenv').config()
var { movies } = require('../public/javascripts/movies');
const { sendMessage, getTemplatedMessageInput } = require("../messageHelper");

router.use(bodyParser.json());

router.post('/', function(req, res, next) {
var movie = movies.filter((v,i) => v.id == req.body.id)[0];

var data = getTemplatedMessageInput(process.env.RECIPIENT_WAID, movie, req.body.seats);

sendMessage(data)
.then(function (response) {
res.redirect('/catalog');
res.sendStatus(200);
return;
})
.catch(function (error) {
console.log(error);
res.sendStatus(500);
return;
});
});

module.exports = router;
The following code to send the ticket purchase message is similar to what you did for the welcome message. Open the messageHelper.js file and add the getTemplatedMessageInput function:

function getTemplatedMessageInput(recipient, movie, seats) {
return JSON.stringify({
"messaging_product": "whatsapp",
"to": recipient,
"type": "template",
"template": {
"name": "sample_movie_ticket_confirmation",
"language": {
"code": "en_US"
},
"components": [
{
"type": "header",
"parameters": [
{
"type": "image",
"image": {
"link": movie.thumbnail
}
}
]
},
{
"type": "body",
"parameters": [
{
"type": "text",
"text": movie.title
},
{
"type": "date_time",
"date_time": {
"fallback_value": movie.time
}
},
{
"type": "text",
"text": movie.venue
},
{
"type": "text",
"text": seats
}
]
}
]
}
}
);
}

module.exports = {
sendMessage: sendMessage,
getTextMessageInput: getTextMessageInput,
getTemplatedMessageInput: getTemplatedMessageInput
};
Note that we are using the sample_movie_ticket_confirmation template above, where you provided the movie thumbnail, the movie title, the date/time, the location, and the number of the seats. You can experiment with other available templates or create new ones, by visiting the Message Templates page.

Now open the app.js file and create a router variable for the buyTicket route:

var buyTicketRouter = require('./routes/buyTicket);
Then make the app use the new welcomeRouter:

app.use('/buyTicket', buyTicketRouter');
Finally, restart the app again to see the new login page, by using CTRL+C and the command:

> npm start
> From the ticket screen, select three seats under a movie card:

Then click the Buy button. This will cause your app to send a template message to your test phone number via

WhatsApp:

Now, open WhatsApp to see the template message.

That’s it!

As you can see, sending messages from Node.js code is straightforward. However, here are some tips and best practices for integrating WhatsApp into applications:

Even if you’re automating your app messages, make sure that communication with customers doesn’t feel robotic. People expect a more personal experience, so try sending personalized messages.
Explore a more relaxed and informal tone.
Keep your text messages clear and to the point.
When using templates, provide rich context information by using links to documents, videos, or images such as those above, where we depicted the movies related to the tickets.
Conclusion
This article demonstrated how to add messaging capability to a Node.js app by integrating it with a WhatsApp Business account.

First, you created a simple Node.js application from scratch. Then, you added a sample login page and configured the application to send basic welcome messages to users via the API. Finally, you added a catalog page and configured it to send template messages with movie ticket details to customers.

But this is only the tip of the iceberg. You can explore our documentation to see how you can get the most out of the Cloud API.

Tags:
Business Tools
Platforms
2022
WhatsApp
RELATED NEWS

Building for Success: How to Visualize WhatsApp Account Metrics
Business Tools
Developer Tools
Platforms
2022
WhatsApp
December 19, 2022

Using QR Codes and Short Links with the WhatsApp Business Platform
Business Tools
Developer Tools
Platforms
2022
WhatsApp
December 12, 2022

Understanding Authorization Tokens and Access for the WhatsApp Business Platform
Business Tools
Platforms
2022
WhatsApp
December 5, 2022
More News

Products
Artificial Intelligence
AR/VR
Business Tools
Gaming
Open Source
Publishing
Social Integrations
Social Presence
Programs
Developer Circles
ThreatExchange
Support
Developer Support
Bugs
Platform Status
Report a Platform Data Incident
Facebook for Developers Community Group
Sitemap
News
Blog
Success Stories
Videos
Meta for Developers Page
Terms and Policies
Platform Initiatives Hub
Platform Terms
Developer Policies
European Commission Commitments
Follow Us
Follow us on FacebookFollow us on InstagramFollow us on TwitterFollow us on LinkedInFollow us on YouTube
© 2023 Meta
About
Create Ad
Careers
Privacy Policy
Cookies
Terms

English (US)

English (US)
