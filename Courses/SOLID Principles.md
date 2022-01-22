SOLID Prinicples (using a C# application as example) by @ardalis on Pluralsight


Code - https://github.com/ardalis/SolidSample
Also check - https://github.com/ardalis/CleanArchitecture

SOLID principles are a set of principles to adhere to while developing a Software solution. These principals talk about how the code should be organized. They talk about modules, which is mostly a class but not always, how they should be related and organized.

It is not necessary to stick to all these principals right from the beggining of product development journey, one should first focus on writing a working solution and then if making modifications to the code starts to seem like a very daunting task that has to done repeatedly then that code is a good candidate to be refactored using these principles.

These principles are turned into actionable patters using something called Design Patterns.

The SOLID Principles -

1. Single Responsibility Principal

Each module should only have one task to do. It is upto the developers to define what are different tasks but a good example can be - Business Logic, Logging, Networking etc.

It comes from an idea called segregation of concern. If two things can be done independently of each other i.e. how one is done is not dependent on how other is done then they should be done by two different modules instead of being done by the same module.

Adhering to this principle makes it easy to write unit tests. If it is difficult to write unit tests for your classes then maybe this principle is being violated.

2. Open Closed Principle

Every module should be open for extension but closed for modification. What this essentially means is that the module can be made to do new things, but that should not need changing the existing source code, except in case of fixing bugs.

A module that adheres to OCP will have few conditonal, if-else, code.

OCP is a balancing act between concreteness and abstraction. If the code is too concrete then it becomes difficult to extend, however if it is too abstract it becomes larger and more complicated.

One can start on the concrete side of things, but if the code needs to be changed more than 3 times, then they should start shifting to the abstract side.

Design pattern - Factory Design pattern.

Links - https://bit.ly/2LSXOuO
        https://bit.ly/2GmxglZ
        https://bit.ly/2AMmprC

3. Liskov Substitution Principle

A sub-class should be substitutable for a base class i.e. anywhere one would expect a base class to work, the sub class will work exactly as expected.

The sub class must provide its definition of all the behaviors expected of the base class and there should be no NotImplementedException in the code.

In general, a high level module should tell a low level module what to do and have no concern over how the low level module achieves the feat.

Indications of violation of LSP -
i. NotImplemented exception
ii. Type Checking in code
iii. Null Checking in code

Design pattern - NullObject pattern

4. Interface Segregation Principle

If a client is being used by a host application then it must only have those endpoints that are used by the host. That is an abstraction of client should not hav end points that the host does not use.

Having large interfaces i.e. abstractions with lots of endpoints are problematic because when new modules need to extend the abstraction they must implement all the behaviour even if the new module does not exhibit those behavior. Usually such modules throw an NotImplemented exception for behaviors they do not exihibit, this violates LSP.

You do not want to give your host applications multiple ways to do the same thing using your client interface, as it will increase the complexity and size of the module.

Design Pattern - Adapter Design Pattern

Also look at -
  1. Pain Driven Development
  2. Creating N-Tier application in C#
  3. Domain Driven Design Fundamentals

5. Dependency Inversion Principal

High level components must not depend on the details of low level components, but on the abstractions. Therefore while instantiating these high level components the controlling code must pass the concrete implementation of the low level components to the high level component. (This is the familiar process of Dependency Injection.)

The implementation details of a module must depend on it's abstraction, but not the other way around i.e. abstractions must not depend on the details. Abstractions must also not leak the details of the module.

The low level components can be supplied to the high level component using 
  1. Setter methods - high level module exposes endpoints to set the low level component they will make use of
  2. Constructor - the constructor of the high level module demands that instances of low level objects it depends on be passed to it

Design Pattern - Strategy Design pattern
