---
title: Dependency management in classes
date: 2018-07-31T00:19:43+03:00
author: "Taras Kushnir"
image: dependency.jpeg
categories:
  - Programming
keywords:
  - oop
  - architecture
  - classes
  - code
---

Object-oriented languages like Ruby, C++ or Java tend to have one common problem among others: dependency injection. It is very rare for a class to be fully self-contained and never depend on anything else in your code. Usually there're multiple connections between them: simple, transitive or even circular. And designing good dependency injection often makes a huge difference in terms of maintainability and testability of your code. This is the thing that should be done properly rather sooner than later.

Let's review our possible options. I'd like to start with least maintainable and testable solutions first:

### God object

"God object" is a class that has all other dependencies accessible from and that all classes in your code depend on. At a first point of view it makes things look simple: all your classes have only one dependency (the god object itself) and this makes their interface simple. God object on the other hand, has all or majority of the classes of your codebase listed as dependency. This approach may work for small projects that do not have many dependencies but on the long run it makes things complicated to test and the god object itself - a huge monster hard to maintain and change.

```
class GodObject {
  public getServiceA() { return serviceA; }
  public getServiceB() { return serviceB; }
  // ...

  private ServiceA serviceA;
  private ServiceB serviceB;
  // ...
}

class A {
  public A (GodObject obj) {
    this.serviceA = obj.getServiceA();
    this.serviceB = obj.getServiceB();
  }
}
```

### Dependency injection containers

This is a very popular way of managing the dependencies but on the inside this is a dynamic variation of a god object. It is slightly more generic in a way that it doesn't depend on classes of your codebase directly, but instead lets you build dependencies "on the fly".

```
container.inject<ServiceA>(new ServiceA());
container.inejct<ServiceB>(new ServiceB());

class A {
  public A (DIContainer container) {
    this.serviceA = container.resolve<ServiceA>();
    this.serviceB = container.resolve<ServiceB>();
  }
}
```

Dependencies of the class A now are implicit: it's hard to tell them without reading it's whole code. Also it's very easy to make a mistake of bringing new dependency to the class which your will forget to satisfy and it will not be checked by the compiler.


### Messaging systems

In this case you can make use of technique used in distributed systems to reduce complecity and make use of dependencies: messages. A message is a self-contained piece of "parameters pack" (or data transfer object) passed to the unknown at a compile time target. Your class accepts "message hub" as a depedency. Message hub is a class that connects senders with receivers of the messages. Arbitrary entity can sign up for certain messages as a receiver and when anybody will produce messages of this kind, message hub will notify all interested receivers. 

```
source.addListener(target);
source.emit(new Message());

class Target {
  public handleMessage(Message msg) {
    // do something
  }
}
```

This sounds pretty much as slightly inverted dependency injection container. Instead of requesting known services to call them directly, you send known messages (parameters) to unknown APIs. Learning dependencies of your class would require to read it's source code completely. If you will not satisfly the dependency the compiler will not let you know about it. 


### Dependency setters

A step aside from dependency injection containers are public setters that inject the dependencies directly. They make interface of a class more explicit but tend to bloat it significantly.

```
class A {
  public void setServiceA(ServiceA serviceA) {
    this.serviceA = serviceA;
  }
  
  public void setServiceB(ServiceB serviceB) {
    this.serviceB = serviceB;
  }
  
  // ...
}
```

Also this approach makes you check if you have injected services A and B in places where you want to make use of them.

### Construction time injection

Injecting the dependencies at construction time is very clean and maintainable way to manage dependencies. A class constructor that accepts generic interfaces of dependencies is very easy to test.

```
class A {
  public A (IServiceA serviceA, IServiceB serviceB) {
    this.serviceA = serviceA;
    this.serviceB = serviceB;
  }
}
```

A contract of this class is explicit, immutable and known on the compile time. You cannot introduce new implicit dependency somewhere inside your class. It is easy to inject fake objects for testing purposes. Also it introduces single place where you check validity of the injected dependencies so you don't have to do it all over your code of class A. If your construction time dependencies begin to overflow one line of code you know it's time to refactor your classes into smaller ones.

## Bottom line

I would say it's obvious why construction time injection should be preffered among others: strict API and compile-time checks. Of course this is not a silver bullet but if you don't have strong arguments for anything else - make yourself a favor and use it.
