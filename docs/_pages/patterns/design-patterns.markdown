# Design patterns
In four categories:
- Creational: help create instances of objects
- Structural: set up communication pathways between objects
- Behavioral: partition behavior of system in classes

Each pattern pursues the same goal: dependency management

Patterns fulfil solid principles

Command pattern is most simple

Interface named command with simple method called execute

Example: controlling a printer requires muiltiple commands
- motor on
- motor off
- clutch engage
- relay close

Decoupling what is done from who does it

Benefit: decoupling actors from actions gives flexibility

We can use this to implement do & undo

Command is object and can hold state
Object that remembers relevant details can undo those

Undo

The actor model

Xerox example: a lot of threads
Threads need stacks

Why not allocate just a lot of space and cross fingers?

We can't afford with these numbers


Run method executes every command in list of actor model
But what if execute of command in the list puts another command in the list?
When you push a button the thread dies

A single thread (the "actor") pulls jobs from the queue and executes them one at a time
No shared state—everything the actor needs is contained in the command object itself

Run to completion model for threads: this solves the stack problem
