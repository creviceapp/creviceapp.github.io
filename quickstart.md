
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

The userscript file is just a C# Scripting file. You can do anything you want by writing your own script in it, or else by just copying codes from [Stack Overflow](https://stackoverflow.com/). See [Overview of C# Scripting](#overview-of-c-scripting) for more details.
