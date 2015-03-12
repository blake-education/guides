## Rspec style guide

- First #describe what you are doing
- Then establish the #context
- #it only expects one thing (within reason)
- Prefer explicitness - #it #its #specify #subject are nice shortcuts but
consider whether they are sacrificing readability
- Having excessive setup (fixtures, database entities, doubles etc.) can cause false-positives. Thus, set up preconditions as close to the test as possible (usually within the context block) and tear them down afterwards. This is really important when working with time for example.
- Unit tests are fast and effective to work with when doing TDD. These are isolated tests, typically
for a single class, and should use stubs and spies via test doubles to replace anything that is
external to the _unit_ being tested. Testing side-effects that occur elsewhere in the system
is incorrect. Not being able to unit test something is typically a signal that something may be wrong
with the design of the unit
- Beware of testing implementation specifics. When this becomes difficult, this typically identifies strong coupling - sometimes there is no way around this (e.g. you are calling an external API)
- Avoid testing things that have already been tested elsewhere. Thin controller actions typically fall
into this category, as do things like associations (e.g. something belongs_to some other thing)
- Integration tests seek to test how various units integrate and thus typically involve assertions
regarding side-effects. These tests can involve subsystems or the entire system - in the latter case, they are called feature tests. These types of tests are generally slow and should be reserved for the
purpose of smoke testing key features of the site

For specifics on what to do and what not to do along with excellent examples, see:

- [betterspecs.org](http://betterspecs.org/)
- [pcreux's tips and tricks](http://eggsonbread.com/2010/03/28/my-rspec-best-practices-and-tips/)
- [carbon five's rspec best practices](http://blog.carbonfive.com/2010/10/21/rspec-best-practices/)

In depth articles on mocks, stubs, doubles and when to use what:

- [Myron Marston on mocking](http://myronmars.to/n/dev-blog/2012/06/thoughts-on-mocking)
- [Martin Fowler on mocks and stubs](http://martinfowler.com/articles/mocksArentStubs.html)
