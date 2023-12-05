---
layout: post
title: Things I Learned Supporting Customers
type: blog
tags: 
- engineer
- knowledge
---

> I am a Software Engineer and the product I work with require Level 2 support and an Engineer is assigned for a month to solve those issues.

During my time, I figured some of the things that is essential for anybody to deal with issues and think of precautions while writing code. A product is written by many developers, and most of the time what happens is until a product gets into shape the code underhood is all messed up. Four different programmers will have four different ways of writing code, and for a startup, these things aren't taken seriously. This happens in most of the cases where product prototyping needs to be achieved without thinking of anything about future like optimization, heavy traffic, memory consumption. The feature goes well until it's about a customer. The customer is the one who will be using the application, not the one who develops it. And with time it is customers and your application has more support tickets than you can imagine. I am not saying this happens everywhere but happens somewhere. 

## Well Tested Code

> Write tests, have as much test coverages as you can. It is really really important. 

A precaution I call it, writing tests allows you to identify issues pretty quickly than a customer does. If I write a test, I am confident that for these cases the application is handled pretty well, things will not break. There are lots of ways to write test but the essential ones are I believe Unit tests and Automated tests. Testing every nooks and corner of the application/code, every unit we call it is Unit Test. Write the test for frontend, backend, services, jobs everything. Don't merge a PR if there aren't any tests. For example, for a simple function to sum: 

```ruby
def sum(a, b)
  a + b
end
```
Yeah, I wrote this function to sum two integers or anything you can sum, but what happens when b is ~nil~, or b is a ~string~. You might miss these always expecting a and b are integers. Now, that's where you need tests, for input is there an expected output? What happens if you don't have expected output? That is what you know when you write tests. And for automated tests, which is necessary I believe should always run in a CI environment, should always let the developer know the UI is broken and there's an error. Cypress, Selenium are some of those frameworks that lets you write automated tests running headless browsers. Write every end to end tests for every path a user follows, you will be able to minimize issues a lot.

## Debug Before Deploy

The feature you write is your responsibility. Anything you write might come back to you, a user can have problems with your feature, then you should be able to help the engineer who is asking you. It depends on how well the support engineer can debug what's going on. Your feature's code depends on that. Write code thinking about debugs, like how well someone will be able to read your code, debug it based on issue. This is essential because it will define how fast you can resolve the user's issues. What can you do to write such thing? I know a few points which might help you: 
-  Write environment variables which open logs, I have seen in many projects. There is an environment variable like DEBUG, when true will have better logs enabled.
-  Write enough functions, each doing only a thing or two to generate output for input. When I try to replicate a user's behaviour, I always end up in a 10-20 lines of method and will have to go through every single line to see what's going on? Instead with enough functions, I could at once see which method for this object is broken and could fix it in a bit.
-  Always think about failure while writing code, that's how you will be to minimize issues.

## Document your steps

If you resolved the issue tell others how you did it. Write up a good document for that. This will be helpful for other engineers. Write up a commit, write enough description in the commit. If you debug it manually in production, write down the steps in a document, link it with the project readme. Let other people know how you did it. I most of the time write a clear commit description with issue link, reported error link, cause of the issue, fix of the issue is the commit and changes you did. 

## Communication

There are people for support, there is/are someone for Quality Assurance, and there are Engineers. People on support will not have any idea what's going on with the system, ticket goes through QA team, while debugging they realize they have missed or a new issue and now you are the pilot. Drive the ticket to safety. First thing you should find out is what the user did. Software's like full story, mouseflow help a lot in these cases. You can get that information from support person or QA person. That's where you start the communication. Your first priority is to unblock the user and let the team know, they will send an email/message to the user, user will be happy and so will the others. Now, take your time resolve the issue and notify your support team that the issue has been fixed. They will let the users know. Have it verified, debug it, test it well before sending it to production.

## Support is Not All About Bugs

There are always customers who will ask of something your app won't have. Like a filter based on timezone, your application won't have it but it can be added in a day or two. It isn't a bug but it is an enhancement. Being a support engineer you need to have the ability to determine what is a bug and what is an enhancement. Enhancement should go into a refinements queue. Bug/Issues should go into a critical/priority queue. There might be time when all the customer issues has been wrapped up and you can jump into refinements. These refinements queue is also a data for the business person to seriously look into, like how much users are asking to have filter by timezone in your app. This fits for a small level business or product, large products will have their own support managers collecting data, reporters etc. But for a business of 9 developers and 3 QA + customer support personnel this is really helpful. 


----
Towards the conclusion, it all depends on the engineer who will write the code. Write clean code, maximize tests, think as a user not as a developer, write clean code. 
> Write up a PR checklist, things that are needed in a pull request without which features won't be approved. Like documentation, test coverage etc.
