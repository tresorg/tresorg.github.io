---
title: where to put your code ... and what code to put where
author: Alexander Bogdanov
date: 2025-02-21
category: MVC, recipies, architecture, cakephp
layout: post
# cover: https://sighingnow.github.io/jekyll-gitbook/assets/dinosaur.gif
---

Code is different depending on what we're talking about, if we're parsing a string this is low-level, but if we're calling a model for a result-set this is high-level, where does business logic go? I get asked this a lot so here is what we can apply to any MVC framework.

The general idea is: Models = fat, Controllers = thin, Views = fat. Fat means lots of details and business logic. Business logic are decisions you make over how your app behaves. Thin does not mean few lines of code, thin means high level code with few decisions.

For my web projects I use CakePHP, a "convention over configuration" approach to build sites fast. 

```
config/
     bootstrap.php
     = global functions, global functions are fine when they need to be global

vendor/
     = besides all vendor code required, there are things we write ourselves that we
     share between repositories/services/containers in a stack.

src/Controller/ and src/Shell/ (shells are essentially controllers, just without the request)
     = This layer should be THIN, manage the request (or arguments), how its processed and return output.
     Avoid: explicit business logic, explicit db schema.
     Why: Controllers are managers, they set rules, delegate and respond.
     How: move queries to tables, pass request/db data to business layer to get an answer

src/Controller/Component
     = Repetitive controller logic should be refactored into components
     Avoid: same as controllers because this only shared controller code.
     Keep in mind: controllers have lifecycle callbacks, use these to your advantage.

src/Middleware
     = Reusable layers of Request handling or response building logic.
     This goes like onion layers on top of controllers and can handle or modify the request/response.
     Uses: tracking cookies, output formats, stuff you add to AppController could be moved into Middleware.

src/Model/Entity
     = Smart database row but not less and not more.
     Avoid: file/db/network operations in properties, accessing related data a->b->c
     Why: besides as the app grows, entities get used in many ways you don't expect, serialization etc

src/Model/Table
     = Models should be FAT. Collection of database rows + app functionality related to these.
     All queries (finders/savers) should go to this layer. Business logic is welcome here.
     Avoid:
          formatting data: views
          slow (file/network) operations unless explicit: behaviors
          looking at request data: the controller should tell you what needs to be done
     How: prefer to let the controller pass in the results of operations into the model to save.
          If a user is something, rather let the controller call a different method on the table, easier to test+debug.

src/Model/Behavior
     = reusable logic + superpowers for models
     Uses: repetitive model code, extensions like file uploads, content generation
     Keep in mind: Behaviors extend models and callbacks, use callback events to your advantage
     
src/YourChosenNamespace
     = business layer, API, namespaced functions
     this layer should be FAT.
     Avoid: coupling to db schema, loading tables
     Why: coupling of schema makes the app harder to develop/test/debug
     How: let controllers (or shells) manage this layer


src/Template (and src/View = reusable template code)
     = this layer should be FAT, you may have a separate front-end - the idea is still the same,
     Business logic is welcome here.

```

The MVC pattern suggests to pass the Model directly to the View and let the View interact with it. You don't have to do this, the Controller can define what is needed from the model at a specific place and give the result to the View.



