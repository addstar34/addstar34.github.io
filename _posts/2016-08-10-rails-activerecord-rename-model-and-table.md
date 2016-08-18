---
layout: post
title: Rails ActiveRecord rename model and table
author: Adam D
---

Renaming a model and table in Rails is easy using a migration and rename_table, but then there's a stack of references to this old name that will also need to be updated. I'm going to cover what those are.

## First create a migration

**But!** In this migration isn't just the `rename_table` statement. You also need to:

- remove and re-add indices
- rename relational tables used in `has_many though` or `habtm` relationships
- rename the foreign key columns in these relational tables

Here's an example of a migration I created:

{% highlight ruby %}
class RenameShopsToBusinesses < ActiveRecord::Migration[5.0]
  def change
    remove_index :shops, :slug, unique: true
    rename_table :shops, :businesses
    add_index :businesses, :slug, unique: true

    rename_table :shop_categories, :business_categories
    rename_column :business_categories, :shop_id, :business_id
  end
end
{% endhighlight %}

Once you've created the migration run it and check your scheme file.

Before you're finished touching the database in this task if you have any polymorphic relationships you will need to rename the records for the `resource_type` attribute.

## Now the fun of renaming files and the code within those files begins

This was the first time I've needed to do this so I didn't want to automate the process and opted to rename everything manually, plus the app was relatively small.

If you're going to do this manually start with your tests or specs and factories or fixtures, then your routes and then start at the top of your app and work your way down. Inside each file after manually renaming I did a search for the table name I changed to make sure I got everything.

## Tests, specs, fixtures, factories

- rename files  
- inside files rename  

## Rename routes

## Assets

- rename javascript and scss files

## Controllers

- rename controller file  
- inside the file rename the class  
- inside the file rename variables  
- inside the file rename references to the old model name  
- inside the file rename params method and the params.require  

## Helpers

- rename the file  
- inside the file rename the class

## Models

- rename the file for the model and the join models  
- inside the file rename the class  
- rename relationships `has_many`, `has_many through` and `has_and_belongs_to_many`

## Policies for pundit

- rename file  
- inside file rename the class

## Views

- rename folders  
- inside files rename

Now run your tests or specs and see how you did.
