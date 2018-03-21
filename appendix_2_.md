  
# Appendix 2. How to introduce gesture function to your application
  
  
## Introduction
  
  
`CreviceLib` is a library which provides basic gesture functions, based on the codes abstracted from core logic of **CreviceApp** 3.x. **CreviceApp** 4.0 or later is using `CreviceLib` as a library. So if you want, you can easily introduce gesture function to your application by adding a reference to `CreviceLib` and following this guidance.
  
_Note: `CreviceLib` is distributed as a nuget package. Visit [NuGet Gallery \| CreviceLib](https://www.nuget.org/packages/Crevice.Core/ ) for more details._
  
  
## Quickstart
  
  
At first, check if the reference to `CreviceLib` is certainly added to your project.
  
The very simple setup code is the following:
  
```cs
using Crevice.Core.Example;
  
var keys = new SimpleKeySetA(maxSize: 10);
var root = new SimpleRootElement();
var gm = new SimpleGestureMachine();
```
  
  
  
Then, you can be able to start writing gesture DSL.
  
```cs
var whenever = root.When(ctx => {
    return true;
});
var action = whenever.On(keys[0]);
action.Do(ctx => {
    // When PressEvent and ReleaseEvent of KeySetA[0] are given to GestureMachine,
    // then this code will be executed.
});
```
  
After writing the gesture DSL, you can now run `GestureMachine`.
  
```cs
gm.Run(root);
```
  
And finally, you should connect user input to `GestureMachine`.
  
```cs
// If the following events are input to `GestureMachine`,
gm.Input(keys[0].PressEvent);
gm.Input(keys[0].ReleaseEvent);
// then the action will be executed here.
```
  
  
## Key
  
  
`CreviceLib` treats two types of key, **KeyA** and **KeyB**, which abstracted from **type A (double throw)** and **type B (single throw)** keys on the real world. For example, any key of a keyboard device is the former, and up (or down) event of wheel button of a mouse device is the later. We do not need to think about the case where we need another type of key, for the peace of mind.
  
![Fig1. Type A (double throw) key.](./images/ap2.fig1.jpg )
<div class="img-caption">Fig1. Type A (double throw) key.</div>
  
![Fig2. Type B (single throw) key.](./images/ap2.fig2.jpg )
<div class="img-caption">Fig2. Type B (single throw) key.</div>
  
![Fig3. A bizarre, unspeakable key.](./images/ap2.fig3.jpg )
<div class="img-caption">Fig3. A bizarre, unspeakable key.</div>
  
**KeyA** which occupies most of use cases, have two events, `PressEvent` and `ReleaseEvent`. 
  
```wavedrom
{ 
    signal: [
        { name: 'KeyA', wave: '0h.l',  node: '.a.b'},
        { name: 'PressEvent', wave: '0l..',  node: '.c..' },
        { name: 'ReleaseEvent', wave: '0..l',  node: '...d' },
    ],
    edge: [
        "a->b Pressing",
        "a=c",
        "b=d",
    ]
}
```
  
**KeyB** is used only in a few cases, have an only event, `FireEvent`. 
  
```wavedrom
{ 
    signal: [
        { name: 'KeyB', wave: '0l',  node: '.a'},
        { name: 'FireEvent', wave: '0l',  node: '.b' },
    ],
    edge: [
        "a=b",
    ]
}
```
  
### The difference between KeyA and KeyB
  
  
It may be seemed strange that an event be treated as a key, but a counterpart of up event of wheel button is not any of the other events which belongs to it. The counterpart of the up event is itself; you can think that it to be compressed. If there is no need to distinguish `PressEvent` and `ReleaseEvent` of a **KeyA**, do not you think that it can be compressed into a **KeyB**?
  
```wavedrom
{ 
    signal: [
        { name: 'KeyA', wave: '0h.l',  node: '.a.b'},
        { name: 'PressEvent', wave: '0l..',  node: '.c..' },
        { name: 'ReleaseEvent', wave: '0..l',  node: '...d' },
        {},
        { name: 'KeyB', wave: '0l..',  node: '.e'},
        { name: 'FireEvent', wave: '0l..',  node: '.f' },
    ],
    edge: [
        "b->e Compress",
        "d->f Compress",
    ]
}
```
  
## KeySet
  
  
`CreviceLib` provides **KeySet** classes managing a set of sequential keys. `SimpleKeySetA` corresponds to `DoubleThrowKey` and `SimpleKeySetB` corresponds to `SingleThrowKey`. These can be used in a simply way, only take an argument `maxSize` which means the maximum size of the sequential key set.
  
```cs
var keySetA = new SimpleKeySetA(maxSize: 10);
Assert.AreEqual(keySetA is PhysicalDoubleThrowKeySet, true);
Assert.AreEqual(keySetA[0] is PhysicalDoubleThrowKey, true);
Assert.AreEqual(keySetA[0].PressEvent is PressEvent, true);
Assert.AreEqual(keySetA[0].ReleaseEvent is ReleaseEvent, true);
```
  
```cs
var keySetB = new SimpleKeySetB(maxSize: 10);
Assert.AreEqual(keySetB is PhysicalSingleThrowKeySet, true);
Assert.AreEqual(keySetB[0] is PhysicalSingleThrowKey, true);
Assert.AreEqual(keySetB[0].FireEvent is FireEvent, true);
```
  
_Note: Regarding the adjective **Physical** commonly held by both names of the type of `SimpleKeySetA` and `SimpleKeySetB`, see [Physical and logical event types](#physical_and_logical_event_types ) for more details._
  
## GestureMachineConfig
  
  
`GestureMachineConfig` holds configuration values for `GestureMachine`. The configuration values are so mutable that you can edit it if you want any time. Also you can use newly customized `GestureMachineConfig` by inheriting it.
  
`SimpleGestureMachineConfig` is a example class, which inherits `GestureMachineConfig` and do not have any change to it.
  
```cs
var config = new SimpleGestureMachineConfig();
config.GestureTimeout = 0; // Set GestureMachine to never timeout.
```
  
## RootElement
  
  
`RootElement<EvaluationContext, ExecutionContext>` is the root element of the tree of gesture DSL. You can start definition of your gestures with `When()` function.
  
`SimpleRootElement` is a class which is simplified about it's generics types.
  
```cs
var root = new SimpleRootElement();
var whenever = root.When(ctx => {
    return true;
});
whenever
.On(KeySetA[0]) 
.Do(ctx => {
    // When PressEvent and ReleaseEvent of KeySetA[0] are given to GestureMachine,
    // then this code will be executed.
});
```
  
## ContextManager
  
  
`ContextManager<EvaluationContext, ExecutionContext>` manages `ctx` in the functions like `When()`, or `Do()` on gesture DSL. If you want to change the initialization of `EvaluationContext` or `ExecutionContext`, you can do it with this class.
  
`SimpleContextManager` is a class which is simplified about it's generics types and overrided the default initializer for `EvaluationContext` and `ExecutionContext` on `CreateEvaluateContext()` and `CreateExecutionContext()`. 
  
_Note: You should override `CreateEvaluateContext()` and `CreateExecutionContext()` if you create a class inherits ContextManager._
  
## CallbackManager
  
  
`CallbackManager<GestureMachineConfig, ContextManager, EvaluationContext, ExecutionContext>` manages callbacks of `GestureMachine`. 
  
`SimpleCallbackManager` is a class which is simplified about it's generics types.
  
## GestureMachine
  
  
`GestureMachine<GestureMachineConfig, ContextManager, EvaluationContext, ExecutionContext>` is the main component of `CreviceLib`. 
  
`SimpleGestureMachine` is a class which is simplified about it's generics types.
  
## Physical and logical event types
  
  
`CreviceLib` supports physical and logical event types, for the abstraction of multiple input devices. In contrast to `Input()` function of `GestureMachine` which only takes physical event, `On()` function in gesture DSL takes both physical and logical events. In case a logical event be given to `On()` as the arguement, and a physical event corresponding to it be given to `Input()`, `GestureMachine` will treat it correctly in their relationship on physical and logical event types.
  