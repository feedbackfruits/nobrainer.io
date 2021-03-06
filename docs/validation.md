---
layout: docs
title: Validations
prev_section: callbacks
next_section: dirty_tracking
permalink: /validations/
---

## Declaring Validators

Validations works the same way as in ActiveRecord because NoBrainer reuses the
ActiveModel validation logic. If you are not familiar with how validations
typically operate, please read the following documentation:
[ActiveRecord validations](http://edgeguides.rubyonrails.org/active_record_validations.html).  
However, there are a some differences with the
associated validator and the uniqueness validator, which are explained below.

There are six ways to declare validations with NoBrainer:

* `validate_presence_of :field_name`
* `validates :field_name1, :field_name2, :presence => true`
* `validate { errors.add(:base, "too many friends") if too_many_friends? }`
* `field :field_name, :validates => { :presence => true }`
* Using shorthands as described below.
* Using types: `field :field_name, :type => Integer`. This will validate that the
  given field is an integer. Read more about the type checking mechanism in the
  [Types](/docs/types) section.

## Shorthands

### required

You may use the `required` shorthand to specify a presence validator, except
with `Boolean` types for which a `not_null` validator is used instead.

{% highlight ruby %}
class Model
  field :name, :required => true
  field :admin, :type => Boolean, :required => true
end
# Equivalent to:
class Model
  field :name, :validates => { :presence => true }
  field :admin, :type => Boolean, :validates => { :not_null => true }
end
{% endhighlight %}

### uniq/unique

You may use the `uniq` (or `unique`) shorthand to specify a uniqueness validator:

{% highlight ruby %}
class Model
  field :name, :uniq => true
end
# Equivalent to:
class Model
  field :name, :validates => { :uniqueness => true }
end
{% endhighlight %}

### in

You may use the `in` shorthand to specify a inclusion validator:

{% highlight ruby %}
class Model
  field :state, :in => %w(start finish)
end
# Equivalent to:
class Model
  field :state, :validates => { :inclusion => { :in => %w(start finish) } }
end
{% endhighlight %}

### format

You may use the `format` shorthand to specify a format validator:

{% highlight ruby %}
class Model
  field :name, :format => { /\A[a-z]+\z/ }
end
# Equivalent to:
class Model
  field :name, :validates => { :format => { :with => /\A[a-z]+\z/ } }
end
{% endhighlight %}

### length/min_lenght/max_length

You may use the `format` shorthand to specify a format validator:

{% highlight ruby %}
class Model
  field :field1, :length => (3..5)
  field :field2, :min_length => 4
  field :field3, :max_length => 10
end
# Equivalent to:
class Model
  field :field1, :validates => { :lenght => (3..5) }
  field :field2, :validates => { :lenght => { :minimum => 4 } }
  field :field3, :validates => { :lenght => { :maximum => 10 } }
end
{% endhighlight %}

## When are validations performed?

### Validations are performed on:

Validations are performed when calling the following methods on an instance:
* `create`
* `create!`
* `save`
* `save!`
* `update`
* `update!`

If you want to bypass validations, you may pass the `:validate => false` option
to these methods, which can be quite handy in a development console. Do not use
such thing in your actual code.

The `!` version of these methods raise a `NoBrainer::Error::DocumentInvalid`
exception when validation fails. If left uncaught in a Rails controller, a 422
status code will be returned.
The vanilla versions populate the errors array attached to the instance.
`save()` and `update()` return true or false depending if the validations
failed. When using `create()`, you may call `persisted?` to check if the
model was valid and persisted.

### Validations are *not* performed on:

* Validations are not performed when updating all documents matching a criteria,
  such as `Model.update_all()`.
* Attribute validations are not run when their corresponding attribute have
  not changed (through [dirty tracking](/docs/dirty_tracking)).

## Presence Validations on belongs\_to Associations

Foreign keys in belongs\_to associations are always validated when the foreign
key is present. If you wish to disable this behavior, you may pass `validates =>
false` on the association declaration.

Additionally, you may add a presence validator as such:

{% highlight ruby %}
class Comment
  belongs_to :post, :required => true
end
{% endhighlight %}

## The Uniqueness Validator

The uniqueness validator ensures that a field value can be present at most once
table wide.

For performance reasons, NoBrainer only performs uniqueness validations when the involved
fields change.

The uniqueness validator uses distributed locks to ensure race-free semantics.
Read more about this in the [Distributed Lock](/docs/types) section.

### Using scopes

The uniqueness validator accept a `:scope` option which can be a field, or an
array of fields. For example:

{% highlight ruby %}
class TeamMember
  field :team_name
  # Do not allow duplicate emails within a team.
  field :email, :validates => {:uniqueness => {:scope => :team_name}}
end
{% endhighlight %}

It is highly recommended that you add a presence validator on the scoped fields,
because RethinkDB considers `nil` and `undefined` to be two different things.

## Differences with ActiveModel

### Associated Validator

The associated validator is not implemented.

### Uniqueness Validator

The uniqueness validator does not accept the `:case_sensitive` option.
Downcasing the attribute in a `before_save/validation` callback is a better idea.

### NotNull Validator

NoBrainer supports an additional validator `not_null`. It rejects undefined and
nil values.
