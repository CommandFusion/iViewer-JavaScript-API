## Script startup and setup

### Script execution and lifetime

When iViewer loads a new GUI, it looks whether the GUI has Javascripts set up in the project. If so, it will load them all in a single space (a hidden webview) and run them. Any code that is not itself inside a function is being executed immediately at load time. Once loaded, your scripts stay in memory for all the duration of the GUI run (that is, until user switches GUI or quits the application).

Much like in an.md page declaring scripts in its <HEAD> tag, your scripts are therefore parsed once when loaded and the top-level code (all the code that is not embedded in a function) gets executed. But beware that by the time the script loads, the _bridge_ between Javascript and iViewer is not fully available yet. Therefore, you should limit the tasks you do in your top-level Javascript code to:

* Create and initialize your global variables
* Assign a function to the **CF.userMain** variable
* or if you are developing a module, add your module information and setup function to the modules list

If there is a syntax error in your top-level Javascript, the script load phase will silently fail. Make sure you run and thoroughly test your code using the [Remote Debugging Monitor](debug.md#remote_Debugging_Monitor_user_interface) first, and use the debugging tools at hand in the desktop web browser to track any error in your code. Additionally and for debugging purposes, you can setup a web view in your GUI pages that is marked in GUI Designer as **Use for script debugging output**. This is where your `CF.log()` calls will display text if you're not currently connected through the Remote Debugging Monitor.

### Startup sequence

iViewer has a definite startup sequence you can rely on. It is designed to allow your scripts and modules to setup everything that it needs even before some parts of your GUI are started. Here is what goes:

* GUI loads
* Scripts load
* JavaScript modules initialization functions are called
* Your `CF.userMain()` function is called
* All the `CF.*` calls you make in your `CF.userMain()` function are executed
* iViewer starts external systems connections

This can have some consequences on what you can do in your `CF.userMain()` function. In particular, you should not try to send data to UDP systems using `CF.send()` in your `CF.userMain()` function, as these are not started yet and the data to send is not enqueued.

Rather, if you need to send data early on (for example, to multicast systems), your options are:

* Use `setTimeout()` to schedule execution of your `CF.send()` calls at a later time,
* or watch the `CF.ConnectionStatusChangeEvent` using `CF.watch()` and when your callback is triggered with a `true` connected status, start sending your data.

Here is an example:

    CF.userMain = function() {
        // ... perform your initializations here ...
        
        // We want to send data to a UDP multicast system as early as possible. UDP systems
        // report that they are "connected" when they are ready to send and receive data
        CF.watch(CF.ConnectionStatusChangeEvent, )
    };

### Module initialization

You can write generic, reusable JavaScript code modules that you share among several projects. This is a good opportunity to maximize your investment in writing code, and minimize the development time for each project. For example, you could develop modules that abstract the dialog with specific hardware you are using in your projects. If you design your module to be generic enough and not be tied to one specific job, then you'll save a lot of time on each project making use of the module.

The [Module Development Guidelines](modules.md) document describes how you can develop modules. To learn more about the startup and initialization sequence, read the [Module registration and startup](modules.md#module_registration_and_startup) chapter.


### One-time initialization of your script

You can provide multiple Javascript files that are loaded along with your GUI. When all the scripts have been cached and loaded, iViewer looks whether you did set a `CF.userMain` function. If you did, it will call it first thing after initial [CF variables](globals.md) setup and modules initialization.

The function you provide in `CF.userMain` can perform all tasks that initially set up your GUI, including watching for events, setting some initial join values, etc.

Refrain from calling any `CF` function directly outside of one of your functions in your Javascript code. If you do this, your call into iViewer will be executed once the environment is ready, but before your `CF.userMain` function is called. Rather, at GUI startup take advantage of iViewer calling your `CF.userMain` function.

Here is an example `CF.userMain` function. If watches for a few events and starts a Bonjour lookup to find some network services:

    CF.userMain = function() {
        // iViewer initialization is now complete and the CF environment is fully available
        
        // Watch for page flips
        CF.watch(CF.PageFlipEvent, onPageFlip);
        
        // Watch for network status, get initial status
        CF.watch(CF.NetworkStatusChangeEvent, onNetworkStatusChange);

        // Watch for important system connects and disconnects
        CF.watch(CF.ConnectionStatusChangeEvent, "TEST-SYSTEM", onTestSystemConnectionStatusChange, true);
        CF.watch(CF.ConnectionStatusChangeEvent, "SERVER SYSTEM", onServerSystemConnectionStatusChange, true);
        
        // Watch for some crucial join changes
        CF.watch(CF.JoinChangeEvent, ["d10", "d11", "d12", "a50"], onImportantJoinChange);
    };
    
    // Callback function for page flips
    function onPageFlip(previousPage, newPage) {
        // We get called when switching pages
        CF.log("Flipped from page " + previousPage + " to page " + newPage);
    }
    
    // Callback function for network status change
    function onNetworkStatusChange(networkStatus) {
        if (networkStatus.hasNetwork) {
            CF.log("Network is connected");
            CF.log("Device IP address is " + networkStatus.ipv4address);
        } else {
            CF.log("Device is disconnected from the network");
        }
    }

    // Callback function for my outgoing TEST-SYSTEM status change
    function onTestSystemConnectionStatusChange(system, connected, remote) {
        if (connected) {
            // Keep the address of the remote we are connected to
            CF.log("TEST-SYSTEM is CONNECTED to " + remote);
        } else if (remote == null ) {
            // Initial status
            CF.log("Initial status: TEST-SYSTEM not connected");
        } else {
            // Lost the connection to TEST-SYSTEM
            CF.log("TEST-SYSTEM disconnected from " + remote);
        }
    }
    
    // Callback function for my incoming SERVER SYSTEM status change
    // We receive notifications for new client connections,
    // and disconnections from connected clients
    function onServerSystemConnectionStatusChange(system, connected, remote) {
        if (connected) {
            CF.log("New client connected to SERVER SYSTEM: " + remote);
        } else {
            CF.log("Client " + remote + " disconnected from SERVER SYSTEM");
        }
    }
    
    // Important join changes
    function onImportantJoinChange(join, value, tokens) {
        CF.log("Important join change: new value for " + join + " is " + value);
    }

## Execution context for GUI Designer Javascript

In GUI Designer, you can define small bits of Javascript code to execute as the result of a button action, in a command, gesture, system handler, etc. In some cases, iViewer creates a *local scope* for you, and predefines a few variables before executing the code you typed in GUI Designer. You can take advantage of this to gather additional information about the context in which your code is executed.

### Button javascript action

Each button can carry a small bit of Javascript, either as a basic action, or in its advanced action. The code you write there will be executed by iViewer in a context that can be seen as a function like this:

    function() {
        var join = "button join";
        var tokens = { /* button tokens */ };
        var list = "list join";     // set to null if button is not in a list
        var listIndex = index;      // number valid only if list != null

        // Your javascript code is inserted here
    }

iViewer automatically creates the bit of code above and inserts the JavaScript code that you provided in guiDesigner. Therefore, you can always gather information about the button that triggered execution of the code.

For example, say you have a button "d1" in a subpage that is the list item subpage for list "l1". Your Javascript can be a function call that passes the aforementioned variables: `onButtonPressed(join, list, listIndex);`

![Javascript code in a Button action](/assets/buttonjs.png "Javascript code in a Button action")


What iViewer will execute when button d1 in the third list item is pressed will be:

    function() {
        var join = "d1";
        var tokens = { "[join]": "d1" };
        var list = "l1";
        var listIndex = 2;
        // Will call onButtonPressed("d1", "l1", 2)
        onButtonPressed(join, list, listIndex);
    }


### Remote system send handler

In GUI Designer, you can configure an external system to pass the data to send to a Javascript handler instead of directly sending it itself. Your script can massage the data any way it needs to (for example, transform the data in an encapsulated binary packet, add header and checksum, etc). While you should use this feature sparingly for performance reasons (going through Javascript to send several tens messages per second could greatly degrade performance), it can be a very useful feature in some cases.

Your Javascript send handler is responsible for calling CF.send() once it has transformed the data (or even to skip sending data if it is determined that it should not be sent, i.e. to avoid redudant messages). iViewer provides you with an execution context where the system name and data to send are defined as local variables, and your code is executed wrapped in a function like this:

    function() {
        var system = "system name";
        var data = "data to send";
        // Your Javascript code is inserted here
    }
    
Here is an example where we call a function named `onSendDataToCustomBox(system name, data);`. The function will in turn take appropriate actions, and call `CF.send()` with the data properly massaged:

![Defining a Javascript send handler for a System](/assets/systemjs.png "Defining a Javascript send handler for a System")

The actual code executed by iViewer when, for example, a command generates a data string `"Hello, world!"` to send to this system is:

    function() {
        var system = "Custom Box";
        var data = "Hello, world!";
        onSendDataToCustomBox(system, data);
    }

Your send handler function could for example add binary packet markets around the data:

    function onSendDataToCustomBox(system, data) {
        if (system == "Custom Box") {
            // Send \xF5\xF5 <data> \xF4\xF4
            CF.send(system, "\u00F5\u00F5" + data + "\u00F4\u00F4");
        }
    }

Note that from your Javascript code, calls to `CF.send()` always send the data straight to the remote system without going through the Javascript send handler.

### Command send handler

Very much like System send handler, each command can define a small bit of Javascript code that executes when the command runs, and can optionally be responsible for sending out the final data to either the system targeted by the command, or even to another system.

The local execution context is the same as for a [System send handler](#remote_system_send_handler): local variables with the system name and the data generated by the command are being set up, and your code is run.

For example, consider the case where you create a command on a Loopback system that generates parameters for an HTTP request to be sent out by Javascript:

![Piping commands to Javascript](/assets/commandjs.png "Piping commands to Javascript")

Note that we have checked the __Javascript sends command value?__ box here, so the loopback system will not directly receive the data that was generated by the command.

Assuming the `[artist]` token value is `"Dire Straits"`, the local execution context for your code will be:

    function() {
        var system = "Loopback";
        var data = "Dire Straits";
        onSendHTTPRequest("artist", data);
    }

Now you can process this command by sending out an HTTP request to a server, and handle the response in your Javascript:

    function onSendHTTPRequest(command, value) {
        var url = "http://" + myServerURL + "?command=" + encodeURIComponent(command) + "&value=" + encodeURIComponent(data);
        CF.request(url, function(status, headers, body) {
           if (command == "artist") {
               // process artist response body here
           }
        });
    }

This is a powerful tool to make it easier to create your user interface, separating the actions you wire in the GUI from the actual actions that are taken by your Javascript.


### Gesture action Javascript

Gestures can be configured to execute Javascript code in GUI Designer:

![Gesture Javascript](/assets/gesturejs.png "Gesture Javascript")

When attaching Javacript to a gesture, a local variable named `gesture` is defined. This is an object which contains properties related to the gesture that was recognized (specific properties depend on the gesture, complete list can be found below). The properties are named after the tokens that are made available for command execution, minus the square brackets in the token name for convenience.

Additionally, the local variables `join`, `tokens`, `list`and `listIndex` are also being set up for you to let your code know about the environment (the object) that contains the gesture.

The standard local variables are defined as:

* `join` (_String_): the join of the on-screen object that defines the gesture
* `tokens` (_Object_): the tokens of the aforementioned object
* `list` (_String_): if the object defining the gesture is in a list item subpage, the join of the list. Otherwise, `null`.
* `listIndex` (_Number_): if the object defining the gesture is in a list item subpage, the index of the item.
* `gesture` (_Object_): an object defining the properties of the gesture that just executed. Contents depend on the gesture, see below for per-gesture definitions.

Your code is wrapped in a function like this (executing the code typed in the **Script** box in the screenshot above):

    function() {
        // This is a swipe left gesture tokens object
        var join = "object join";
        var tokens = { /* object tokens */ };
        var list = "list join";     // set to null if object defining the gesture is not in a list
        var listIndex = index;      // number valid only if list != null
        var gesture = {
            type: "swipe",
            direction: "left",
            x: 150,
            y: 200,
            startx: 230,
            starty: 200,
            deltax: -80,
            deltay: 0
        };
        // your gesture Javascript inserted here
        onSwipeLeft(Math.abs(gesture.deltax));
    }

Here is how your could handle this gesture in your `onSwipeLeft()` function:

    function onSwipeLeft(pixels) {
        if (pixels > 200) {
            // Fast, long swipe on left: could scroll two subpages
        } else {
            // Short left swipe: could scroll one subpage
        }
    }

Each gesture provides a minimal set of properties:

* `type` (_string_): the gesture type. Can be one of "swipe", "press", "tap", "rotate", "pinch"
* `x` (_number_): the current horizontal position
* `y` (_number_): the current vertical position
* `startx` (_number_): the touch-down horizontal position when the gesture started
* `starty` (_number_): the touch-down vertical position when the gesture started
* `deltax` (_number_): the horizontal distance beween the current position and the touch-down position
* `deltay` (_number_): the vertical distance between the current position and the touch-down position

The `"swipe"` gesture adds the following properties:

* `direction` (_string_): the gesture direction (one of "left", "right", "up", "down")


The `"rotate"` gesture adds the following properties:

* `velocity` (_number_): the velocity of the rotation gesture in radians per second
* `rotation` (_number_): the rotation of the gesture in radians since its last change.


The `"pinch"` gesture adds the following properties:

* `scale` (_number_): the gesture scale. `1.0` represents two fingers at the original distance. `2.0` represents two fingers spread twice the original distance. `0.5` represents two fingers spread half the original distance.
* `velocity` (_number_): the velocity of the pinch in scale factor per second. 


The `"pan"` gesture adds the following properties:

* `velocity` (_number_): the velocity of the pan gesture in points per second. 

