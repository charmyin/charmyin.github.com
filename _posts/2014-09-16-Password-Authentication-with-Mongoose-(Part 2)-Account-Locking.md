---
layout: post
title: "Password Authentication with Mongoose (Part 2): Account Locking"
description: "Password Authentication with Mongoose (Part 2): Account Locking"
category: [Nodejs]
tags: [nodejs, javascript, authenticate]
---

Via [1.Password Authentication with Mongoose (Part 2): Account Locking](http://devsmash.com/blog/implementing-max-login-attempts-with-mongoose)

This post is Part 2 (of 2) on implementing secure username/password authentication for your Mongoose User models. In Part 1 we implemented one-way password encryption and verification using bcrypt. Here in Part 2 we'll discuss how to prevent brute-force attacks by enforcing a maximum number of failed login attempts.

### Quick Review

If you haven't done so already, I recommend you start with reading Part 1. However, if you're like me and usually gloss over the paragraph text looking for code, here's what our User model looked like when we left off:

		var mongoose = require('mongoose'),
		    Schema = mongoose.Schema,
		    bcrypt = require('bcrypt'),
		    SALT_WORK_FACTOR = 10;

		var UserSchema = new Schema({
		    username: { type: String, required: true, index: { unique: true } },
		    password: { type: String, required: true }
		});

		UserSchema.pre('save', function(next) {
		    var user = this;

		    // only hash the password if it has been modified (or is new)
		    if (!user.isModified('password')) return next();

		    // generate a salt
		    bcrypt.genSalt(SALT_WORK_FACTOR, function(err, salt) {
		        if (err) return next(err);

		        // hash the password using our new salt
		        bcrypt.hash(user.password, salt, function(err, hash) {
		            if (err) return next(err);

		            // override the cleartext password with the hashed one
		            user.password = hash;
		            next();
		        });
		    });
		});

		UserSchema.methods.comparePassword = function(candidatePassword, cb) {
		    bcrypt.compare(candidatePassword, this.password, function(err, isMatch) {
		        if (err) return cb(err);
		        cb(null, isMatch);
		    });
		};

		module.exports = mongoose.model('User', UserSchema);

As can be seen, there's not much too it - we hash passwords before documents are saved to MongoDB, and we provide a basic convenience method for comparing passwords later on.

###  Why do we Need Account Locking?

While our code from Part 1 is functional, it can definitely be improved upon. Hashing passwords will save your bacon if a hacker gains access to your database, but it does nothing to prevent brute-force attacks against your site's login form. This is where account locking comes in: after a specific number of failed login attempts, we simply ignore subsequent attempts, thereby putting the kibosh on the brute-force attack.

Unfortunately, this still isn't perfect. As stated by OWASP:

Password lockout mechanisms have a logical weakness. An attacker that undertakes a large numbers of authentication attempts on known account names can produce a result that locks out entire blocks of application users accounts.
The prescribed solution, then, is to continue to lock accounts when a likely attack is encountered, but then unlock the account after some time has passed. Given that a sensible password policy puts the password search space into the hundreds of trillions (or better), we don't need to be too worried about allowing another five guesses every couple of hours or so.

### Requirements

In light of the above, let's define our account locking requirements:

A user's account should be "locked" after some number of consecutive failed login attempts
A user's account should become unlocked once a sufficient amount of time has passed
The User model should expose the reason for a failed login attempt to the application (though not necessarily to the end user)

### Step 1: Keeping Track of Failed Login Attempts and Account Locks

In order to satisfy our first and second requirements, we'll need a way to keep track of failed login attempts and, if necessary, how long an account is locked for. An easy solution for this is to add a couple properties to our User model:

		var UserSchema = new Schema({
		    // existing properties
		    username: { type: String, required: true, index: { unique: true } },
		    password: { type: String, required: true },
		    // new properties
		    loginAttempts: { type: Number, required: true, default: 0 },
		    lockUntil: { type: Number }
		});

loginAttempts will store how many consecutive failures we have seen, and lockUntil will store a timestamp indicating when we may stop ignoring login attempts.

### Step 2: Defining Failed Login Reasons

In order to satisfy our third requirement, we'll need some way to represent why a login attempt has failed. Our User model only has three reasons it needs to keep track of:

The specified user was not found in the database
The provided password was incorrect
The maximum number of login attempts has been exceeded
Any other reason for a failed login will simply be an error scenario. To describe these reasons, we're going to kick it old school with a faux-enum:

		// expose enum on the model
		UserSchema.statics.failedLogin = {
		    NOT_FOUND: 0,
		    PASSWORD_INCORRECT: 1,
		    MAX_ATTEMPTS: 2
		};

Please note that it is almost always a bad idea to tell the end user why a login has failed. It may be acceptable to communicate that the account has been locked due to reason 3, but you should consider doing this via email if at all possible.

### Step 3: Encapsulating the Login Process

Lastly, let's make life easier on the consuming code base by encapsulating the whole login process. Given that our security requirements have become much more sophisticated, we'll allow external code to interact through a single User.getAuthenticated() static method. This method will operate as follows:

User.getAuthenticated() accepts a username, a password, and a callback (cb)
If the provided credentials are valid, then the matching user is passed to the callback
If the provided credentials are invalid (or maximum login attempts has been reached), then null is returned instead of the user, along with an appropriate enum value
If an error occurs anywhere in the process, we maintain the standard "errback" convention
We'll also be adding a new helper method (user.incLoginAttempts()) and a virtual property (user.isLocked) to help us out internally.

Because our User model is starting to get somewhat large, I'm just going to jump straight to the end result with everything included:

		var mongoose = require('mongoose'),
		    Schema = mongoose.Schema,
		    bcrypt = require('bcrypt'),
		    SALT_WORK_FACTOR = 10,
		    // these values can be whatever you want - we're defaulting to a
		    // max of 5 attempts, resulting in a 2 hour lock
		    MAX_LOGIN_ATTEMPTS = 5,
		    LOCK_TIME = 2 * 60 * 60 * 1000;

		var UserSchema = new Schema({
		    username: { type: String, required: true, index: { unique: true } },
		    password: { type: String, required: true },
		    loginAttempts: { type: Number, required: true, default: 0 },
		    lockUntil: { type: Number }
		});

		UserSchema.virtual('isLocked').get(function() {
		    // check for a future lockUntil timestamp
		    return !!(this.lockUntil && this.lockUntil > Date.now());
		});

		UserSchema.pre('save', function(next) {
		    var user = this;

		    // only hash the password if it has been modified (or is new)
		    if (!user.isModified('password')) return next();

		    // generate a salt
		    bcrypt.genSalt(SALT_WORK_FACTOR, function(err, salt) {
		        if (err) return next(err);

		        // hash the password using our new salt
		        bcrypt.hash(user.password, salt, function (err, hash) {
		            if (err) return next(err);

		            // set the hashed password back on our user document
		            user.password = hash;
		            next();
		        });
		    });
		});

		UserSchema.methods.comparePassword = function(candidatePassword, cb) {
		    bcrypt.compare(candidatePassword, this.password, function(err, isMatch) {
		        if (err) return cb(err);
		        cb(null, isMatch);
		    });
		};

		UserSchema.methods.incLoginAttempts = function(cb) {
		    // if we have a previous lock that has expired, restart at 1
		    if (this.lockUntil && this.lockUntil < Date.now()) {
		        return this.update({
		            $set: { loginAttempts: 1 },
		            $unset: { lockUntil: 1 }
		        }, cb);
		    }
		    // otherwise we're incrementing
		    var updates = { $inc: { loginAttempts: 1 } };
		    // lock the account if we've reached max attempts and it's not locked already
		    if (this.loginAttempts + 1 >= MAX_LOGIN_ATTEMPTS && !this.isLocked) {
		        updates.$set = { lockUntil: Date.now() + LOCK_TIME };
		    }
		    return this.update(updates, cb);
		};

		// expose enum on the model, and provide an internal convenience reference 
		var reasons = UserSchema.statics.failedLogin = {
		    NOT_FOUND: 0,
		    PASSWORD_INCORRECT: 1,
		    MAX_ATTEMPTS: 2
		};

		UserSchema.statics.getAuthenticated = function(username, password, cb) {
		    this.findOne({ username: username }, function(err, user) {
		        if (err) return cb(err);

		        // make sure the user exists
		        if (!user) {
		            return cb(null, null, reasons.NOT_FOUND);
		        }

		        // check if the account is currently locked
		        if (user.isLocked) {
		            // just increment login attempts if account is already locked
		            return user.incLoginAttempts(function(err) {
		                if (err) return cb(err);
		                return cb(null, null, reasons.MAX_ATTEMPTS);
		            });
		        }

		        // test for a matching password
		        user.comparePassword(password, function(err, isMatch) {
		            if (err) return cb(err);

		            // check if the password was a match
		            if (isMatch) {
		                // if there's no lock or failed attempts, just return the user
		                if (!user.loginAttempts && !user.lockUntil) return cb(null, user);
		                // reset attempts and lock info
		                var updates = {
		                    $set: { loginAttempts: 0 },
		                    $unset: { lockUntil: 1 }
		                };
		                return user.update(updates, function(err) {
		                    if (err) return cb(err);
		                    return cb(null, user);
		                });
		            }

		            // password is incorrect, so increment login attempts before responding
		            user.incLoginAttempts(function(err) {
		                if (err) return cb(err);
		                return cb(null, null, reasons.PASSWORD_INCORRECT);
		            });
		        });
		    });
		};

		module.exports = mongoose.model('User', UserSchema);

### Sample Usage

Assuming that you've saved the above code as user-model.js, here's how you would go about using it:

		var mongoose = require('mongoose'),
		    User = require('./user-model');

		var connStr = 'mongodb://localhost:27017/mongoose-bcrypt-test';
		mongoose.connect(connStr, function(err) {
		    if (err) throw err;
		    console.log('Successfully connected to MongoDB');
		});

		// create a user a new user
		var testUser = new User({
		    username: 'jmar777',
		    password: 'Password123'
		});

		// save user to database
		testUser.save(function(err) {
		    if (err) throw err;

		    // attempt to authenticate user
		    User.getAuthenticated('jmar777', 'Password123', function(err, user, reason) {
		        if (err) throw err;

		        // login was successful if we have a user
		        if (user) {
		            // handle login success
		            console.log('login success');
		            return;
		        }

		        // otherwise we can determine why we failed
		        var reasons = User.failedLogin;
		        switch (reason) {
		            case reasons.NOT_FOUND:
		            case reasons.PASSWORD_INCORRECT:
		                // note: these cases are usually treated the same - don't tell
		                // the user *why* the login failed, only that it did
		                break;
		            case reasons.MAX_ATTEMPTS:
		                // send email or otherwise notify user that account is
		                // temporarily locked
		                break;
		        }
		    });
		});