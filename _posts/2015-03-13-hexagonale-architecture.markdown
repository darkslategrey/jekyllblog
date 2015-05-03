---
layout: post
title:  "Hexagonale architecture"
date:   2015-03-21 12:01:04
categories: softwarese 
published: false
---
C'est un sujet que j'aborde récemment. 

Une demande d'évoultion sur un projet que j'ai écrit recemment m'a ammené à réfléchir à
une architecture plus modulaire. 
C'est naturelleemnt que j'en suis venu à m'interesser aux messages.
Ainsi le protocole pubsub m'a paru être adéquat.
Après avoir programmé pendant quelques années sur le principe de de
demandé à faire quelque chose à mes objets, procédures, fonctions. 
Le pasage à une conception basée sur les messages n'est pas facile.
Mais l'effort en vaux la peine. On se propulse d'un coup dans une
nouvelle façon de designer ses apps. 

Je m'avance peut être car je n'ai pas vraimeent creusé ce qui se cache
derrière _les micros services_. Mais de ce j'en sais ils s'inscrivent
dans ce mouvement (perpetuel ?) d'architecture distribuée
Certains_


Un nom + un moment + un contenu + un emeteur + un/des recepteurs + un média

J'ai croisé les micros services, la programmation par envoi de
messages.



# Hexagonal 


## Adapters (Port)


class RunIdService
   attr_reader :run_id

   def next!
      @run_id = redis_consume_run_id!

   end

   private
   def redis_consume_run_id!

      redis = RedisRepo.db_client
      loop do

        # ... fiddly Redis-specific transaction code ...
         redis.unwatch and return last if success
      end
   end
end

## Tests

 Few lets for Values
 Often stubs
 Asserts on mocks for outgoing queries/commands
 Asserts on results probably not worth much

Adapters tend to be thin wrappers around external services.

  We'll often use stubs to fake out the results of calling an
  external service, and we'll use mocks for the specific

  purpose of making sure that the call to that external service
  was well-formed.


And there's not a lot of benefit to checking results, because

  your test would usually look like
 1. Set up a mock returning some value

 2. Call the object and make sure the result was the value we

  just mocked


So that's Adapters. And like Values can't call Entities,
  Values and Entities don't have side effects and can't
  call Adapters or else they'd have side effects to.


## Shells


class MonthCountWorker
   def perform serialized_sym
      store(monthcount(of_month(identified_by(serialized_sym)))
   end
   def identified_by serialized_sym
      SymRepo.deserialize serialized_sym
   end
   def of_month sym
      ThreadRepo.month(sym)
   end
   def monthcount month
      MonthCount.from month
   end
   def store mc
      MonthCountRepo.new(mc).store
   end

end



get '/:slug/?' do
   @slug = Slug.new params[:slug]
   @list = ListRepo.find(@slug)

   @title = "#{@list.title_name} archive"
   @years = MonthCountRepo.years_of_month_counts @slug

   haml :'list/show.html'
end
 

Web front


These little programs are called Shells. They coordinate the

  specialiazed objects to do meaningful work. It's their job to
  manage the dependencies and the classes they use make

  all the decisions. We keep them little because they don't
  have restrictions on their mutability and side effects, so

  they're harder to reason about. A Shell may not be a single
  method or a single object like the MonthCountWorker, it

  may have several, like the web frontend.

### Testing


 Use fixtures to capture real-world input
 Might need factories to create enough Entities
 Expect on results, and state, and mocks
 Have to be integrated ­ the scary mock pitfalls
 If you have one integrated test, can stub Adapters later



## Entities

class Message
   def initialize call_number, email slug
      @call_number = CallNumber.new(call_number)
      @email = email
      @slug = Slug.new(slug)
   end
end



One of the interesting things is how little code there tends to

  be in Entities. Their job is to have an identity and wrap up
  Values, not run a lot of code.


Extracting Entities from ActiveRecord

   Find the identity (probably an int primary key)
   Extract Values
   Drive out side effects to Adapters and Shells

### Testing

 Few lets for Values
 Maybe factory methods
 Maybe stub entities, but not Values
 Asserts on results
 Asserts on object state

The real difference from testing Values is that rather than only assert
   a method call returned the right thing, we tend to assert that after
   calling a method the Entity is now in the right state, it has an
   attribute we expect.


## Objects values

New class for methods that reuse instance variables.

Use gem (include) Adamantium (immutable) to raise exception when object's
mutations occur.

Use (extend) Forwardable module to delegate some methods

Use gem (include) Equalizer(:attr) to include equlities (?)

Use another gem to add ruby methods like hash, to_s, inspect, ==, <=>
if the value can be used inplace of a String / Integer

Exemple the Subject in Message Object from Chibrary

class Message

def initialize raw
   @raw, @header = raw, Headers.new raw
end

def subject
   Subject.new self[:subject]
dnd

def subject= subj
   self[:subject] = subj.to_s
end

end


* Immutable
* Variable (mutable) <=> Value (immutable)
* Same answer each call
* Immutable code cannot cal mutable code because of above

* No side effect : if we call a method, no other object change

### How to test objects values

* No lets
* No stubs
* No factories
* Integrate down
* Asserts on results
* Property-based testing, like QuickCheck


When we test Values we can do without a lot of things
  because they're very small and predicatble. If a Value is
  built up out of other Values, it's trivial for our tests to
  integrate down through them.

We don't use mocks or stubs because we don't need to
  ensure that side effects take place. We only write
  assertions on the result of calling methods.

*** Additionally, we can do really cool things like automatically
  generate property-based tests, though I don't have time to
  get into it. ***



# Conf Peter Harkins

## Tools

### Values

* adamantium
* equalizer
* lift

### Entities

* virtus
* concord
* vanguard

### Other

* generative, theft
* mutant


## Explore

* Chibrary
* TwoFactorAuth
* microrb.com
* cuba, lotus
* trailblazer
* datamapper
* ruby object mapper
* haskell


# To read about

Liskov Substitution Principle.
The Law of Demeter

# References

* Eric Evans: Domain Driven Design
* Gary Bernhardt: Boundaries
* Sandi Metz: Magic Tricks of Testing
* J.B. Rainsberger: Integrated Tests are a Scam
