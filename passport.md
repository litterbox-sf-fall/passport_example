## Objectives:

1. Explain our previous problem from the earlier bcrypt lesson (users have to keep logging in and we can't just stop anyone from going to /home)
2. Understand the basics of authorization and how passport.js helps.
3. Learn what cookies and sessions are and how passport uses them to remember users

### Start with the concepts

What are we trying to do here? Keep logged users in, make sure not logged in users don't get to our welcome page. Let's tackle each problem individually

Implement basic authorization:

1. Need some sort of strategy to do that (that's what a passport strategy is)
2. Assign a token of sorts to users who are 

Keep users logged in:

1. Sessions - stored on the server
2. Cookies - stored in the browser

So we need to store that user information somewhere...but where? A database? In memory? A cookie? Explain the benefits and downsides of each:

- Memory - lots of sessions, lots of memory taken up, and what happens when the server goes down?
- Database - very reasonable and common but potentially time consuming
- Cookie - easy but not always secure, what happens if someone grabs our cookie and is able to get session data out of it? That's why we need a secret key. There also isn't a lot of memory available per cookie, so we should be careful that we don't store too much. What's a good thing to store as a unique identifier for each user? An ID!
	
Alright, so we want to store our ID on a cookie and then have our server parse that data at a later time when the user comes back and we will know if that user has previously logged in...that's exactly what serializing and deserializing does! Compare to JSON.parse() and JSON.stringify()

So with that in mind, what do we need to build?

- a way to store sessions
- a way to add special data to users who we create (compare functions)
- a different way of displaying flash messages
- a way to check and see if a user is logged in

## Questions
- What does secret do with sessions? keys? -> a secret key for us to parse the session

- What does req.isAuthenitcated do? -> assigns an object to the req to authenticate

- What is req.user? -> ???

- What does req.logout() do? -> kills the users session

## PROCESS
1. set up a strategy with passport
2. Add session that writes to a cookie
3. serialize and deserialize users
4. set up some validation to make sure our users are logged in

## STEPS WE NEED TO DO
1. Include middlware (npm install --save passport, passport local)
2. refactor our authorize function to include passport's strategy
3. include passport use
4. in our log in, authenticate our users

## Coding Steps
1. NPM install and add middleware necessary
var express = require("express"),
  bodyParser = require("body-parser"),
  passport = require("passport"),
  passportLocal = require("passport-local").Strategy,
  cookieParser = require("cookie-parser"),
  cookieSession = require("cookie-session"),
  db = require("./models/index"),
  flash = require('connect-flash'),
  app = express();

2. Middleware for ejs, grabbing HTML and including static files
	app.set('view engine', 'ejs');
	app.use(bodyParser.urlencoded({extended: true}) ); 
	app.use(express.static(__dirname + '/public'));

3. Store our session data in a cookie:
store our session data in a cookie
app.use(cookieSession( {
  secret: 'thisismysecretkey',
  name: 'cookie created by elie'
  })
);

4. // get passport started
app.use(passport.initialize());
app.use(passport.session());
app.use(flash());

5. // prepare our serialize functions
passport.serializeUser(function(user, done){
  done(null, user.id);
});

passport.deserializeUser(function(id, done){
  db.user.find({
      where: {
        id: id
      }
    })
    .done(function(error,user){ 
      done(error, user);
    });
});

6. add passport strategy 
passport.use(new passportLocal.Strategy({
      usernameField: 'username',
      passwordField: 'password',
      passReqToCallback : true
    },

    function(req, username, password, done) {
      // find a user in the DB
      User.find({
          where: {
            username: username
          }
        })
        // when that's done, 
        .done(function(error,user){
          if(error){
            console.log(error);
            return done (err, req.flash('loginMessage', 'Oops! Something went wrong.'));
          }
          if (!user){
            return done (null, false, req.flash('loginMessage', 'Username does not exist.'));
          }
          if ((User.comparePass(password, user.password)) !== true){
            return done (null, false, req.flash('loginMessage', 'Invalid Password'));
          }
          done(null, user); 
        });
    }));
7. // authenticate users when logging in
app.post('/login', passport.authenticate('local', {
  successRedirect: '/home', 
  failureRedirect: '/login', 
  failureFlash: true
}));
8. Catch all for 404's
9. // catch-all for 404 errors 
app.get('*', function(req,res){
  res.status(404);
  res.render('404');
});
9. Build our views
10. Include bootstrap
	
