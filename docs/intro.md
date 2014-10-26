## Introduction to iViewer scripting


iViewer is a powerful and versatile generic home automation remote control software. Without any scripting, you can perform complex tasks like parsing data from remote systems, filling your UI elements, responding to events, sending commands, etc.

In some cases, though, you want to have more control and more flexibility over the way you respond to events, process and generate data. To this end, you can use JavaScript to drive iViewer, talk to remote systems, watch network events and perform more complex tasks than could be done with simply parsing input with regular expressions.

iViewer defines a complete set of Application Programming Interfaces (APIs) that allow your JavaScript code to interact with the software. Your JavaScript files are loaded along with your GUI, and stay in memory for as long as your GUI is opened. This means that you can create scripts that keep context data in the long run. To keep information for longer periods of time (across sessions), you can [use Persistent Global Tokens](intro.md#how_do_I_store_information_accross_launch_sessions) (as defined in the GUI file). Since there is no actual limit to the size of the data a token can contain, you can take advantage of this to store complex structures of data that will be saved to storage and be restored the next time your GUI is loaded.


The APIs defined by iViewer allow for:

* Getting and setting joins and tokens
* Sending data to remote systems
* Receiving data from remote systems (through Feedback Items)
* Watching and being notified when select joins change values
* Manipulating lists
* Performing external network requests (HTTP and HTTPS) in a very easy to use fashion
* Manipulating and animating GUI element properties (position, opacity, scale) in order to build compelling dynamic user interfaces
* Computing hashes and CRCs
* Being notified of network condition changes (network addresses, network status)
* Publishing and discovering network services
* Controlling video elements

These are just the topics covered by the APIs provided by iViewer itself. Your JavaScript code can range from very simple to large and complex, from simple single file JavaScripts to complex, modular applications with reusable code modules.

To help you with developing and debugging your scripts, iViewer provides an easy to use Remote Debugger that runs in your computer's web browser. Using the Remote Debugger, you can inspect live changes to joins, connections to remote systems, see various events happening in your GUI and monitor cache status.



## Fundamentals of JavaScript in iViewer

With iViewer, you write JavaScript code very much the same way than you'd write it for a web browser. Similarly to a web page, your code:

* is loaded when the GUI loads,
* is put to sleep, as well as the rest of the GUI when iViewer goes to background,
* stays in memory and will run for as long as the GUI is open,
* calls into iViewer in an asynchronous fashion (calls you make are not executed immediately, but start executing in iViewer once your code returns. Your calls, however, are being executed in the order you wrote them),
* makes calls into iViewer and waits for returned data asynchronously, very much the same way you'd make requests to a remote website from the browser.

This last point is particularly important to understand when you write JavaScript code for iViewer. All the API calls that return information do so through a *callback function* that you provide. When the data is available, iViewer will call your callback function with the data you requested.

### Load-time code execution

Similar to the way JavaScript code loads in a web page, any code at the top level of your JavaScript files (code that isn't itself enclosed in a function) is executed as soon as the script loads in iViewer. This is important, as this implies that by the time this code is executed, the iViewer environment is not fully operational yet. The only thing you can assume with certainty at startup time is that iViewer's own environment basic setup has been performed by the time your script loads, as it is being loaded prior to loading your own scripts. Consequently, the `CF` global variable representing the iViewer environment is guaranteed to be available at that time.

We very strongly recommend not having any code sitting at the root level of your scripts, other than code that sets up the `CF.userMain` function, and for code that performs module setup.

For more information about the startup mechanism, see the associated [Startup Documentation](startup.md).
    
### Asynchronous code execution

Asynchronous code execution is a concept where iViewer will:

* enqueue the `CF.*` calls that you make from your code, and start executing them at a later time (usually by the time your currently executing code returns)
* provide requested data in an asynchronous fashion, by the means of a *callback function* that you provide.

Essentially, every time you need to get information back by calling into iViewer (for example using the `CF.request()`, `CF.getJoin()`, `CF.getJoins()` etc. calls), 


Take, for example, this code snippet:

    // Get a join value, show the result
    CF.log("First set s1");
    CF.setJoin("s1", "Hello, world!");
    CF.log("Now get s1 back");
    CF.getJoin("s1", function(join, value, tokens) {
       CF.log("Join s1 is " + value);
    });
    CF.log("Change value of s1");
    CF.setJoin("s1", "Another value");

Now let's look at what you can see in the [Remote Debugger](debug.md)'s log panel:

    > First set s1
    > Now get s1 back
    > Change value of s1
    > Join s1 is Hello, world!

Even though the string `Join s1 is Hello, world!` is correct and shows up as expected, notice how the string `Change value of s1` appears **before** the `CF.log` call in the line above. Here is the flow of code execution:

![JavaScript execution flow](/assets/jsflowexample.png "JavaScript execution flow")

Steps **1** through **6** are scheduled for execution in iViewer. They will start executing sequentially once your code returns to iViewer. When iViewer reaches step 4, it schedules your callback for execution, with the correct value, but the actual execution will only happen after all the outstanding calls into iViewer finish executing.

This can take some time to get comfortable with. Fundamentally, this is not very different from the way traditional JavaScript executes in a web browser, particularly when it makes requests whose result can come back at a later time.

### JavaScript compatibility

Your JavaScript code is executed by your device's WebKit engine, in the context of a hidden.md webview. This has several implications and some limitations:

* Your JavaScript code is loaded along with the GUI, and stays in memory for as long as the GUI is not closed
* Consequently, your code can keep contextual information that remains valid for the whole duration of the GUI use
* You can use timers (JavaScript's `setTimeout()` function) to schedule code execution at regular intervals.
* Your are tied to the WebKit JavaScript implementation, which is the same as the one used in Apple's Safari desktop browser.
* Your code runs concurrently to the main (User Interface) thread of execution. This means that long JavaScript operations like iterating over a large amount of data could potentially block the user interface for as long as your loop runs.

This last point is important, and deserves an explanation: due to technical constraints, your JavaScript code is being run in the main thread of the application where user interface events are also processed. This means that any lengthy operation you perform in JavaScript will block the main thread (and will prevent iViewer from responding to user interaction) until your operation completes.

An additional constraint enforced by the operating system is that any bit of javascript code executed **must not** take longer than 10 seconds to execute. If it does, on iOS devices the operating system will kill the application, thinking the app has crashed or blocked.


### Debugging

One of the major issues with working on mobile platforms is the complexity of debugging code that runs on the device. For JavaScript, in particular, it is very difficult, particulary when the code runs embedded in an application.

To make debugging easier, we have implemented a [Remote Debugging Monitor](debug.md) which serves two purpose: to provide internal information about iViewer (join states, remote systems states, etc), and to let you debug your JavaScript code as you could do for code that runs in a web page.

For in-depth information about debugging JavaScript in iViewer, please refer to the [Remote Debugging Monitor](debug.md) documentation.


### JavaScript modules

iViewer provides a mechanism allowing you to develop reusable JavaScript code modules: you can code a generic module, and reuse it in several applications without any modification.

To this end, iViewer has a loading mechanism where modules can specify a setup function. All modules have their `setup` function called prior to your `CF.userMain` function being called.

To learn more about how you can make reusable code modules for iViewer, please read the [Module Development Guidelines](modules.md) document.
