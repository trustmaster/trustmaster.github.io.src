+++
categories = ["Development"]
comments = true
date = "2017-07-26T14:48:53+03:00"
description = "Taking the best of the 2 worlds: rapidness of development with dynamic languages and reliability of static languages with TypeScript. Contains some Go inspiration"
draft = false
image = "/img/post-bg.jpg"
tags = ["development"]
title = "TypeScript: JavaScript way to Go"

+++

# TypeScript: JavaScript way to Go

My journey at [The Grid](https://thegrid.io) was over and I had 3 months before the start of a new contract, so I took an opportunity to fill a position of an interim CTO at [Dater.com](https://dater.com), a new dating app that is going to change the industry by moving from "post & search" approach to online & offline game-alike activity.

The main challenge was setting higher standards of development culture within the team and facilitating the development cycle. But speaking of the technical side of things, the project is rather interesting too: TypeScript / Angular 4 / Ionic / Firebase / Google Cloud Services.

In this article I'd like to share a couple of patterns that I found myself useful in this project and share some general thoughts on TypeScript.

## Less is more

One of my colleagues said that TypeScript is a rather successful attempt to turn JavaScript into C#. I have to admit that such a statement has some reason behind it. With TypeScript you have full power of Inheritance, Access Modifiers, Mixins, Generics, Decorators, Interfaces, Union types, and what not.

It is very easy to find yourself spending a lot of time trying to choose a proper language construct for the problem, or just blindly mimicking the .NET habits in a project that is actually driven by a JavaScript engine.

Instead, how about applying the &quot;less is more&quot; principle to use only the essential features and focus on code rather than language? So that we could benefit from the dynamic nature of JavaScript and reliability of static languages.

### The must haves

So, let's pick the very essential features of TypeScript first.

Here is the Top 5 features that I like in TypeScript:

1. *ES2015+* - it's nice to keep writing just modern JavaScript in first place. Bonus: you don't have to deal with Babel.
2. *Type Annotations* - this actually brings 3 benefits:
  - static analysis makes the code more reliable;
  - the code is more self-documenting;
  - you get nice Code Intelligence features not only in Visual Studio but also in editors like SublimeText and Vim. For example, here is how it looks in SublimeText:
3. *Interfaces* - they can help writing more generic and testable code, see below.
4. *TSLint* - I actually use a combination of [TSLint + ESLint + AirBnB JavaScript Code Style Guide](https://www.npmjs.com/package/tslint-config-airbnb) to enforce more good practices in the code than a TypeScript compiler provides out of the box.
5. OK, it's hard to pick the last one. Especially because things like `async/await` are already included in #1.

![TypeScript autocomplete in ST3](/img/post/typescript_sublime_autocomplete_2.png)

*Fig. 1: TypeScript autocomplete and linter example in SublimeText3*

Really, all the features in the above list are cool, so if TypeScript is an acceptable part of your toolbox, you should try them to make work with JavaScript projects more enjoyable. Just fire up your favorite editor and try them!

*Note: I suspect that this is partially or fully applicable to [Flowtype](https://flowlang.org) as well, but I won't claim it this time because I need to do some practical Flow first*.

### Minimalistic type annotations

A common thread that many TypeScripters share is this: *define and annotate everything*. Some say it helps them feel more confident, some use this approach just for sake of IntelliSence. But for those who care of the balance between quality of code and development pace, there a simple alternative: *use the power of automatic type inference*. The most common places where type annotations come handy are:

- interface definitions;
- function arguments;
- class properties.

There the annotations hit two birds with one stone: make the code more self-documenting and give the compiler enough information for type inference. Inside the methods, if decomposed properly, the compiler is capable to infer a type for most of the variables. If you think this is too dynamic and unsafe, have a look at how Go compiler does a similar thing while keeping the code strictly typed.

### Do you really need a generic here?

Another common overengineering practice with TypeScript is using Generics to turn every possible type into a container and every possible function into a template function. This is a waste of time not only for those who write such code but also for those innocent souls who have to read and maintain it onwards.

Here are a couple of questions that need to be answered before using a generic:

- Is it really a generic or something that complies with a certain interface? In latter case, use an `interface` instead.
- Is it really a generic or just `any`? If there's no pattern in how the data is used or if it's just an `Object`, then it's likely to be `any`.

OK, time to put the Occam's razor aside and dive deeper into one of the fundamental parts of TypeScript: interfaces.

## Using interfaces to make code more generic and testable

In Go, interfaces are often used to implement polymorphic functions and Dependency Injection pattern. We can do a similar thing in TypeScript.

Let's implement a function that persists a `User` record in [FireBase](https://firebase.google.com/).

### Interface for a domain model

First, let's agree that a `User` is any object that conforms with the following interface:

```js
interface User {
  key: string;
  email: string;
  fullName?: string;
  birthday?: number;
}
```

A downside of interfaces in TypeScript is that they only exist at compile time. Meanwhile it makes whole lot of sense to check the data against interfaces at run time. A proposed method to solve this is to provide a type guard per interface:

```js
function isUser(object: any): object is User {
  return ('key' in object && 'email' in object);
}
```

Type guards are used just as other functions:

```js
if (!isUser(user)) {
  throw new Error('Invalid user object');
}
```

Writing a type guard per interface is not at all convenient, so a better solution remains an open question for now.

### Interface for a dependency

Interfaces can be used effectively to abstract the way dependencies are used from their implementation.

For instance, here are a couple of interfaces kindly provided by the `firebase` library itself:

```js
// Example extracts from firebase.d.ts, (c) Google
interface Database {
  app: firebase.app.App;
  goOffline(): any;
  goOnline(): any;
  ref(path?: string): firebase.database.Reference;
  refFromURL(url: string): firebase.database.Reference;
}

interface Reference extends firebase.database.Query {
  child(path: string): firebase.database.Reference;
  key: string|null;
  // ... (snap) ...
  push(value?: any, onComplete?: (a: Error|null ) => any): firebase.database.ThenableReference;
  remove(onComplete?: (a: Error | null) => any): firebase.Promise<any>;
  root: firebase.database.Reference;
  set(value: any, onComplete?: (a: Error|null) => any): firebase.Promise <any>;
  // ... (snap) ...
  update(values: Object, onComplete?: (a: Error|null) => any): firebase.Promise <any>;
}
```

We can just import them in our project:

```js
import { database } from 'firebase';

// Now we can type database.Database or database.Reference and get some nice code hints
```

Firebase is a good example of an API that provides the reusable TypeScript interfaces out of the box (I could brag that `firebase` interfaces are not compatible with `firebase-admin` interfaces, but that's a different issue). Another option is obtaining `.d.ts` files from the [DefinitelyTyped](http://definitelytyped.org/) project. Though, not all services and libraries come with definitions provided, so if your project relies on some 3rd party API that you want to be interchangeable and testable, you should consider implementing a minimal interface for it yourself. This is also the case if the definitions exist but are not up to date or contain tightly coupled code that depends on a certain way of use. So, I have to cry this out loud:

*Dear TypeScript library maintainers, please consider using `interface` instead of `class` to describe your API. In many cases it's just a matter of writing a few lines of code more, or even just replacing `class` with `interface`, but it makes coupling of client code more loose and your consumers won't have to maintain interfaces for your API themselves.*

### Using the interfaces

Consuming interfaces is almost identical to using "normal" types. Here is our `addUser` function:

```js
async function addUser(db: database.Database, user: User): Promise<User> {
  if (!user.email) {
    throw new Error('User email is required');
  }
  let savedUser = user;
  savedUser.key = db.ref('Users').push(user);
  return db.ref('/Users').set(savedUser);
}
```

This is how we could call it passing real objects:

```js
import * as firebase from 'firebase';
import { addUser, User } from './lib/user';

// Somewhere down the line
const userInput = <User>{
  email: input.email,
  fullName: input.fullName,
  birthday: input.birthday,
};

try {
  const user = await addUser(firebase.database(), userInput);
  sendResult(user);
} catch (err) {
  sendError(err);
}
```

Dealing with `User` is really trivial here, all you need to notice is how we pass `firebase.database()` as a dependency service.

#### Alternative ways to organize containers

Passing everything as the arguments of a function may be inconvenient. In many cases dependency containers are passed as class `constructor` arguments, so that the containers are reused through the state of `this`. Example:

```js
import { database } from 'firebase';
import { Bucket } from './lib/google-cloud-storage'; // Custom service interface

class UserStorage {
  constructor(private db: firebase.Database, private bucket: Bucket) { }
  async function addUser(user: User) {
    // The implementation is very similar to a function written before
  }
}
```

In functional or dataflow programming dependencies are passed as function arguments or as combined objects (messages/IPs/etc.). In the latter case it makes sense to separate dependencies from actual input:

```js
interface Deps {
  db: firebase.Database,
  bucket: Bucket,
}

// Factory function that returns an addUser implementation using the `deps`
function makeAddUser(deps: Deps): (user: User) => Promise<User> {
  return (user: User): Promise<User> => {
    // Here we use `database` and `bucket` from `deps` to save `user`
    // Let's write an example promise chain
    return findUserFiles(deps, user)
      .then((files) => uploadFiles(deps, files))
      .then(() => saveUser(deps, user));
  }
}

// Example usage
const addUser = makeAddUser({ db: firebase.database(), bucket: gcsBucket });
addUser(userData)
.then((user) => {
  // Success
})
.catch((err) => {
  // Error
});
```

It's a matter of specific application design choice how to wrap the concept. Underneath it remains the same.

### Mocking a service

Now we get closer to what this is all about. Let's write a mock for local Firebase testing using the [FirebaseServer](htts://npmjs.com/package/firebase-server) package.

```js
import * as firebase from 'firebase';
import * as FirebaseServer from 'firebase-server';

export default class FirebaseMock {
  private app;
  private server;
  constructor(private port = 5151) {
    this.server = new FirebaseServer(this.port, 'test-app-local.firebaseio.com');
    this.app = firebase.initializeApp({
      apiKey: 'fake-key',
      databaseURL: `ws://test-app-local.firebaseio.com:${this.port}`,
    });
  }
  database(): firebase.database.Database {
    return firebase.database(this.app);
  }
  stop(callback) {
    firebase.app().delete();
    this.server.close(callback);
  }
}
```

Mocking Firebase with `firebase-server` is as easy as that. In other cases you might actually need to implement some mocking methods that mimick the interface.

### Testing

This is bread and butter of Dependency Injection. Let's write a Mocha/Chai.js test for our function:

```js
import { database } from 'firebase';
import { addUser, User } from './lib/user';
import FirebaseMock from './mocks/firebase';

