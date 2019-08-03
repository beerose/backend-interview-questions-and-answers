# Backend interview questions and answers

Based on the [arialdomartini's](https://github.com/arialdomartini) [list](https://github.com/arialdomartini/Back-End-Developer-Interview-Questions#patterns).

## Table of contents

1. [Design patterns](#patterns)

## <a name='patterns'>Design patterns:</a>

> 1.  Why are global and static objects evil? Can you show it with a code example?

(static variables are used to represent global state)

- **Bugs from the mutable global state** — hard to debug what caused a bug,
- **Makes testing harder,**
- **Function impurity,**
- **Code is harder to understand,**
- **Concurrency issues.**

Code example:

```c
int globalValue;

void doSomething() {
    globalValue = 2;
}

int main() {
  globalValue = 1;

  doSomething();

  // Programmer may not know that doSomething changes globalValue
  // and still expects globalValue to be 1

  if (globalValue == 1)
      // handle globalValue = 1
  else
      // handle globalValue = 2
  return 0;
}
```

> 2. Tell me about Inversion of Control and how it improves the design of code.

Build on the _Hollywood Principle:_ **Don’t call us, we’ll call you**. It means you don't have to go to Hollywood to be famous, Hollywood will find you and make you a star. So the control over you being a star is inverted.

In the programming world _Inversion of Control_ means that instead of having **callee** controlling the program flow, it's the **caller** who controls it.

Class `UserController` without IoC:

```c#
class UserController {
  void connectSQL() {
    SQLConnection connection = new PostgreSQLConnection()
      connection.connect()
  }
}
```

In the above example `UserController` depends on the implementation of `PostgreSQLConnection`, so the high-level module is dependent on the low-level module. Changes in the `PostgreSQLConnection` would affect `UserController`, which may cause some mess.

Class `UserController` with IoC:

```c#
class UserController {
  private SQLConnection _connection;

  UserController(SQLConnection connection) {
    _connection = connection;
  }

  connectSQL() {
    _connection.connect()
  }
}
```

Now `UserController` depends on the abstraction of `SQLConnection`, not implementation. In the constructor, you can pass anything that implements `SQLConnection` interface, so let's say for example `MySQLConnection`, `PostgreSQLConnection`, etc.

### Pros of using IoC:

- Loosely coupled system's components,
- Helps with code duplication,
- Provides the ability to swap dependency implementations,
- Components can be tested through mocking their dependencies.

> 3. The Law of Demeter (the Principle of Least Knowledge) states that each unit should have only limited knowledge about other units and it should only talk to its immediate friends (sometimes stated as "don't talk to strangers"). Would you write code violating this principle, show why it is a bad design and then fix it?

So, let's say we have a Person, which has property _backpack_.

```c#
class Person {
  Backpack backpack;

  void setBackpack(Backpack newBackpack) {
    backpack = newBackpack;
  }

  Backpack getBackpack() {
    return backpack;
  }
}
```

We have also the Backpack class:

```c#
class Backpack {
  Dictionary<string, Item> items;

  Item getItem(string itemName) {
    return items[itemName];
  }
}
```

Now, let's say someone asks for the lighter from some person's backpack:

```c#
Person jon = new Person();
Backpack backpack = jon.getBackpack();
Item item = backpack.getItem('lighter');
```

**Why is this bad?**

Imagine standing at a bus stop, some stranger approaches you and asks for a lighter. So you're giving him the whole backpack, possibly containing a lighter. I guess that's not what most of the people would do. The stranger can basically take whatever he wants.

1. _jon_ exposed much more information that he could have,
2. These two classes are now _tightly coupled._ If we change the Backpack class, we may
   have to make change the Person class and the way backpack and the item are extracted.
3. Another problem would be if the stranger decided to take all of the items and throw it in the garbage: `jon.setBackpack(null)`. Person class assumes that it has a backpack, so when someone would try to access it again, we would get runtime exception for calling a method on a null pointer.

**How to fix this?**

What would you do if someone asks you for a lighter? I would take a lighter from my backpack and give to this person.

So, let's modify Person class:

```c#
class Person {
  Backpack backpack;

  Item getItem(itemName) {
    return backpack.items[itemName];
  }
}
```

Now, when some asks for an item it would look in the following way:

```c#
Person jon = new Person();
Item lighter = jon.getItem('lighter');
```

**Why is this better?**

It surely better models the real-world scenario. You don't have to be afraid that someone would steal from you. Another thing is when Backpack class would be changed, only the Person class may require some updates, not the _stranger asking for stuff_ code (as long as the interface of the Person class stays the same).

> 4. Active-Record is the design pattern that promotes objects to include functions such as Insert, Update, and Delete, and properties that correspond to the columns in some underlying database table. In your opinion and experience, which are the limits and pitfalls of this pattern?

- This approach plays well if you have almost no business logic, when you only perform operations such as creates, reads, updates, deletes — basic CRUD. But in most of the cases you would have some business logic, if not now, then most likely in the future. So your application would go beyond the simple CRUD.
- Domain becomes tightly coupled to a particular persistence mechanism. If that mechanism requires some changes, for example you're switching from no-sql to sql database, every class that uses Active-Record would require changes as well.
- Because your database is so tightly coupled with your objects it’ll not make it easy to use your objects without the database. So since it's impossible to separate them, it makes the unit testing impossible without using a database.

> 5. Data-Mapper is a design pattern that promotes the use of a layer of Mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself. On the contrary, in Active-Record objects directly incorporate operations for persisting themselves to a database, and properties corresponding to the underlying database tables. Do you have an opinion on those patterns? When would you use one instead of the other?

Active-Record may be a good choice for simple CRUD apps with no business logic. It requires much less setup than Data-Mapper, so allows to have an app up and running fast. It's easy to learn and easy to use.

In any other cases (which is -- most), when the app is a little bit more complex than simple CRUD, Data-Mapper would be a better choice. It allowes to have more flexibilty between domain and database than Active-Record.

Some thoughts on those patterns and ORMs in general:

- Less SQL and relying on the ORM may cause worse performance -- you can do inefficient not knowing they are inefficient.
- Each ORM is different -- every time you switch to another project, which uses different ORM, you start learning it from scratch. It may not apply to Active-Record, though.
- You can't do _everything_ with ORM. They are more or less limited when it comes to writing some complex queries.
- Domain is tightly coupled to the database in Active-Record.
- In general Data-Mappers are little bit harder to learn and require more setup than AR.
