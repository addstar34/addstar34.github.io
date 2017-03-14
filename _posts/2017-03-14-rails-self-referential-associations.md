---
layout: post
title: Rails Self Referential Associations
author: Adam D
---

I was building a Rails app which required a Contacts model to be able to relate to many other contacts where the relationship was reciprocal.

# The Database

Because contacts can have relationships with many other contacts I knew I needed another table to track this and knew I needed a `has_many :through` association.

Rails has another association called `has_and_belongs_to_many` which also could have worked but it's not as easy to work with as it doesn't have any intervening model and doesn't offer flexibility like adding extra columns.

Creating the migration.

{% highlight ruby %}
class CreateRelationships < ActiveRecord::Migration[5.0]
  def change
    create_table :relationships do |t|
      t.integer :contact_id
      t.integer :relation_id

      t.timestamps
    end
  end
end
{% endhighlight %}

# The associations

Next I needed to setup the associations.

{% highlight ruby %}
# relationship.rb
class Relationship < ApplicationRecord
  belongs_to :contact
  belongs_to :relation, class_name: 'Contact'
end

# contact.rb
class Contact < ApplicationRecord
  has_many :relationships
  has_many :relations, through: :relationships
end
{% endhighlight %}

Great I could now create the relationships but the problem was the relationship would only be viewable from the contact who created the relationship and not the other way around.

So on the contact model I created an association for the inverse of the relationship.

{% highlight ruby %}
# contact.rb
class Contact < ApplicationRecord
  has_many :relationships
  has_many :relations, through: :relationships
  has_many :inverse_relationships, class_name: 'Relationship', foreign_key: 'relation_id'
  has_many :inverse_relations, through: :inverse_relationships, source: :contact
end
{% endhighlight %}

But then to get both sides of a relationship I had to go through 2 associations or do lookups on both columns:

{% highlight ruby %}
# using the associations
@contact.relationships
@contact.inverse_relationships

# or query the model for both sides
Relationship.where("contact_id = :contact OR relation_id = :contact", contact: @contact)
{% endhighlight %}

While this was worked it just didn’t seem efficient or elegant.

# A better way

So I decided to refactor this inverse association and instead when a relationship is created I would use callback to create (and destroy) the inverse relationship in the database.

{% highlight ruby %}
# relationship.rb
class Relationship < ApplicationRecord
  belongs_to :contact
  belongs_to :relation, class_name: 'Contact'

  after_create :create_inverse, unless: :inverse_exists?
  after_destroy :destroy_inverse, if: :inverse_exists?

  def create_inverse
    self.class.create(inverse_relationship)
  end

  def destroy_inverses
    inverses.destroy_all
  end

  def inverse_exists?
    self.class.exists?(inverse_relationship)
  end

  def inverses
    self.class.where(inverse_relationship)
  end

  def inverse_relationship
    { relation_id: contact_id, contact_id: relation_id }
  end
end
{% endhighlight %}

While this creates more database records in the relationships table the lookup queries are so much better now.
