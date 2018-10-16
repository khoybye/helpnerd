# Observer and Decorator Patterns

The goal of this assignment is to give you practice with the *Observer*
and *Decorator* patterns. To complete this assignment, you will need
to fork this project skeleton, follow the instructions below to
complete the implementation, then commit/push your changes back to
**your** GitHub fork (no pull requests, please!)

**Note**: The assignment must be completed in the specific manner described in the instructions. 
Any deviations from the instructions will possibly cause the automated grading to fail and will 
be considered incorrect. Following the instructions includes, but is not limited to:

* Do not change the test code in any way. If the test code does not compile, change your implementation of the classes being tested, not the tests! (you may add additional tests)
* Do not modify interfaces and abstract classes provided to you in any way unless specifically instructed to do so.
* Do not modify class constructors, method signatures, or local field names unless specifically instructed to do so, or if it is required to conform to the tests provided.
* Do not add fields to any classes unless specifically instructed to do so.

## Note About Academic Honesty

You should keep your code and repositories private and should not be sharing code under any circumstances. 
**When forking your projects, you should make sure the repository is set to private and you should immediately 
remove the class's team CSC430 from your project and add me as a collaborator so that I will be the only 
person who can access your code other than you! If you do not do this and check in large amounts of code, 
I will consider this a violation of the academic honesty policies of MSU.**

## Instructions

Please read through the following instructions at least once before 
actually starting your work on the project. Read through each step and
the corresponding interfaces and classes in the project skeleton so
that you will have at least a vague understanding of what you will need
to do.

### High Level Goal

In this project we will be continuing with the blob game that we are 
developing by adding a **Game** class which will store the blobs
that are a part of the game and will also manage reporting game state
to any observers that might be interested. Additionally, we will be 
creating a decorator **LimitedVisionObserver** that will limit the 
visibility of our observers to a certain distance so that the individual
observers won't have a complete view of the game state when desired.

Please note that we will be using code that is *similar* to what you
have already seen, but some interfaces, etc. have changed and some code
has been moved into the API dependency.

### Code Changes

Below are some (but, perhaps not all) changes that have been made to
the API that you should be aware of.

#### MoveStrategy

The *MoveStrategy* interface has been modified so that its move method
now takes two arguments:

* A *BlobView* of the *current blob* which needs to be moved
* A *List<BlobView>* which contains views of *other* *Blob*s in the game.

This allows our strategies to take the position of other *Blobs* into
consideration when selecting a move.

#### Blob

The *Blob* interface no longer has a *move()* method. Instead, we have
added a *move(final List<BlobView> views)* method which accepts a list
of views for other *Blob*s in the game. 

#### BaseBlob

Some of the methods that were implemented in the *SimpleBlob* class have
been moved to the *BaseBlob* class.

### Testing

You are provided with a very inadequate test suite and will need to write
additional tests in order to make sure that your code works properly.
To help with this task, I have added testing notes below to get you thinking
in the right direction. The recommended tests do not guarantee sufficient
coverage. They are merely meant as a starting point for your testing.

### Add API Dependency

In order to use the current API code, you will need to add a
dependency for the API library (as we did in the previous project), but
be sure to set your dependency to version *2.0*.

## Implementation

### LimitedVisionObserver

First, we will use the *decorator pattern* to create a decorator
which will decorate a given *GameStateObserver* so that when it is passed
a list of *BlobView*s the *GameStateObserver* will actually only see the
*BlobView*s within a certain distance. 

To do this, we need to introduce a new interface *HasLocation*. The purpose
of this interface is to indicate which *GameStateObserver*s have a location
that can be used to limit visibility. This interface is provided to
you.

#### Implementation Details

* Add a constructor to initialize the given fields
* Implement the *GameStateObserver* interface
* *updateLocations* should be filtering its input list according to the vision
limit before delegating to the decorated observer. In other words, you
should remove all views outside of the specified range before passing the
data on to the wrapped observer. 
* The max vision value is *inclusive*, so if a *BlobView* is exactly that
distance away, it should still be visible.
* We can not compute a distance if the wrapped observer does not have
a location, so we should check and see if the wrapped observer implements
the *HasLocation* interface. If it does **not**, then *updateLocations*
should be called with an empty list.
* When we are working with lists of *GameStateObservers*, we will need to
be able to check for equivalence. Accordingly, you must override the
*equals* and *hashCode* methods in the *LimitedVisionObserver* class. Note
that for this project, we do not need to consider the *maxDistance* value
in these computations.