describe('addUser()', () => {
  let firebase: FirebaseMock;
  let db: firebase.Database;
  before((done) => {
    firebase = new FirebaseMock();
    db = firebase.database();
    done();
  });
  after((done) => {
    firebase.stop(done);
  });

  it('adds a user', async () => {
    const user = <User>{
      email: 'john@example.com',
      fullName: 'John Appleseed'
    };
    const savedUser = await addUser(db, user);
    expect(savedUser.key).to.be.a('string');
    expect(savedUser.email).to.equal(user.email);
    expect(savedUser.fullName).to.equal(user.fullName);
  });

  it('discards non-user objects', async() => {
    const user = <User>{};
    const didThrow = false;
    try {
      await addUser(db, user);
    } catch (e) {
      expect(e).to.be.an('error');
      didThrow = true;
    } finally {
      expect(didThrow).to.be.true;
    }
  });
});
```

If we omit the testing boilerplate, there is nothing here that would be different from the normal consumer code, meanwhile a mock is being used. All we have to do is to pass the mock object to the code being tested instead of a real service.

## Bottom line

There are many topics which I haven't covered in this article. For example, *Union types* which you can combine with *strict null checks* to make the use of `null` as a "third state" explicit:

```js
type OrderRecord interface {
  id: number;
  user_id: number;
  comment: string|null;
}
```

Also *Union types* are useful for the *return value or error* pattern:

```js
function getItem(id: number): Item|Error {
  if (somethingWentWrong()) {
    return new Error('Oops');
  }
  const item = loadItemFromSomewhere(id);
  return item;
}
```

But the main thought is this: it's not about how many language features are used in your projects, it's about how the most essential onces can be applied to achieve your team's goals.
