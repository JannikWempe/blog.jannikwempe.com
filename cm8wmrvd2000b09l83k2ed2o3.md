---
title: "Keep Your Functions Clean & Focused: Context Provision with Node.js AsyncLocalStorage"
seoTitle: "Keep Functions Clean: Context Provision with Node.js AsyncLocalStorage"
seoDescription: "Learn how to use Node.js AsyncLocalStorage to implement the open/closed principle, enhancing code maintainability and scalability"
datePublished: Mon Mar 31 2025 05:30:38 GMT+0000 (Coordinated Universal Time)
cuid: cm8wmrvd2000b09l83k2ed2o3
slug: nodejs-async-local-storage-context
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743399027018/a96d6a52-9ddf-4f57-8619-5430e784915c.png
tags: nodejs

---

You have probably heard of the **SOLID principles** in programming. By applying them to your codebase, you can create a solid (🤡) architecture that is both easy to maintain and extend.

SOLID stands for:

* Single Responsibility
    
* Open/Closed
    
* Liskov Substitution
    
* Interface Segregation
    
* Dependency Inversion
    

In this post, I will demonstrate how to leverage Node.js’s built-in `AsyncLocalStorage` to implement the **Open/Closed principle**.

## What is the Open/Closed Principle?

The open/closed principle states that software entities (classes, modules, functions, etc.) should be **open for extension but closed for modification**.

**But what does that mean?**

In this post, I will focus on two examples to showcase how you can apply the open/closed principle using Node.js’s `AsyncLocalStorage`:

1. Database transactions
    
2. Logging
    

First, I'll show you a violation of the open/closed principle, and then I will demonstrate how to adhere to it using Node.js’s built-in `AsyncLocalStorage`.

## Violation of the Open/Closed Principle

### Example 1: Database Transactions

Imagine you have a repository function that creates a user profile in the database.

```javascript
import { db } from './db';

async function main() {
  await createUserProfile({ id: '1', name: 'John Doe' });
}

async function createUserProfile(profile) {
  await db.insert(userProfileTable).values(profile);
};
```

Alright, now suppose you want to create the user profile together with a user entity stored in a different table. You should do that in a single transaction because neither should exist without the other. This is what you might do:

```javascript
import { db } from './db';

async function main() {
  await db.transaction(async (transaction) => {
    await Promise.all([
      createUserProfile({ id: '1', name: 'John Doe' }, transaction),
      createUser({ id: '1', name: 'John Doe' }, transaction),
    ]);
  });
}

// ❌ we have added the transaction argument to the function (= extended the function)
// ❌ this violates the open/closed principle
async function createUserProfile(profile, transaction) {
  await (transaction ?? db).insert(userProfileTable).values(profile);
};

async function createUser(user, transaction) {
  await (transaction ?? db).insert(userTable).values(user);
}
```

That works, but it is a violation of the open/closed principle ❌ You are modifying the `createUserProfile` function to accommodate the new requirement. Every time you decide to run a repository function within a transaction, you must modify the function to pass the transaction argument.

### Example 2: Logging

