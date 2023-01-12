# How to Create a Global Store in React (without Context)?

https://www.youtube.com/watch?v=GMeQ51MCegI

Most React applications need **global state** – state that is stshared between components and not owned by a single component. Common example use cases are authentication or a color mode.

Technically you could just place all our states at our top-level component and pass it down the React element tree to the components that need access to the state. But in any application but a very simple one that would require us to pass down the state several levels down the tree and through components that are not using the state themselves at all. It would pollute the code and ruin the Developer Experience (DX). That problem is known as **prop-drilling**. It is the reason why other solutions exist to share state between multiple components being further apart in the component tree. Other solutions include state management libraries (React Redux, Zustand, Jotai and many more), React Context API, and a **global store using a subscription mechanism – the solution we will explore in this post**.

React Context API is more widely used than the solution we will be creating. But the React Context API is often not the best fit for global state.

## The Downside of React Context API for Global State

The intro of the [Context API docs](https://reactjs.org/docs/context.html) read like a direct solution to the aforementioned prop-drilling problem:

> Context provides a way to pass data through the component tree without having to pass props down manually at every level.

It is indeed exactly made for the purpose of preventing prop-drilling and being able to create multiple, separated contexts for different parts of the component tree (the context provider provides access to the context for all components below it in the component tree; the closest provider is used when consuming the context).

BUT, it has one downside, which is mentioned in the [docs of useContext](https://reactjs.org/docs/hooks-reference.html#usecontext):

> When the nearest `<MyContext.Provider>` above the component updates, this Hook will trigger a rerender with the latest context value passed to that `MyContext` provider. **Even if an ancestor uses** `React.memo` or `shouldComponentUpdate`, a rerender will still happen starting at the component itself using `useContext`.

If a component consumes a context using `useContext` it will re-render every time the value provided by the context changes – even if the component does is not actually using the piece of the value that changed.

Let me give you an example: imagine we need a state holding the current user. The `type User` is defined as follows:

```typescript
type Social = {
  twitterUrl?: string
  githubUrl?: string
}

type Contact = {
  email: string
  phone?: string
}

type User = {
  id: string
  username: string
  firstName: string
  lastName: string
  birthDate?: string
  contact: Contact
  social?: Social
}
```

* disadvantages of context
    
* closure -&gt; need UI update (useState/useReducer)
    
* React 18 hook useExternalStore
    

## References