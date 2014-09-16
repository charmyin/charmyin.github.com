---
layout: post
title: "Password Authentication with Mongoose (Part 1): bcrypt"
description: "Password Authentication with Mongoose (Part 1): bcrypt"
category: [Nodejs]
tags: [nodejs, javascript, authenticate]
---

Via [Password Authentication with Mongoose (Part 1): bcrypt](http://devsmash.com/blog/password-authentication-with-mongoose-and-bcrypt)


This post is Part 1 (of 2) on implementing secure username/password authentication for your Mongoose User models. In this first installment, we will discuss how to implement one-way encryption of user passwords with bcrypt, and how to subsequently use the encrypted password for login verification.

Update: Password Authentication with Mongoose (Part 2): Account Locking is now live!

### Cast of Characters

Mongoose
From the Mongoose GitHub repo: "Mongoose is a MongoDB object modeling tool designed to work in an asynchronous environment." In other words, Mongoose provides a model layer for interacting with your MongoDB collections from Node. This model layer provides a common location for implementing document validation, persistence indirection, and other logic that should be abstracted from the business layer.

#### Website: http://mongoosejs.com/

### node.bcrypt.js

node.bcrypt.js is, well, bcrypt for Node. If you're not familiar with bcrypt and why it's a good thing, then I highly recommended Coda Hale's excellent article on how to safely store a password.

#### Website: https://github.com/ncb000gt/node.bcrypt.js/

### Objectives

Before we get into the code, let's identify some objectives/requirements in our initial username/password authentication implementation:

The User model should fully encapsulate the password encryption and verification logic
The User model should ensure that the password is always encrypted before saving
The User model should be resistant to program logic errors, like double-encrypting the password on user updates
bcrypt interactions should be performed asynchronously to avoid blocking the event loop (bcrypt also exposes a synchronous API)
Step 1: The User Model
Even if you aren't too familiar with Mongoose schemas and models, the code below should be fairly easy to follow. It wouldn't be a bad idea to read through some of the basics though.

To start things off, let's create our bare bones representation of a user; for the sake of this article, all we need is a username and a password:

	var mongoose = require('mongoose'),
	    Schema = mongoose.Schema,
	    bcrypt = require('bcrypt'),
	    SALT_WORK_FACTOR = 10;

	var UserSchema = new Schema({
	    username: { type: String, required: true, index: { unique: true } },
	    password: { type: String, required: true }
	});

	module.exports = mongoose.model('User', UserSchema);

### Step 2: Password Hashing Middleware

You might have noticed the unused bcrypt and SALT_WORK_FACTOR references above - we'll be using them in this step. As a quick refresher, the purpose of the salt is to defeat rainbow table attacks and to resist brute-force attacks in the event that someone has gained access to your database. bcrypt in particular uses a key setup phase that is derived from Blowfish. For the purposes of this article, all you need to know about that is that the key setup phase is very computationally expensive, which is actually a good thing when trying to thwart brute-force attacks. How expensive depends on how many rounds or iterations the key setup phase uses - this is where our SALT_WORK_FACTOR comes into play. The default used by node.bcrypt.js is 10, so I went ahead and made our explicit value the same.

Alright, let's get back to the code. The first thing we want to add to our User model is some Mongoose middleware that will automatically hash the password before it's saved to the database. Here's what that looks like:

		UserSchema.pre('save', function(next) {
		    var user = this;

		    // only hash the password if it has been modified (or is new)
		    if (!user.isModified('password')) return next();

		    // generate a salt
		    bcrypt.genSalt(SALT_WORK_FACTOR, function(err, salt) {
		        if (err) return next(err);

		        // hash the password along with our new salt
		        bcrypt.hash(user.password, salt, function(err, hash) {
		            if (err) return next(err);

		            // override the cleartext password with the hashed one
		            user.password = hash;
		            next();
		        });
		    });
		});
The above code will accomplish our goal of always hashing the password when a document is saved to the database. There are a couple things to be aware of though:

Because passwords are not hashed until the document is saved, be careful if you're interacting with documents that were not retrieved from the database, as any passwords will still be in cleartext.
Mongoose middleware is not invoked on update() operations, so you must use a save() if you want to update user passwords.

### Step 3: Password Verification

Now that we have our User model and we're hashing passwords, the only thing left is to implement password verification. Adding this to our model turns out to be just a few more lines of code:

	UserSchema.methods.comparePassword = function(candidatePassword, cb) {
	    bcrypt.compare(candidatePassword, this.password, function(err, isMatch) {
	        if (err) return cb(err);
	        cb(null, isMatch);
	    });
	};
	Simple enough.

	Altogether Now
	Here's what our User model looks like after adding our middleware and password verification method:

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
	Sample Usage
	Assuming that you've saved the above code as user-model.js, here's how you would go about testing it:

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

	    // fetch user and test password verification
	    User.findOne({ username: 'jmar777' }, function(err, user) {
	        if (err) throw err;

	        // test a matching password
	        user.comparePassword('Password123', function(err, isMatch) {
	            if (err) throw err;
	            console.log('Password123:', isMatch); // -> Password123: true
	        });

	        // test a failing password
	        user.comparePassword('123Password', function(err, isMatch) {
	            if (err) throw err;
	            console.log('123Password:', isMatch); // -> 123Password: false
	        });
	    });
	});

### Next Steps

This post was just Part 1 on implementing secure username/password authentication for your Mongoose User models. Stay tuned for Part 2, in which we'll discuss preventing brute-force attacks by enforcing a maximum number of failed login attempts.

Thanks for reading!

Update: Password Authentication with Mongoose (Part 2): Account Locking is now live!