#### Testing

* What happens when the input is empty?
* What happens if *all* elements are too far away?
* What happens if the input is null?
* What happens if the constructor receives null values?

### SimpleBlob

*SimpleBlob* should extend the functionality of the *BaseBlob* and
should also act as a *GameStateObserver*. By implementing this 
interface, your *SimpleBlob* will be able to subscribe to a 
*GameStateObservable* that will periodically provide your 
*SimpleBlob* with a list of *BlobView*s from the game.

When provided with these *BlobView*s, your *SimpleBlob*
should use them to decide where to move to. We will, again,
leverage the *Strategy* pattern for this purpose.

#### Implementation Details 

* You should add *two* constructors. One constructor should accept
a *Point* and a *MoveStrategy*. The other constructor should accept
a *Point*, *MoveStrategy* and *GameStateObservable*. 
* If a *GameStateObservable* is provided, we should initialize our 
*SimpleBlob* by calling a super constructor, *then* call *subscribe*
on the observable and subscribe our *SimpleBlob* to future updates.
* You should not add *any* instance variables.
* Your *updateLocations* method will need to use your *move* method.

#### Testing
* What happens if the constructor gets null values?
* What happens if *updateLocations* gets a null value?
* What happens if the move takes you in different directions?

### Implement the Game Class

The *Game* class will store a list of *GameStateObserver*s,
a list of *Blob*s in the game and a double representing the maximum 
"visibility" that a *GameStateObserver* should have.

The *Game* class should also implement the *GameStateObservable*
interface which will allow *GameStateObserver*s to subscribe to
and unsubscribe from game state updates.

Please note that *subscribing* does not mean the subscriber is
part of the game (the subscriber may not even *be* a *Blob*)! In
order for a *Blob* to be a part of the game, it must be passed
into the *join* method.

When *executeMove* is called, it should get a read only copy of
all *Blob*s (i.e., a list of *BlobView*s) in the game and send 
this list to all of the currently subscribed *GameStateObserver*s.

Remember, though, that we don't want to expose *all* of the
*BlobView*s to every subscriber. We actually only want them to see
*BlobView*s which are within a certain distance of them.

To make sure that this happens, whenever a *GameStateObserver* 
subscribes to the *Game*, we will decorate it with a *LimitedVisionObserver*
with the appropriate maximum vision. By doing this, our *executeMove*
method can simply pass all *BlobView*s and not worry about the distance,
because the decorator is handling that for us!

#### Implementation Details

* Your constructor should only accept a double for the max vision, be
sure other variables get initialized, though!
* *join* should not allow a *Blob* to join more than once! All elements
in the blobs list should be unique.
* *executeMove* should *not* perform any vision distance filtering! It
should pass all *BlobView*s to every subscriber.
* *subscribe* should not allow a *GameStateObserver* to subscribe
multiple times. All subscribers should be unique.
* All subscribers should be decorated with a *LimitedVisionObserver* before
being added to the subscription list.
* You may notice that no *SimpleBlob*s are seeing updates. This may
be because the decorator only provides data when it wraps an observer
which implements the *HasLocation* interface, so add that to the
*SimpleBlob* if you haven't already.

#### Testing

* What happens if your constructor and methods receive null inputs?
* What happens if the *subscribe*/*unsubscribe*/*join* methods are
called multiple times.
* What happens if you try to *executeMove* with no subscribers and/or
blobs?

### Testing, In General

* In general, you need to think about different scenarios that
could happen and try to test for them, then harden your code if necessary.
* You should think about what problems might be caused by null values.
* You may not have concrete classes for all of your types, so you will need
to use anonymous classes or mocks to test with (e.g., there are no 
*MoveStrategy*s in the project to pass to your *SimpleBlob*, so you need
to create them on the fly in your test or mock them).
* Mocking is new to you and you are not *expected* to use mocks. However,
they are very handy so if you are interested you should check out the
Mockito library.

