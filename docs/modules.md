
# JavaScript code modules in iViewer

You can write generic, reusable JavaScript code modules that you share among several projects. This is a good opportunity to maximize your investment in writing code, and minimize the development time for each project. For example, you could develop modules that abstract the dialog with specific hardware you are using in your projects. If you design your module to be generic enough and not be tied to one specific job, then you'll save a lot of time on each project making use of the module.

There are several ways you can make a module generic. A recommended design is to create a top level object for your module, with project-specific parameters that can be set or updated from your `CF.userMain` function. This could be the IP addresses or names of systems to talk to, specific data about a particular setup, etc.

## Single or multiple instance modules

When designing your module, you need to decide whether the module will be used as a single instance (singleton), or multiply instantiated in your project. For example, you may have a generic module designed to drive a single system, and you know that your projects will always have at most one of these systems on the network (could be heating or air conditioning control, a network server, etc). In this case, you'll make your module a *singleton* which is simpler to define and use.

In other cases, you plan on having multiple systems that are controlled the same way. Each system has its own unique parameters, and there may be several of them on the network at the same time. In this case, you'll create your module using a pattern that allows for multiple instances to be created.

## Module registration and startup

During the startup phase, iViewer will call each module's `setup` function first. Then once all the modules are setup, the `CF.userMain` function is called to finalize script startup.

Each module registers itself in the `CF.modules` array, providing information about the module:

* the module name
* the module setup function
* the global object to pass as `this` to the module setup function

This last bit of information calls for some explanation. JavaScript is an object-oriented language, but the `this` object (accessible from within your setup function) can be properly set only if the callers knows which object provides the function.

Here is a simplified, example module, using the *singleton* pattern:

    // Our module global object. This objects holds all of the module variables and
    // functions to avoid polluting the global name space. This sample module simply
    // performs an HTTP request, does some processing on the returned body and stores
    // the result in a 'data' variable. This simple module simply processed the result
    // of http requests by splitting the text it reads based on a separator and adding
    // the results to a data array.
    var SimpleModule = {
        // Cumulative array of all processed data
        data: undefined,
        
        // Our module setup function. Note how each function is defined like
        // a variable. Also don't forget the comma at the end of the function
        // definition, because we really are declaring a list of object properties
        // here (variables and functions)
        setup: function() {
            // Initialize data with an empty array
            // (useless, but that's for purpose of this demo)
            this.data = [];
        },

        // A public function. We can call it using DemoModule.getData();
        getData: function(url, dataReceivedCallback) {
            // to be able to access our variable, we need to keep a reference to
            // "this" because by the time the callback is called, the "this" value
            // won't point to our module anymore. This is common JavaScript practice.
            var that = this;
            
            // perform the request
            CF.request(url, function(status, headers, body) {
                if (status == 200) {
                    // ... process body here, extract the data we are looking for ...
                    var processed = body.split("##");
                    
                    // Add the processed body to our data array
                    that.data.push(processed);
                    
                    // If a callback was provided, call it now, handing over the
                    // last bit of data we processed.
                    if (dataReceivedCallback !== undefined) {
                        dataReceivedCallback.apply(null, [processed]);
                    }
                }
            });
        }
    };
    
    // At load time, the code below is executed. This is a good occasion to add our
    // module to the modules list.
    CF.modules.push({
        name: "Simple module",      // the name of the module (mostly for display purposes)
        setup: SimpleModule.setup,  // the setup function to call
        object: SimpleModule,       // the object to which the setup function belongs ("this")
        version: 1.0                // An optional module version number that is displayed in the Remote Debugger
    });

The example above shows an example of a complete, functional module that you can reuse in multiple projects. Just add it to your project's scripts in GUI Designer, and you can directly call `SimpleModule.getData(url, optional callback)`.

This examples shows a few things and good practices about module development:

* **object oriented**: your module encapsulates all its variables and functions in one or more global variables, trying to limit the global namespace pollution, and using meaningful names that make it unique.
* **reusable**: your module is reusable. If you have specific parameters that can vary from project to project (URLs, login names, password, etc), prepare variables in your module that can be initialized by your `CF.userMain` function.
* **singleton**: this module is explicitely designed to be used as-is in your project
* **commented**: when you make a module, remember to add meaningful comments for yourself, and for others to use it.


