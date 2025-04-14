---
title: "Consuming a C++ DLL in dotnet core"
datePublished: Mon Sep 14 2020 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0irxi500xvzfs1hdju0zoy
slug: consuming-a-c-dll-in-dotnet-core
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380809570/hC_KS8sXV.jpeg
tags: cpp, csharp, async, dotnetcore

---


For a client, I need to open a drawing in a specialised program. When the user closes the program it should upload the drawing to a server. What looks like a straightforward start of a process, turned into a mindbogglingly number of failures. Journey with me through all the steps I took to get this seemingly simple task to work.

After a quick `Process.Start('program', 'path/to/drawing')` didn't work, I knew something was wrong. So I contacted the developers of the software and asked for clarification. They sent over very helpful documentation and I was on my merry way again.

Apparently, they have a .dll that is included with their program and that is how an external application can communicate with it. Phew, not so hard after all. So I modelled the external calls to methods and decorated them with the `[DllImport]` attribute.

```
private delegate int Callback(int code, string message);
[DllImport("path/to/dll", CharSet = CharSet.Unicode, SetLastError = true, CallingConvention = CallingConvention.Cdecl)]
private external static int RegisterCallback(Callback method);

[DllImport(/\*see previous\*/)]
private external static int StartApplication();

[DllImport(/\*see previous\*/)]
private external static int DoWork(string command);
```

In this flow, which is apparently more common in C++ than dotnet, I first register a callback method, start the application and then send work to it. The registered callback method handles status updates and other information that gets sent back. This way, a single marshalling needs to be done and multiple `DoWork` methods can be called, even different external functions, and they'll all report back to the registered callback method.

The `code` parameter in the callback method determines what will be in the `message` and how to interpret it. For example, the `code` 0 can refer to application initialisation and the message then is "SUCCESS" or "FAILURE: with a specific reason". More complex information can be sent back as well. For example, `code` 2 could be that the application has saved the drawing, the `message` then contains XML with the location of the drawing and some meta information about the changes that were made.

Although this is quite an interesting approach, as most examples of interop use a specific callback in each function. This approach however, does pose a few interesting challenges. I found that the callback was never invoked. To get more error information, I had to enable `SetLastError = true`. This then allows me to call `Marshal.GetLastWin32Error()`. That function returns a number that I can then look up in the [official Microsoft docs](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-).

Unfortunately, I forgot the error so I can't reference it here. It did lead me to an article to set the `CallingConvention` to `Cdecl`. This is to indicate that I am responsible to clean the stack and not the code that I'm calling. This has to do with defaults in both runtimes, C++ uses __cdecl convention while the default for C# is __stdcall. The conventions need to align to allow the processes to talk to each other.

Huzzah, done... No wait, too soon, still no callback. The application that I am calling is starting, but it also gets cleaned up too fast. I'm not sure what C++ techniques are used, but either Windows or the GC are cleaning up the application before it could properly start.

What I did notice is that the `StartApplication()` method returns almost immediately and I have to build a way to wait for the callback that says that the full initialisation of the application is done. I assume that the `StartApplication` method sets a process in motion and reports that the process kicked off correctly. Dotnet on the other hand figures that the invoked method was done and doesn't need all those resources anymore, including the application that is halfway through its startup process.

If anybody is interested in the wait mechanic that I tried here: I used a [`TaskCompletionSource<bool>`](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcompletionsource-1?view=netcore-3.1) to return whether the initialisation would have completed succesfully based on the message the callback would receive. This way, I could use `await` to wait for the external program to finish.

Here's where the developers of that program gave me a helping hand: they told me dotnet needs to keep a reference to the method that is being called and the way to do that is to use functions from the _Kernel32_ library to load the functions into memory and only release them when they are done.

```
[DllImport("kernel32.dll", SetLastError = true)]
private static extern IntPtr LoadLibrary(string pathToLibrary);

[DllImport("kernel32.dll", SetLastError = true)]
private static extern IntPtr GetProcAddress(IntPtr libraryReference, string procedureName);

[DllImport("kernel32.dll")]
private static extern bool FreeLibrary(IntPtr libraryReference);
```

So instead of using a `DllImport`, I have to do this by hand.

The `LoadLibrary` function is used to load the library into the running application. The `IntPtr` that is returned is a pointer to a memory address where the library is loaded. The `GetProcAddress` function takes the pointer returned by `LoadLibrary` and the name of the exposed method. When the library has fulfilled it's purpose, so after the `DoWork` function is done and the callback method has received the done signal, I need to clean up the references to the library by feeding the pointer to the library to the `FreeLibrary` function.

It's comparable to how `DllImport` works behind the scenes. Except that `DllImport` is applied to a method that can be called. The `GetProcAddress` only returns another pointer to where the function I want to call is loaded into memory. To convert the pointer to an callable function, I need to marshal it. In full dotnet framework, there is a static function `Marshal.GetDelegateForFunctionPointer` which takes the `IntPtr` and the `Type` of a `delegate`. It then returns an object that can be cast to the delegate. If you are using dotnet core, there is a generic function `Marshal.GetDelegateForFunctionPointer<T>`.

