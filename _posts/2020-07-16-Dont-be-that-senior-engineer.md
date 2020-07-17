---
layout: post
title: Don't be that engineer
---

The ongoing lockdown and work-from-home has many affects. One of the side effect is being able to eavesdrop to the meetings or discussions your partner/apartment-mate
is having with his/her team.

I am in that type of a situation, as I am sure many others would be. The below is a compendium of various anecdotes I came across about my partner's team's senior
engineers, while listening (intentionally or un-intentionally) to their video conference discussions. Here, senior engineers mean 10+ years of experience in software development and architecture.
 
# A Debugger port is NOT the server port

So, there was a debug session going on. The spring server was not able to connect to a database. Database was throwing certain authentication error, even when the username and passwords were correct.

After trying a bunch of things here and there, our senior engineer starts the IntelliJ debugging session, and sees the debugger port as 8000. And Spring server is running at 8080. His reaction: "There, there looks like some weird config errors. How can it connect at port 8000 when server is running at 8080? Something is weird. Fix it!"

Don't be that engineer.

A debugger port is completely different port from a server port. A server port is the port where Spring serves its REST APIs. That's the port at which clients communicate with the spring server. A debugger port is the one which is used by IntelliJ to communicate with the running JVM process. It uses JPDA and it is NOT the same as the Spring server port.

To Be Continued...
