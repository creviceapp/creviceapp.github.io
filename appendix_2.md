
# Appendix 2. How to introduce gesture function to your application

## Introduction to Crevice.Core library

`Crevice.Core` is a library which provides basic gesture functions, based on the codes abstracted from core logic of **CreviceApp** version 3.x. **CreviceApp** version 4.0 or later is using `Crevice.Core` as a library.


### Key
#### DoubleThrowKey and SingleThrowKey

`Crevice.Core` treats two types of **Key**s, abstracted from **double throw type** and **single throw type** keys on the real world. For example, left (or right) button of a mouse device is the former, and up (or down) event of wheel button is the later. 

Standard `DoubleThrowKey` have two events, `PressEvent` and `ReleaseEvent`. 

```wavedrom
{ 
    signal: [
        { name: 'Key', wave: '0h.l',  node: '.a.b'},
        { name: 'PressEvent', wave: '0l..',  node: '.c..' },
        { name: 'ReleaseEvent', wave: '0..l',  node: '...d' },
    ],
    edge: [
        "a->b Pressing",
        "a=>c",
        "b=>d",
    ],
    foot : {
        text: "DoubleThrowKey",
    }
}
```

And `SingleThrowKey` have an only event, `FireEvent`. 

```wavedrom
{ 
    signal: [
        { name: 'Key', wave: '0l',  node: '.a'},
        { name: 'FireEvent', wave: '0l',  node: '.b' },
    ],
    edge: [
        "a=>b",
    ],
    foot : {
        text: "SingleThrowKey",
    }
}
```

It may be seemed strange that an event but a button to be treated as a key, a counterpart of up event of wheel button is not any of the events of it. The counterpart of up event of wheel button is itself; you can think that it to be compressed. If there is no need to distinguish `PressEvent` and `ReleaseEvent`, do not you think that it can be compressed into a `FireEvent` ?

#### SimpleKeySetA and SimpleKeySetB

`Crevice.Core` provides two example classes managing a set of **Key**s as `SimpleKeySetA` and `SimpleKeySetB`. `SimpleKeySetA` corresponds to `DoubleThrowKey` and `SimpleKeySetB` corresponds to `SingleThrowKey`.



---

 three types of events.


`SimpleKeySetA`
`SimpleKeySetB`
`SimpleGestureMachineConfig`
`SimpleRootElement`
`SimpleContextManager`
`SimpleCallbackManager`
`SimpleGestureMachine`

#### Physical and Logical 
## 
