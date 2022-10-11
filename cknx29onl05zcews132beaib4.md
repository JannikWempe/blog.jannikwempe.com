# TypeScript class decorators incl. Dependency Injection example

By using class decorators, we have **access to the constructor and also its prototype** (for explanation about constructors and prototype see [this MDN explanation of inheritance](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Inheritance)). Therefore, we can modify the whole class. We could add methods using its prototype, set defaults for parameters passed to the constructor, add attributes and also remove or wrap these.

# Class decorator with generic constraint
In the [TypeScript decorators basic post](https://blog.jannikwempe.com/typescript-decorators-basics#overview-over-signatures) I already described the signature of the different types of decorators including the class decorator. We can use TypeScripts `extends` keyword to ensure the target is a constructor. That enables us to treat `target` as a constructor (that is why I renamed it to `constructor` in the following example) and use features like extending `constructor`.

```typescript
type Constructor = {
  new (...args: any[]): {}
}
function classDecorator <T extends Constructor>(constructor: T): T | void {
  console.log(constructor)
  return class extends constructor {} // exentds works
}

// original signature as in typescript/lib/lib.es5.d.ts
// not only restricted to target being a constructor, therefore extending target does not work
// function classDecorator<TFunction extends Function>(target: TFunction): TFunction | void  {
//   console.log(target)
//   return class extends target {}
// }

@classDecorator
class User {
  constructor(public name: string) {}
}

// Output:
//   [LOG]: class User {
//      constructor(name) {
//        this.name = name;
//      }
//    }
```
[Open example in Playground](https://www.typescriptlang.org/play?#code/C4TwDgpgBAwg9gOwM7AE4FcDGw6qgXigG8BYAKCigQgHcoAKAOmYENUBzJALihYRADaAXQCUPIgF9yUsgDN0CbAEtEUTABsWSJABEImXCxx4APABUoEAB7AICACZJYiFBmy4AfPQPI0WYzxmYlAWAD5QAG5wSvbE5JQ+SHDqEIzqcOzeLn7uqCLxUKgQwOioCGqa2pY2do5q2W7GxBJQAPSt1XbAdTS4ANZI0uTk7VC4SuxKCCzqUEgT0yVFvE5TUKCQSJioSmDArepKAEYHx4wQSACsjPaMwINkowhwwGMI6iCFF2hK2BCxOHWbHYxSgRwgU3YvHqvkauAANOsABYQIqyXDQay2ByQoEcUH2OAXKgvKC9VB9EYdeSKYAqcoaLS6fSGYzmABiCmUqixtScnNp9K8wGBxUCAu5CGCZgldNU4SiMUopEeHQSLmSqXSmRF+OA+VVlC+SwZlScvIcTl1INekipUBk5AAAoztHoDKgjLhyK6nABVJCouIUGGufy4ehgdBHQ6YKgsAC2EB4rkhIma0iAA)

# Limitations
There is a limitation of modifying the class using a class decorator, which you should be aware of:

> TypeScript supports the runtime semantics of the decorator proposal, but **does not currently track changes to the shape of the target**. Adding or removing methods and properties, for example, will not be tracked by the type system.

You can modify the class, but it's type will not be changed. Open the examples in the next section in the Playground to get an idea of what that means.

There is an ongoing [open issue](https://github.com/Microsoft/TypeScript/issues/4881) (since 2015) in the TypeScript repo regarding that limitation.

There is a workaround using interface merging, but having to do that somehow misses the point of using the decorator in the first place.
```typescript
function printable <T extends { new (...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    print() {
      console.log(constructor.name)
    }
  }
}

// workaround to fix typing limitation
// now print() exists on User
interface User {
  print: () => void;
}

@printable
class User {
  constructor(public name: string) {}
}

const jannik = new User("Jannik");
console.log(jannik.name)
jannik.print() // without workaround: Property 'print' does not exist on type 'User'.

// Output:
//   [LOG]: "Jannik"
//   [LOG]: "User"
```
[Open example in Playground](https://www.typescriptlang.org/play?#code/GYVwdgxgLglg9mABABwE4zFAhgIwDYCmiAPACqIEAeUBYAJgM6IDeiYBA7ogBQB0-WVAHMGALkRYwATwDaAXQCU45gF9EKgHzcICBlFQhocVONIKWAWABQiRKgJQQqJBDxYGTKjXpMdYPQZGqJY2tijomNzmzNZhYX4McIS8eHBC2rr6hlDGvGBYALYECrFhKqXlVpXWAPQ1iBzGANaCcOB0iDmIwDCUnVLIGEKIeDAFMNiwCLX1YHBcaBhQURSUMHpMCIgAqgwEqNZL+8BYEES7+yG2i5jiKwC8GogAbnAwdADc1tVWAAI32HwBGsrncTAuwRioQSWSC3GQIHwMAgbEKBHEASG0UqPxhiAAVpIwDAmoh7mxODs9qhuAAiABSRJJtIUXysCSSBBSaW4hLAxKaeTRJSsfIFvABKzqDQmAAs2lAGs1Wu1xAAFVBwZD7KBSRAAcgB+sQdDgBCYc0VVHWiq2uu1Boh+t4QA)

# Examples
Finally, some examples to get an idea of what you can do. There are very few limitations of what you can do since you essentially could just replace the whole class.

## Add properties
The following example shows how to add additional attributes to the class and modifying them by passing a function to the decorator factory (see [TypeScript decorators basics post](https://blog.jannikwempe.com/typescript-decorators-basics#overview-over-signatures#decorator-factories) for the concept of decorator factories).

```typescript
interface Entity {
  id: string | number;
  created: Date;
}

function Entity(generateId: () => string | number) {
  return function <T extends { new (...args: any[]): {} }>(constructor: T) {
    return class extends constructor implements Entity {
      id = generateId();
      created = new Date();
    }
  }
}

@Entity(Math.random)
class User {
  constructor(public name: string) {}
}

const jannik = new User("Jannik");
console.log(jannik.id)
console.log(jannik.created)

// Output:
//   [LOG]: 0.48790990206152396
//   [LOG]: Date: "2021-01-23T10:36:12.914Z"
```
[Open example in Playground](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgKLmGAnsg3gWAChllgATALmQGcwpQBzZAH2RAFcBbAI2gG4iJBFAhxIlZABExEAYQC+RIjHYgEYYAHsQaDNgAUDCCGgyAkhP0BKZAF4AfDTqMWbLryg2CxZCLDsoHRU1DW1kAB4AFWQIAA9IEDJqPDYIAHdkfQA6HLgoBmoqOBAsAG0AXSsqXHlkeXt9BG1aKHZ1TSgqSK9BEl8If0DkBAAbOGpkuISk4ea6NrAO0k4ABxGITmMwZPQNbDxevtIyO2QjEyhzMms5I6ERGRPbVIzpSBvDut7FBSVCAAFdpgsPoALJiAAWWUuiU0nCsRFG42SAFVqNADj4miAWgsOvoVuxuCNgAg2HBNlQWowvD8foi5sgAFbFEDAADWpxMGTR0H0ACIAFKsjn8qxybHUTTrLIjTQMfQskBs9lZcg2AD0GuQAAUoJoVtB9gBycjG5BkTQQZIgTRgGKxYC0ZBhbCG5DG3lQc0AWhoEBQ-IAMsBOJgxFocfz-epIwycdKILL5YqRarhKJxJrtXqDUacMaM49zZbrWw7Q6nfbXVh3Z70d7kH70YGQ2GwBHmtH0bHtEA)

This can be quite handy for entities, which you want to store somewhere. You can pass the method to generate the entities `id` and the `created` timestamp will automatically be set. You could also extend these example for example by passing a function to format the timestamp.

## Prevent modifications of a class
In this example we use `Object.seal()` on the constructor itself and on its prototype in order to prevent adding/removing properties and make existing properties non-configurable. This could be handy for (parts of) libraries, which should be modified.

```typescript
function sealed<T extends { new (...args: any[]): {} }>(constructor: T) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class User {
  constructor(public name: string) {}
}

User.prototype.isAdmin = true; // changing the prototype

const jannik = new User("Jannik");
console.log(jannik.isAdmin) // without @sealed -> true
```
[Open example in Playground](https://www.typescriptlang.org/play?#code/GYVwdgxgLglg9mABAZwKYEMA2qAmAeAFUVQA8pUwdlEBvRMVAd0QAoA6D9AJwHNkAuROjABPANoBdAJSCaAX0RyAfCwgJkULiGhwugglNoBYAFCJEAeQBGAK1TQ2aLKvWbtUXVIDcp89bsOTpguYBpaOlxsAA5ccB5QIlGo3qZypqYAAkG4phCY6MjUAKpoXMZmiGqhbhEsUSBWmDAQ9OgAtqiCYTBgPIbyqekmJaiRMXFxiahsMMgAgjhtPYgAvIhuqF6IAPTblQAWwjw9POv7qIjj8VNDVRqINsJgMADWq-RMiCNcLABEAFJPV6-FImO5wbBsTBwHgsR5gZ4vGbzRY9Qy7RCMGBQfZwEBQRBZDDYHCIAC0SnWWlQQA)

## Dependency Injection
An advanced usage of class decorators (in synergy with parameter decorators) would be Dependency Injection (DI). This concept is heavily used by frameworks like [Angular](https://angular.io/) and [NestJS](https://nestjs.com/). I will provide a minimal working example. Hopefully you get an idea of the overall concept after that.

DI can be achieved by three steps:
1. Register an instance of a class that should be injectable in other classes in a `Container` (also called `Registry`)
2. Use a parameter decorator to mark the classes to be injected (here: `@inject()`; commonly done in the constructor of that class, called **constructor based injection**).
3. Use a class decorator (here: `@injectionTarget`) for a class that should be the target of injections.

The following example shows the `UserRepository` being injected into the `UserService`. The created instance of `UserService` has access to an instance of `UserRepository` without having a repository passed to its constructor (it has been injected). You can find the explanation as comments in the code.

```typescript
class Container {
  // holding instances of injectable classes by key
  private static registry: Map<string, any> = new Map();

  static register(key: string, instance: any) {
    if (!Container.registry.has(key)) {
      Container.registry.set(key, instance);
      console.log(`Added ${key} to the registry.`);
    }
  }

  static get(key: string) {
    return Container.registry.get(key)
  }
}

// in order to know which parameters of the constructor (index) should be injected (identified by key)
interface Injection {
  index: number;
  key: string;
}

// add to class which has constructor paramteters marked with @inject()
function injectionTarget() {
  return function injectionTarget <T extends { new (...args: any[]): {} }>(constructor: T): T | void {
    // replacing the original constructor with a new one that provides the injections from the Container
    return class extends constructor {
      constructor(...args: any[]) {
        // get injections from class; previously created by @inject()
        const injections = (constructor as any).injections as Injection[]
        // get the instances to inject from the Container
        // this implementation does not support args which should not be injected
        const injectedArgs: any[] = injections.map(({key}) => {
          console.log(`Injecting an instance identified by key ${key}`)
          return Container.get(key)
        })
        // call original constructor with injected arguments
        super(...injectedArgs);
      }
    }
  }
}

// mark constructor parameters which should be injected
// this stores the information about the properties which should be injected
function inject(key: string) {
  return function (target: Object, propertyKey: string | symbol, parameterIndex: number) {
    const injection: Injection = { index: parameterIndex, key }
    const existingInjections: Injection[] = (target as any).injections || []
    // create property 'injections' holding all constructor parameters, which should be injected
    Object.defineProperty(target, "injections", {
      enumerable: false,
      configurable: false,
      writable: false,
      value: [...existingInjections, injection]
    })
  }
}

type User = { name: string; }

// example for a class to be injected
class UserRepository {
  findAllUser(): User[] {
    return [{ name: "Jannik" }, { name: "Max" }]
  }
}

@injectionTarget()
class UserService {
  userRepository: UserRepository;

  // an instance of the UserRepository class, identified by key 'UserRepositroy' should be injected
  constructor(@inject("UserRepository") userRepository?: UserRepository) {
    // ensures userRepository exists and no checks for undefined are required throughout the class
    if (!userRepository) throw Error("No UserRepository provided or injected.")
    this.userRepository = userRepository;
  }

  getAllUser(): User[] {
    // access to an instance of UserRepository
    return this.userRepository.findAllUser()
  }
}

// initially register all classes which should be injectable with the Container
Container.register("UserRepository", new UserRepository())

const userService = new UserService()
// userService has access to an instance of UserRepository without having it provided in the constructor
// -> it has been injected!
console.log(userService.getAllUser())

// Output:
//   [LOG]: "Added UserRepository to the registry."
//   [LOG]: "Injecting an instance identified by key UserRepository"
//   [LOG]: [{"name": "Jannik"}, {"name": "Max"}]
```
[Open in Playground](https://www.typescriptlang.org/play?#code/MYGwhgzhAEDCD2A7ALmAlogpgJ2gbwFgAoaaAejOgAt4QATDAc2gwlUWExngDMXEAVpmCoARiEzRQkCF2iiAntADWmBcVIAHbGgBuYZJLYG0waNkyM0bbAoBc0ALJhNAHhtMANNDCIFAPmgAXmgsAHcnFwAKAEoAbmINaGNkU3NLa0NsKNV7ZOQdREZvVnZOB18FGPwk0jQ+KIBCBBR0LGwAOgsrGwUOqkgctRjqwhJSCZbUDBwujN6O2WQhhRLEYw5MeNqJ4CQIWkwOkHhGKIADAEE6Okw6aAASPFyAX2hkeHeqSW7M2w7zttxqQXklQUQkik0oxMMtcg4PEVRjsLMgAK7YRBwJDTdpzHoFPowuHDMHEcHECj8aDwbC3XAfFSIeARMJUUxUaCaMDYMAAW1hOG4fGQ3yk+wKaJEtOgUQwtwAHtUIDQ0fR5JIMEIRHdZWhbih6mhdYoVKSiBgsjwwJxoABJQTCVJIGrjeWYBUORBovmiHAJcbw-KFRgBilEKlgG7vT7SKDQNkc6iQcXrSXS3Dc3l8wxZGB8nmqe5hNCi6AAAS1TtixB4aI4zqxVZEaCQABUecTYq7SKiMVi6w3W03HS3253YdBXG3oB7DIg6DA8KFMBEoh0N52IBU-ABtAC6MQceDeL38UT2aewUo+2AcbaP0BnAB9oLp4PqexMqRZNOBgEwXySLSaBWIgYAgKmNg3jKJZlmAK4REgkiigYXLYPAuj6nIoqaqOjYwDwGF8kB2KtDM2AorC-ZSOA8ZzpgC4wJe0EZl+ExQemt7rpu2CMNuPh7oe7EceQlDEvw2oEdARHwCRcYQHE6GYFh8BohAIBKMAFgGCaSiVvhNbAqJLHIJJTrDjAIQXhK15sSmlQxB0zbSSmDpScOB47BxVISbh-AbJwMCMi5MnEaRUxtDg3nfpQorWCwfJ-pgAqtI20B0PAcjMmZEBopomi0mZW4JuywCciqanqjlGrmTqdAxaQpl1YYdCXHxAmVAewQtZZHQFpoURRM8agvNUQSBGMokmfshzHKcFzuRZRSCQFZSagaqQ8Ma9ymrkjwjQoLyAo1Ex9piZG4rMXa5DEp1jadVLABBkEgWBEGcXZt4JqWnIubqnY+oxyAQKdeWaDgPHOfhdztfxQLTeCHFI9A4LhlSBbYMon0wZmPL8oK2AwIm5XJKq6p+i1dyUnF7IwGwtI4WKGA8LSBbpWAohqWZ-naPAEPYKkcgkxV5O7XhUnU0Qg5jiOUkrAiBRMMi4znQO9ay7KqB8bCDgAPKiFJ3h8wLyAKAA0moishtAr4QAovq0Mb+MClkDqKl6Pp+tgKscc1LnDg4S2ayEy7up6XIu4T7set4+0o01EqzgqmRMMHBFB-hnn7j1UTaxJDl+E5Af7Lbr5ecZT06YY6H8zgZvQAA5CX6yN9QtAMCtL042xWYE3m3gi2TVXi1TDXGQbUkdLc21YAAChhpsKHnE7IN4ABELcQOv3hTaJjFA7y4iYA41ogLIngxZe22MBinMSKfEEXzFYQ6GID8yU-mCX8ZpD6CAaIT7QF3BuDoHpU5FHTpZNYHkkD7h2A9cYaNEhEDNhDaAABVWQuBQ6hAJtbJgSl0aUA9PyZKMkZQIQUjGWq-17jEGoVgnAAAlTAhUICllpEoPes82ogBAEw7Ij5BHdT3r2aiF1dzLnAgKBw68ABSvhEBoGUOvVGu88GyOgOvZwCo1EvHgUg8kKCDKwMQB2HWyw7pEEYdggAyjgLCto97qRYWw+AHDbx5EEaw9hnDbABiSJGEcgVgIijFD49xniuG0RkCUTaRo9JmiUI3SJfiCjwAUG3SqapR50KSKZL6tIoimOrOvNJHj-EKHXtUVx2BfGVK8QAfgcBU6JthfaxVnOsDEcg6kNPaUocBbAYC+HuMyKQ3xgDKEIjKesM8Zj3B5D8TAABHNEaALD3FFBhNEjBVQ8zFApHY9RZSNH6VEqp1QdksmgAAUWwBhbI68ABynw2lVNrlhW49wZR0I6DUnY8UIAdAuekmJIQwWNK4QGEEKDSDEkuPwwRsRWnYNETsSMwAgrBU+L4NavhbS8EwdggZVSqLogusC0FpLLleI6LwpFAjsFGThUQYh-BSxoBekoX4bAcA+H4bEqAwsyqixHrQ-C99JBwU5P5SKFFiAKrxHyrIURym0vBbYHeiESVuK1cvEYKDmp1IcdgJxkgQjhD1dgM1FqjJUlNY40wkgBijOxVwXFq1SiErCTaslXifqim5smLCK1SxfOwvcDApFCm4xptAAAtIECNbqNSMTHo0Bhs0JDzTOE681LqOiIuRSykYQA)

Of course this is a basic example with a lot of missing features, but it showcases the potential of class decorators and the concept of DI quite well.

There a few libraries implementing DI:
🔷 [InversifyJS](https://github.com/inversify/InversifyJS)
🔷 [typedi](https://github.com/typestack/typedi)
🔷 [TSyringe](https://github.com/microsoft/tsyringe)

# Wrap Up
Class decorators can be very powerful, because you can change the whole class it is decorating. There is a limitation, because the type of a class changed by a decorator will not reflect that change.

Have you ever written your own class decorators? What class decorators have you used?

# Feedback welcome
I'd really appreciate your feedback. **What did you (not) like? Why?** Please let me know, so I can improve the content.
