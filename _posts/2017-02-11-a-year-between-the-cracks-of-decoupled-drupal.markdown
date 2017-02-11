---
layout: post
title: "Between the cracks of decoupled (Drupal) architecture"
date: 2017-02-11 11:18:44 +0100
comments: true
categories: 
- drupal
- drupalplanet
- headless
- decoupled
- reactjs
- graphql
- migrate
---
In any decoupled architecture, people tend to focus on the pieces that will fit together. But what nobody ever tells you is: *watch out for the cracks!* 

The cracks are the integration points between the different components. It's not GraphQL as a communication layer; it's that no one thinks to log GraphQL inconsistencies when they occur. It's not "what's my development environment", it's "how do these three development environments work on my localhost at the same time?". It's the thousand little complexities that you don't think about, basically because they aren't directly associated with a noun. We've discovered "crack" problems like this in technical architecture and devops, communication, and even project management. They add up to a lot of unplanned time, and they have presented some serious project risks.

A bit more about my recent project with [Amazee Labs](https://amazeelabs.com). It's quite a cool stack: several data sources feed into [Drupal 8](https://drupal.org), which offers an editorial experience and [GraphQL](https://graphql.org) endpoints. Four [React](https://facebook.github.io/react/)/[Relay](https://facebook.github.io/relay/) sites sit in front, consuming the data and even offering an authenticated user experience ([Auth0](https://auth0.com)). I've been working with brilliant people: [Sebastian Siemssen](https://www.drupal.org/u/fubhy), [Moshe Weitzman](https://www.drupal.org/u/moshe-weitzman), [Philipp Melab](https://github.com/pmelab), and others. It has taken all of us to deal with the crack complexity.

The first crack appeared as we were setting up environments for our development teams. How do you segment repositories? They get deployed to different servers, and run in very different environments. But they are critically connected to each other. We decided to have a separate "back end" repo, and separate repos for each "front end" site. Since Relay needs to compile the entire data schema on startup, this means that every time the back end is redeployed with a data model change, we have to automatically redeploy the front end(s). For local development, we ended up building a mock data backend in MongoDB running in Docker. Add one more technology to support to your list, with normal attendant support and maintenance issues.

DevOps in general is more complicated and expensive in a decoupled environment. It's all easy at first, but at some point you have to start connecting the front- and back-ends on peoples' local development environments. Cue obvious problems like port conflicts, but also less obvious ones. The React developers don't know anything about drupal, drush, or php development environments. This means your enviroment setup needs to be VERY streamlined, even idiot-proof. Your devops team has to support a much wider variety of users than normal. Two of our front-enders had setups that made spinning up the back-end take more than 30 minutes. 30 minutes! We didn't even know that was possible with our stack.  The project coordinater has to budget significant time for this kind of support and maintenance.

Some of the cracks just mean you have to code *very* carefully. At one point we discovered that certain kinds of invalid schema are perfectly tolerable to the GraphQL module. We could query everything just fine - but React couldn't compile the schema, and gave cryptic errors that were hard to track down. Or what about the issues where there *are* no error messages to work with? CORS problems were notoriously easy to miss, until everything broke without clear errors. Some of these are impossible to avoid. The best you can do is be thorough about your test coverage, add integration tests which consider all environments, and *document all the things*.

Not all the cracks are technological; some are purely communication. In order to use a shared data service, we need a shared data model and API. So how do you communicate and coordinate that between 5 teams and 5 applications? We found this bottleneck extremely difficult. At first, it simply took a long time to get API components built. We had to coordinate so many stakeholders, that the back-end data arch and GraphQL endpoints got way behind the front-end sites. At another point, one backender organically became the go-to for everything GraphQL. He was a bottleneck within weeks, and was stuck with all the information silo'ed in his head. This is still an active problem area for us. We're working on thorough and well-maintained documentation as a reference point, but this costs time as well.

Even project managers and scrum masters found new complexities. We had more than 30 people working on this project, and everyone had to be well coordinated and informed. You certainly can't do scrum with 30 people together - the sprint review would take days! But split it out into many smaller teams and your information and coordination problems just got much harder. Eventually we found our solution: we have 3 teams, each with their own PO, frontender(s) and backender(s), who take responsibility for whole features at a time. Each team does its own, quite vanilla, scrum process. Layered on top of this, developers are in groups which cut across the scrum teams, which have coordination meetings and maintain documentation and code standards. All the back-enders meet weekly and work with the same standards, but the tightest coordination is internal to a feature. So far this is working well, but ask me again in a few months. :) 

Working in a fully decoupled architecture and team structure has been amazing. It really is possible, and it really does provide a lot more flexibility. But it demands a harder focus on standards, communication, coordination, and architecture. Sometimes it's not about the bricks; it's about the mortar between them. So the next time you start work on a decoupled architecture, *watch out for the cracks!*
