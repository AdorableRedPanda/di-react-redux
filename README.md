# di-react-redux

As a frontend developer, I have one most painful side in React everyday life. 
This issue often comes up in casual conversations or job interviews so, I understand, 
it is a burning question for many developers.

I'm going to focus on the problem associated with the common practice of using a state manager in React 
applications - not about any specific state-manager, but about approaches of its usages.

## Average React state-management app troubles

Let's begin by considering a diagram of a React application.

![arbitrary react app using store diagram](img/before_tree.png)

We have a data store that is being used by five red components: A, B, C, D, and E. Each component has direct access 
to the store, like through the hooks like `useReducer` or `useSelector` in Redux. The details of the state management
implementation are not important; what matters is that the components have knowledge of the inner structure of the 
store and need to know the exact location of the data they require.

It's important to note that A and B are related to the same business logic and are completely independent of C, D, 
and E. They work with different interface elements and different store fields. This information is not evident from 
the diagram and highlights the need for clear documentation and design to ensure easy understanding and maintenance 
of the codebase.

What's wrong with this app?
* It is difficult to determine which components are related by business logic and which are independent. For example, 
  there is no clear indication of the relationship between D and  E components, making them difficult to understand 
  without investigating their store usage.
* All components share state, which can lead to bugs in one component affecting others
* Testing any component requires setting up a test scenario, creating mock data for the store, and checking expected 
  store state. This can be time-consuming and difficult to manage.
* New team members can be challenging because they need to have a deep understanding of the entire store and how 
  it's used by all components. It's challenging to localize work and onboard team members part by part.
* Reusing components is difficult - the C component is tied to specific paths in the store. The needs to get the similar
view often leads to code duplication of C, D and E components. 
have 

Why this happened? Let's analyze the component's relations diagram in terms of 
[coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming)) and 
[cohesion](https://en.wikipedia.org/wiki/Cohesion_(computer_science)).

To put it simply, the coupling is metric that refers to how much different modules of a system rely on each other, 
cohesion - refers to how closely related and focused the elements within a module are. Low coupling and high cohesion
are better in software because they make the code easier to work with and reduce the chances of unintended consequences 
when making changes.

![arbitrary react app relations diagram](img/before_relations.png)

The diagram depicts a simple structure with six components, but there are some issues with it:
* There are no clear modules in the code, with components A and B related in business-logic being lumped together
  with the others.
* Every component is aware of the structure of the store.

Despite these issues, it is still possible to work with this structure thanks to React's unidirectional data flow 
and the independence of components. However, this approach can lead to more bugs, a larger codebase, and a more 
fragile development process, as well as longer onboarding for new team members.

## How to fix it?

### Theoretically 
The issues in the example are caused by a violation of the Dependency Inversion principle, which can be summarized 
as follows:

* High-level modules should not depend on low-level modules. Both should depend on abstractions.
* Abstractions should not depend on implementations. Implementations should depend on abstractions.

When a component directly accesses the store, it depends on the specific implementation of the store, including its 
paths and data. To address this, we need to make the component dependent only on abstractions and create a special 
way for the store to realize these abstractions.

### In practice

In other frameworks such as Ninject for .NET or Angular (lets you get a feel for how DI works), a DI container is
operates and replaces interfaces with class realizations during compile time.
How to make it in React way - without classical compilation?

[React Context](https://react.dev/learn/passing-data-deeply-with-context) allows us to provide a value to a
component tree without having to pass it down through every level of the tree - it can be used in conjunction with 
other techniques to create a simple form of DI container.
We will use the context object as dependency marker (instead of interface) to resolving dependency inside container.
`useContext` hook or `Context.Consumer` can be used to require realization through the context, while `Context.
Provider` can create a container for it.

To simplify component-store relationships, we will adopt a straightforward approach. Components that use the store will
require it as a specific abstraction based on business logic. For example, we can add two providers in the picture 
below, one for data used by A and B components and another for data used by C, D, and E components. The providers have 
direct access to the store, and therefore are colored red, while the other components are colored based on the 
corresponding provider.

![react app tree with providers diagram](img/after_tree.png)

What we got now:
* Separating components into independent green and blue groups, technical level dependencies between components are 
minimized and the risk of bugs is reduced (we almost exclude this kind of bugs).
* Writing tests for these components is simplified, as they don't rely on any store knowledge.
* These components can be easily reused with different data by changing the realization on the provider level.
* This approach makes it easier to onboard new team members since they can understand and work on each module 
  separately.
* By analyzing the abstraction level used in components, it's easier to understand whether they are related or not.

The diagram of relations doesn't require any comments - we can see two separated modules now/
![react app relations diagram](img/after_relations.png)