## Module development guidelines

Depending on the type of module you develop, user (yourself or another developer) may need to use a single instance of your module (directly reuse your module object as-is), or create multiple instances of the object to use them in different context. For example, you may be developing a module that controls one instance of a piece of hardware, and need multiple instances of it to control multiple pieces at the same time.

To this end, JavaScript offers several possible design patterns. Of these, you'll have to choose how you want your JavaScript object to be reused, and provide guidelines and examples as to how one should use it.

Here is an example of a design for a module object that is designed to be instantiated multiple times at once. It allows for public (accessible by users of the instance) and private (completely hidden) variables and methods. An instance on the module can be created by calling `MyModule(parameters)`, where you can specify initial parameters for the module instance to be returned.

You'll note that since this module is designed to have multiple instances, there is no need for one time setup so we don't need to add the module to the `CF.modules` list.

    // A skeleton module that encapsulates private variables and functions,
    // while making public variables and functions accessible by users of
    // module instances
    var MyModule = function(parameters) {
        // Private variables
        var somePrivateVariable = "something";
        var otherPrivateVariable = 5;
        
        // Define the object that will be returned and is the
        // new instance of the module with its public functions
        // and variables.
        var module = {
            login: "",
            password: "",
            url: "",
            dataCallback: undefined
        };

        // Private function
        function prepareInternalConnection() {
            // ... do something here ...
        }
        
        // Private function
        function processData(data) {
            // ... do something with the data ...
            var processedData = data.split(":");
            for (var i=0; i < processedData.length; i++) {
                // ... here we can individually process each received entry ....
            }
            return processedData;
        }
        
        function dataReceivedFromHardware(data) {
            // When we receive data from the hardware, we process it
            // and call the callback function for user to receive its
            // processed data
            if (module.dataCallback !== undefined) {
                module.dataCallback(module, processData(data));
            }
        }
        
        // Initialize the instance with provided parameters if any
        // When creating the module instance, user can provide a login name,
        // password, URL and callback function. If login, password or URL are
        // not provided, we use default values.
        module.login = parameters["login"] || "defaultLoginName";
        module.password = parameters["password"] || "defaultPassword";
        module.url = parameters["url"] || "http://192.168.0.1";
        module.dataCallback = parameters["callback"];
        
        // Public function that starts up this module instance, for example
        // by sending some data to the piece of hardware
        module.Startup = function() {
            // call our internal function that prepares stuff for connecting
            prepareInternalConnection();

            // ... perform the remaining of the startup sequence ...
            
            sendMessage("ready");
        };
        
        // Public function that does something remotely with the hardware
        // we're talking to
        module.DoSomethingWithHardware() {
            // ... here we do something with the hardware ...
            // ... when we get data back, we can call the provided
        };

        // Public function that cleans up (i.e. sends a message to a remote system)
        // before we're done using the instance
        module.Cleanup = function() {
            // ... 
        };
        
        // This is the most important part: we return the module object that
        // we just created. This object contains the public functions and
        // variables, and by the magic of JavaScript scoping, can still access
        // its private functions and variable (while code that uses the module
        // can't see them)
        return module;
    };

With such a module design, it becomes easy for developers to integrate your module in their code. Below is an example of use for your generic module. Note that similar to module development, we encapsulate all our script variables in a single GUI variable, which reduces namespace pollution and allows for clearer grouping of your script's variables.

    var GUI = {};

    CF.userMain = function() {
        // Create two instances of MyModule
        GUI.hardware1 = MyModule({
            login:"test",
            password:"test",
            url:"192.168.0.7",
            callback: hardwareReceivedCallback
        });
        GUI.hardware1.Startup();
        
        GUI.hardware2 = MyModule({
            login:"test",
            password:"test",
            url:"192.168.0.8",
            callback: hardwareReceivedCallback
        });
        GUI.hardware2.Startup();
    };
    
    function hardwareReceivedCallback(instance, data) {
        if (instance.url == GUI.hardware1.url) {
            // ... we received data from the first piece of hardware
        } else {
            // ... we received data from the second piece of hardware
        }
    }

In the example above, at startup time we create module instance for two pieces of hardware on our network. We define a common callback function to process data from the hardware. The module passes the instance as an argument, so we can differentiate between the two remote hardware.
