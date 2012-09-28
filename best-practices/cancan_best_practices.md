CanCan Best Practices
=====================

<dl>
  <strong>can·can</strong>
  /ˈkanˌkan/
  <br/>
  Noun:
  <dd>
    <ol>
      <li>
        A lively, high-kicking stage dance originating in 19th-century Parisian
        music halls and performed by women in long skirts and petticoats.
       </li>
      <li>
        A simple authorization plugin that offers a lot of flexibility.
      </li>
    </ol>
  </dd>
 </dl>

Defining Abilities
------------------

> _Main article:
> [Defining Abilities](https://github.com/ryanb/cancan/wiki/defining-abilities)_

* Use the hash syntax when defining rule conditions.

  ```ruby
  can :read, Article, published: true
  ```

* Use nested hashes to define conditions on associations.

  ```ruby
  can :read, Article, author: { approved: true }
  ```

* Use scopes for complicated rules and when the logic has already been defined
  on the model.

  ```ruby
  can :update, Article, Article.written_by(user)
  ```

* Avoid using blocks to define conditions when possible. Prefer the hash syntax
  or use scopes for more complicated checks.

  ```ruby
  #bad
  can :update, Project do |project|
    user.owned_projects.include? project
  end

  #good
  can :update, Project, owner_id: user.id

  #also good
  can :update, Project, user.owned_projects
  ```

  The reason for using the hash syntax and scopes is that when checking abilities,
  CanCan will only run the block on instances, not for checks against the class.

  This means that while the first example above might work for `:edit` and
  `:update`, it would not work for `:index`. See the
  [Only for Object Attributes](https://github.com/ryanb/cancan/wiki/defining-abilities-with-blocks)
  section of the Defining Abilities With Blocks page on the CanCan wiki for more
  information.

  Block rule definitions also cannot be used when fetching records, which will
  be discussed later.

* Avoid using `:manage`.

  ```ruby
  #bad
  can :manage, Resource

  #good
  can :read, Resource
  can :create, Resource
  can :update, Resource
  can :destroy, Resource
  ```

  The `:manage` shortcut will actually return true to any `can?` call for
  `Resource`, regardless of what the action is.

  ```ruby
  can? :show, @resource
  can? Object, Resource
  can? false, Resource
  can? StandardError, Resource
  ```

  This also makes it very difficult to test your ability.

* Similarly, avoid using `:all`

  ```ruby
  # bad
  can :read, :all

  #good
  can :read, Article
  can :read, Comment
  ```

* For the sake of all that is holy and readable, don't use arrays for the action
  or subject in a rule definition.

  ```ruby
  #bad
  can [:create, :update], Comment

  #bad
  can :show, [Article, Comment]

  #superbad
  can [:do, :whatever], [To, These, Things]

  #good
  can :create, Comment
  can :update, Comment
  can :show, Article
  can :show, Comment
  ```

  This makes changing conditions, adding and removing actions or subjects, and
  refactoring much easier as each line contains a unit of information. Git diffs
  are also much easier to grok this way.



Integration with Rails Controllers
----------------------------------

> _Related articles:_
>
> * _[Authorizing controller actions](https://github.com/ryanb/cancan/wiki/Authorizing-controller-actions)_
> * _[Controller Authorization Example][controller-authorization]_
> * _[Fetching Records](https://github.com/ryanb/cancan/wiki/Fetching-Records)_


* Let `load_and_authorize_resource` handle loading and authorizing of resources.

  ```ruby
  class RobotOverlordsController < ActionController::Base
    load_and_authorize_resource
    ...
  end
  ```

  This automatically handles loading allowed `@robot_overlords` for the index
  page, and will `find`, `create`, or `new` `@robot_overlord` based on the
  conditions in the ability.

  See the [controller authorization][controller-authorization] example on the
  CanCan wiki for a more detailed example.

[controller-authorization]: https://github.com/ryanb/cancan/wiki/Controller-Authorization-Example

Testing Abilities
-----------------

> _Main article:
> [Testing Abilities](https://github.com/ryanb/cancan/wiki/Testing-Abilities)_

* Don't use aliases in tests.
* Use the `be_able_to` rspec matcher if you are using specs.

Multiple rules for the same action and subject
----------------------------------------------

* Abilities will be `OR`ed together.
* Be aware of [ability precedence](https://github.com/ryanb/cancan/wiki/Ability-Precedence).


Miscellaneous
-------------

* Expose the user with an `attr_accessor` and set it in `Ability#initialize`.
  Doing so will make life easier when you need to determine why your rules
  aren't working the way you intended.
* Avoid defining your own action aliases. Use only
  [built-in aliases](https://github.com/ryanb/cancan/wiki/Aliases).
* Consider handling `CanCan::AccessDenied` in an optional, specific way per
  controller or action, and fall back to a default way in
  `ApplicationController`.

  For example, check for `handle_access_denied` in `@controller`, and use the default handling if that method is not present.
