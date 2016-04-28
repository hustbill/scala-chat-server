# Scala chat server

write a scalable, fault-tolerant, network chat server and client (Scala)  

Akka uses the Actor Model together with Software Transactional Memory to raise the abstraction level and provide
a better platform to build correct concurrent and scalable applications.  
For fault-tolerance Akka adopts the “Let it crash”, also called “Embrace failure”, model which has been used with
great success in the telecom industry to build applications that self-heal, systems that never stop.  
Actors also provides the abstraction for transparent distribution and the basis for truly scalable and fault-tolerant
applications.  

Akka is Open Source and available under the Apache 2 License.  
In this project, we can utilize Akka to build a highly concurrent, scalable and fault-tolerant network server.

##**Let it crash: Implementing fault-tolerance**

Akka’s approach to fault-tolerance; the “let it crash” model, is implemented by linking Actors. It is very different
to what Java and most non-concurrency oriented languages/frameworks have adopted. It’s a way of dealing with
failure that is designed for concurrent and distributed systems.  
If we look at concurrency first. Now let’s assume we are using non-linked Actors. Throwing an exception in
concurrent code, will just simply blow up the thread that currently executes the Actor. There is no way to find
out that things went wrong (apart from see the stack trace in the log). There is nothing you can do about it. Here
linked Actors provide a clean way of both getting notification of the error so you know what happened, as well as
the Actor that crashed, so you can do something about it.  


##Supervisor hierarchies
A supervisor is a regular Actor that is responsible for starting, stopping and monitoring its child Actors. The basic
idea of a supervisor is that it should keep its child Actors alive by restarting them when necessary. This makes for
a completely different view on how to write fault-tolerant servers. Instead of trying all things possible to prevent
an error from happening, this approach embraces failure. It shifts the view to look at errors as something natural
and something that will happen and instead of trying to prevent it; embrace it. Just “let it crash” and reset the
service to a stable state through restart.

Akka has two different restart strategies; All-For-One and One-For-One.  
• OneForOne: Restart only the component that has crashed.  
• AllForOne: Restart all the components that the supervisor is managing, including the one that have crashed.  
The latter strategy should be used when you have a certain set of components that are coupled in some way that if
one is crashing they all need to be reset to a stable state before continuing.


##Chat server: Supervision, Traits and more
There are two ways you can define an Actor to be a supervisor; declaratively and dynamically. In this example we
use the dynamic approach. There are two things we have to do:
• Define the fault handler by setting the ‘faultHandler’ member field to the strategy we want.
• Define the exceptions we want to “trap”, e.g. which exceptions should be handled according to the fault
handling strategy we have defined. This in done by setting the ‘trapExit’ member field to a ‘List’ with all
exceptions we want to trap.
The last thing we have to do to supervise Actors (in our example the storage Actor) is to ‘link’ the Actor. Invoking
‘link(actor)’ will create a link between the Actor passed as argument into ‘link’ and ourselves. This means that we
will now get a notification if the linked Actor is crashing and if the cause of the crash, the exception, matches one
of the exceptions in our ‘trapExit’ list then the crashed Actor is restarted according the the fault handling strategy
defined in our ‘faultHandler’. We also have the ‘unlink(actor)’ function which disconnects the linked Actor from
the supervisor.

##Session management
The session management is defined in the ‘SessionManagement’ trait in which we implement the two abstract
methods in the ‘ChatServer’; ‘sessionManagement’ and ‘shutdownSessions’.  
The ‘SessionManagement’ trait holds a ‘HashMap’ with all the session Actors mapped by user name as well as a
reference to the storage (to be able to pass it in to each newly created ‘Session’).  

##Chat message management
Chat message management is implemented by the ‘ChatManagement’ trait. It has an abstract ‘HashMap’ session
member field with all the sessions. Since it is abstract it needs to be mixed in with someone that can provide this
reference. If this dependency is not resolved when composing the final component, you will get a compilation
error.  
It implements the ‘chatManagement’ function which responds to two different messages; ‘ChatMessage’ and
‘GetChatLog’. It simply gets the session for the user (the sender of the message) and routes the message to this
session. Here we also use the ‘forward’ function to make sure the original sender reference is passed along to
allow the end receiver to reply back directly  

##Chat storage: Backed with simple in-memory
To keep it simple we implement the persistent storage, with a in-memory Vector, i.e. it will not be persistent. We
start by creating a ‘ChatStorage’ trait allowing us to have multiple different storage backend. For example one
in-memory and one persistent.  

