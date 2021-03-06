  
```force_js
(function(){
    WaveDrom.ProcessAll();
})();
```
  
  
  
## Installation
  
  
  
### Install Crevice 4 From Windows Store
  
  
  
Crevice 4 has been released on Windows Store. You can now get Crevice more easily!
  
[Get Crevice 4 \- Microsoft Store](https://www.microsoft.com/en-us/store/p/crevice-4/9nzxvrj73xj0 )
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/2dyD36-TMog" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  
  
### Install Crevice 4 From GitHub
  
  
  
[Releases · creviceapp/creviceapp](https://github.com/creviceapp/creviceapp/releases )
  
Extract zip file to any location, and click `crevice4.exe`.
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/PQg8h_6h62w" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  
  
  
  
## Quickstart
  
  
  
After the first execution of the application, you can find `default.csx` in the directory `%APPDATA%\Crevice4`. It's the userscript file. Please open it with a text editor and take a look through it.
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/VcUqzj0K9lY" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  
After several `using` declaring lines, you can see `Browser` definition as following (but here, a little bit shortened):
  
```cs
var Browser = When(ctx =>
{
    return ctx.ForegroundWindow.ModuleName == "chrome.exe" ||
           ctx.ForegroundWindow.ModuleName == "firefox.exe" ||
           ctx.ForegroundWindow.ModuleName == "iexplore.exe");
});
```
  
When the `ModuleName` of `ForegroundWindow` is one of follows, `chrome.exe`, `firefox.exe`, or `iexplore.exe`, then `When` returns true; this is the declaration of initialization of a context which specialized to `Browser`. 
  
After that, the declaration of gestures follows. Let's see the first one:
  
```cs
Browser.
On(Keys.RButton).
On(Keys.WheelUp).
Do(ctx =>
{
    SendInput.Multiple().
    ExtendedKeyDown(Keys.ControlKey).
    ExtendedKeyDown(Keys.ShiftKey).
    ExtendedKeyDown(Keys.Tab).
    ExtendedKeyUp(Keys.Tab).
    ExtendedKeyUp(Keys.ShiftKey).
    ExtendedKeyUp(Keys.ControlKey).
    Send(); // Previous tab
});
```
  
This is a mouse gesture definition; when you press and hold `Keys.RButton`, and then if you `Keys.WheelUp` it, the actions declared in `Do` will be executed.
  
As long as Crevice4 is executing, you can edit userscript file at any time. Of cause it's ok even if while you are reading the following sections. Crevice4 supports **hotloading** feature. Whenever Crevice4 detects an update of the userscript file, it will be compiled and evaluated immediately, then it will be loaded if the compilation is successful. 
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/NDZc8hArVd8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  
The userscript file is just a C# Scripting file. You can do anything you want by writing your own script in it, or else by just copying codes from [Stack Overflow](https://stackoverflow.com/ ). See [Overview of C# Scripting](#overview-of-c-scripting ) for more details.
  
  
  
  
# Gesture
  
  
  
## Button gesture
  
  
As you may know, mouse gestures with it's buttons are called "rocker gesture" in mouse gesture utility communities. But we will call it, including gestures with keyboard keys, as **Button gesture**, **Key gesture**, or simply **Gesture** here. 
  
```cs
// Button gesture.
Browser.
On(Keys.RButton). // If you press mouse's right button,
Do(ctx => // and release mouse's right button,
{
    // then this code will be executed.
});
```
  
```cs
// Button gesture with two mouse buttons.
Browser.
On(Keys.RButton). // If you press mouse's right button,
On(Keys.LButton). // and press mouse's left button,
Do(ctx => // and release mouse's left or right button,
{
    // then this code will be executed.
});
```
  
Even if after pressing a button or a key which means the start of a gesture, you can cancel it by holding and pressing it until it to be timeout.
  
```cs
Browser.
On(Keys.RButton). // If you WRONGLY pressed mouse's right button,
Do(ctx => // you hold the button until it to be timeout and release it,
{
    // then this code will NOT be executed.
});
```
  
This means actions declared in `Do()` clause is not assured it's execution.
  
Above three gestures are **Button gesture** by standard (*double throw*) buttons. `On()` clause with standard buttons can be used for declare `Do()` clause but also `Press()` and `Release()` clauses.
  
_Note: See [Appendix 2. - Key](#key ) for the details about the type of keys (single throw and double throw)._
  
## Button gesture with Press() / Release()
  
  
  
`Do()` clause fits to many cases, but there are cases do not fit to. For example, where there is need to hook to the press or release event of a button or a key. `Press()` and `Release()` clauses fit to this case. These can be written just after `On()` clause.
  
```cs
Browser.
On(Keys.RButton).
Press(ctx => // If you press mouse's right button,
{
    // then this code will be executed.
}).
Release(ctx => // And release it,
{
    // then this code will be executed.
});
```
  
For `Release()` clause, it can be after `Do()` clause. 
  
```cs
Browser.
On(Keys.RButton).
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
  
_Note: Be careful that there are cases which Inappropriate when you use this for conversion of buttons or keys as is. See [Convert a button or a key into an arbitrary one](#convert-a-button-or-a-key-into-an-arbitrary-one ) for more details._
  
## Button gesture with single throw button
  
  
  
Few of the buttons in `Keys`: `WheelUp`, `WheelDown`, `WheelLeft`, and `WheelRight`, are different from the standard ones. These are *single throw* buttons, have only one state and only one event. So, `On()` clauses with these can not be used with `Press()` and `Release()` clauses.
  
```cs
Browser.
On(Keys.WheelUp).
Press(ctx => { }); // Compilation error
```
  
```cs
Browser.
On(Keys.WheelUp).
Do(ctx => { }); // OK
```
  
```cs
Browser.
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
Browser.
On(Keys.RButton). // If you press right button,
On(Keys.MoveDown, Keys.MoveRight). // and draw stroke to down and to right by the pointer,
Do(ctx => // and release right button,
{
    // then this code will be executed.
});
```
  
**Stroke gesture** represents special case when a standard button is pressed, so it have the same grammatical limitation to **Button gesture with single throw button**.
  
  
```cs
Browser.
On(Keys.RButton).
On(Keys.MoveDown).
Press(ctx => { }); // Compilation error
```
  
```cs
Browser.
On(Keys.RButton).
On(Keys.MoveDown).
Do(ctx => { }); // OK
```
  
```cs
Browser.
On(Keys.RButton).
On(Keys.MoveDown).
Release(ctx => { }); // Compilation error
```
  
  
##### Grammatical limitations:
  
  
* `On()` clause with `Keys.Move*` does not have `Press()` and `Release()` functions.
* `On()` clause with `Keys.Move*` should have `On()` caluse with standard (double throw) button as the previous context.
* `On()` clause with `Keys.Move*` should be the last element of the sequence of `On()` clauses.
  
  
  
  
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
OnDecomposed(Keys.RButton). // If you press mouse's right button many times,
Press(ctx => 
{
    // then this code will be executed, each time the PRESS event will be published,
}). 
Release(ctx => {
    // and then this code will be executed, each time the RELEASE event will be published.
});
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
  
  
  
  
# Overview of C# Scripting
  
  
  
C# Scripting is a powerful and flexible scripting feature provided by [Microsoft Roslyn](https://github.com/dotnet/roslyn ). It designed to make scripts you cut and pasted from C# code work as-is. So, you can build your own gesture environment easily just only with a little of knowledge of C# language. But C# Scripting has only a few special feature C# language does not have. The following sections are the introduction of the features very useful if you know it when you need to do it.
  
## #r directive
  
  
  
You can add reference to assemblies by `#r` directive. 
  
```cs
// Add a reference to dll file.
#r "path_to/Assembly.dll"
  
// Add a reference to an assembly.
#r "System.Speech"
  
// Error occurrs unless you have installed Visual Studio.
#r "Microsoft.VisualStudio" 
```
  
_Note 1: This directive should be placed on the top of your C# Scripting code._
_Note 2: This directive does not support to load NuGet package automatically. You can do it by download NuGet package by yourself, extract dll files, and add refereces to it by using `#r` directive._
  
## #load directive
  
  
  
You can load the content in another C# Scripting file by `#load` directive.
  
```cs
#load "path_to/another.csx"
```
  
_Note : This directive should be placed on the top of your C# Scripting code except for `#r` directive._
  
  
## Tutorial of C# Language
  
  
  
There are a lot of good tutorials of C# language that you can pick to learn. The followings are just an example:
  
<a href="https://docs.microsoft.com/en-us/dotnet/csharp/quick-starts/"><img title="C# interactive tutorials - C# quickstarts | Microsoft Docs" src="images/docs.microsoft.com_en-us_dotnet_csharp_quick-starts.png" alt="https://docs.microsoft.com/en-us/dotnet/csharp/quick-starts/" width="400" /></a>
<div class="img-caption"><a href="https://docs.microsoft.com/en-us/dotnet/csharp/quick-starts/">C# interactive tutorials - C# quickstarts | Microsoft Docs</a></div>
  
<a class="img-caption" href="http://www.learncs.org/"><img title="Learn C# - Free Interactive C# Tutorial" src="images/www.learncs.org.png" alt="http://www.learncs.org/" width="400" /></a>
<div class="img-caption"><a href="http://www.learncs.org/">Learn C#</a></div>
  
<a class="img-caption" href="https://www.sololearn.com/Course/CSharp/"><img title="C# Tutorial | SoloLearn: Learn to code for FREE!" src="images/www.sololearn.com_course_csharp.png" alt="https://www.sololearn.com/Course/CSharp/" width="400" /></a>
<div class="img-caption"><a href="https://www.sololearn.com/Course/CSharp/">C# Tutorial | SoloLearn</a></div>
  
<a class="img-caption" href="https://www.tutorialspoint.com/csharp/index.htm"><img title="C# Tutorial" src="images/www.tutorialspoint.com_csharp.png" alt="https://www.tutorialspoint.com/csharp/index.htm" width="400" /></a>
<div class="img-caption"><a href="https://www.tutorialspoint.com/csharp/index.htm">C# Tutorial | TutorialsPoint</a></div>
  
<a class="img-caption" href="https://mva.microsoft.com/en-US/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949"><img title="Learn C# for Beginners – Microsoft Virtual Academy" src="images/mva.microsoft.com_en-us_training-courses.png" alt="https://mva.microsoft.com/en-US/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949" width="400" /></a>
<div class="img-caption"><a href="https://mva.microsoft.com/en-US/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949">Learn C# for Beginners | Microsoft Virtual Academy</a></div>
  
<a class="img-caption" href="https://docs.microsoft.com/en-us/dotnet/csharp/"><img title="C# Guide | Microsoft Docs" src="images/docs.microsoft.com_en-us_dotnet_csharp.png" alt="https://docs.microsoft.com/en-us/dotnet/csharp/" width="400" /></a>
<div class="img-caption"><a href="https://docs.microsoft.com/en-us/dotnet/csharp/">C# Guide | Microsoft Docs</a></div>
  
  
  
  
# Edit user script
  
  
  
Once open your user script with a text editor, you can edit it whenever you like, and the changes given will be reflect immediately as mentioned before with **hot reloading** feature.
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/NDZc8hArVd8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> 
  
## Debugging with Console
  
  
  
Crevice4 supports `--verbose` option which means output detail informations to console: see [Command line interface](#command-line-interface ) for the details.
  
When crevice4 starts with `--verbose` option, invocation of functions like `Console.WriteLine()` in the user script will be activated and the messages will be printed to console.
  
```bat
> crevice4.exe --verbose
```
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/c8bRyaE1iC0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> 
  
  
## Edit user script with Visual Studio Code
  
  
  
Simple text editors can do this, but IDE which supports IntelliSense is very helpful. Cevice4 provides IDE integrate feature. 
  
[Visual Studio Code](https://code.visualstudio.com/ ) and [C\# Plugin](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp ) are required. 
  
And also, `IDESupport` files must be placed in Crevice 4 userscript directory. The way to setup it is simple, extract Crevice 4 zip archive distributed on GitHub as the name `IDESupport` and then, place it in Crevice 4 userscript directory.
  
The following instruction movie includes the way to setup `IDESupport` files.
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/VkRK_w7ad80" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  
After the setup, you can edit Crevice 4 user script with Visual Studio Code.
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/4mK8CgCwSFw" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  
  
  
  
  
  
# Practical example
  
  
  
## Send keystrokes
  
  
  
The following code input the message to the foreground application.
  
```cs
Do(ctx =>
{
    SendInput.UnicodeKeyStroke("This text will be input.");
});
```
  
But `UnicodeKeyStroke` method is slow. So if there is no need to use it, you should better to do it with `Clipboard` and Ctrl+V method. See [Paste text message with clipboard](#paste-text-message-with-clipboard ).
  
## Paste text message with clipboard
  
  
  
```cs
using System.Windows.Forms;
```
  
```cs
Do(ctx =>
{
    Clipboard.SetText("This text will be pasted.");
    SendInput.Multiple().
    ExtendedKeyDown(Keys.ControlKey).
    ExtendedKeyDown(Keys.V).
    ExtendedKeyUp(Keys.V).
    ExtendedKeyUp(Keys.ControlKey).
    Send(); // Ctrl+V
});
```
_Note: To use `Clipboard`, you should declare loading namespace `System.Windows.Forms` by **using** statement._
  
## Convert a button or a key into an arbitrary one
  
  
  
You can use `Keys.XButton1` as `Keys.Lwin` by the following code.
  
```cs
// Convert Keys.XButton1 to Keys.LWin.
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
  
But be careful that this conversion is incomplete. You should use `OnDecomposed` instead of `On` clause in case you need to support the repeat function which keyboard's keys possess.
  
The following code supports conversion of repeated press event of `Keys.XButton1` into it of `Keys.LWin`.
  
```cs
OnDecomposed(Keys.XButton1).
Press(ctx =>
{
    SendInput.ExtendedKeyDown(Keys.LWin);
}).
Release(ctx =>
{
    SendInput.ExtendedKeyUp(Keys.LWin);
});
```
  
## Create global gesture
  
  
  
If you need to declare **global** gesture, you can do it by simply declare `When` clause which always returns true.
  
```cs
Whenever = When(ctx => { return true });
```
  
This declaration always is active, returns true, but has one big problem. This can be shadowed by the other gesture. To make this problem solved, you need to add some lines:
  
```cs
DeclareProfile("Whenever"); 
  
Whenever = When(ctx => { return true });
  
// ...
  
DeclareProfile("Other"); 
  
// ...
```
  
See [Profile](#profile ) for more details.
  
## Refer the definition of `When` from another `When`
  
  
  
`When()` returns an instance of [WhenElement](https://github.com/creviceapp/creviceapp/blob/51efbd2f9a62717956cf42aedfbed625aa3e13ae/CreviceLib/Core.DSL.cs#L61 ) which holds a function which given when it was declared. The value is read only and public. So, it is accsesible from another `WhenElement`.
  
In the following example, [WhenElement.WhenEvaluator](https://github.com/creviceapp/creviceapp/blob/51efbd2f9a62717956cf42aedfbed625aa3e13ae/CreviceLib/Core.DSL.cs#L76 ) is referenced.
  
```cs
var Browser = When(ctx =>
{
    return ctx.ForegroundWindow.ModuleName == "chrome.exe" ||
           ctx.ForegroundWindow.ModuleName == "firefox.exe" ||
           ctx.ForegroundWindow.ModuleName == "opera.exe" ||
           ctx.ForegroundWindow.ModuleName == "iexplore.exe";
});
  
var WheneverExceptForBrowser = When(ctx =>
{
    return !Browser.WhenEvaluator(ctx);
});
```
  
##### Warning:
  
  
Application will hang up if `WhenElement.WhenEvaluator` loops. So be carefull when use it.
  
## Use Win32 API
  
  
  
You can use Win32 APIs simply importing it.
  
```cs
[DllImport("winmm.dll")]
public static extern uint timeGetTime();
  
uint lastExecutionTime = 0;
```
  
```cs
Do(ctx => 
{
    var current = timeGetTime();
    Console.WriteLine($"Time passed from last execution: {lastExecutionTime - current}");
    lastExecutionTime = current;
});
```
  
##### References:
  
  
 [DllImportAttribute Class (System.Runtime.InteropServices)](https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.dllimportattribute%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396 ) 
[pinvoke.net: the interop wiki!](https://www.pinvoke.net/ )
  
## Find out the target application / window
  
  
  
The following userscript shows the way to dig into target application / window:
  
```cs
using System;
  
var Whenever = When(ctx =>
{
    return true;
});
  
Whenever.
On(Keys.MButton).
Do(ctx =>
{
    Console.WriteLine($"ForegroundWindow.ThreadId: 0x{ctx.ForegroundWindow.ThreadId:X}");
    Console.WriteLine($"ForegroundWindow.ProcessId: 0x{ctx.ForegroundWindow.ProcessId:X}");
    Console.WriteLine($"ForegroundWindow.Text: {ctx.ForegroundWindow.Text}");
    Console.WriteLine($"ForegroundWindow.ClassName: {ctx.ForegroundWindow.ClassName}");
    Console.WriteLine($"ForegroundWindow.ModulePath: {ctx.ForegroundWindow.ModulePath}");
    Console.WriteLine($"ForegroundWindow.ModuleName: {ctx.ForegroundWindow.ModuleName}");
  
    Console.WriteLine($"PointedWindow.ThreadId: 0x{ctx.PointedWindow.ThreadId:X}");
    Console.WriteLine($"PointedWindow.ProcessId: 0x{ctx.PointedWindow.ProcessId:X}");
    Console.WriteLine($"PointedWindow.Text: {ctx.ForegroundWindow.Text}");
    Console.WriteLine($"PointedWindow.ClassName: {ctx.PointedWindow.ClassName}");
    Console.WriteLine($"PointedWindow.ModulePath: {ctx.PointedWindow.ModulePath}");
    Console.WriteLine($"PointedWindow.ModuleName: {ctx.PointedWindow.ModuleName}");
});
  
```
  
If you using an application which is at the foreground. For example, it is Chrome, a browser. 
  
Activate the above userscript, and then press the middle button. The following messages will be output:
  
```
ForegroundWindow.ThreadId: 0xB20
ForegroundWindow.ProcessId: 0x1058
ForegroundWindow.Text: Crevice Documentation - Google Chrome
ForegroundWindow.ClassName: Chrome_WidgetWin_1
ForegroundWindow.ModulePath: C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
ForegroundWindow.ModuleName: chrome.exe
PointedWindow.ThreadId: 0xB20
PointedWindow.ProcessId: 0x1058
PointedWindow.Text: Crevice Documentation - Google Chrome
PointedWindow.ClassName: Chrome_RenderWidgetHostHWND
PointedWindow.ModulePath: C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
PointedWindow.ModuleName: chrome.exe
```
  
Next, if you move the cursor and focus the Windows' taskbar, and press the middle button, then, the following messages will be output:
  
```
ForegroundWindow.ThreadId: 0xB20
ForegroundWindow.ProcessId: 0x1058
ForegroundWindow.Text: Crevice Documentation - Google Chrome
ForegroundWindow.ClassName: Chrome_WidgetWin_1
ForegroundWindow.ModulePath: C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
ForegroundWindow.ModuleName: chrome.exe
PointedWindow.ThreadId: 0x1DF8
PointedWindow.ProcessId: 0x1D2C
PointedWindow.Text: Crevice Documentation - Google Chrome
PointedWindow.ClassName: MSTaskListWClass
PointedWindow.ModulePath: C:\Windows\explorer.exe
PointedWindow.ModuleName: explorer.exe
```
  
`ForegroundWindow` and `PointedWindow` are `WindowInfo`, so these have more powerful functions like `GetChildWindows()`.
  
```cs
  
using System;
  
var Whenever = When(ctx =>
{
    return true;
});
  
Whenever.
On(Keys.MButton).
Do(ctx =>
{
    Console.WriteLine($"ForegroundWindow.ThreadId: 0x{ctx.ForegroundWindow.ThreadId:X}");
    Console.WriteLine($"ForegroundWindow.ProcessId: 0x{ctx.ForegroundWindow.ProcessId:X}");
    Console.WriteLine($"ForegroundWindow.Text: {ctx.ForegroundWindow.Text}");
    Console.WriteLine($"ForegroundWindow.ClassName: {ctx.ForegroundWindow.ClassName}");
    Console.WriteLine($"ForegroundWindow.ModulePath: {ctx.ForegroundWindow.ModulePath}");
    Console.WriteLine($"ForegroundWindow.ModuleName: {ctx.ForegroundWindow.ModuleName}");
  
    var children = ctx.ForegroundWindow.GetChildWindows();
    for (var i = 0; i < children.Count; i++)
    {
        Console.WriteLine($"ForegroundWindow.Children[{i}].ThreadId: 0x{children[i].ThreadId:X}");
        Console.WriteLine($"ForegroundWindow.Children[{i}].ProcessId: 0x{children[i].ProcessId:X}");
        Console.WriteLine($"ForegroundWindow.Children[{i}].Text: {children[i].Text}");
        Console.WriteLine($"ForegroundWindow.Children[{i}].ClassName: {children[i].ClassName}");
        Console.WriteLine($"ForegroundWindow.Children[{i}].ModulePath: {children[i].ModulePath}");
        Console.WriteLine($"ForegroundWindow.Children[{i}].ModuleName: {children[i].ModuleName}");
    }
});
```
  
The following logs are the dump data of Chrome and Explorer:
  
```
ForegroundWindow.ThreadId: 0xB20
ForegroundWindow.ProcessId: 0x1058
ForegroundWindow.Text: Crevice Documentation - Google Chrome
ForegroundWindow.ClassName: Chrome_WidgetWin_1
ForegroundWindow.ModulePath: C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
ForegroundWindow.ModuleName: chrome.exe
ForegroundWindow.Children[0].ThreadId: 0xB20
ForegroundWindow.Children[0].ProcessId: 0x1058
ForegroundWindow.Children[0].Text: Chrome Legacy Window
ForegroundWindow.Children[0].ClassName: Chrome_RenderWidgetHostHWND
ForegroundWindow.Children[0].ModulePath: C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
ForegroundWindow.Children[0].ModuleName: chrome.exe
ForegroundWindow.Children[1].ThreadId: 0xB20
ForegroundWindow.Children[1].ProcessId: 0x1058
ForegroundWindow.Children[1].Text: Chrome Legacy Window
ForegroundWindow.Children[1].ClassName: Chrome_RenderWidgetHostHWND
ForegroundWindow.Children[1].ModulePath: C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
ForegroundWindow.Children[1].ModuleName: chrome.exe
ForegroundWindow.Children[2].ThreadId: 0xB20
ForegroundWindow.Children[2].ProcessId: 0x1058
ForegroundWindow.Children[2].Text: Chrome Legacy Window
ForegroundWindow.Children[2].ClassName: Chrome_RenderWidgetHostHWND
ForegroundWindow.Children[2].ModulePath: C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
ForegroundWindow.Children[2].ModuleName: chrome.exe
```
  
```
ForegroundWindow.ProcessId: 0xFE0
ForegroundWindow.Text:
ForegroundWindow.ClassName: Shell_TrayWnd
ForegroundWindow.ModulePath: C:\Windows\explorer.exe
ForegroundWindow.ModuleName: explorer.exe
ForegroundWindow.Children[0].ThreadId: 0xD04
ForegroundWindow.Children[0].ProcessId: 0xFE0
ForegroundWindow.Children[0].Text: Start
ForegroundWindow.Children[0].ClassName: Start
ForegroundWindow.Children[0].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[0].ModuleName: explorer.exe
ForegroundWindow.Children[1].ThreadId: 0xD04
ForegroundWindow.Children[1].ProcessId: 0xFE0
ForegroundWindow.Children[1].Text: Type here to search
ForegroundWindow.Children[1].ClassName: TrayButton
ForegroundWindow.Children[1].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[1].ModuleName: explorer.exe
ForegroundWindow.Children[2].ThreadId: 0xD04
ForegroundWindow.Children[2].ProcessId: 0xFE0
ForegroundWindow.Children[2].Text:
ForegroundWindow.Children[2].ClassName: TrayDummySearchControl
ForegroundWindow.Children[2].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[2].ModuleName: explorer.exe
ForegroundWindow.Children[3].ThreadId: 0xD04
ForegroundWindow.Children[3].ProcessId: 0xFE0
ForegroundWindow.Children[3].Text: Type here to search
ForegroundWindow.Children[3].ClassName: Button
ForegroundWindow.Children[3].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[3].ModuleName: explorer.exe
ForegroundWindow.Children[4].ThreadId: 0xD04
ForegroundWindow.Children[4].ProcessId: 0xFE0
ForegroundWindow.Children[4].Text: Type here to search
ForegroundWindow.Children[4].ClassName: Static
ForegroundWindow.Children[4].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[4].ModuleName: explorer.exe
ForegroundWindow.Children[5].ThreadId: 0xD04
ForegroundWindow.Children[5].ProcessId: 0xFE0
ForegroundWindow.Children[5].Text:
ForegroundWindow.Children[5].ClassName: ToolbarWindow32
ForegroundWindow.Children[5].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[5].ModuleName: explorer.exe
ForegroundWindow.Children[6].ThreadId: 0xD04
ForegroundWindow.Children[6].ProcessId: 0xFE0
ForegroundWindow.Children[6].Text: Task View
ForegroundWindow.Children[6].ClassName: TrayButton
ForegroundWindow.Children[6].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[6].ModuleName: explorer.exe
ForegroundWindow.Children[7].ThreadId: 0xD04
ForegroundWindow.Children[7].ProcessId: 0xFE0
ForegroundWindow.Children[7].Text:
ForegroundWindow.Children[7].ClassName: ReBarWindow32
ForegroundWindow.Children[7].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[7].ModuleName: explorer.exe
ForegroundWindow.Children[8].ThreadId: 0xD04
ForegroundWindow.Children[8].ProcessId: 0xFE0
ForegroundWindow.Children[8].Text: Running applications
ForegroundWindow.Children[8].ClassName: MSTaskSwWClass
ForegroundWindow.Children[8].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[8].ModuleName: explorer.exe
ForegroundWindow.Children[9].ThreadId: 0xD04
ForegroundWindow.Children[9].ProcessId: 0xFE0
ForegroundWindow.Children[9].Text: Running applications
ForegroundWindow.Children[9].ClassName: MSTaskListWClass
ForegroundWindow.Children[9].ModulePath: C:\Windows\explorer.exeForegroundWindow.Children[9].ModuleName: explorer.exe
ForegroundWindow.Children[10].ThreadId: 0xD04
ForegroundWindow.Children[10].ProcessId: 0xFE0
ForegroundWindow.Children[10].Text:
ForegroundWindow.Children[10].ClassName: PeopleBand
ForegroundWindow.Children[10].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[10].ModuleName: explorer.exe
ForegroundWindow.Children[11].ThreadId: 0xD04
ForegroundWindow.Children[11].ProcessId: 0xFE0
ForegroundWindow.Children[11].Text: People
ForegroundWindow.Children[11].ClassName: TrayButton
ForegroundWindow.Children[11].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[11].ModuleName: explorer.exe
ForegroundWindow.Children[12].ThreadId: 0xD04
ForegroundWindow.Children[12].ProcessId: 0xFE0
ForegroundWindow.Children[12].Text:
ForegroundWindow.Children[12].ClassName: TrayNotifyWnd
ForegroundWindow.Children[12].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[12].ModuleName: explorer.exe
ForegroundWindow.Children[13].ThreadId: 0xD04
ForegroundWindow.Children[13].ProcessId: 0xFE0
ForegroundWindow.Children[13].Text:
ForegroundWindow.Children[13].ClassName: Button
ForegroundWindow.Children[13].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[13].ModuleName: explorer.exe
ForegroundWindow.Children[14].ThreadId: 0xD04
ForegroundWindow.Children[14].ProcessId: 0xFE0
ForegroundWindow.Children[14].Text:
ForegroundWindow.Children[14].ClassName: Button
ForegroundWindow.Children[14].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[14].ModuleName: explorer.exe
ForegroundWindow.Children[15].ThreadId: 0xD04
ForegroundWindow.Children[15].ProcessId: 0xFE0
ForegroundWindow.Children[15].Text: System Promoted Notification Area
ForegroundWindow.Children[15].ClassName: ToolbarWindow32
ForegroundWindow.Children[15].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[15].ModuleName: explorer.exe
ForegroundWindow.Children[16].ThreadId: 0xD04
ForegroundWindow.Children[16].ProcessId: 0xFE0
ForegroundWindow.Children[16].Text:
ForegroundWindow.Children[16].ClassName: SysPager
ForegroundWindow.Children[16].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[16].ModuleName: explorer.exe
ForegroundWindow.Children[17].ThreadId: 0xD04
ForegroundWindow.Children[17].ProcessId: 0xFE0
ForegroundWindow.Children[17].Text: User Promoted Notification Area
ForegroundWindow.Children[17].ClassName: ToolbarWindow32
ForegroundWindow.Children[17].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[17].ModuleName: explorer.exe
ForegroundWindow.Children[18].ThreadId: 0xD04
ForegroundWindow.Children[18].ProcessId: 0xFE0
ForegroundWindow.Children[18].Text: Tray Input Indicator
ForegroundWindow.Children[18].ClassName: TrayInputIndicatorWClass
ForegroundWindow.Children[18].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[18].ModuleName: explorer.exe
ForegroundWindow.Children[19].ThreadId: 0xD04
ForegroundWindow.Children[19].ProcessId: 0xFE0
ForegroundWindow.Children[19].Text:
ForegroundWindow.Children[19].ClassName: IMEModeButton
ForegroundWindow.Children[19].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[19].ModuleName: explorer.exe
ForegroundWindow.Children[20].ThreadId: 0xD04
ForegroundWindow.Children[20].ProcessId: 0xFE0
ForegroundWindow.Children[20].Text:
ForegroundWindow.Children[20].ClassName: InputIndicatorButton
ForegroundWindow.Children[20].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[20].ModuleName: explorer.exe
ForegroundWindow.Children[21].ThreadId: 0xD04
ForegroundWindow.Children[21].ProcessId: 0xFE0
ForegroundWindow.Children[21].Text: System Clock, 9:47 PM, ?5/?26/?2018
ForegroundWindow.Children[21].ClassName: TrayClockWClass
ForegroundWindow.Children[21].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[21].ModuleName: explorer.exe
ForegroundWindow.Children[22].ThreadId: 0xD04
ForegroundWindow.Children[22].ProcessId: 0xFE0
ForegroundWindow.Children[22].Text: Action Center
ForegroundWindow.Children[22].ClassName: TrayButton
ForegroundWindow.Children[22].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[22].ModuleName: explorer.exe
ForegroundWindow.Children[23].ThreadId: 0xD04
ForegroundWindow.Children[23].ProcessId: 0xFE0
ForegroundWindow.Children[23].Text:
ForegroundWindow.Children[23].ClassName: TrayShowDesktopButtonWClass
ForegroundWindow.Children[23].ModulePath: C:\Windows\explorer.exe
ForegroundWindow.Children[23].ModuleName: explorer.exe
```
  
Children of `WindowInfo` are also the instance of `WindowInfo` which provides same features, so you can dig into the tree of Windows' window more deeper and deeper.
  
## Notify gesture status changes
  
  
  
### Gesture timeout
  
  
```cs
{
    Config.Callback.GestureTimeout += (sender, e) => { 
        Tooltip("Gesture Timeout");
    };
}
```
  
### Gesture stroke status
  
  
  
The following codes will help you to know current status of gesture stroke.
  
Matched stroke definition will be shown with prefix "✔ ", upcoming candidates will be shown with "➡ ", and if there is no matched stroke definition, current stroke will be shown with "❓ ".
  
```cs
{
    string StrokesToDirectionString(IEnumerable<Crevice.Core.Stroke.Stroke> strokes)
        => string.Join("", strokes.Select(x => DirectionToString(x.Direction)));
  
    string StrokeSequenceToDirectionString(Crevice.Core.Stroke.StrokeSequence strokeSequence)
        => string.Join("", strokeSequence.Select(x => DirectionToString(x)));
  
    string DirectionToString(Crevice.Core.Stroke.StrokeDirection d) 
    {
        switch (d) 
        {
            case Crevice.Core.Stroke.StrokeDirection.Up: return "↑";
            case Crevice.Core.Stroke.StrokeDirection.Down: return "↓";
            case Crevice.Core.Stroke.StrokeDirection.Left: return "←";
            case Crevice.Core.Stroke.StrokeDirection.Right: return "→";
            default: return "";
        };
    }
  
    string lastDirections = "";
  
    Config.Callback.StrokeUpdated += (sender, e) => {
        if (lastDirections.Length == e.Strokes.Count) return;
        var directions = string.Join("", StrokesToDirectionString(e.Strokes));
        var state = CurrentProfile.GestureMachine.CurrentState;
        if (state.IsStateN) {
            var stateN = state.AsStateN();
            var trigger = stateN.InversedStrokeTrigger;
            var matches = trigger.Keys.Select(x => StrokeSequenceToDirectionString(x)).Where(x => x.StartsWith(directions)).ToList();
            if (matches.Any())
            {
                Tooltip($"{string.Join("\r\n", matches.Select(s => s == directions? "✔ " + s : "➡ " + s))}");
            }
            else
            {
                Tooltip($"❓ {directions}");
            }
        }
        lastDirections = directions;
    };
}
```
  
## Change the state of window
  
  
  
### Maximize window
  
  
  
```cs
Do(ctx =>
{
    var SC_MAXIMIZE = 0xF030;
    ctx.ForegroundWindow.PostMessage(WM_SYSCOMMAND, SC_MAXIMIZE, 0);
});
```
  
### Minimize window
  
  
  
```cs
Do(ctx =>
{
    var SC_MINIMIZE = 0xF020;
    ctx.ForegroundWindow.PostMessage(WM_SYSCOMMAND, SC_MINIMIZE, 0);
});
```
  
### Restore window
  
  
  
```cs
Do(ctx =>
{
    var SC_RESTORE = 0xF120;
    ctx.ForegroundWindow.PostMessage(WM_SYSCOMMAND, SC_RESTORE, 0);
});
```
  
### Activate window
  
  
  
```cs
Do(ctx =>
{
    var SC_RESTORE = 0xF120;
    ctx.ForegroundWindow.Activate();
});
```
  
### Close window
  
  
  
```cs
Do(ctx =>
{
    var SC_CLOSE = 0xF060;
    ctx.ForegroundWindow.PostMessage(WM_SYSCOMMAND, SC_CLOSE, 0);
});
```
  
There are more a lot of numbers of parameters can be used for operate the window. See [WM\_SYSCOMMAND message](https://msdn.microsoft.com/library/windows/desktop/ms646360%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396 ) for more details.
  
## Draw mouse gesture trail
  
  
  
If you want to know whether mouse gesture is certainly activated or not, introducing the following extension is help you.
[GestureStrokeOverlay\.cs](https://gist.github.com/rubyu/1be7e0594945e5d304764372aaaf1a0d )
  
<iframe width="560" height="315" src="https://www.youtube.com/embed/1b8raQthC_o" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  
## Change gesture behavior by modifier keys
  
  
  
In this case, you can use keyboard's modifier keys for declaring the gesture definition, but there is one problem. Gesture definition can only be declared with **ordered** `On` clauses. For example, for a gesture definition takes two modifier keys as it's modifier, you should declare the gesture definition with all pattern of the combination of modifier keys.
  
```cs
// Pattern 1/2.
On(Keys.ControlKey).
On(Keys.ShiftKey).
On(Keys.RButton).
Do(ctx => {
  
});
```
  
```cs
// Pattern 2/2.
On(Keys.ShiftKey).
On(Keys.ControlKey).
On(Keys.RButton).
Do(ctx => {
  
});
```
  
By using win32 API `GetKeyState()` instead of the above way, you can easily declare gesture declaration which supports modifier keys.
  
```cs
[System.Runtime.InteropServices.DllImport("user32.dll")]
static extern short GetKeyState(int nVirtKey);
```
  
```cs
On(Keys.RButton).
Do((tx =>
{
    // When shift and control modifier key is pressed
    if (GetKeyState(Keys.ShiftKey) < 0 && 
        GetKeyState(Keys.ControlKey) < 0) 
    {
        // this code will be executed.
    }
});
```
  
## Shortcut for fixed phrase
  
  
  
If you are typing boring boilerplate on your computer everyday, Crevice can automate it by assigning it as a gesture.
  
```cs
using System.Windows.Forms;
using Crevice.Core.DSL;
using Crevice.Core.Keys;
using Crevice.GestureMachine;
  
class KeyCommandManager
{
    private List<LogicalDoubleThrowKey> currentKeys 
        = new List<LogicalDoubleThrowKey>();
  
    private List<(List<LogicalDoubleThrowKey>, Action)> key2Action 
        = new List<(List<LogicalDoubleThrowKey>, Action)>();
  
    public void Setup(
        DoubleThrowElement<ExecutionContext> onElement)
    {
        var uniqueKeys = key2Action.
            Select(t => t.Item1).
            Aggregate(new List<LogicalDoubleThrowKey>(), (a, b) => { a.AddRange(b); return a; }).
            Distinct();
        foreach (var key in uniqueKeys)
        {
            onElement.
            OnDecomposed(key).
            Press(ctx => 
            {
                AddKey(key);
            });
        }
    }
  
    public void Register(Action action, params LogicalDoubleThrowKey[] keyArr)
        => key2Action.Add((keyArr.ToList(), action));
  
    public void Reset() => currentKeys.Clear();
  
    public void AddKey(LogicalDoubleThrowKey key) => currentKeys.Add(key);
  
    public void ExecuteCommand()
    {
        var actions = key2Action.
            Where(t => currentKeys.SequenceEqual(t.Item1)).
            Select(t => t.Item2);
        foreach (var action in actions)
        {
            action();
        }
    }
}
```
  
By the following setting, if you press `Keys.T` and `Keys.I` sequecially while press and holding `Keys.RControlKey`, then a registered text will be pasted from clipboard to foreground application.
  
```cs
var keyCommandManager = new KeyCommandManager();
  
// Register a pair of an action that sends fixed message to the foreground application 
// and a sequence of keys, `Keys.T` - `Keys.I`.
keyCommandManager.Register(() => 
{
    Clipboard.SetText("Thank you for your inquiry asking about our product!");
    SendInput.Multiple().
    ExtendedKeyDown(Keys.ControlKey).
    ExtendedKeyDown(Keys.V).
    ExtendedKeyUp(Keys.V).
    ExtendedKeyUp(Keys.ControlKey).
    Send(); // Ctrl+V
}, Keys.T, Keys.I);
  
var Whenever = When(ctx => { return true; });
var WheneverOn = Whenever.On(Keys.RControlKey);
  
// Declare gesture definition generated automatically from registered data;
// On(Keys.RControlKey).OnDecomposed(Keys.T).Press() and OnDecomposed(Keys.RControlKey).On(Keys.I).Press().
keyCommandManager.Setup(WheneverOn);
  
WheneverOn.
Release(ctx => {
    keyCommandManager.ExecuteCommand();
    keyCommandManager.Reset();
});
```
  
## Define C-x C-x command like Emacs
  
  
  
The following code is the example of Emacs like C-x C-x gesture definition. It seems like previous `KeyCommandManager`, but is more complicated in it's behavior. `EmacsLikeCommandManager` has a state that can be permanent. 
  
```cs
  
using System.Windows.Forms;
using Crevice.Core.DSL;
using Crevice.Core.Keys;
using Crevice.GestureMachine;
  
class EmacsLikeCommandManager
{
    private readonly Action<string> showStatus;
    private readonly List<LogicalDoubleThrowKey> currentKeys 
        = new List<LogicalDoubleThrowKey>();
    private readonly List<(List<LogicalDoubleThrowKey>, Action)> key2Action 
        = new List<(List<LogicalDoubleThrowKey>, Action)>();
  
    public EmacsLikeCommandManager(Action<string> showStatus)
    {
        this.showStatus = showStatus;
    }
    public void Setup(
        DoubleThrowElement<ExecutionContext> onElement)
    {
        var uniqueKeys = key2Action.
            Select(t => t.Item1).
            Aggregate(new List<LogicalDoubleThrowKey>(), (a, b) => { a.AddRange(b); return a; }).
            Distinct();
        foreach (var key in uniqueKeys)
        {
            onElement.
            OnDecomposed(key).
            Press(ctx => 
            {
                AddKey(key);
                showStatus(string.Join("->", currentKeys.Select(k => k.KeyId)));
                if (ExecuteCommand())
                {
                    Reset();
                }
            });
        }
    }
    public void Register(Action action, params Crevice.Core.Keys.LogicalDoubleThrowKey[] keyArr){
        key2Action.Add((keyArr.ToList(), action));
    }
    public void Reset() => currentKeys.Clear();
    public void AddKey(Crevice.Core.Keys.LogicalDoubleThrowKey key) => currentKeys.Add(key);
    public bool ExecuteCommand()
    {
        var actions = key2Action.
            Where(t => currentKeys.SequenceEqual(t.Item1)).
            Select(t => t.Item2);
        foreach (var action in actions)
        {
            action();
        }
        return actions.Any();
    }
}
```
  
If you press `Keys.A` while pressing `Keys.RControlKey`, the first condition to be filled. After that, you can release `Keys.RControlKey`. If you press `Keys.P` while pressing `Keys.RControlKey` again, then all condition to be filled.  A registered text will be pasted from clipboard to foreground application. Like Emacs, you can use C-g to reset the manager's state.
  
```cs
var keyCommandManager = new EmacsLikeCommandManager(str => 
{
    // You need to use Crevice API remotely, if a class uses it.
    Tooltip(str);
});
  
// Register a pair of an action that sends fixed message to the foreground application 
// and a sequence of keys, `Keys.A` - `Keys.P`.
keyCommandManager.Register(() => 
{
    Clipboard.SetText("I appreciate your commitment in promoting the program!");
    SendInput.Multiple().
    ExtendedKeyDown(Keys.ControlKey).
    ExtendedKeyDown(Keys.V).
    ExtendedKeyUp(Keys.V).
    ExtendedKeyUp(Keys.ControlKey).
    Send(); // Ctrl+V
}, Keys.A, Keys.P); // Ctrl+A -> Ctrl+P
  
var Whenever = When(ctx => { return true; });
var WheneverOn = Whenever.On(Keys.RControlKey);
  
// Declare gesture definition generated automatically from registered data;
// On(Keys.RControlKey).OnDecomposed(Keys.A).Press() and On(Keys.RControlKey).OnDecomposed(Keys.P).Press().
keyCommandManager.Setup(WheneverOn);
  
WheneverOn. // Ctrl+G to reset.
OnDecomposed(Keys.G).
Press(ctx => 
{
    Tooltip("");
    keyCommandManager.Reset();
});
```
  
## Fix super sensitive mouse scroll wheel
  
  
  
In recent years, mouse devides that can be purchased in the market include those specially designed for internet browsing. For example, there are major manufacturer product with very sensitive scroll wheel, Logitech M545/546. It is very convenient for the purpose, but it is a bit difficult to handle with other uses. Here we try to make the wheel scroll easier to use for other purposes by introducing a class that adjusts sensitivity.
```cs
class VerticalWheelManager
{
    public readonly int SuppressionDuration;
    public readonly int ForceEmitThreshold;
    public readonly VerticalWheel Up;
    public readonly VerticalWheel Down;
    public VerticalWheelManager(int duration, int threshold)
    {
        this.SuppressionDuration = duration;
        this.ForceEmitThreshold = threshold;
        this.Up = new VerticalWheel(this);
        this.Down = new VerticalWheel(this);
    }
  
    public class VerticalWheel
    {
        private int count = 0;
        private readonly System.Threading.Timer timer;
        private readonly VerticalWheelManager manager;
        public VerticalWheel(VerticalWheelManager manager)
        {
            this.manager = manager;
            this.timer = new System.Threading.Timer(new System.Threading.TimerCallback( delegate { Reset(); } ));
        }
        private VerticalWheel GetCounterpart()
        {
            return manager.Up == this ? manager.Down : manager.Up;
        }
        public bool Check()
        {
            GetCounterpart().Reset();
            count += 1;
            if (count == (manager.ForceEmitThreshold < 2 ? 1 : 2))
            {
                timer.Change(manager.SuppressionDuration, System.Threading.Timeout.Infinite);
                return true;
            }
            if (count > manager.ForceEmitThreshold)
            {
                Reset();
                return Check();
            }
            else
            {
                return false;
            }
        }
        public void Reset()
        {
            count = 0;
            timer.Change(System.Threading.Timeout.Infinite, System.Threading.Timeout.Infinite);
        }
    }
}
  
var VWManager = new VerticalWheelManager(
    400, // Duration in milliseconds for suppression of wheel events.
    6    // Max number of the wheel events which will be suppressed in the duration for suppression.
);
```
  
```cs
Do(ctx =>
{
    if (VWManager.Up.Check())
    {
        // This code will be executed when `cumulative number of Up event > ForceEmitThreshold` or
        // `time passed after previous execution > SuppressionDuration`.
    }
});
```
  
```cs
Do(ctx =>
{
    if (VWManager.Down.Check())
    {
        // This code will be executed when `cumulative number of Down event > ForceEmitThreshold` or
        // `time passed after previous execution > SuppressionDuration`.
    }
});
```
  
Also, it is difficult to only perform wheel click on these devices. You can solve this problem by assigning wheel click to an arbitrary gesture.
  
  
  
  
# Profile
  
  
  
`Profile` is the concept extends the specification of former gesture definition in parallel direction. 
  
## Single context problem on Crevice3
  
  
  
Gesture definition on Crevice3 had a limitation. If you want to use `Keys.XButton1` as `Keys.LWin`, you can do it by using `Press` and `Release` clauses.
```cs
// Simple conversion gesture.
When(ctx => { return true; }).
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
  
But at the same time, if your userscript file have another gesture definition which has it's own context, do you think above gesture defintion works properly always?
  
```cs
// Another gesture.
When(ctx => 
{ 
    return ctx.ForegroundWindow.ModuleName == "chrome.exe"; 
}).
On(Keys.RButton).
On(Keys.MoveDown).
Do(ctx =>
{
    SendInput.Multiple().
    ExtendedKeyDown(Keys.Home).
    ExtendedKeyUp(Keys.Home).
    Send(); // then send Home to Chrome.
});
```
  
When you press `Keys.RButton` and hoding it, the convertion `Keys.XButton1` to `Keys.LWin` will not work. For coexisting both gesture definition, you should add redundant codes to the latter.
  
```cs
// Another gesture (redundant but correct).
var Chrome = When(ctx => 
{ 
    return ctx.ForegroundWindow.ModuleName == "chrome.exe"; 
});
  
Chrome.
On(Keys.RButton).
On(Keys.XButton1).
Press(ctx =>
{
    SendInput.ExtendedKeyDown(Keys.LWin);
}).
Release(ctx =>
{
    SendInput.ExtendedKeyUp(Keys.LWin);
});
  
Chrome.
On(Keys.RButton).
On(Keys.MoveDown).
Do(ctx =>
{
    SendInput.Multiple().
    ExtendedKeyDown(Keys.Home).
    ExtendedKeyUp(Keys.Home).
    Send(); // then send Home to Chrome.
});
```
  
## Solving the problem with DeclareProfile
  
  
  
`DeclareProfile()` declares new profile which has completely new `Config` and empty gesture definition. Changes added to `Config` and gesture definition declared by gesture DSL starting with `When` before that point will be held as the before profile.
  
```cs
Config.Core.GestureTimeout = 5000; // Change 1000ms to 5000ms.
DeclareProfile("NewProfile"); // Config and gesture definition was reset.
ToolTip(Config.Core.GestureTimeout.ToString()); // 1000 will be shown.
```
  
Using this function, you can represent multiple gesture definition working in parallel.
  
```cs
DeclareProfile("Whenever");
  
When(ctx => { return true; }).
On(Keys.XButton1).
Press(ctx =>
{
    SendInput.ExtendedKeyDown(Keys.LWin);
}).
Release(ctx =>
{
    SendInput.ExtendedKeyUp(Keys.LWin);
});
  
DeclareProfile("Chrome");
  
When(ctx => 
{ 
    return ctx.ForegroundWindow.ModuleName == "chrome.exe"; 
}).
On(Keys.RButton).
On(Keys.MoveDown).
Do(ctx =>
{
    SendInput.Multiple().
    ExtendedKeyDown(Keys.Home).
    ExtendedKeyUp(Keys.Home).
    Send(); // then send Home to Chrome.
});
```
  
Even when you press and holding `Keys.RButton`, the convertion `Keys.XButton1` to `Keys.LWin` will work perfectly.
  
Note: The input event will be passed sequencely from the head of the profiles to the bottom, but if the event to be consumed in the `GestureMachine` of each profile, then it will not to be passed to the next profile.
  
  
  
  
# Config
  
  
  
The system default parameters can be configured by using `Config` as following:
  
## Values
  
  
```cs
// When moved distance of the cursor exceeds this value, the first stroke 
// will be established.
Config.Core.StrokeStartThreshold = 10;
  
// When moved distance of the cursor exceeds this value, and the direction is changed,
// new stroke for new direction will be established.
Config.Core.StrokeDirectionChangeThreshold = 20;
  
// When moved distance of the cursor exceeds this value, and the direction is not changed, 
// it will be extended.
Config.Core.StrokeExtensionThreshold = 10;
  
// Interval time for updating strokes.
Config.Core.WatchInterval = 10; // ms
  
// When stroke is not established and this period of time has passed, 
// the gesture will be canceled and the original click event will be reproduced.
Config.Core.GestureTimeout = 1000; // ms
  
// The period of time for showing a tooltip message.
Config.UI.TooltipTimeout = 3000; // ms
  
// The period of time for showing a balloon message.
Config.UI.BalloonTimeout = 10000; // ms
```
  
## Bindings
  
  
  
```cs
// Binding for the position of tooltip messages.
Config.UI.TooltipPositionBinding = (point) =>
{
    return point;
}
```
  
## Events
  
  
  
### StrokeReset
  
  
  
```cs
Config.Callback.StrokeReset += (sender, e) => { };
```
This event activated when the state of mouse's stroke to be reset.
`e` is `StrokeResetEventHandler`, and it does not have special properties.
  
### StrokeUpdated
  
  
```cs
Config.Callback.StrokeUpdated += (sender, e) => { };
```
  
This event activated when the state of mouse's stroke to be changed.
`e` is `StrokeUpdatedEventHandler `.
  
Type | Property Name | Description |
-----|-----|------
IReadOnlyList&lt;Stroke&gt; | Strokes | 
  
### StateChanged
  
  
```cs
Config.Callback.StateChanged += (sender, e) => { };
```
  
This event activated when the state of GestureMachine to be changed. 
`e` is `StateChangedEventHandler`.
  
Type | Property Name | Description |
-----|-----|------
State | LastState | 
State | CurrentState |
  
### GestureCanceled
  
  
```cs
Config.Callback.GestureCanceled += (sender, e) => { };
```
  
This event activated when the gesture to be canceled.
`e` is `GestureCanceledEventHandler`.
  
Type | Property Name | Description |
-----|-----|------
StateN | LastState | 
  
### GestureTimeout
  
  
  
```cs
Config.Callback.GestureTimeout += (sender, e) => { };
```
  
This event activated when the gesture to be timeout.
`e` is `GestureTimeoutEventHandler`.
  
Type | Property Name | Description |
-----|-----|------
StateN | LastState | 
  
### MachineStart
  
  
  
```cs
Config.Callback.MachineStart += (sender, e) => { };
```
This event activated when GestureMachine to be started.
`e` is `MachineStartEventHandler`, and it does not have special properties.
  
### MachineReset
  
  
  
```cs
Config.Callback.MachineReset += (sender, e) => { };
```
  
This event activated when GestureMachine to be reset for some reasons.
`e` is `MachineResetEventHandler`.
  
Type | Property Name | Description |
-----|-----|------
State | LastState | 
  
### MachineStop
  
  
  
```cs
Config.Callback.MachineStop += (sender, e) => { };
```
  
This event activated when GestureMachine to be stopped.
`e` is `MachineStopEventHandler`, and it does not have special properties.
  
  
  
  
# Core API
  
  
  
## EvaluationContext
  
  
  
`Crevice.GestureMachine.EvaluationContext` have following properties:
  
##### Properties
  
  
  
Type | Property Name | Description |
-----|-----|-----|
System.Drawing.Point | GestureStartPosition
ForegroundWindowInfo | ForegroundWindow | The window which was on the foreground when a gesture started. This is an instance of `WindowInfo`.
PointedWindowInfo | PointedWindow | The window which was under the cursor when a gesture started. This is an instance of `WindowInfo`.
  
These values are guaranteed that same values are provided as `ExecutionContext`'s property in `Press`, `Do`, and `Release` clauses.
  
## ExecutionContext
  
  
  
`Crevice.GestureMachine.ExecutionContext` have following properties:
  
##### Properties
  
  
  
Type | Property Name | Description |
-----|-----|-----|
System.Drawing.Point | GestureStartPosition
System.Drawing.Point | GestureEndPosition
WindowInfo | ForegroundWindow | The window which was on the foreground when a gesture started. This is an instance of `WindowInfo`.
WindowInfo | PointedWindow | The window which was under the cursor when a gesture started. This is an instance of `WindowInfo`.
  
These values, except for `GestureEndPosition`, are guaranteed that same values are provided as `EvaluationContext`'s property in `When` clause. 
  
## GestureMachineProfile
  
  
  
`Crevice.GestureMachine.GestureMachineProfile` have following properties:
  
##### Properties
  
  
Type | Property Name | Description
-----|-----|------
RootElement | RootElement
GestureMachine | GestureMachine
UserConfig | UserConfig
string | ProfileName
  
## WindowInfo
  
  
  
`WindowInfo` is a thin wrapper of the handle of a window. This class provides properties and methods to use window handles more easily.
  
##### Properties
  
  
  
Type | Property Name | Description |
-----|-----|-----|
IntPtr | WindowHandle | Window's handle.
int | ThreadId | Window's thread id.
int | ProcessId | Window's process id.
IntPtr | WindowId | Window's window id.
string | Text | Window's title.
string | ClassName | Window's class name.
WindowInfo | Parent | `WindowInfo` of window's parent window.
string | ModulePath | Window's module path.
string | ModuleName | Window's module name.
  
##### Methods
  
  
  
Return Value | Method Definition | Description |
-----|-----|-----|
long | SendMessage(int Msg, int wParam, int lParam) | A shortcut to win32 API `SendMessage(WindowHandle, Msg, wParam, lParam)`.
bool | PostMessage(int Msg, int wParam, int lParam) | A shortcut to win32 API `PostMessage(WindowHandle, Msg, wParam, lParam)`.
bool | BringWindowToTop() | A shortcut to win32 API `BringWindowToTop(WindowHandle)`.
WindowInfo | FindWindowEx(IntPtr hwndChildAfter, string lpszClass, string lpszWindow) | A shortcut to win32 API `FindWindowEx(WindowHandle, hwndChildAfter, lpszClass, lpszWindow)`. 
WindowInfo | FindWindowEx(string lpszClass, string lpszWindow) | A shortcut to win32 API `FindWindowEx(WindowHandle, IntPtr.Zero, lpszClass, lpszWindow)`.
IReadOnlyList&lt;WindowInfo&gt; | GetChildWindows() | A shortcut to win32 API `EnumChildWindows(WindowHandle, EnumWindowProc, IntPtr.Zero)`.
IReadOnlyList&lt;WindowInfo&gt; |  GetPointedDescendantWindows(Point point, Window.WindowFromPointFlags flags) | A shortcut to win32 API `ChildWindowFromPointEx(hWnd, point, flags)`. This function recursively calls `ChildWindowFromPointEx` until reach to the last descendant window.
IReadOnlyList&lt;WindowInfo&gt; | GetPointedDescendantWindows(Point point) | A shortcut to win32 API `ChildWindowFromPointEx(hWnd, point, Window.WindowFromPointFlags.CWP_ALL)`. This function recursively calls `ChildWindowFromPointEx` until reach to the last descendant window.
void | Activate() | Brings window into the foreground and activates the window.
  
And, you can use a static utility class `Window` for accessing and manipulating a window. See [Extension API - Window](#window ) for more details.
  
## SendInput
  
  
  
Send mouse and keyboard input events to the foreground window. This API provides single and multiple sending method. The events sent by single sending method is guaranteed to arrive to the window in order, but this does not necessarily mean it will not be interrupted by the other events. Multiple sending method guarantees that the events sent by it will not be interrupted by the other events.Both methods support the same API for sending mouse and keyboard events except that multiple sending method is need to be called `Send()` at last.
  
```cs
SendInput.ExtendedKeyDown(Keys.LWin);
// When D key interrupts here,
// Win+D will be invoked unintentionally.
SendInput.ExtendedKeyUp(Keys.LWin); 
```
  
```cs
SendInput.Multiple().
ExtendedKeyDown(Keys.LWin).
ExtendedKeyUp(Keys.LWin).
Send(); // This won't be interrupted by any other input.
```
  
### Mouse event
  
  
  
`Down`, `Up`, and `Click` events are supported for the standard push-release type buttons of mouse devices. For example, the provided API for mouse's left button is `LeftDown()`, `LeftUp()` and `LeftClick()`. For single state buttons, `WheelUp()`, `WheelDown()`, `WheelLeft()` and `WheelRight()` are provided. In addition to these, for move event of mouse cursor, `Move(int dx, int dy)` and `MoveTo(int x, int y)` are provided.
  
##### Methods
  
  
  
Button | Method Definition | Description
-----|-----|-----
Keys.LButton | LeftDown()
Keys.LButton | LeftUp()
Keys.LButton | LeftClick() | Shortcut to `LeftDown()` and `LeftUp()`.
Keys.RButton | RightDown()
Keys.RButton | RightUp()
Keys.RButton | RightClick() | Shortcut to `RightDown()` and `RightUp()`.
 - | Move(int dx, int dy) | Move the cursor relatively. `dx` and `dy` are relative values.
 - | MoveTo(int x, int y) | Move the cursor to the specified point. `x` and `y` are absolute values.
Keys.MButton | MiddleDown()
Keys.MButton | MiddleUp()
Keys.MButton | MiddleClick() | Shortcut to `MiddleDown()` and `MiddleUp()`.
- | VerticalWheel(int delta) | Send vertical wheel message. If `delta` is positive value, the direction of the wheel is up, otherwise down.
Keys.WheelDown | WheelDown() | Shortcut to `VerticalWheel(-120)`.
Keys.WheelUp | WheelUp() | Shortcut to `VerticalWheel(120)`.
- | HorizontalWheel(int delta) | Send horizontal wheel message. If `delta` is positive value, the direction of the wheel is right, otherwise left.
Keys.WheelLeft | WheelLeft() |  Shortcut to `HorizontalWheel(-120)`.
Keys.WheelRight | WheelRight() |  Shortcut to `HorizontalWheel(120)`.
Keys.XButton1 | X1Down()
Keys.XButton1 | X1Up()
Keys.XButton1 | X1Click() | Shortcut to `X1Down()` and `X1Up()`.
Keys.XButton2 | X2Down()
Keys.XButton2 | X2Up()
Keys.XButton2 | X2Click() | Shortcut to `X2Down()` and `X2Up()`.
  
### Keyboard event
  
  
  
A keyboard event is synthesized from a key code and two logical flags, `ExtendedKey` and  `ScanCode`. For sending `Up` and `Down` events, `KeyDown(int keyCode)` and `KeyUp(int keyCode)` are provided. 
  
```cs
SendInput.KeyDown(Keys.A);
SendInput.KeyUp(Keys.A); // Send `A` to the foreground application.
```
  
`ExetendedKeyDown(int keyCode)` and `ExtentedKeyUp(int keyCode)` are provided when `ExtendedKey` flag is needed to be set.
  
```cs
SendInput.ExetendedKeyDown(Keys.LWin);
SendInput.ExtentedKeyUp(Keys.LWin); // Send `Win` to the foreground application.
```
  
For four API above mentioned, combined it with `ScanCode` flag,
`KeyDownWithScanCode(int keyCode)`, `KeyUpWithScanCode(int keyCode)`, `ExtendedKeyDownWithScanCode(int keyCode)` and `ExtendedKeyUpWithScanCode(int keyCode)` are provided.
  
```cs
SendInput.ExtendedKeyDownWithScanCode(Keys.LControlKey);
SendInput.KeyDownWithScanCode(Keys.S);
SendInput.KeyUpWithScanCode(Keys.S);
SendInput.ExtendedKeyUpWithScanCode(Keys.LControlKey); // Send `Ctrl+S` with scan code to the foreground application.
```
  
And finally, for to support `Unicode` flag, following functions are provided; `UnicodeKeyDown(char c)`, `UnicodeKeyUp(char c)`,  `UnicodeKeyStroke(string str)`.
  
```cs
SendInput.UnicodeKeyDown('🍣');
SendInput.UnicodeKeyUp('🍣'); // Send `Sushi` to the foreground application.
```
  
##### Methods
  
  
  
Flag | Method Definition | Description
-----|-----|-----
- | KeyDown(int keyCode) |
- | KeyUp(int keyCode) |
Extended | ExetendedKeyDown(int keyCode)
Extended | ExtentedKeyUp(int keyCode)
ScanCode | KeyDownWithScanCode(int keyCode)
ScanCode | KeyUpWithScanCode(int keyCode)
Extended & ScanCode | ExtendedKeyDownWithScanCode(int keyCode)
Extended & ScanCode | ExtendedKeyUpWithScanCode(int keyCode)
- | UnicodeKeyDown(char c)
- | UnicodeKeyUp(char c)
- | UnicodeKeyStroke(string str)
  
## Notification
  
  
  
### Tooltip
  
  
  
```cs
Tooltip("This is tooltip.");
```
  
##### Methods
  
  
  
 Method Definition | Description
-----|-----
Tooltip(string text) | Show tooltip message at the right bottom corner of the display on the cursor, by default. You can configure the position by changing `Config.UI.TooltipPositionBinding`, see [Config - Bindings](#bindings ).
Tooltip(string text, Point point) | Show a tooltip message at the specified position.
Tooltip(string text, Point point, int duration) | Show a tooltip message at the specified position for a specified period.
  
### Balloon
  
  
  
```cs
Balloon("This is balloon.");
```
##### Methods
  
  
  
 Method Definition | Description
-----|-----
Balloon(string text) | Show a balloon message.
Balloon(string text, string title) | Show a balloon message and a title.
Balloon(string text, string title, int timeout) | Show a balloon message and a title for a specified period.
Balloon(string text, string title, ToolTipIcon icon) | Show a balloon message, a title, and a icon.
Balloon(string text, string title, ToolTipIcon icon, int timeout) | Show a balloon message, a title, and a icon for a specified period.
  
## Keys
  
  
  
`Keys` provides the definition of all buttons and keys of mouse and keyboard for it's property. This is almost all same as [System.Windows.Forms.Keys](https://msdn.microsoft.com/library/system.windows.forms.keys(v=vs.110 ).aspx) but for some extentions, wheel and stroke events.
  
### Differences from System.Windows.Forms.Keys
  
  
  
#### Extended properties
  
  
  
Property Name | Description
-----|-----
WheelUp |
WheelDown |
WheelLeft |
WheelRight |
MoveUp |
MoveDown | 
MoveLeft |
MoveRight |
  
These extended properties are differ than the other properties; these can not be treated as a int value.
  
```cs
var n = 1 + Keys.A; // n == 65
```
  
```cs
var n = 1 + Keys.WheelUp; // Compilation error.
```
  
#### Indexer
  
  
  
`Keys` supports indexer for getting a key represents specified keyCode.
  
```cs
Assert.AreEquals(Keys[64], Keys.A);
```
  
This is useful for getting a key which is not assigned as a `Keys`'s property, but be careful to that the keyCode have the range, 0 to 255.
  
```cs
var key = Keys[256]; // This throws an IndexOutOfRangeException(); 
```
  
  
  
  
# Extension API
  
  
  
## Window
  
  
  
`Window` is a utility static class about Windows's window.
To use this class, declare as following:
```cs
using Crevice.WinAPI.Window;
```
  
##### Methods
  
  
  
Return Value | Method Definition | Description
-----|-----|-----
WindowInfo | From(IntPtr hWnd) | This function wraps `IntPtr` and returns an instance of `WindowInfo`.
System.Drawing.Point | GetCursorPos() | A shortcut to win32 API `GetCursorPos()`.
System.Drawing.Point | GetLogicalCursorPos() | Returns logical cursor position culculated based on win32 API `GetPhysicalCursorPos()` and physical and logical screen size.
System.Drawing.Point | GetPhysicalCursorPos() | A shortcut to win32 API `GetPhysicalCursorPos()`.
WindowInfo | WindowFromPoint(Point point) | Returns a window under the cursor.
WindowInfo | FindWindow(string lpClassName, string lpWindowName) | Find a window matches given class name and window name.
IReadOnlyList&lt;WindowInfo&gt; | GetTopLevelWindows() | Enumerates all windows.
IReadOnlyList&lt;WindowInfo&gt; | GetThreadWindows(int threadId) | Enumerates all windows belonging specified thread.
  
## VirtualKeys
  
  
  
This class provides the virtual key constants. 
  
Note: for `VK_0` to `VK_9` and `VK_A` to `VK_Z`, this is an extension for convenience limited in this application.
  
To use this class, declare as following:
```cs
using static Crevice.WinAPI.Constants.VirtualKeys;
```
  
For more details, see [Virtual-Key Codes (Windows)](https://msdn.microsoft.com/ja-jp/library/windows/desktop/dd375731(v=vs.85 ).aspx).
  
## WindowsMessages
  
  
  
This class provides the windows message constants. 
To use this class, declare as following:
```cs
using static Crevice.WinAPI.Constants.WindowsMessages;
```
  
For more details, see [Window Messages (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/ff381405(v=vs.85 ).aspx).
  
## VolumeControl
  
  
  
`VolumeControl` is a utility class about system audio volume.
To use this class, declare as following:
```cs
using Crevice.WinAPI.CoreAudio;
var VolumeControl = new VolumeControl();
```
  
##### Methods
  
  
  
Return Value | Method Definition | Description
-----|-----|-----
float | VolumeControl.GetMasterVolume() | Returns window's current master mixer volume. This function returns a `float` value, within the range between 0 and 1.
 void | VolumeControl.SetMasterVolume(float value) | Sets window's current master mixer volume. The value should be within the range between 0 and 1.
  
  
  
  
# Comamand line interface
  
  
  
```
Usage:
  crevice4.exe [--nogui] [--script path] [--help]
  
  -g, --nogui       (Default: False) Disable GUI features. Set to true if you 
                    use Crevice as a CUI application.
  
  -n, --nocache     (Default: False) Disable user script assembly caching. 
                    Strongly recommend this value to false because compiling 
                    task consumes CPU resources every startup of application if
                    true.
  
  -s, --script      (Default: default.csx) Path to user script file. Use this 
                    option if you need to change the default location of user 
                    script. If given value is relative path, Crevice will 
                    resolve it to absolute path based on the default directory 
                    (%USERPROFILE%\Crevice4).
  
  -p, --priority    (Default: High) Process priority. Acceptable values are the
                    following: AboveNormal, BelowNormal, High, Idle, Normal, 
                    RealTime.
  
  -V, --verbose     (Default: False) Show details about running application.
  
  -v, --version     (Default: False) Display product version.
  
  --help            Display this help screen.
```
  
  
  
  
# Appendix 1. Complete list of some API
  
  
  
## Complete list of Keys
  
  
  
Property Name | Value | Description
-----|-----|-----
WheelUp | - | -
WheelDown | -
WheelLeft | -
WheelRight | -
MoveUp | -
MoveDown | -
MoveLeft | -
MoveRight | -
KeyCode | 0x0000FFFF
Modifiers | 0xFFFF0000
Shift | 0x00010000
Control | 0x00020000
Alt | 0x00040000
None | 0x00
LButton | 0x01
RButton | 0x02
Cancel | 0x03
MButton | 0x04
XButton1 | 0x05
XButton2 | 0x06
Back | 0x08
Tab | 0x09
LineFeed | 0x0A
Clear | 0x0C
Enter | 0x0D
Return | 0x0D
ShiftKey | 0x10
ControlKey | 0x11
Menu | 0x12
Pause | 0x13
CapsLock | 0x14
Capital | 0x14
KanaMode | 0x15
HangulMode | 0x15
JunjaMode | 0x17
FinalMode | 0x18
KanjiMode | 0x19
HanjaMode | 0x19
Escape | 0x1B
IMEConvert | 0x1C
IMENonconvert | 0x1D
IMEAccept | 0x1E
IMEModeChange | 0x1F
Space | 0x20
Prior | 0x21
PageUp | 0x21
Next | 0x22
PageDown | 0x22
End | 0x23
Home | 0x24
Left | 0x25
Up | 0x26
Right | 0x27
Down | 0x28
Select | 0x29
Print | 0x2A
Execute | 0x2B
PrintScreen | 0x2C
Snapshot | 0x2C
Insert | 0x2D
Delete | 0x2E
Help | 0x2F
D0 | 0x30
D1 | 0x31
D2 | 0x32
D3 | 0x33
D4 | 0x34
D5 | 0x35
D6 | 0x36
D7 | 0x37
D8 | 0x38
D9 | 0x39
A | 0x41
B | 0x42
C | 0x43
D | 0x44
E | 0x45
F | 0x46
G | 0x47
H | 0x48
I | 0x49
J | 0x4A
K | 0x4B
L | 0x4C
M | 0x4D
N | 0x4E
O | 0x4F
P | 0x50
Q | 0x51
R | 0x52
S | 0x53
T | 0x54
U | 0x55
V | 0x56
W | 0x57
X | 0x58
Y | 0x59
Z | 0x5A
LWin | 0x5B
RWin | 0x5C
Apps | 0x5D
Sleep | 0x5F
NumPad0 | 0x60
NumPad1 | 0x61
NumPad2 | 0x62
NumPad3 | 0x63
NumPad4 | 0x64
NumPad5 | 0x65
NumPad6 | 0x66
NumPad7 | 0x67
NumPad8 | 0x68
NumPad9 | 0x69
Multiply | 0x6A
Add | 0x6B
Separator | 0x6C
Subtract | 0x6D
Decimal | 0x6E
Divide | 0x6F
F1 | 0x70
F2 | 0x71
F3 | 0x72
F4 | 0x73
F5 | 0x74
F6 | 0x75
F7 | 0x76
F8 | 0x77
F9 | 0x78
F10 | 0x79
F11 | 0x7A
F12 | 0x7B
F13 | 0x7C
F14 | 0x7D
F15 | 0x7E
F16 | 0x7F
F17 | 0x80
F18 | 0x81
F19 | 0x82
F20 | 0x83
F21 | 0x84
F22 | 0x85
F23 | 0x86
F24 | 0x87
NumLock | 0x90
Scroll | 0x91
LShiftKey | 0xA0
RShiftKey | 0xA1
LControlKey | 0xA2
RControlKey | 0xA3
LMenu | 0xA4
RMenu | 0xA5
BrowserBack | 0xA6
BrowserForward | 0xA7
BrowserRefresh | 0xA8
BrowserStop | 0xA9
BrowserSearch | 0xAA
BrowserFavorites | 0xAB
BrowserHome | 0xAC
VolumeMute | 0xAD
VolumeDown | 0xAE
VolumeUp | 0xAF
MediaNextTrack | 0xB0
MediaPreviousTrack | 0xB1
MediaStop | 0xB2
MediaPlayPause | 0xB3
LaunchMail | 0xB4
SelectMedia | 0xB5
LaunchApplication1 | 0xB6
LaunchApplication2 | 0xB7
Oem1 | 0xBA
OemSemicolon | 0xBA
Oemplus | 0xBB
Oemcomma | 0xBC
OemMinus | 0xBD
OemPeriod | 0xBE
OemQuestion | 0xBF
Oem2 | 0xBF
Oemtilde | 0xC0
Oem3 | 0xC0
Oem4 | 0xDB
OemOpenBrackets | 0xDB
OemPipe | 0xDC
Oem5 | 0xDC
Oem6 | 0xDD
OemCloseBrackets | 0xDD
Oem7 | 0xDE
OemQuotes | 0xDE
Oem8 | 0xDF
Oem102 | 0xE2
OemBackslash | 0xE2
ProcessKey | 0xE5
Packet | 0xE7
Attn | 0xF6
Crsel | 0xF7
Exsel | 0xF8
EraseEof | 0xF9
Play | 0xFA
Zoom | 0xFB
NoName | 0xFC
Pa1 | 0xFD
OemClear | 0xFE
  
## Complete list of Crevice.WinAPI.Constants.VirtualKeys
  
  
  
  
  
  
  
Property Name | Value | Description
-----|-----|-----
 VK_LBUTTON | 0x01 | Left mouse button
 VK_RBUTTON | 0x02 | Right mouse button
 VK_CANCEL | 0x03 | Control-break processing
 VK_MBUTTON | 0x04 | Middle mouse button (three-button mouse)
 VK_XBUTTON1 | 0x05 | X1 mouse button
 VK_XBUTTON2 | 0x06 | X2 mouse button
 - | 0x07 | Undefined
 VK_BACK | 0x08 | BACKSPACE key
 VK_TAB | 0x09 | TAB key
 - | 0x0A-0B | Reserved
 VK_CLEAR | 0x0C | CLEAR key
 VK_RETURN | 0x0D | ENTER key
 - | 0x0E-0F | Undefined
 VK_SHIFT | 0x10 | SHIFT key
 VK_CONTROL | 0x11 | CTRL key
 VK_MENU | 0x12 | ALT key
 VK_PAUSE | 0x13 | PAUSE key
 VK_CAPITAL | 0x14 | CAPS LOCK key
 VK_KANA | 0x15 | IME Kana mode
 VK_HANGUEL | 0x15 | IME Hanguel mode (maintained for compatibility; use VK_HANGUL)
 VK_HANGUL | 0x15 | IME Hangul mode
 - | 0x16 | Undefined
 VK_JUNJA | 0x17 | IME Junja mode
 VK_FINAL | 0x18 | IME final mode
 VK_HANJA | 0x19 | IME Hanja mode
 VK_KANJI | 0x19 | IME Kanji mode
 - | 0x1A | Undefined
 VK_ESCAPE | 0x1B | ESC key
 VK_CONVERT | 0x1C | IME convert
 VK_NONCONVERT | 0x1D | IME nonconvert
 VK_ACCEPT | 0x1E | IME accept
 VK_MODECHANGE | 0x1F | IME mode change request
 VK_SPACE | 0x20 | SPACEBAR
 VK_PRIOR | 0x21 | PAGE UP key
 VK_NEXT | 0x22 | PAGE DOWN key
 VK_END | 0x23 | END key
 VK_HOME | 0x24 | HOME key
 VK_LEFT | 0x25 | LEFT ARROW key
 VK_UP | 0x26 | UP ARROW key
 VK_RIGHT | 0x27 | RIGHT ARROW key
 VK_DOWN | 0x28 | DOWN ARROW key
 VK_SELECT | 0x29 | SELECT key
 VK_PRINT | 0x2A | PRINT key
 VK_EXECUTE | 0x2B | EXECUTE key
 VK_SNAPSHOT | 0x2C | PRINT SCREEN key
 VK_INSERT | 0x2D | INS key
 VK_DELETE | 0x2E | DEL key
 VK_HELP | 0x2F | HELP key
 VK_0 | 0x30 | 0 key
 VK_1 | 0x31 | 1 key
 VK_2 | 0x32 | 2 key
 VK_3 | 0x33 | 3 key
 VK_4 | 0x34 | 4 key
 VK_5 | 0x35 | 5 key
 VK_6 | 0x36 | 6 key
 VK_7 | 0x37 | 7 key
 VK_8 | 0x38 | 8 key
 VK_9 | 0x39 | 9 key
 - | 0x3A-40 | Undefined
 VK_A | 0x41 | A key
 VK_B | 0x42 | B key
 VK_C | 0x43 | C key
 VK_D | 0x44 | D key
 VK_E | 0x45 | E key
 VK_F | 0x46 | F key
 VK_G | 0x47 | G key
 VK_H | 0x48 | H key
 VK_I | 0x49 | I key
 VK_J | 0x4A | J key
 VK_K | 0x4B | K key
 VK_L | 0x4C | L key
 VK_M | 0x4D | M key
 VK_N | 0x4E | N key
 VK_O | 0x4F | O key
 VK_P | 0x50 | P key
 VK_Q | 0x51 | Q key
 VK_R | 0x52 | R key
 VK_S | 0x53 | S key
 VK_T | 0x54 | T key
 VK_U | 0x55 | U key
 VK_V | 0x56 | V key
 VK_W | 0x57 | W key
 VK_X | 0x58 | X key
 VK_Y | 0x59 | Y key
 VK_Z | 0x5A | Z key
 VK_LWIN | 0x5B | Left Windows key (Natural keyboard)
 VK_RWIN | 0x5C | Right Windows key (Natural keyboard)
 VK_APPS | 0x5D | Applications key (Natural keyboard)
 - | 0x5E | Reserved
 VK_SLEEP | 0x5F | Computer Sleep key
 VK_NUMPAD0 | 0x60 | Numeric keypad 0 key
 VK_NUMPAD1 | 0x61 | Numeric keypad 1 key
 VK_NUMPAD2 | 0x62 | Numeric keypad 2 key
 VK_NUMPAD3 | 0x63 | Numeric keypad 3 key
 VK_NUMPAD4 | 0x64 | Numeric keypad 4 key
 VK_NUMPAD5 | 0x65 | Numeric keypad 5 key
 VK_NUMPAD6 | 0x66 | Numeric keypad 6 key
 VK_NUMPAD7 | 0x67 | Numeric keypad 7 key
 VK_NUMPAD8 | 0x68 | Numeric keypad 8 key
 VK_NUMPAD9 | 0x69 | Numeric keypad 9 key
 VK_MULTIPLY | 0x6A | Multiply key
 VK_ADD | 0x6B | Add key
 VK_SEPARATOR | 0x6C | Separator key
 VK_SUBTRACT | 0x6D | Subtract key
 VK_DECIMAL | 0x6E | Decimal key
 VK_DIVIDE | 0x6F | Divide key
 VK_F1 | 0x70 | F1 key
 VK_F2 | 0x71 | F2 key
 VK_F3 | 0x72 | F3 key
 VK_F4 | 0x73 | F4 key
 VK_F5 | 0x74 | F5 key
 VK_F6 | 0x75 | F6 key
 VK_F7 | 0x76 | F7 key
 VK_F8 | 0x77 | F8 key
 VK_F9 | 0x78 | F9 key
 VK_F10 | 0x79 | F10 key
 VK_F11 | 0x7A | F11 key
 VK_F12 | 0x7B | F12 key
 VK_F13 | 0x7C | F13 key
 VK_F14 | 0x7D | F14 key
 VK_F15 | 0x7E | F15 key
 VK_F16 | 0x7F | F16 key
 VK_F17 | 0x80 | F17 key
 VK_F18 | 0x81 | F18 key
 VK_F19 | 0x82 | F19 key
 VK_F20 | 0x83 | F20 key
 VK_F21 | 0x84 | F21 key
 VK_F22 | 0x85 | F22 key
 VK_F23 | 0x86 | F23 key
 VK_F24 | 0x87 | F24 key
 - | 0x88-8F | Unassigned
 VK_NUMLOCK | 0x90 | NUM LOCK key
 VK_SCROLL | 0x91 | SCROLL LOCK key
 - |0x92-96 | OEM specific
 - | 0x97-9F | Unassigned
 VK_LSHIFT | 0xA0 | Left SHIFT key
 VK_RSHIFT | 0xA1 | Right SHIFT key
 VK_LCONTROL | 0xA2 | Left CONTROL key
 VK_RCONTROL | 0xA3 | Right CONTROL key
 VK_LMENU | 0xA4 | Left MENU key
 VK_RMENU | 0xA5 | Right MENU key
 VK_BROWSER_BACK | 0xA6 | Browser Back key
 VK_BROWSER_FORWARD | 0xA7 | Browser Forward key
 VK_BROWSER_REFRESH | 0xA8 | Browser Refresh key
 VK_BROWSER_STOP | 0xA9 | Browser Stop key
 VK_BROWSER_SEARCH | 0xAA | Browser Search key
 VK_BROWSER_FAVORITES | 0xAB | Browser Favorites key
 VK_BROWSER_HOME | 0xAC | Browser Start and Home key
 VK_VOLUME_MUTE | 0xAD | Volume Mute key
 VK_VOLUME_DOWN | 0xAE | Volume Down key
 VK_VOLUME_UP | 0xAF | Volume Up key
 VK_MEDIA_NEXT_TRACK | 0xB0 | Next Track key
 VK_MEDIA_PREV_TRACK | 0xB1 | Previous Track key
 VK_MEDIA_STOP | 0xB2 | Stop Media key
 VK_MEDIA_PLAY_PAUSE | 0xB3 | Play/Pause Media key
 VK_LAUNCH_MAIL | 0xB4 | Start Mail key
 VK_LAUNCH_MEDIA_SELECT | 0xB5 | Select Media key
 VK_LAUNCH_APP1 | 0xB6 | Start Application 1 key
 VK_LAUNCH_APP2 | 0xB7 | Start Application 2 key
 - | 0xB8-B9 | Reserved
 VK_OEM_1 | 0xBA | Used for miscellaneous characters; it can vary by keyboard. For the US standard keyboard, the ';:' key
 VK_OEM_PLUS | 0xBB |  For any country/region, the '+' key
 VK_OEM_COMMA | 0xBC | For any country/region, the ',' key
 VK_OEM_MINUS | 0xBD | For any country/region, the '-' key
 VK_OEM_PERIOD | 0xBE | For any country/region, the '.' key
 VK_OEM_2 | 0xBF | Used for miscellaneous characters; it can vary by keyboard. For the US standard keyboard, the '/?' key
 VK_OEM_3 | 0xC0 | Used for miscellaneous characters; it can vary by keyboard. For the US standard keyboard, the '`~' key
 - | 0xC1-D7 | Reserved
 - | 0xD8-DA | Unassigned
 VK_OEM_4 | 0xDB | Used for miscellaneous characters; it can vary by keyboard. For the US standard keyboard, the '[{' key
 VK_OEM_5 | 0xDC | Used for miscellaneous characters; it can vary by keyboard. For the US standard keyboard, the '&#x5c;&#x7c;' key
 VK_OEM_6 | 0xDD | Used for miscellaneous characters; it can vary by keyboard. For the US standard keyboard, the ']}' key
 VK_OEM_7 | 0xDE | Used for miscellaneous characters; it can vary by keyboard. For the US standard keyboard, the 'single-quote/double-quote' key
 VK_OEM_8 | 0xDF | Used for miscellaneous characters; it can vary by keyboard.
 - | 0xE0 | Reserved
 - | 0xE1 | OEM specific
 VK_OEM_102 | 0xE2 | Either the angle bracket key or the backslash key on the RT 102-key keyboard
 - | 0xE3-E4 | OEM specific
 VK_PROCESSKEY | 0xE5 | IME PROCESS key
 - | 0xE6 | OEM specific
 VK_PACKET | 0xE7 | Used to pass Unicode characters as if they were keystrokes. The VK_PACKET key is the low word of a 32-bit Virtual Key value used for non-keyboard input methods. For more information, see Remark in KEYBDINPUT, SendInput, WM_KEYDOWN, and WM_KEYUP
 - | 0xE8 | Unassigned
 - | 0xE9-F5 | OEM specific
 VK_ATTN | 0xF6 | Attn key
 VK_CRSEL | 0xF7 | CrSel key
 VK_EXSEL | 0xF8 | ExSel key
 VK_EREOF | 0xF9 | Erase EOF key
 VK_PLAY | 0xFA | Play key
 VK_ZOOM | 0xFB | Zoom key
 VK_NONAME | 0xFC | Reserved
 VK_PA1 | 0xFD | PA1 key
 VK_OEM_CLEAR | 0xFE | Clear Key
  
## Crevice.WinAPI.Constants.WindowsMessages
  
  
  
  
  
  
Property Name | Value 
-----|-----
 WM_NULL | 0x0000
 WM_CREATE | 0x0001
 WM_DESTROY | 0x0002
 WM_MOVE | 0x0003
 WM_SIZE | 0x0005
 WM_ACTIVATE | 0x0006
 WM_SETFOCUS | 0x0007
 WM_KILLFOCUS | 0x0008
 WM_ENABLE | 0x000A
 WM_SETREDRAW | 0x000B
 WM_SETTEXT | 0x000C
 WM_GETTEXT | 0x000D
 WM_GETTEXTLENGTH | 0x000E
 WM_PAINT | 0x000F
 WM_CLOSE | 0x0010
 WM_QUERYENDSESSION | 0x0011
 WM_QUERYOPEN | 0x0013
 WM_ENDSESSION | 0x0016
 WM_QUIT | 0x0012
 WM_ERASEBKGND | 0x0014
 WM_SYSCOLORCHANGE | 0x0015
 WM_SHOWWINDOW | 0x0018
 WM_WININICHANGE | 0x001A
 WM_SETTINGCHANGE | WM_WININICHANGE
 WM_DEVMODECHANGE | 0x001B
 WM_ACTIVATEAPP | 0x001C
 WM_FONTCHANGE | 0x001D
 WM_TIMECHANGE | 0x001E
 WM_CANCELMODE | 0x001F
 WM_SETCURSOR | 0x0020
 WM_MOUSEACTIVATE | 0x0021
 WM_CHILDACTIVATE | 0x0022
 WM_QUEUESYNC | 0x0023
 WM_GETMINMAXINFO | 0x0024
 WM_PAINTICON | 0x0026
 WM_ICONERASEBKGND | 0x0027
 WM_NEXTDLGCTL | 0x0028
 WM_SPOOLERSTATUS | 0x002A
 WM_DRAWITEM | 0x002B
 WM_MEASUREITEM | 0x002C
 WM_DELETEITEM | 0x002D
 WM_VKEYTOITEM | 0x002E
 WM_CHARTOITEM | 0x002F
 WM_SETFONT | 0x0030
 WM_GETFONT | 0x0031
 WM_SETHOTKEY | 0x0032
 WM_GETHOTKEY | 0x0033
 WM_QUERYDRAGICON | 0x0037
 WM_COMPAREITEM | 0x0039
 WM_GETOBJECT | 0x003D
 WM_COMPACTING | 0x0041
 WM_COMMNOTIFY | 0x0044
 WM_WINDOWPOSCHANGING | 0x0046
 WM_WINDOWPOSCHANGED | 0x0047
 WM_POWER | 0x0048
 WM_COPYDATA | 0x004A
 WM_CANCELJOURNAL | 0x004B
 WM_NOTIFY | 0x004E
 WM_INPUTLANGCHANGEREQUEST | 0x0050
 WM_INPUTLANGCHANGE | 0x0051
 WM_TCARD | 0x0052
 WM_HELP | 0x0053
 WM_USERCHANGED | 0x0054
 WM_NOTIFYFORMAT | 0x0055
 WM_CONTEXTMENU | 0x007B
 WM_STYLECHANGING | 0x007C
 WM_STYLECHANGED | 0x007D
 WM_DISPLAYCHANGE | 0x007E
 WM_GETICON | 0x007F
 WM_SETICON | 0x0080
 WM_NCCREATE | 0x0081
 WM_NCDESTROY | 0x0082
 WM_NCCALCSIZE | 0x0083
 WM_NCHITTEST | 0x0084
 WM_NCPAINT | 0x0085
 WM_NCACTIVATE | 0x0086
 WM_GETDLGCODE | 0x0087
 WM_SYNCPAINT | 0x0088
 WM_NCMOUSEMOVE | 0x00A0
 WM_NCLBUTTONDOWN | 0x00A1
 WM_NCLBUTTONUP | 0x00A2
 WM_NCLBUTTONDBLCLK | 0x00A3
 WM_NCRBUTTONDOWN | 0x00A4
 WM_NCRBUTTONUP | 0x00A5
 WM_NCRBUTTONDBLCLK | 0x00A6
 WM_NCMBUTTONDOWN | 0x00A7
 WM_NCMBUTTONUP | 0x00A8
 WM_NCMBUTTONDBLCLK | 0x00A9
 WM_NCXBUTTONDOWN | 0x00AB
 WM_NCXBUTTONUP | 0x00AC
 WM_NCXBUTTONDBLCLK | 0x00AD
 WM_INPUT_DEVICE_CHANGE | 0x00FE
 WM_INPUT | 0x00FF
 WM_KEYFIRST | 0x0100
 WM_KEYDOWN | 0x0100
 WM_KEYUP | 0x0101
 WM_CHAR | 0x0102
 WM_DEADCHAR | 0x0103
 WM_SYSKEYDOWN | 0x0104
 WM_SYSKEYUP | 0x0105
 WM_SYSCHAR | 0x0106
 WM_SYSDEADCHAR | 0x0107
 WM_UNICHAR | 0x0109
 WM_KEYLAST | 0x0109
 WM_IME_STARTCOMPOSITION | 0x010D
 WM_IME_ENDCOMPOSITION | 0x010E
 WM_IME_COMPOSITION | 0x010F
 WM_IME_KEYLAST | 0x010F
 WM_INITDIALOG | 0x0110
 WM_COMMAND | 0x0111
 WM_SYSCOMMAND | 0x0112
 WM_TIMER | 0x0113
 WM_HSCROLL | 0x0114
 WM_VSCROLL | 0x0115
 WM_INITMENU | 0x0116
 WM_INITMENUPOPUP | 0x0117
 WM_MENUSELECT | 0x011F
 WM_MENUCHAR | 0x0120
 WM_ENTERIDLE | 0x0121
 WM_MENURBUTTONUP | 0x0122
 WM_MENUDRAG | 0x0123
 WM_MENUGETOBJECT | 0x0124
 WM_UNINITMENUPOPUP | 0x0125
 WM_MENUCOMMAND | 0x0126
 WM_CHANGEUISTATE | 0x0127
 WM_UPDATEUISTATE | 0x0128
 WM_QUERYUISTATE | 0x0129
 WM_CTLCOLORMSGBOX | 0x0132
 WM_CTLCOLOREDIT | 0x0133
 WM_CTLCOLORLISTBOX | 0x0134
 WM_CTLCOLORBTN | 0x0135
 WM_CTLCOLORDLG | 0x0136
 WM_CTLCOLORSCROLLBAR | 0x0137
 WM_CTLCOLORSTATIC | 0x0138
 WM_MOUSEFIRST | 0x0200
 WM_MOUSEMOVE | 0x0200
 WM_LBUTTONDOWN | 0x0201
 WM_LBUTTONUP | 0x0202
 WM_LBUTTONDBLCLK | 0x0203
 WM_RBUTTONDOWN | 0x0204
 WM_RBUTTONUP | 0x0205
 WM_RBUTTONDBLCLK | 0x0206
 WM_MBUTTONDOWN | 0x0207
 WM_MBUTTONUP | 0x0208
 WM_MBUTTONDBLCLK | 0x0209
 WM_MOUSEWHEEL | 0x020A
 WM_XBUTTONDOWN | 0x020B
 WM_XBUTTONUP | 0x020C
 WM_XBUTTONDBLCLK | 0x020D
 WM_MOUSEHWHEEL | 0x020E
 WM_MOUSELAST | 0x020E
 WM_PARENTNOTIFY | 0x0210
 WM_ENTERMENULOOP | 0x0211
 WM_EXITMENULOOP | 0x0212
 WM_NEXTMENU | 0x0213
 WM_SIZING | 0x0214
 WM_CAPTURECHANGED | 0x0215
 WM_MOVING | 0x0216
 WM_POWERBROADCAST | 0x0218
 WM_DEVICECHANGE | 0x0219
 WM_MDICREATE | 0x0220
 WM_MDIDESTROY | 0x0221
 WM_MDIACTIVATE | 0x0222
 WM_MDIRESTORE | 0x0223
 WM_MDINEXT | 0x0224
 WM_MDIMAXIMIZE | 0x0225
 WM_MDITILE | 0x0226
 WM_MDICASCADE | 0x0227
 WM_MDIICONARRANGE | 0x0228
 WM_MDIGETACTIVE | 0x0229
 WM_MDISETMENU | 0x0230
 WM_ENTERSIZEMOVE | 0x0231
 WM_EXITSIZEMOVE | 0x0232
 WM_DROPFILES | 0x0233
 WM_MDIREFRESHMENU | 0x0234
 WM_IME_SETCONTEXT | 0x0281
 WM_IME_NOTIFY | 0x0282
 WM_IME_CONTROL | 0x0283
 WM_IME_COMPOSITIONFULL | 0x0284
 WM_IME_SELECT | 0x0285
 WM_IME_CHAR | 0x0286
 WM_IME_REQUEST | 0x0288
 WM_IME_KEYDOWN | 0x0290
 WM_IME_KEYUP | 0x0291
 WM_MOUSEHOVER | 0x02A1
 WM_MOUSELEAVE | 0x02A3
 WM_NCMOUSEHOVER | 0x02A0
 WM_NCMOUSELEAVE | 0x02A2
 WM_WTSSESSION_CHANGE | 0x02B1
 WM_TABLET_FIRST | 0x02c0
 WM_TABLET_LAST | 0x02df
 WM_CUT | 0x0300
 WM_COPY | 0x0301
 WM_PASTE | 0x0302
 WM_CLEAR | 0x0303
 WM_UNDO | 0x0304
 WM_RENDERFORMAT | 0x0305
 WM_RENDERALLFORMATS | 0x0306
 WM_DESTROYCLIPBOARD | 0x0307
 WM_DRAWCLIPBOARD | 0x0308
 WM_PAINTCLIPBOARD | 0x0309
 WM_VSCROLLCLIPBOARD | 0x030A
 WM_SIZECLIPBOARD | 0x030B
 WM_ASKCBFORMATNAME | 0x030C
 WM_CHANGECBCHAIN | 0x030D
 WM_HSCROLLCLIPBOARD | 0x030E
 WM_QUERYNEWPALETTE | 0x030F
 WM_PALETTEISCHANGING | 0x0310
 WM_PALETTECHANGED | 0x0311
 WM_HOTKEY | 0x0312
 WM_PRINT | 0x0317
 WM_PRINTCLIENT | 0x0318
 WM_APPCOMMAND | 0x0319
 WM_THEMECHANGED | 0x031A
 WM_CLIPBOARDUPDATE | 0x031D
 WM_DWMCOMPOSITIONCHANGED | 0x031E
 WM_DWMNCRENDERINGCHANGED | 0x031F
 WM_DWMCOLORIZATIONCOLORCHANGED | 0x0320
 WM_DWMWINDOWMAXIMIZEDCHANGE | 0x0321
 WM_GETTITLEBARINFOEX | 0x033F
 WM_HANDHELDFIRST | 0x0358
 WM_HANDHELDLAST | 0x035F
 WM_AFXFIRST | 0x0360
 WM_AFXLAST | 0x037F
 WM_PENWINFIRST | 0x0380
 WM_PENWINLAST | 0x038F
 WM_APP | 0x8000
 WM_USER | 0x0400
 WM_CPL_LAUNCH | WM_USER + 0x1000
 WM_CPL_LAUNCHED | WM_USER + 0x1001
 WM_SYSTIMER | 0x118
  
  
  
  
# Appendix 2. How to introduce gesture function to your application
  
  
  
## Introduction
  
  
  
`CreviceLib` is a library which provides basic gesture functions, based on the codes abstracted from core logic of **CreviceApp** 3.x. **CreviceApp** 4.0 or later is using `CreviceLib` as a library. So if you want, you can easily introduce gesture function to your application by adding a reference to `CreviceLib` and following this guidance.
  
_Note: `CreviceLib` is distributed as a nuget package. Visit [NuGet Gallery \| CreviceLib](https://www.nuget.org/packages/Crevice.Core/ ) for more details._
  
  
## Quickstart
  
  
  
At first, check if the reference to `CreviceLib` is certainly added to your project.
  
A bit simplified, but sufficiently practical classes are provided in `Crevice.Core.Example`. The very simple setup code is the following:
  
```cs
using Crevice.Core.Example;
  
var keysA = new SimpleKeySetA(maxSize: 10);
var root = new SimpleRootElement();
var gm = new SimpleGestureMachine();
```
  
Then, you can be able to start writing gesture DSL.
  
```cs
var whenever = root.When(ctx => {
    return true;
});
var action = whenever.On(keysA[0]);
action.Do(ctx => {
    // When PressEvent and ReleaseEvent of keysA[0] are given to GestureMachine,
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
gm.Input(keysA[0].PressEvent);
gm.Input(keysA[0].ReleaseEvent);
// then the action will be executed here.
```
  
After using it, `GestureMachine` should be disposed.
  
```cs
gm.Dispose();
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
<div class="img-caption">Fig4. KeyA.</div>
  
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
<div class="img-caption">Fig5. KeyB.</div>
  
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
<div class="img-caption">Fig6. Image of the compression KeyA to KeyB.</div>
  
## KeySet
  
  
  
`CreviceLib` provides **KeySet** classes, which inherit `Crevice.Core.Keys.KeySet<KeyType>`, managing a set of sequential keys of **KeyA** (`DoubleThrowKey`) or **KeyB** (`SingleThrowKey`). 
  
`SimpleKeySetA` corresponds to **KeyA**, and `SimpleKeySetB` corresponds to **KeyB**. These can be used in a simply way, only take an argument `maxSize` which means the maximum size of the sequential key set.
  
```cs
var keysA = new SimpleKeySetA(maxSize: 10);
Assert.AreEqual(keysA is PhysicalDoubleThrowKeySet, true);
Assert.AreEqual(keysA[0] is PhysicalDoubleThrowKey, true);
Assert.AreEqual(keysA[0].PressEvent is PressEvent, true);
Assert.AreEqual(keysA[0].ReleaseEvent is ReleaseEvent, true);
```
  
```cs
var keysB = new SimpleKeySetB(maxSize: 10);
Assert.AreEqual(keysB is PhysicalSingleThrowKeySet, true);
Assert.AreEqual(keysB[0] is PhysicalSingleThrowKey, true);
Assert.AreEqual(keysB[0].FireEvent is FireEvent, true);
```
  
_Note: Regarding the adjective **Physical** commonly held by both names of the type of `SimpleKeySetA` and `SimpleKeySetB`, see [Physical and logical event types](#physical-and-logical-event-types ) for more details._
  
## GestureMachineConfig
  
  
  
`Crevice.Core.FSM.GestureMachineConfig` holds configuration values for `GestureMachine`. The configuration values are so mutable that you can edit it any time. Also you can use newly customized `GestureMachineConfig` by inheriting it.
  
`SimpleGestureMachineConfig` is a example class which inherits `GestureMachineConfig`, and do not have any change from it.
  
```cs
var config = new SimpleGestureMachineConfig();
```
  
Available properties are the following:
  
```cs
// When moved distance of the cursor exceeds this value, the first stroke 
// will be established.
config.StrokeStartThreshold = 10;
  
// When moved distance of the cursor exceeds this value, and the direction is changed,
// new stroke for new direction will be established.
config.StrokeDirectionChangeThreshold = 20;
  
// When moved distance of the cursor exceeds this value, and the direction is not changed, 
// it will be extended.
config.StrokeExtensionThreshold = 10;
  
// Interval time for updating strokes.
config.WatchInterval = 10; // ms
  
// When stroke is not established and this period of time has passed, 
// the gesture will be canceled and the original click event will be reproduced.
config.GestureTimeout = 1000; // ms
```
  
_Note: If you want to use customized `GestureMachineConfig`, it should be given as the generics type parameter to classes which extend these classes: [CallbackManager](#callbackmanager ), and [GestureMachine](#gesturemachine )._
  
## EvaluationContext
  
  
  
`Crevice.Core.Context.EvaluationContext` is a class to be passed to a function which declared with `When()` on gesture DSL, as the argument. By default, it is empty and does not have any value. You can inherits and extends this class so that values you need will be given to `Evaluator` on it's evaluation.
  
```cs
public class OriginalEvaluationContext : EvaluationContext
{
    public readonly DateTime Created;
  
    public EvaluationContext()
    {
        Created = System.DateTime.Now;
    }
}
```
  
_Note: If you want to use customized `EvaluationContext`, it should be given as the generics type parameter to classes which extend these classes: [RootElement](#rootelement ), [ContextManager](#contextmanager ), [CallbackManager](#callbackmanager ), and [GestureMachine](#gesturemachine )._
  
## ExecutionContext
  
  
  
`Crevice.Core.Context.ExecutionContext` is a class to be passed to a function which declared with `Press()`, `Do()`, and `Release()` on gesture DSL, as the argument. By default, it is empty and does not have any value. You can inherits and extends this class so that values you need will be given to `Executor` on it's evaluation.
  
```cs
public class OriginalExecutionContext : ExecutionContext
{
    public readonly DateTime Created;
    public readonly OriginalEvaluationContext EvaluationContext;
  
    public EvaluationContext(OriginalEvaluationContext evaluationContext)
    {
        Created = System.DateTime.Now;
        EvaluationContext = evaluationContext;
    }
}
```
  
_Note: If you want to use customized `ExecutionContext`, it should be given as the generics type parameter to classes which extend these classes: [RootElement](#rootelement ), [ContextManager](#contextmanager ), [CallbackManager](#callbackmanager ), and [GestureMachine](#gesturemachine )._
  
## RootElement
  
  
  
`Crevice.Core.DSL.RootElement<EvaluationContext, ExecutionContext>` is the root element of the tree of gesture DSL. You can start definiting of your gestures with `When()` function. See [Gesture DSL](#gesture-dsl ) for more details about it.
  
`SimpleRootElement` is a class which is simplified about it's generics types, and is able to be created without generics parameters.
  
```cs
var root = new SimpleRootElement();
var whenever = root.When(ctx => {
    return true;
});
whenever
.On(keysA[0]) 
.Do(ctx => {
    // When PressEvent and ReleaseEvent of keysA[0] are given to GestureMachine,
    // then this code will be executed.
});
```
  
## ContextManager
  
  
  
`Crevice.Core.Context.ContextManager<EvaluationContext, ExecutionContext>` manages `ctx` in the functions like `When()`, or `Do()` on gesture DSL. If you want to initialize `EvaluationContext` or `ExecutionContext` in your own way, you can do it with inheriting and extending this class.
  
```cs
public class OriginalContextManager : ContextManager<OriginalEvaluationContext, OriginalExecutionContext>
{
    public override OriginalEvaluationContext CreateEvaluateContext()
        => // Initialization code for `OriginalEvaluationContext`.
  
    public override OriginalExecutionContext CreateExecutionContext(OriginalEvaluationContext evaluationContext)
        => // Initialization code for `OriginalExecutionContext`.
}
```
  
`SimpleContextManager` is a class which is simplified about it's generics types, is able to be created without generics parameters, and is overrided the default initializer for `EvaluationContext` and `ExecutionContext`. 
  
```cs
var contextManager = new SimpleContextManager();
```
  
_Note1: You should override `CreateEvaluateContext()` and `CreateExecutionContext()` if you create a class inherits ContextManager, or else these functions throw an `NotImplementedException`._
  
_Note2: If you want to use customized `ContextManager`, it should be given as the generics type parameter to classes which extend these classes: [CallbackManager](#callbackmanager ), and [GestureMachine](#gesturemachine )._
  
## CallbackManager
  
  
  
`Crevice.Core.Callback.CallbackManager<GestureMachineConfig, ContextManager, EvaluationContext, ExecutionContext>` manages callbacks of `GestureMachine`. 
  
`SimpleCallbackManager` is a class which is simplified about it's generics types, and is able to be created without generics parameters.
  
  
```cs
var callbackManager = new SimpleCallbackManager();
```
  
Avaliable event properties are the following:
  
```cs
callbackManager.StrokeReset += (sender, e) => { };
callbackManager.StrokeUpdate += (sender, e) => { };
callbackManager.StateChange += (sender, e) => { };
callbackManager.GestureCancel += (sender, e) => { };
callbackManager.GestureTimeout += (sender, e) => { };
callbackManager.MachineStart += (sender, e) => { };
callbackManager.MachineReset += (sender, e) => { };
callbackManager.MachineStop += (sender, e) => { };
```
  
See [Config - Events](#events ) for the details.
  
## GestureMachine
  
  
  
`Crevice.Core.FSM.GestureMachine<GestureMachineConfig, ContextManager, EvaluationContext, ExecutionContext>` is the main component of `CreviceLib`. 
  
`SimpleGestureMachine` is a class which is simplified about it's generics types, and is able to be created without generics parameters.
  
```cs
var keysA = new SimpleKeySetA(maxSize: 10);
var root = new SimpleRootElement();
var gm = new SimpleGestureMachine();
gm.Run(root);
gm.Input(keysA[0].PressEvent);
gm.Input(keysA[0].ReleaseEvent);
// ...snip...
gm.Dispose();
```
  
_Note: This class is `IDisposable` so it should be disposed by calling `Dispose()` after using it._
  
## Physical and logical event types
  
  
  
`CreviceLib` supports physical and logical event types, for the abstraction of multiple input devices. In contrast to `Input()` function of `GestureMachine` which only takes physical event, `On()` function in gesture DSL takes both physical and logical events. In case a logical event be given to `On()` as the arguement, and a physical event corresponding to it be given to `Input()`, `GestureMachine` will treat it correctly in their relationship on physical and logical event types.
  
## FSM of GestureMachine
  
  
  
`GestureMachine` has a state representing it's current context. The initial state is the instance of `State0`, and the others are of `StateN`. `N` of `StateN` means an natural number grater than 0. So, there is no limit for the depth of the context.
  
```mermaid
graph TD;
    A["State0 (Initial state)"]-->B["StateN"];
    B-->A;
    B-->C["StateN'"];
    C-->B;
    C-->D["StateN''"];
    D-->C;
    D-->E["..."];
    E-->D;
```
  
  
_Note: Crevice 3.x has the limit for the depth of the context. This limitation has been relaxed on Crevice 4.0._
  
`CreviceLib` supports several features which naturally be in need of like the following: **cancel**, **timeout**, and  **reset**. `GestureMachine`, `State0`, and `StateN` should support this; therefore the flow of process and the implimentations of these classes are a bit complexed, but these are in reason, so the following gudance may be sutable even for those new to FSM (Finite state machine) to learn it.
  
### Result
  
  
  
`Crevice.Core.FSM.Result` is a class representing the result of `Input()` function which is implemented in `State`, `State0`, and `StateN`.
  
`Result` has two properties, `EventIsConsumed` and `NextState`. `EventIsConsumed` represents the event given to `Input()` was consumed or not, and `NextState` represents the next state of `GestureMachine`; this can be itself and can be `State0` directly from deeper one, for example `StateN` (N=3).
  
`Result` can be initialized as following:
  
```cs
Result.Create(eventIsConsumed: false, nextState: this);
```
  
 _Note: `GestureMachine` also has `Input()` function, but it is not same to this; it calls `Input()` of internal `State` and returns boolean value, `Result.EventIsConsumed`._
  
### State
  
  
  
`Crevice.Core.FSM.State` is parent abstract class for `State0` and `StateN`.
  
`State` have a only one property `Depth`,
```cs
public int Depth { get; }
```
  
a constructor,
```cs
public State(int depth)
{
    Depth = depth;
}
```
  
virtual functions,
```cs
public virtual Result Input(IPhysicalEvent evnt)
    => Result.Create(eventIsConsumed: false, nextState: this);
  
public virtual State Timeout()
    => this;
  
public virtual State Reset()
    => this;
```
  
and functions for utility, used by child classes,
```cs
protected static bool HasPressExecutors(
    IReadOnlyList<IReadOnlyDoubleThrowElement> doubleThrowElements)
    => doubleThrowElements.Any(d => d.PressExecutors.Any());
  
protected static bool HasDoExecutors(
    IReadOnlyList<IReadOnlyDoubleThrowElement> doubleThrowElements)
    => doubleThrowElements.Any(d => d.DoExecutors.Any());
  
protected static bool HasReleaseExecutors(
    IReadOnlyList<IReadOnlyDoubleThrowElement> doubleThrowElements)
    => doubleThrowElements.Any(d => d.ReleaseExecutors.Any());
```
  
and used mainly in user script; `State` may be returned as a parameter of callback events. The following functions are useful for treating convertion of `State` to it's child class. It is hard to do it without these, because `State`, `State0`, and `StateN` are heavily parameterlized by generics types.
  
```cs
public bool IsState0 => GetType() == typeof(State0<...snip...>);
public bool IsStateN => GetType() == typeof(StateN<...snip...>);
  
public State0<...snip...> AsState0()
    => this as State0<...snip...>;
  
public StateN<...snip...> AsStateN()
    => this as StateN<...snip...>;
```
  
Here, `State` is already supports basic functions, `Input()`, `Timeout()`, and `Reset()`.
  
`Input()` of `State` always returns `Result(eventIsConsumed: false, nextState: this)`. This means the event given to this will never be consumed.
  
```mermaid
sequenceDiagram
    participant A as GestureMachine
    participant B as State
    A->>B: Input(event)
    B->>A: Result(eventIsConsumed: false, nextState: this)

```
  
`Timeout()` and `Reset()` of `State` always returns itself. This means if the state of a `GestureMachine` is `State`, it will not be effected by these functions.
  
```mermaid
graph TD;
    A["State"];;
    A-->|"Timeout()"|A;
    A-->|"Reset()"|A;
```
  
_Note: Generics type patameters are abbreviated on the avobe codes for readability. If you want to read the original code, see [Core.FSM.State.cs](https://github.com/creviceapp/creviceapp/blob/9a090645bb5255680e50331fb3f1f9cdfcdb7e63/CreviceLib/Core.FSM.State.cs )._
  
### State0
  
  
  
`Crevice.Core.FSM.State0` is the initial state. This class inherits `State`.
  
  
### StateN
  
  
  
`Crevice.Core.FSM.StateN` represents states in which the depth grater than 0. This class inherits `State`.
  
  
  