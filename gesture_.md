  
# Gesture
  
  
## Button gesture
  
As you may know, mouse gestures with it's buttons are called "rocker gesture" in mouse gesture utility communities. But we will call it, including gestures with keyboard keys, as **Button gesture**, **Key gesture**, or simply **Gesture** here. 
  
```cs
// Button gesture.
Chrome.
On(Keys.RButton). // If you press mouse's right button,
Do(ctx => // and release mouse's right button,
{
    // then this code will be executed.
});
```
  
```cs
// Button gesture with two mouse buttons.
Chrome.
On(Keys.RButton). // If you press mouse's right button,
On(Keys.LButton). // and press mouse's left button,
Do(ctx => // and release mouse's left or right button,
{
    // then this code will be executed.
});
```
  
Even if after pressing a button or a key which means the start of a gesture, you can cancel it by holding and pressing it until it to be timeout.
  
```cs
Chrome.
On(Keys.RButton). // If you WRONGLY pressed mouse's right button,
Do(ctx => // you hold the button until it to be timeout and release it,
{
    // then this code will NOT be executed.
});
```
  
This means actions declared in `Do()` clause is not assured it's execution.
  
Above three gestures are **Button gesture** by *the standard (double throw) buttons*. `On()` clause with standard buttons can be used for declare `Do()` clause but also `Press()` and `Release()` clauses.
  
_Note: See [Appendix 2. - Key](#key ) for the details about the type of keys (single throw and double throw)._
  
## Button gesture with Press() / Release()
  
  
`Do()` clause fits to many cases, but there are cases do not fit to. For example, where there is need to hook to the press or release event of a button or a key. `Press()` and `Release()` clauses fit to this case. These can be written just after `On()` clause.
  
```cs
// Convert Keys.XButton1 to Keys.LWin.
Chrome.
On(Keys.XButton1).
Press(ctx =>
{
    SendInput.ExtendedKeyDown(Keys.LWin);
}).
Release(ctx =>
{
    SendInput.ExtendedKeyUp(Keys.LWin);
});
```
  
For `Release()` clause, it can be after `Do()` clause.
  
```cs
Chrome.
On(Keys.XButton2).
Press(ctx =>
{
    // Assured.
}).
Do(ctx =>
{
    // Not assured. 
    // e.g. When the gesture to be timeout,
    //      this action will not be executed.
}).
Release(ctx =>
{
    // Assured.
});
```
  
Actions declared in `Press()` and `Release()` clauses are different from it of `Do()` clause, the execution of these are assured.
  
_Note: Be careful that this conversion is incomplete. See [Convert a button or a key into an arbitrary one](#convert-a-button-or-a-key-into-an-arbitrary-one ) for more details._
  
## Button gesture with single throw button
  
  
Few of the buttons in `Keys`: `WheelUp`, `WheelDown`, `WheelLeft`, and `WheelRight`, are different from the standard ones. These are *double throw*, have only one state and only one event. So, `On()` clauses with these can not be used with `Press()` and `Release()` clauses.
  
```cs
Chrome.
On(Keys.WheelUp).
Press(ctx => { }); // Compilation error
```
  
```cs
Chrome.
On(Keys.WheelUp).
Do(ctx => { }); // OK
```
  
```cs
Chrome.
On(Keys.WheelUp).
Release(ctx => { }); // Compilation error
```
  
_Note: See [Appendix 2. - Key](#key ) for the details about the type of keys (single throw and double throw)._
  
##### Grammatical limitations:
  
* `On()` clause with single state button does not have `Press()` and `Release()` functions.
  
Single state buttons are `Keys.WheelUp`,  `Keys.WheelDown`,  `Keys.WheelLeft`, and  `Keys.WheelRight`.
  
## Stroke gesture
  
  
"Mouse gestures by strokes", namely **Stroke gesture** is the most important part in the functions of mouse gesture utilities.
  
`On()` clause takes arguments that consist of combination of `Keys.MoveUp`, `Keys.MoveDown`, `Keys.MoveLeft` and `Keys.MoveRight`. These are representing directions of movements of the mouse pointer.
  
```cs
Chrome.
On(Keys.RButton). // If you press right button,
On(Keys.MoveDown, Keys.MoveRight). // and draw stroke to down and to right by the pointer,
Do(ctx => // and release right button,
{
    SendInput.Multiple().
    ExtendedKeyDown(Keys.ControlKey).
    ExtendedKeyDown(Keys.W).
    ExtendedKeyUp(Keys.W).
    ExtendedKeyUp(Keys.ControlKey).
    Send(); // then send Ctrl+W to Chrome.
});
```
  
**Stroke gesture** represents special case when a standard button is pressed, so it have the same grammatical limitation to **Button gesture with single throw button**.
  
  
```cs
Chrome.
On(Keys.RButton).
On(Keys.MoveDown).
Press(ctx => { }); // Compilation error
```
  
```cs
Chrome.
On(Keys.RButton).
On(Keys.MoveDown).
Do(ctx => { }); // OK
```
  
```cs
Chrome.
On(Keys.RButton).
On(Keys.MoveDown).
Release(ctx => { }); // Compilation error
```
  
  
##### Grammatical limitations:
  
* `On()` clause with `Keys.Move*` does not have `Press()` and `Release()` functions.
* `On()` clause with `Keys.Move*` should have `On()` caluse with standard (double throw) button as the previous context.
* `On()` clause with `Keys.Move*` should be the last element of the sequence of `On()` clauses.
  