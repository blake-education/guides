Code Conventions
================

A guide for using small composable classes in Rails.

Contents
--------

1. [Service Objects](#service-objects)
1. [Workflow Objects](#workflow-objects)
1. [Query Objects](#query-objects)

Service Objects
---------------

Objects that *do an action*, and only one action (as opposed to [workflow objects](#workflow-objects), which compose service objects).

Service classes:

* Live in `app/services/service/`
* Named like `Service::MaybeSomeContext::VerbNoun`, e.g. `Service::ConsumeResetToken` (if this was `User` specific, could also be `Service::User::ConsumeResetToken`)
* Have one public method, `#call`
* May be initialized with many objects, encapsulating the business logic to do something with them
* Using flags, modes, etc as parameters to switch upon probably means you want a separate class 

Example:

```ruby
module Service
  class ConsumeResetToken
    def initialize(user, token)
      @user = user
      @token_validity = valid_token?(user, token)
    end

    # [g] Check validity of a token and reset the token regardless.
    # - Return true if the token passed into service matched the users token
    def call
      reset_token!
      token_validity
    end

    private
    attr_reader :user, :token_validity

    def valid_token?(user, token)
      user && user.single_access_token == token
    end

    def reset_token!
      Adapter::AuthLogic.reset_token!(user)
    end
  end
end
```

Resources:
* [Code Climate Blog - 7 Patterns to Refactor Fat ActiveRecord Models - 2. Extract Service Objects](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/#service-objects)
* [brewhouse blog - Gourmet Service Objects](http://brewhouse.io/blog/2014/04/30/gourmet-service-objects.html)

Workflow Objects
----------------

Objects that compose multiple [service objects](#service-objects) (and possibly also using other objects too), to complete a workflow of actions related to a business process.

Workflow classes:

* Live in `app/workflows/workflow/`
* Named similarly to service objects, like `Workflow::MaybeSomeContext::VerbNoun`, e.g. `Workflow::SignupTeacher`, `Workflow::Teacher::DeleteClass`
* Have one public method, `#call`
* May call several service objects to fulfill a business process
* Errors should be handled such changes from earlier service calls should be rolled back when later service calls fail

Example:

```ruby
module Workflow
  module Teacher
    class CopyClass
      def initialize(original_class, new_product)
        @original_class, @new_product = original_class, new_product
        @teacher = @original_class.teacher
      end

      def call
        if has_capacity?
          new_class = Service::CreateSchoolClass.new(teacher, product).call
          original_class.students.each do |student|
            # Adds student to class, notifies them
            Workflow::SchoolClass::AddStudent.new(new_class, student).call
          end
          Service::SchoolClass::Created::NotifySubCo.new(new_class).call
          new_class
        end
      end

      private

      attr_reader :original_class, :teacher, :new_product

      def has_capacity?
        Policy::Subscription::Capacity::Validator.new(teacher, product).has_capacity?
      end
    end
  end
end

```

Query Objects
-------------

Objects that provide a query interface, usually including business rules.

Query classes:

* Live in `app/queries/query/`
* Named like `Query::MaybeSomeContext::Descriptor`, e.g. `Query::Subscription::TeacherTrials`
* Provide a method that returns a base scope, as well as possibly a number of chainable scope methods and/or methods that return values
* May be initialized with zero, one, or many objects or base scopes, e.g:
  * a school to query teachers for that school
  * a date to find subscriptions that expire on that date
  * an ActiveRecord::Relation to scope add mode scopes to
  * Some combination of these
* It may be prudent to ignore rules of DRY in some circumstances, e.g. reimplementing a scope that already exists on a model, so it is clear how it's specified *in the context of this query*, even if it may be the same as that on the model (for the time being)

Example:

```ruby
module Query
  module Subscription
    module TeacherTrials
      # Provides the base set of teacher trials, with the scopes providing further specificity
      def subscriptions
        # We use the school flag for teacher trials
        ::Subscription.school.is_trial.not_deprecated.extending(Scopes)
      end
      module_function :subscriptions

      module Scopes
        def product(product)
          where(subscription_type: product)
        end

        def default(product)
          product(product).merge(::Subscription.default).first
        end
      end
    end
  end
end
```

Note that it is necessary to use the provided scopes in the context of an existing `Query::Subscription::TeacherTrials.subscriptions` instance, e.g.:

```ruby
teacher_trials  = ::Query::Subscription::TeacherTrials.subscriptions
reading_default = teacher_trials.default(:reading)
wf_trials       = teacher_trials.product(:wf)
```

Resources:
* [Turn simple with Query Objects](http://helabs.com.br/blog/2014/01/18/turn-simple-with-query-objects/) - Excellent example of Query objects with scopes
* [Code Climate Blog - 7 Patterns to Refactor Fat ActiveRecord Models - 4. Extract Query Objects](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/#query-objects)

