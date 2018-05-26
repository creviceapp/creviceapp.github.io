  
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
  
If you using an application, it is on the foreground. For example, it to be Chrome here. Activate the above userscript, and then press the middle button. The following messages will be output:
  
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
  
Next, if you move the cursor and point the Windows' taskbar and press the middle button, the following messages will be output:
  
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
  
`ForegroundWindow` and `PointedWindow` are `WindowInfo`, so there are more powerful functions like `GetChildWindows()` they have.
  
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
  
The following logs are dump data of Chrome and Explorer:
  
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
  
Children of `WindowInfo` is also provides the functions, so that you can dig into more deeper and deeper.
  
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
  