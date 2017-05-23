---
layout: post
title:  Two-way data binding vs. One-way data flow
date:   2017-05-23 14:29:57 +0000
---


In a recent technical interview, I was asked to discuss the difference between two-way data binding and one-way data flow. From what I gather, this has become a fairly common interview question, and beyond that, I think it's just an important concept to understand as a developer.

The issue of one-way vs. two-way data binding is frequently discussed in the context of React vs. Angular, two of the most popular Javascript frameworks currently in use. While there are certainly other examples, these two have the benefit of both being widely used, and providing clear differences in how they handle data binding.

Let's start with Angular, which features two-way data binding. Essentially, this means that input fields and model data are bound together, and will each automatically update as the other is changed. So as a user is typing into an input field, the corresponding model will be updated simultaneously.

This can be achieved using the ng-model directive. Here's an example of a very simple form, starting with the view.

```
<div ng-app="myApp" ng-controller="myController">
    Name: <input ng-model="name">
    <h1>{{name}}</h1>
</div>
```

And then the controller:

```
function MyController($scope) {
	$scope.name = "John";
}
```

Any change to either the model or the input field will immediately update the other.

React handles data binding a bit differently. In the above example, changes from the HTML are directly making changes in the code. This does not happen in React. Of course, we can still receive user input and use it to update a component's state, but the input is not *directly* manipulating the state. Rather, the input will trigger an event, which the component can listen for, and then update its state. The view is then re-rendered as necessary based on the new state. In React, a similarly basic form might look something like this:

```
class App extends React.Component {
  constructor() {
    super();
 
    this.handleChange = this.handleChange.bind(this);
 
    this.state = {
      value: '',
    };
  }
 
  handleChange(event) {
    this.setState({
      value: event.target.value,
    });
  }
 
  render() {
    return (
      <input type="text" value={this.state.value} onChange={this.handleChange} />
    );
  }
}
```
Here, we're taking the onChange event from the input, and passing it into the handleChange function in the component, where the actual change to the model occurs. Although the signal to make the change initiated from the view, the component itself is doing the work. This is why the state is sometimes referred to as the "single source of truth." Visually, this kind of unidirectional data flow can be imagined like this:


![](http://i.imgur.com/hJmGMqu.jpg)

While two-way data binding as in Angular would look more like this:

![](http://i.imgur.com/cPYiWsZ.jpg)

This is a simplified look at the issue, and it's entirely possible to achieve one way data flow in Angular and two-way data binding in React, but out of the box at least, this is one of the primary differences. There is plenty of debate about the relative merits of each method which I won't wade into here, but below are a couple of good resources for further reading.

[Two-Way Data Binding: Angular 2 and React](https://www.accelebrate.com/blog/two-way-data-binding-angular-2-and-react/)

[The Case for Unidirectional Data Flow](https://www.exclamationlabs.com/blog/the-case-for-unidirectional-data-flow/)

[Databinding, from the Angular Documentation](https://docs.angularjs.org/guide/databinding)