```
internal class CustomApplicationIntegration : IDisposable
{
  // kernel32 imports here
  private readonly IntPtr _dllPointer;
  public CustomApplicationIntegration(string pathToLibrary)
  {
    _dllPointer = LoadLibrary(pathToLibrary);
  }

  public void Dispose()
  {
    if (_dllPointer != IntPtr.Zero)
      FreeLibrary(_dllPointer);
  }

  private delegate void Callback(int code, string message);
  private Callback _keepAlive;
  private bool _applicationDone = false;
  private delegate int RegisterCallback(Callback method);
  private delegate int StartApplicationDelegate();
  public void Interact()
  {
    _keepAlive = CallbackMethod;
    var registerPointer = GetProcAddress(_dllPointer, "RegisterCallback");
    if (registerPointer == IntPtr.Zero)
      throw new Exception("Something went wrong!");
    // dotnet 4.X aka full framework
    var registerCallback = (RegisterCallback) Marshal.GetDelegateForFunctionPointer(registerPointer, typeof(RegisterCallback));
    var registerResult = registerCallback(_keepAlive);
    // check that registerResult returns ok

    var startPointer = GetProcAddresss(_dllPointer, "StartApplication");
    // dotnet core
    var startApplication = Marshal.GetDelegateForFunctionPointer<StartApplicationDelegate>(startPointer);

    var startResult = startApplication();
    // check that startResult returns ok
    while (!_applicationDone)
      Thread.Sleep(100);
  }

  private static void CallbackMethod(int code, string message)
  {
    // handle callback
    _applicationDone = code == StopCode;
  }
}
```

If something goes wrong in either the `LoadLibrary` or `GetProcAddress`, they return the value `IntPtr.Zero`. That's how I can check for success, everything that is not the zero value is a valid pointer. Because the `SetLastError` is set on the `DllImports` of these functions, I can get the id of the error with the `Marshal.GetLastWin32Error` function, just like I did earlier.

At the end of the `Interact` method, there is a loop to keep waiting on the external application to send the stop code. Only when that code is received, can I continue with my application. There are functions in the library to do specific tasks and to stop the external application, but I'm omitting them here for brevity.

In the actual app, there are several loops waiting for the external application to finish some work or signal some status changes. For example, I have to wait for the application to start, before I can assign any work to it.

Now lets run this thing and see... still... no... callbacks... What I can see, is that the application is still exiting, but now it takes longer. The startup logs indicate that the library is doing it's job correctly, but that the process that is started by the library is being terminated. Windows, stop killing my processes!

The `Task.Run` is to blame here! I run the `Interact` function in a `Task` and it runs a while longer, but it doesn't protect the process the library is starting. When I changed this to a `Thread`, it solved the matter. To be honest, I have no idea why the `Task` terminates early, while the `Thread` runs as intended. If anybody knows why this is, do [get in touch](https://kenbonny.net/contact/).

```
// bad way
await Thread.Run(() =>
{
  var integration = new CustomApplicationIntegration();
  integration.Interact();
}

// the good way
var workThread = new Thread(() =>
{
  var integration = new CustomApplicationIntegration();
  integration.Interact();
});
workThread.Start();
```

I hope this runs, I hope this runs and... I wait forever on the callback function. Don't give up hope, this is the last problem to solve, but what a problem it is. The issue is that I created a deadlock: the external application invokes the callback and waits for it to return (eventually it would crash after a looong time) and my code waits for the callback but is too busy waiting to notice the callback.

Because it's a console application, there is no main event loop. The console expects there to be only one way through the application. It can wait on async methods, but it does not expect any application events. What this means is that it does not see the event of the callback, because it does not originate from within the application.

Simple callbacks that happen within the execution of the called C++ code expect the callback method to be invoked. Because I registered the callback and then went on to other functions that do not have that callback explicitly referenced, the console is not ready for these invocations. Instead, the Windows event loop is used. Since the console app does not have a main event loop, they simple do not get processed. This is the same reason why [Timers and Sockets do not work as expected in console applications](https://docs.xojo.com/ConsoleApplication.DoEvents).

The most [simple solution to this problem](https://stackoverflow.com/questions/63409981/the-c-sharp-callback-function-is-not-called-from-c-code/) is to convert the console app to a WinForms app and call `Application.DoEvents()` in the `while` loop in the `CustomApplicationIntegration` class.

```
while (!_applicationDone)
{
  Thread.Sleep(100);
  Application.DoEvents();
}
```

To me, it feels a bit hacky. That is because in a normal WinForms app, the `Application.DoEvents()` method gets handled by the framework. It has a number of guards to handle concurrency when multiple events fire in quick succession. The official Microsoft documentation seems to [discourage manually waiting on `Application.DoEvents`](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.application.doevents?view=netcore-3.1#remarks). Since I'm only expecting events from one external source, there should be no problems. Should...

Wow, that was a lot to take in. From `DllImports`, over marshalling functions and waiting for an asynchronous response to finally transforming the console app to a WinForms app so Windows events could be handled. Somebody has earned their favourite beverage, so go on and treat yourself to a nice beer, wine or two fingers of whiskey. I know I'm going to enjoy all myself now.
