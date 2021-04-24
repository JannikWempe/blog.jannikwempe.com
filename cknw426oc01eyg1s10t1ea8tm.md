## TypeScript decorators basics

⁉️ **What** are decorators? **What** types of decorators are there?

⁉️ **How** can they be used?

⁉️ **When** are they executed?

# Decorators
Decorators are a stage 2 ECMAScript proposal ("_draft"_; purpose: _"Precisely describe the syntax and semantics using formal spec language."_). Therefore, the feature isn't included in the ECMAScript standard yet. TypeScript (early) adopted the feature of decorators as an **experimental feature**.

But what are they? In the [ECMAScript proposal](https://github.com/tc39/proposal-decorators) they are described as follows:

> Decorators @ decorator are functions **called on class elements or other JavaScript syntax forms during definition**, **potentially wrapping or replacing them with a new value** returned by the decorator.

In the [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/decorators.html) decorators are described as:

> Decorators provide a way to add both **annotations and a meta-programming syntax** for class declarations and members.

To put it in a more general way: **you can change the behaviour of certain parts of the code by annotating them with a decorator.** The parts of the code, which can be annotated with decorators are described in the [Types of decorators](#types-of-decorators) section.

**BONUS:** There is even a **decorator pattern** described in the [Design Patterns book by the Gang of Four](https://en.wikipedia.org/wiki/Design_Patterns). Its intent is described as:
> to add responsibilities to individual objects **dynamically and transparently**, that is, without effecting other objects.

# Enable decorators <a name="enable-decorators"></a>
Since decorators are an experimental feature, they are **disabled by default**. You must enable them by either enabling it in the `tsconfig.json` or passing it to the TypeScript compiler (`tsc`). You should also at least use ES5 as a target (default is ES3).

`tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true
  }
}
```

**CLI**
```bash
tsc -t ES5 --experimentalDecorators
```

You might also want to have a look at the related [Emit Decorator Metadata](https://www.typescriptlang.org/tsconfig#emitDecoratorMetadata) setting (which is not in scope of this post.)

# Types of decorators <a name="types-of-decorators"></a>
There are **5 different types of decorators**:
- class decorators
- property decorators
- method decorators
- accessor decorators _(== method decorator applied to getter / setter function)_
- parameter decorators

The following example shows where they can be applied:
```typescript
// this is no runnable code since the decorators are not defined

@classDecorator
class Polygon {
  @propertyDecorator
  edges: number;
  private _x: number;

  constructor(@parameterDecorator edges: number, x: number) {
    this.edges = edges;
    this._x = x;
  }

  @accessorDecorator
  get x() {
    return this._x;
  }

  @methodDecorator
  calcuateArea(): number {
    // ...
  }
}
```

Class constructors can not have a decorator applied.

## Overview over signatures <a name="overview-over-signatures"></a>
Each of the decorator functions receives different parameters. The accessor decorator is an exception, because it is essentially just a method decorator, which is applied to an accessor (getter or setter).

The different signatures are defined in `node_modules/typescript/lib/lib.es5.d.ts`:
```typescript
interface TypedPropertyDescriptor<T> {
  enumerable?: boolean;
  configurable?: boolean;
  writable?: boolean;
  value?: T;
  get?: () => T;
  set?: (value: T) => void;
}

declare type ClassDecorator = <TFunction extends Function>(target: TFunction) => TFunction | void;
// also applies for accessor decorators
declare type PropertyDecorator = (target: Object, propertyKey: string | symbol) => void;
declare type MethodDecorator = <T>(target: Object, propertyKey: string | symbol, descriptor: TypedPropertyDescriptor<T>) => TypedPropertyDescriptor<T> | void;
declare type ParameterDecorator = (target: Object, propertyKey: string | symbol, parameterIndex: number) => void;
```

## Order of evaluation
The different types of decorators are evaluated in the following order:

⬇️ instance members: Property Decorators first and after that Accessor, Parameter or Method Decorators

⬇️ static members: Property Decorators first and after that Accessor, Parameter or Method Decorators

⬇️ Parameter Decorators are applied for the constructor.

⬇️ Class Decorators are applied for the class.

Bringing the different types, their signatures and order of evaluation together:
```typescript

function propertyDecorator(target: Object, propertyKey: string | symbol) {
  console.log("propertyDecorator", propertyKey);
}
function parameterDecorator(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  console.log("parameterDecorator", propertyKey, parameterIndex);
}
function methodDecorator<T>(target: Object, propertyKey: string | symbol, descriptor: TypedPropertyDescriptor<T>) {
  console.log("methodDecorator", propertyKey);
}
function accessorDecorator<T>(target: Object, propertyKey: string | symbol, descriptor: TypedPropertyDescriptor<T>) {
  console.log("accessorDecorator", propertyKey);
}
function classDecorator(target: Function) {
  console.log("classDecorator");
}

@classDecorator
class Polygon {
  @propertyDecorator
  private static _PI: number = 3.14;
  @propertyDecorator
  edges: number;
  private _x: number;

  constructor(@parameterDecorator edges: number, x: number) {
    this.edges = edges;
    this._x = x;
  }

  @methodDecorator
  static print(@parameterDecorator foo: string): void {
    // ...
  }

  @accessorDecorator
  static get PI(): number {
    return Polygon._PI;
  }

  @accessorDecorator
  get x() {
    return this._x;
  }

  @methodDecorator
  calcuateArea(@parameterDecorator bar: string): number {
    return this.x * 2;
  }
}

console.log("instantiating...")
new Polygon(3, 2)

// Output:
//   [LOG]: "propertyDecorator",  "edges"
//   [LOG]: "accessorDecorator",  "x"
//   [LOG]: "parameterDecorator",  "calcuateArea",  0
//   [LOG]: "methodDecorator",  "calcuateArea"
//   [LOG]: "propertyDecorator",  "_PI"
//   [LOG]: "parameterDecorator",  "print",  0
//   [LOG]: "methodDecorator",  "print"
//   [LOG]: "accessorDecorator",  "PI"
//   [LOG]: "parameterDecorator",  undefined,  0
//   [LOG]: "classDecorator"
//   [LOG]: "instantiating..."
```
[Open example in Playground](https://www.typescriptlang.org/play?#code/LAKAZgrgdgxgLgSwPZQAQAcBOT0FNNwCeAIrjEpgIZwUAUclmA5rnAFyoDyARgFZlwANBmx4ChANK5CHAM5xMCKE1QAfVLMIBbbkgA2ASlQBvUKlTkos-bgB0epE1oAiLDnxFS5KjUzPhbmJEUoQGANygAL6gkLCIKBiMlFqs+F4U1HQMzKwcPPzwAaIektJyCkoq6po6+gFJKXD4AJJQACa4AB4cUBA6+EbGqGYWKNZ6dg5Org2pmOk+FP4i7uIh9VSNLe1d4VEx0PDIaI0AFkhtC5mYADwAKgB89Iws7Fx8AkWrwWUaFcpqDTaXR6YQdWQwRToXwcO6EPBtAAKxXEpAhUN89wegxGlnGk0cLjOFyuvmWgRKIXCwxA0XAh3iaEoMBguFk1nmZAymMezxyb3ynxWQVKMj+igB1WBdVQ4MhCGhFFh8NwSJRnjZ8sVt0eOJA5jxNnshOczNZ7IopKWXxFVIitIOcWOFj0lHZVswfNeHAAYgzjnqDWMjVMXDBXe6uYs-HsHSBQAABcNu2Qe0DJ9moRH6QhMBKmfWoBMU1FR64jLAIABu1Fwf2oCBgqAA+ojmj0+tx8KgALyoADMtgAjAAWe3mYvqkhl3wjVUsWQd-qYcciau1lvdVC9Zf23FjBQQeB0YuzJqc7zXVDztlLruYYRbnf3wPmVBwU4IWS2G+yXvXtoF1XcwPy-Wxm06f9OlXOkRgTYlLhnCgRnkBsm0rKA4FoU9NjmD1UDAJAkHKCUmAMDgqyQBA2hMEZzAAeno1BbBYkZYMLBMzTZDk00LVDECbV4s2aWhyO3TtuwLN9UEwVgIEwNBsz0XMUHAtsYNAOCuItC9uWQwshM6UTaMLcxZLgeS0FA78II0+MOIQ3iDUoPQYAgWsAEFZMobD0DPNIkMwVBuEYEjKjE59JLomS5IU99P2-SCACpUAAJjs9jDQmY1piUVDMIQBtlBY2xnAMUAoFwAB3LMczzKBaH7YRUoMIA)

# Decorator factories
Maybe you already asked yourself, after having a look at the different signatures, how to pass additional properties to the decorator functions. The answer to that is: with decorator factories.

Decorator factories are just functions wrapped around the decorator function itself. With that in place, you are able to pass parameters to the outer function in order to modify the behavior of the decorator.

Example:
```typescript
function log(textToLog: string) {
  return function (target: Object, propertyKey: string | symbol) {
    console.log(textToLog);
  }
}

class C {
  @log("this will be logged")
  x: number;
}

// Output:
//   [LOG]: "this will be logged"
```
[Open example in Playground](https://www.typescriptlang.org/play?#code/LAKAZgrgdgxgLgSwPZQAQBskHMAUcCmAHnACpIAy2AXKgM5wBOCUWAlKgN6ioP5wQM0kWIhSo8AQwZY+NAPIAjAFb54AGlQAHBkk34GcAJ4BpfIZr0mLVAB86hgLYKk6dlxCpPqGClov8AHSYuATEZJRsANzcAL6gcSCgMOgStLSoAMKc3KgAAsE4AERwABYI6QDuCOjoqAr4GNgyACaFrDmENFAQTvrRIDFAA):

I know this example isn't too exciting, but it opens the door for a lot of possibilities. But I will keep some of them for the following parts of this series 😉

# Decorator composition
Can you apply multiple decorators at once? Yes! In what order are they executed? Have a look:

```typescript
function log(textToLog: string) {
  console.log(`outer: ${textToLog}`)
  return function (target: Object, propertyKey: string | symbol) {
    console.log(`inner: ${textToLog}`)
  }
}

class C {
  @log("first")
  @log("second")
  x: number;
}

// Output:
//   [LOG]: "outer: first"
//   [LOG]: "outer: second"
//   [LOG]: "inner: second"
//   [LOG]: "inner: first"
```
[Open example in Playground](https://www.typescriptlang.org/play?#code/LAKAZgrgdgxgLgSwPZQAQBskHMAUcCmAHnACpIAy2AXKgM5wBOCUWAlKgN6iqowq1J0+AHSZcAAyQQCDGgBIOBYmUpYAvuNbdUDfHAgM0kWIhSo8AQwZY9NAPIAjAFb54AGlQAHBkk-4GcACeANL4gTT0TCyoAD50gQC2DoLsXCA8PHxQAkKi2DjizFD+8opEpBTYGlrpqGqg9SCgMOgWtLSoAMKc2gACYjgARGAIDPSDNTz9+YO0rigAJhPahDRQEEn+ANwNQA)

The decorator factories are executed in the order of their occurrence and the decorator functions are executed in reversed order.

# Resources
🔗 [TypeScript Handbook - Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)

🔗 [GitHub issue discussion about adding Decorators to TypeScript](https://github.com/Microsoft/TypeScript/issues/2249)

## ECMAScript proposals <a name="ecmascript-proposals"></a>
🔗 [ECMAScript proposals](https://github.com/tc39/proposals)

🔗 [ECMAScript decorator proposal](https://github.com/tc39/proposal-decorators)


# Feedback welcome
I'd really appreciate your feedback. **What did you (not) like? Why?** Please let me know, so I can improve the content.
