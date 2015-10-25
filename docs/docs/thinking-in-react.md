---
id: thinking-in-react
title: Thinking in React
prev: tutorial.html
next: conferences.html
redirect_from: 'blog/2013/11/05/thinking-in-react.html'
---

React is, in my opinion, the premier way to build big, fast Web apps with JavaScript. It has scaled very well for us at Facebook and Instagram.

One of the many great parts of React is how it makes you think about apps as you build them. In this post, I'll walk you through the thought process of building a searchable product data table using React.

## Start with a mock

Imagine that we already have a JSON API and a mock from our designer. Our designer apparently isn't very good because the mock looks like this:

![Mockup](/react/img/blog/thinking-in-react-mock.png)

Our JSON API returns some data that looks like this:

```
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## Step 1: break the UI into a component hierarchy

The first thing you'll want to do is to draw boxes around every component (and subcomponent) in the mock and give them all names. If you're working with a designer, they may have already done this, so go talk to them! Their Photoshop layer names may end up being the names of your React components!

But how do you know what should be its own component? Just use the same techniques for deciding if you should create a new function or object. One such technique is the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle), that is, a component should ideally only do one thing. If it ends up growing, it should be decomposed into smaller subcomponents.

Since you're often displaying a JSON data model to a user, you'll find that if your model was built correctly, your UI (and therefore your component structure) will map nicely. That's because UI and data models tend to adhere to the same *information architecture*, which means the work of separating your UI into components is often trivial. Just break it up into components that represent exactly one piece of your data model.

![Component diagram](/react/img/blog/thinking-in-react-components.png)

You'll see here that we have five components in our simple app. I've italicized the data each component represents.

  1. **`FilterableProductTable` (orange):** contains the entirety of the example
  2. **`SearchBar` (blue):** receives all *user input*
  3. **`ProductTable` (green):** displays and filters the *data collection* based on *user input*
  4. **`ProductCategoryRow` (turquoise):** displays a heading for each *category*
  5. **`ProductRow` (red):** displays a row for each *product*

If you look at `ProductTable`, you'll see that the table header (containing the "Name" and "Price" labels) isn't its own component. This is a matter of preference, and there's an argument to be made either way. For this example, I left it as part of `ProductTable` because it is part of rendering the *data collection* which is `ProductTable`'s responsibility. However, if this header grows to be complex (i.e. if we were to add affordances for sorting), it would certainly make sense to make this its own `ProductTableHeader` component.

Now that we've identified the components in our mock, let's arrange them into a hierarchy. This is easy. Components that appear within another component in the mock should appear as a child in the hierarchy:

  * `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
      * `ProductCategoryRow`
      * `ProductRow`

## Step 2: Build a static version in React

<iframe width="100%" height="300" src="https://jsfiddle.net/reactjs/yun1vgqb/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Now that you have your component hierarchy, it's time to implement your app. The easiest way is to build a version that takes your data model and renders the UI but has no interactivity. It's best to decouple these processes because building a static version requires a lot of typing and no thinking, and adding interactivity requires a lot of thinking and not a lot of typing. We'll see why.

To build a static version of your app that renders your data model, you'll want to build components that reuse other components and pass data using *props*. *props* are a way of passing data from parent to child. If you're familiar with the concept of *state*, **try to refrain from using state** when building this static version. State is reserved for component behavior that changes either by the client ("user input"), over time (`setInterval`, `setTimeout`), or whatever quality of the component that may alter over the duration of its lifecycle, otherwise called the ["component lifecycle"](https://facebook.github.io/react/docs/component-specs.html). Keyword: state is data that changes _during_ the lifecycle.

You can build top-down or bottom-up. That is, you can either start with building the components higher up in the hierarchy (i.e. starting with `FilterableProductTable`) or with the ones lower in it (`ProductRow`). In simpler examples, it's usually easier to go top-down, and on larger projects, it's easier to go bottom-up and write tests as you build.

At the end of this step, you'll have a library of reusable components that render your data model. The components will only have `render()` methods since this is a static version of your app. The component at the top of the hierarchy (`FilterableProductTable`) will take your data model as a prop. If you make a change to your underlying data model and call `ReactDOM.render()` again, the UI will be updated. It's easy to see how your UI is updated and where to make changes since there's nothing complicated going on. React's **one-way data flow** (also called *one-way binding*) keeps everything modular and fast.

Simply refer to the [React docs](/react/docs/) if you need help executing this step.

### A brief interlude: props vs state

There are two types of "model" data in React: props and state. It's important to understand the distinction between the two; skim [the official React docs](/react/docs/interactivity-and-dynamic-uis.html) if you aren't sure what the difference is.

## Step 3: Identify the minimal (but complete) representation of UI state

To make your UI interactive, you need to be able to trigger changes to your underlying data model. React makes this easy with **state**.

To build your app correctly, you first need to think of the minimal set of mutable state that your app needs. The key here is DRY: *Don't Repeat Yourself*. Figure out what the absolute minimal representation of the state of your application needs to be and compute everything else you need on-demand. For example, if you're building a TODO list, just keep an array of the TODO items around; don't keep a separate state variable for the count. Instead, when you want to render the TODO count, simply take the length of the TODO items array.

Think of all of the pieces of data in our example application. We have:

  a) The original list of products
  b) The search text the user has entered
  c) The value of the checkbox
  d) The filtered list of products

