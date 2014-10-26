## General / Javascript

#### How do I store information accross launch sessions?

If you want to keep data that persists between launch sessions of your GUI, you can define Persistent Global Tokens in the GUI, store information in them and restore it when your script reloads. You are not limited to mere strings: it is perfectly acceptable to encode objects to a string (for exemple, using JSON conversion) as long as their size is not too big. You can store large data structures this way, but this is not a recommended method to store megabytes of data.

Here is an example of saving and restoring an object to a global token:

    /* Assuming that a persistent global token named 'JS_CONTEXT' has been defined in the GUI
     */
    CF.userMain = function() {
        // Read the global context that we need
        restoreContext();
    }
    
    // This is the context object in which you keep context information
    var myContext = {
        // ... arrays, objects, strings, numbers are all acceptable here ...
    };
    
    function saveContext() {
        // Convert the context object to JSON
        var contextStr = JSON.stringify(myContext);
        
        // Write it to our JS_CONTEXT global token
        CF.setToken(CF.GlobalTokensJoin, "JS_CONTEXT", contextStr);
    }
    
    function restoreContext() {
        // Read the contents of the JS_CONTEXT token
        CF.getJoin(CF.GlobalTokensJoin, function(join, value, tokens) {
            var contextStr = tokens["JS_CONTEXT"];
            if (contextStr != null && contextStr.length > 0) {
                // Convert the context string from JSON back to an actual object
                var decodedObject = JSON.parse(tokens["JS_CONTEXT"]);
                if (typeof(decodedObject) == "object") {
                    // Set our context to the decoded object
                    myContext = decodedObject;
                }
            }
        });
    }

#### How do I create strings with binary data?

Preparing strings with binary data (non-ASCII characters) is very easy. In string litterals, you can include binary bytes by writting their hexadecimal value prefixed by `\x`, for example `\x0A`.

    // A simple string with two prefix bytes: 0xF5 0xF4 and a linefeed at the end
    var str = "\xF5\xF4Hello, world\n";


You can also use Javascript's [String.fromCharCode()](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/String/fromCharCode) function to assemble binary strings using arrays. To do so, you create an array with the list of bytes you want in the string and use Javascript's [apply](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/function/apply) method to call `String.fromCharCode()` with an array of values. Since `String.fromCharCode()` creates a single string from any number of parameters, this trick is very useful to quickly assemble binary strings:

    // Convert a list of values to a string of bytes
    var array = [0x01, 0x7f, 0xAA];
    var str = String.fromCharCode.apply(null, array);

    
Now to make things even more useful, we can factor such methods into a function that will contatenate array entries into a string, while keeping an option to pass full strings in the array to minimize the work to do to prepare the array. You can use this function in your code as a generic way to assemble complex strings mixing text and binary data. We first use [Array.map()](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/map) to process each item in the array and turn it into an individual string if needed, then contatenate all strings using [Array.join()](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/join).

    // Helper function that takes an array of values and generates a string
    // If the value is a number, a byte will be added to the string
    // If the value is a string, it will be copied verbatim to the resulting string
    function binaryString(array) {
        return array.map(function(item) {
            return (typeof(item) == "number") ? String.fromCharCode(item) : item;
        }).join("");
    }
    
    // Example of use: assemble a text string prefixed with a 0x01 byte and suffixed with a 0xff byte
    var str = binaryString([0x01, "data contents", 0xff]);
    // -> this will create a string: "\x01data contents\xff"


## Joins and tokens

##### How do I check whether a join exists in a GUI?

When developing scripts that are shared among multiple GUI files, you may need to check whether your GUI has some features that others don't. Use [CF.getJoin()](gui.md#cF.getJoin) or [CF.getJoins()](gui.md#cF.getJoins) and check whether the join value. If it is `null`, the join doesn't exist in this GUI.

Example:

    // Check whether join d1000 exists in this GUI file
    var hasD1000 = false;
    CF.getJoin("d1000", function(join, value) {
        if (value != null)
            hasD1000 = true;
    });


##### How do I access global tokens?

You can access the join that contains all global tokens using it's identifer, `CF.GlobalTokensJoin`. To read global tokens, use one of `CF.getJoin()` or `CF.getJoins()`. To set global tokens, use `CF.setToken()`. You can also set multiple tokens on the global tokens join when using `CF.setJoins()`.

Example:

    // Increment a global counter token named [mycounter]
    CF.getJoin(CF.GlobalTokensJoin, function(j, v, tokens) {
       var counter = parseInt(tokens["[mycounter]"], 10);
       CF.setToken(CF.GlobalTokensJoin, "[mycounter]", counter+1);
    });
    
    // Set multiple global tokens using CF.setJoins()
    CF.setJoins([{
        join: CF.GlobalTokensJoin,
        tokens: {
            "[mycounter]": 0,
            "SERVER_URL": "http://www.commandfusion.com",
            "CURRENT_ALBUM": ""
        }
    }]);


## Modules

#### How do I debug code that runs just after a module is loaded?

Early code execution is tricky to debug, because the code that's not inside functions or objects will execute right away after it has been loaded. In this (advanced) situations, `CF.log()` may not be usable because you're running too early in the startup process. Instead, you may want to resort to using `console.log()` and running with the Remote Debugging Monitor connected, so that your early messages are logged to Safari's error console.
