---
layout: post
title: "Understanding node.js"
description: "Understanding node.js"
category: [Nodejs]
tags: [Nodejs]
---

Reprint from [Understanding node.js](http://debuggable.com/posts/understanding-node-js:4bd98440-45e4-4a9a-8ef7-0f7ecbdd56cb/)


<hr/>



Node.js has generally caused two reactions in people I've introduced it to. Basically people either "got it" right away, or they ended up being very confused.

If you have been in the second group so far, here is my attempt to explain node:

It is a command line tool. You download a tarball, compile and install the source.
It let's you run JavaScript programs by typing 'node my_app.js' in your terminal.
The JS is executed by the V8 javascript engine (the thing that makes Google Chrome so fast).
Node provides a JavaScript API to access the network and file system
"But I can do everything I need in: ruby, python, php, java, ... !".

I hear you. And you are right! Node is no freaking unicorn that will come and do your work for you, sorry. It's just a tool, and it probably won't replace your regular tools completely, at least not for now.

"Get to the point!"

Alright, I will. Node is basically very good when you need to do several things at the same time. Have you ever written a piece of code and said "I wish this would run in parallel"? Well, in node everything runs in parallel, except your code.

"Huh?"

That's right, everything runs in parallel, except your code. To understand that, imagine your code is the king, and node is his army of servants.

The day starts by one servant waking up the king and asking him if he needs anything. The king gives the servant a list of tasks and goes back to sleep a little longer. The servant now distributes those tasks among his colleagues and they get to work.

Once a servant finishes a task, he lines up outside the kings quarter to report. The king lets one servant in at a time, and listens to things he reports. Sometimes the king will give the servant more tasks on the way out.

Life is good, for the king's servants carry out all of his tasks in parallel, but only report with one result at a time, so the king can focus. *

"That's fantastic, but could you quit the silly metaphor and speak geek to me?"

Sure. A simple node program may look like this:

var fs = require('fs')
  , sys = require('sys');

fs.readFile('treasure-chamber-report.txt', function(report) {
  sys.puts("oh, look at all my money: "+report);
});

fs.writeFile('letter-to-princess.txt', '...', function() {
  sys.puts("can't wait to hear back from her!");
});
Your code gives node the two tasks to read and write a file, and then goes to sleep. Once node has completed a task, the callback for it is fired. But there can only be one callback firing at the same time. Until that callback has finished executing, all other callbacks have to wait in line. In addition to that, there is no guarantee on the order in which the callbacks will fire.

"So I don't have to worry about code accessing the same data structures at the same time?"

You got it! That's the entire beauty of JavaScripts single-threaded / event loop design!

"Very nice, but why should I use it?"

One reason is efficiency. In a web application, your main response time cost is usually the sum of time it takes to execute all your database queries. With node, you can execute all your queries at once, reducing the response time to the duration it takes to execute the slowest query.

Another reason is JavaScript. You can use node to share code between the browser and your backend. JavaScript is also on its way to become a really universal language. No matter if you did python, ruby, java, php, ... in the past, you've probably picked up some JS along the way, right?

And the last reason is raw speed. V8 is constantly pushing the boundaries in being one of the fastest dynamic language interpreters on the planet. I can't think of any other language that is being pushed for speed as aggressively as JavaScript is right now. In addition to that, node's I/O facilities are really light weight, bringing you as close to fully utilizing your system's full I/O capacities as possible.

"So you are saying I should write all my apps in node from now on?"

Yes and no. Once you start to swing the node hammer, everything is obviously going to start looking like a nail. But if you're working on something with a deadline, you might want to base your decision on:

Are low response times / high concurrency important? Node is really good at that.
How big is the project? Small projects should be fine. Big projects should evaluate carefully (available libraries, resources to fix a bug or two upstream, etc.).
"Does node run on Windows?"

No. If you are on windows, you need to run a virtual machine (I recommend VirtualBox) with Linux. Windows support for node is planned, but don't hold your breath for the next few months unless you want to help with the port.

"Can I access the DOM in node?"

Excellent question! No, the DOM is a browser thingy, and node's JS engine (V8) is thankfully totally separate from all that mess. However, there are people working on implementing the DOM as a node module, which may open very exciting possibilities such as unit testing client-side code.

"Isn't event driven programming really hard?"

That depends on you. If you already learned how to juggle AJAX calls and user events in the browser, getting used to node shouldn't be a problem.

Either way, test driven development can really help you to come up with maintainable designs.

"Who is using it?"

There is a small / incomplete list in the node wiki (scroll to "Companies using Node"). Yahoo is experimenting with node for YUI, Plurk is using it for massive comet and Paul Bakaus (of jQuery UI fame) is building a mind-blowing game engine that has some node in the backend. Joyent has hired Ryan Dahl (the creator of node) and heavily sponsors the development.

Oh, and Heroku just announced (experimental ) hosting support for node.js as well.

"Where can I learn more?"

Tim Caswell is running the excellent How To Node blog. Follow #nodejs on twitter. Subscribe to the mailing list. And come and hang out in the IRC channel, #node.js (yes, the dot is in the name). We're close to hitting the 200 lurker-mark there soon : ).

I'll also continue to write articles here on debuggable.com.

That's it for now. Feel free to comment if you have more questions!

--fg

*: The metaphor is obviously a simplification, but if it's hard to find a counterpart for the concept of non-blocking in reality.