Let's go through each one and figure out which one is state. Simply ask three questions about each piece of data:

  1. Is it passed in from a parent via `this.props`? If so, it's props! Duh.
  2. Can you derive it from its props? Props!
  3. Does it change over the duration of the component lifecycle? This is state.

The a)list of products is passed in as props, so that's obviously not state. d)The filtered list of products isn't state, because you compute it by combining the original list of products with the user's search string. And finally, b)the search text and c)checkbox are state since they change _specifically_ over the duration of the lifecycle.

Here's our app's state:

  * The search text the user has entered
  * The value of the checkbox

## Step 4: Identify the component that owns the state

<iframe width="100%" height="300" src="https://jsfiddle.net/reactjs/zafjbw1e/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

OK, so we've identified what the minimal app state is. Next, we need to identify which *owns* this state. Wait... own?

Let's run through this strategy for our application:

  * `ProductTable` needs the state to filter the products and `SearchBar` needs the state to display the current user's search string and whether the checkbox is checked.
  * `FilterableProductTable` contains both of those components
  * Therefore, the owner is `FilterableProductTable`

*Note:* A good rule of thumb: designate the parent component of all the state-using child components to be the "owner."

As you develop more robust React apps over time, you'll come across conventions like "container components" or "smart and dumb" components. In short, most React architectures will implement a bunch of "smart" components that have the primary task overseeing their particular domain of state and propogating that state to it's components.

Cool, so we've decided that our state lives in `FilterableProductTable`. First, add a `getInitialState()` method to `FilterableProductTable` that returns `{filterText: '', inStockOnly: false}` to reflect the initial state of your application. Then, pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as a prop. Finally, use these props to filter the rows in `ProductTable` and set the values of the form fields in `SearchBar`.

You can start seeing how your application will behave: set `filterText` to `"ball"` and refresh your app. You'll see that the data table is updated correctly.

## Step 5: Add inverse data flow

<iframe width="100%" height="300" src="https://jsfiddle.net/reactjs/n47gckhr/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

So far, we've built an app that renders correctly as a function of props and state flowing down the hierarchy. Now it's time to support data flowing the other way: the form components deep in the hierarchy need to update the state in `FilterableProductTable`.

React makes this data flow explicit to make it easy to understand how your program works, but it does require a little more typing than traditional two-way data binding. React provides an add-on called `ReactLink` to make this pattern as convenient as two-way binding, but for the purpose of this post, we'll keep everything explicit.

If you try to type or check the box in the current version of the example, you'll see that React ignores your input. This is intentional, as we've set the `value` prop of the `input` to always be equal to the `state` passed in from `FilterableProductTable`.

Let's think about what we want to happen. We want to make sure that whenever the user changes the form, we update the state to reflect the user input. Since components should only update their own state, `FilterableProductTable` will pass a callback to `SearchBar` that will fire whenever the state should be updated. We can use the `onChange` event on the inputs to be notified of it. And the callback passed by `FilterableProductTable` will call `setState()`, and the app will be updated.

Though this sounds complex, it's really just a few lines of code. And it's really explicit how your data is flowing throughout the app.

## And that's it

Hopefully, this gives you an idea of how to think about building components and applications with React. While it may be a little more typing than you're used to, remember that code is read far more than it's written, and it's extremely easy to read this modular, explicit code. As you start to build large libraries of components, you'll appreciate this explicitness and modularity, and with code reuse, your lines of code will start to shrink. :)