Imagine you have a logger with an `appendKeys` method that appends keys to every subsequent log statement (just like the [logger from Powertools for AWS Lambda](https://docs.powertools.aws.dev/lambda/typescript/latest/core/logger/#temporary-keys)). This is very useful to find all logs that are related.

Consider a scenario where you have a function that processes an order. You want to easily find all logs related to a specific order. To achieve that, you use `logger.appendKeys` to append the `orderId` to every log statement:

```javascript
import { logger } from './logger';

async function main() {
  await processOrder(order);
}

async function processOrder(order) {
	logger.appendKeys({ orderId: order.id });
  // ...
  const total = calculateOrderTotal(order.lineItems);
  // ...
}

function calculateOrderTotal(lineItems) {
  // calculation...

  // this will also log the orderId since we have appended it to the logger instance
  logger.debug('calculated order total', { total: 100 });
}
```

Now, suppose you want to process multiple orders concurrently. You cannot simply call `appendKeys` on the global logger instance because the `orderId` key would be overwritten, as all `processOrder` executions share the same logger instance.

One way to deal with this, would be to create a separate logger instance for every `processOrder` call:

```javascript
import { logger } from './logger';

async function main() {
  await Promise.all(orders.map(order => {
    const childLogger = logger.createChild();
    return processOrder(order, childLogger);
  }));
}

// ❌ we had to add the logger argument to the function
// ❌ this violates the open/closed principle
async function processOrder(order, logger) {
	logger.appendKeys({ orderId: order.id });
  // ...
}
```

Another solution is to not use `logger.appendKeys` altogether and instead pass the `orderId` to every log statement:

```javascript
import { logger } from './logger';

async function main() {
  await Promise.all(orders.map(order => processOrder(order, childLogger)));
}

async function processOrder(order) {
  // ...
  const total = calculateOrderTotal(order);
  // ...
}

// ❌ we now pass the whole order because we need access to the orderId for logging
// ❌ this violates the open/closed principle
function calculateOrderTotal(order) {
  // calculation...

  // this will also log the orderId since we have appended it to the logger instance
  logger.debug('calculated order total', { total: 100, orderId: order.id });
}
```

This approach is not only tedious but also error-prone, since you must consistently use the same `orderId` key in every log statement to correlate logs associated with a specific order.

You can imagine that both of these approaches quickly become unmanageable—especially when `processOrder` calls many other functions in a deeply nested manner.

## Applying the Open/Closed Principle with AsyncLocalStorage

Now that we know what a violation of the open/closed principle looks like, let's explore how to apply the open/closed principle using Node.js’s built-in `AsyncLocalStorage`.

### Context Sharing with AsyncLocalStorage

We want the `createUserProfile` and `processOrder` functions to work in different scenarios without needing to modify them. For instance, `createUserProfile` shouldn't have to know whether it is running in a transaction or not, and `processOrder` should not change just because it is executed concurrently.

So how do we solve this?

We can leverage `AsyncLocalStorage` to create a context that is available to all functions executed within the same scope. This allows us to modify the behavior of functions without actually changing their implementations.

```typescript
// context.ts
import { AsyncLocalStorage } from 'node:async_hooks';

export function createContext<T>() {
	const context = new AsyncLocalStorage<T>();
	return {
		use() {
			const result = context.getStore();
			if (!result) {
				throw new Error('No context available');
			}
			return result;
		},
		provide<R>(value: T, callback: () => R) {
			return context.run<R>(value, callback);
		},
	};
}
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">ℹ</div>
<div data-node-type="callout-text">I have learned about the usage of `AsyncLocalStorage` from the codebase of <a target="_self" rel="noopener noreferrer nofollow" href="https://github.com/terminaldotshop/terminal/blob/229bbd69d1a2576fbd3302ac8864b87fb4d184ed/packages/core/src/context.ts" style="pointer-events: none">terminaldotshop/terminal</a>. This is where I took the code for `createContext` from and also the transaction example (not the logging example though).</div>
</div>

`createContext` creates a private storage `context` that is only accessible within the `createContext` function. It returns an object with a `use` method that returns the current context and a `provide` method that allows you to provide a context (`value`) that is available with the `callback` function.

Example usage:

```typescript
const userContext = createContext<{ userId: string }>();

userContext.provide({ userId: '123' }, async () => {
  // this can be somewhere deeply nested in your codebase
  const { userId } = userContext.use();
});
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This is probably familiar to you if you know React. React also has `useContext` and `createContext` to provide data to (potentially deeply nested) components without having to pass props through every component.</div>
</div>

How can we leverage the `createContext` function to solve our examples?

### Example 1 Revisited: Database Transactions

Lets use the `createContext` function to create a context for the database transaction.

```typescript
type TransactionOrDb = Transaction | typeof db;

const TransactionContext = createContext<{
	transaction: TransactionOrDb;
}>();

async function provideTransaction<T>(
	callback: (transaction: Transaction) => Promise<T>,
	transactionOptions?: TransactionOptions,
): Promise<T> {
	try {
    // if there already is a transaction context, we use that
		const { transaction } = TransactionContext.use();
		return callback(transaction);
	} catch {
    // otherwise we start a new transaction that we provide to the callback
		const result = await db.transaction(
			async (transaction) => {
				return TransactionContext.provide({ transaction }, () => callback(transaction));
			},
			transactionOptions,
		);
		return result as T;
	}
}

async function useTransactionOrDb<T>(callback: (transactionOrDb: TransactionOrDb) => Promise<T>) {
	try {
    // this throws outside of the context (see `createContext().use()`)
		const { transaction } = TransactionContext.use();
		return callback(transaction);
	} catch {
    // if not within a context, we use the db instance
		return callback(db);
	}
}
```

We have two functions that essentially wrap the `use` and `provide` methods of the `createContext` function:

* `provideTransaction` is used to provide a transaction to the callback function. It either uses an already existing transaction or starts a new one.
    
* `useTransactionOrDb` returns the transaction if it is within a transaction context or the db instance otherwise.
    

Now, instead of using the `db` instance directly in our app, we can just always use `useTransactionOrDb`.

Back to our example, we have the following code before deciding to use a transaction:

```javascript
import { db } from './db';

async function main() {
  await createUserProfile({ id: '1', name: 'John Doe' });
}

async function createUserProfile(profile) {
  await useTransactionOrDb().insert(userProfileTable).values(profile);
};
```

Here, `createUserProfile` will just use the `db` instance directly since it is not within a transaction context.

Now, decide that we have to run `createUserProfile` in a transaction together with `createUser`. We can do that by providing a transaction to the `createUserProfile` function:

```javascript
import { db } from './db';

async function main() {
  await provideTransaction(async () => {
    await Promise.all([
      createUserProfile({ id: '1', name: 'John Doe' }),
      createUser({ id: '1', name: 'John Doe' }),
    ]);
  });
}

// ✅ the arguments are unchanged – no modification needed
async function createUserProfile(profile) {
  await useTransactionOrDb().insert(userProfileTable).values(profile);
};

async function createUser(user) {
  await useTransactionOrDb().insert(userTable).values(user);
}
```

Now we are following the open/closed principle! We did not have to modify the `createUserProfile` function to accommodate the new requirement. Only the caller of the function has to know about the context.

### Example 2 Revisited: Logging

Now, let's see how we can apply the open/closed principle to the logging example.

Again, we are creating some helper functions that wrap the `use` and `provide` methods of the `createContext` function:

```typescript
const LoggerContext = createContext<{
  logger: Logger;
}>();

export function createLoggerContext<T>(callback: (logger: Logger) => T): T {
  try {
    // if there is already a LoggerContext, use it
    const { logger } = LoggerContext.use();
    return callback(logger);
  } catch {
    // if there is no LoggerContext, create a new one with a new child logger
    let childLogger = logger.createChild();
    const result = LoggerContext.provide({ logger: childLogger }, () =>
      callback(childLogger),
    );
    return result as T;
  }
}

export function getLogger() {
  try {
    const { logger } = LoggerContext.use();
    return logger;
  } catch {
    return logger;
  }
}
```

This is essentially the same as in the previous example:

* `createLoggerContext` is used to create a child logger and provide it to the callback function.
    
* `getLogger` is used to return the logger. If there is no logger in the context, it will return the global logger.
    

Now we can use that to follow the open/closed principle in our second example. Instead of using the `logger` instance directly, just always use `getLogger`. This is how our initial code looks like:

```javascript
import { getLogger } from './logger';

async function main() {
  await processOrder(order);
}

async function processOrder(order) {
	getLogger().appendKeys({ orderId: order.id });
  // ...
  const total = calculateOrderTotal(order.lineItems);
  // ...
}

function calculateOrderTotal(lineItems) {
  // calculation...

  // this will also log the orderId since we have appended it to the logger instance
  getLogger().debug('calculated order total', { total: 100 });
}
```

Okay, now we change it to run `processOrder` in parallel:

```javascript
import { logger } from './logger';

async function main() {
  await Promise.all(orders.map(order => {
    return createLoggerContext(() => {
      return processOrder(order);
    });
  }));
}

// ✅ the arguments are unchanged – no modification needed
async function processOrder(order) {
	getLogger().appendKeys({ orderId: order.id });
  // ...
  const total = calculateOrderTotal(order.lineItems);
  // ...
}

function calculateOrderTotal(lineItems) {
  // calculation...

  // this will also log the orderId since we have appended it to the logger instance
  getLogger().debug('calculated order total', { total: 100 });
}
```

We now create a child logger for every `processOrder` call. `processOrder` can just call `getLogger().appendKeys` to append the `orderId` to the logger as it did before. Now the keys don't overwrite each other because `getLogger()` returns different a child logger instances.

By using `getLogger()` everywhere in throughout your codebase instead of the global `logger` instance, we don't have to modify any functions if they are run differently. We are following the open/closed principle!

## Conclusion

In this post, we have seen how to apply the open/closed principle using Node.js’s built-in `AsyncLocalStorage`. We created a `createContext` function that allows us to establish a context for any type, which we then leveraged to solve our two examples.

Here are some key takeaways:

* Decoupled Context Management: By using AsyncLocalStorage, you can inject contextual data (like transaction handles or logging metadata) without modifying the core business logic.
    
* Adherence to the Open/Closed Principle: Functions remain closed for modification while still being open for extension, minimizing the need to alter function signatures as new requirements arise.
    
* Elimination of Manual Parameter Drag: AsyncLocalStorage prevents the tedious and error-prone process of manually threading context through multiple function calls.
    
* Improved Maintainability: By isolating context handling from your repository and logging logic, your codebase stays cleaner, more modular, and easier to maintain.
    
* Enhanced Debugging and Consistency: Automatically propagated context ensures that logs and database transactions are consistent, making debugging and analytics more straightforward.
    
* Versatile Application: This approach benefits various scenarios, including managing database transactions and setting up consistent logging across parallel asynchronous processes. These takeaways emphasize how leveraging Node.js AsyncLocalStorage can lead to more robust, maintainable, and scalable code while adhering to fundamental design principles.
    

## Further Reading

* [Node.js AsyncLocalStorage API](https://nodejs.org/api/async_context.html)
    
* [AsyncLocalStorage Usage in the terminaldotshop codebase](https://github.com/terminaldotshop/terminal/blob/229bbd69d1a2576fbd3302ac8864b87fb4d184ed/packages/core/src/context.ts)
    
* [Open/Closed Principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)
    
* [SOLID Principles](https://en.wikipedia.org/wiki/SOLID_principle)