  
# Gesture DSL
  
  
## When
  
  
All gesture definition starts from `When()` clause, representing the condition for the activation of a gesture. And also `When()` clause is the first context of a gesture.
```cs
var Chrome = When(ctx =>
{
    return ctx.ForegroundWindow.ModuleName == "chrome.exe";
});
```
  
_Note: `ctx` is EvaluationContext, see [Core API - EvaluationContext](#evaluationcontext ) for more details._
  
The next to `When()` are `On()` and `OnDecomposed()` clauses.
  
## On
  
  
`On()` clause takes a button or a sequence of stroke as it's argument. This clause can be declared successively if you needed. So, `On()` clause is the second or later context of a gesture. 
  
```cs
// Button gesture.
Chrome.
On(Keys.RButton). // If you press mouse's right button,
```
  
```cs
// Button gesture with two buttons.
Chrome.
On(Keys.RButton). // If you press mouse's right button,
On(Keys.LButton). // and press mouse's left button,
```
  
```cs
// Storke gesture.
Chrome.
On(Keys.RButton). // If you press mouse's right button,
On(Keys.MoveUp, Keys.MoveDown). // and draw stroke to up and to down by the pointer,
```
  
Other than itself, this clause takes `Press()`, `Do()`, and `Release()` clauses as the next clause. 
  
```cs
Chrome.
On(Keys.RButton).
Press(ctx => {});
```
```cs
Chrome.
On(Keys.RButton).
Do(ctx => {});
```
```cs
Chrome.
On(Keys.RButton).
Release(ctx => {});
```
  
And also, these clauses are can be declared at the same time.
  
```cs
Chrome.
On(Keys.RButton).
Press(ctx => {}).
Do(ctx => {}).
Release(ctx => {});
```
  
##### Grammatical limitations:
  
* `On()` and `OnDecomposed()` clauses given the same button **can not** be declared on the same context. See [OnDecomposed](#ondecomposed ) for more details.
  
## OnDecomposed
  
  
`OnDecomposed()` clause takes a button as It's argument. Like `On()`, `OnDecomposedOn()` clause is the second or later context of a gesture, too. But, in contrast to `On()` clause, this clause **can not** be declared successively, and **can not** take `Do()` clause as the next clause. This clause exists for the cases that you want to simply hook the press and release events to an action. This clause takes `Press()` and `Release()` clauses as the next clause. These clauses will directly be connected to the action, and if a button pressed, each of all the events published while it will invoke it.
  
```cs
Chrome.
OnDecomposed(Keys.RButton). // If you press and hold mouse's right button,
Press(ctx => 
{
    // then this code will be executed, each time the press event will be published.
}). 
Release(ctx => {});
```
  
```cs
// This clause CAN NOT be declared successively.
Chrome.
OnDecomposed(Keys.RButton). 
OnDecomposed(Keys.MButton). // Compilation error.
```
  
```cs
// This clause CAN NOT be declared on the same context with `On` clause given the same button.
Chrome.
On(Keys.RButton).
OnDecomposed(Keys.RButton). // Runtime error will be thrown and warning messsage will be shown.
```
  
  
##### Grammatical limitations:
  
* `OnDecomposed()` clause does not have `Do()` functions.
* `On()` and `OnDecomposed()` clauses given the same button **can not** be declared on the same context.
  
## Do
  
  
`Do()` clause declares an action which will be executed only when the conditions, given by the context it to be declared, are fully filled. `Do()` clause is the last context of a gesture. 
  
```cs
On(Keys.RButton). // If you press mouse's right button,
Do(ctx => // and release mouse's right button,
{
    // then this code will be executed.
});
```
  
_Note: `ctx` is `ExecutionContext`, see [Core API - ExecutionContext](#executioncontext ) for more details._
  
## Press
  
  
`Press()` clause declares an action which will be executed only when the conditions, given by the context it to be declared, are fully filled, except for the last clause's release event. `Press()` clause is the last context of a gesture. 
  
```cs
On(Keys.RButton). // If you press mouse's right button,
Press(ctx => // without waiting for release event,
{
    // then this code will be executed.
});
```
  
_Note: `ctx` is `ExecutionContext`, see [Core API - ExecutionContext](#executioncontext ) for more details._
  
## Release
  
  
`Release()` clause declares an action which will be executed only when the conditions, given by the context it to be declared, are fully filled. `Release()` clause is the last context of a gesture. 
  
```cs
On(Keys.RButton). // If you press mouse's right button,
Release(ctx => // and release mouse's right button,
{
    // then this code will be executed.
});
```
  
_Note: `ctx` is `ExecutionContext`, see [Core API - ExecutionContext](#executioncontext ) for more details._
  