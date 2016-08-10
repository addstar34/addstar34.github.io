---
layout: post
title: Rails rename a model (table)
author: Adam D
---

DRAFT------------------

create a migration
- remove indexes
- rename table
- add indexes back
- rename relational tables has_many through or habtm
- rename foreign keys for relational tables

POLYMORPHIC TABLES NEED RECORDS TO BE RENAMED

run the migration
- check schema

rename files and contents

assets
- rename javascript and scss files

controllers
- rename the file
- inside the file rename the class
- inside the file rename variables
- inside the file rename references to the model
- inside the file rename params method and the params.require

- check other controllers for references

helpers
- rename the file
- inside the file rename the class

models
- rename the file for model and the join models
- rename the class
- rename relationships has_many, has_many through

- inside models that didn't change check for references i.e. categories

policies for pundit
- rename file
- inside file rename the class

views
- rename folders
- inside files rename

tests, specs, fixtures, factories
- rename files
- inside files rename

update routes

run your tests
