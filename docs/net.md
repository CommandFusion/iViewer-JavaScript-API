# Network API

The network API give you access to the systems defined in your GUI, as well as powerful functionality that let you perform external network queries in order to fetch and process data. With the network API, you can be notified when systems connect and disconnect, when incoming connections are accepted, send data to remote systems and perform network lookups.

## Constants and variables

#### CF.ipv4address

As a matter of convenience, the IPv4 address of the device running your script is available as a string in `CF.ipv4address`. When network status changes (see `CF.NetworkStatusChangeEvent`), the address is updated. If the device currently does not have a network connection, the address will be an empty string.

#### CF.ipv4netmask

The IPv4 subnet mask for the device network connection (i.e. 255.255.255.0)

#### CF.ipv6address

Similarly to `CF.ipv4address`, this variable contains the up-to-date IPv6 address of the device, or an empty string if no IPv6 address is available.

#### CF.ipv6netmask

Similarly to `CF.ipv4netmask`, this variable contains the up-to-date IPv6 subnet mask of the device, or an empty string if no IPv6 address is available.

#### CF.MACaddress

The MAC address of the device's network interface (typically, its Wifi adapter). Note that in some cases, the MAC address may not be known. This can happen when running over a cellular connection (the cellular interface does not have a MAC address) or when a VPN connection is active (the VPN connection could have been established over 3G, then WiFi turned on but VPN didn't jump links and stayed over 3G). In these cases, `CF.MACaddress` will be an empty string.

#### CF.networkType

The current network type. Value is a string and can be one of `None`, `VPN`, `WiFi`, `WAN` (for 3G / 4G connections) or `AdHoc`. Typically, if your device currently has an open VPN connection, the network type will be `VPN` even if you elected to NOT send all the traffic over the VPN connection.

#### CF.networkSSID

The current network SSID. If the device is connected to a WiFi network, then this is the SSID (network name) of the network. Otherwise, an empty string.

#### CF.systems

The global `CF` object defines `CF.systems` which is a dictionary of the systems defined in your GUI. In some cases, particularly when writing generic scripts that are common to several GUIs, it may be practical to explore the list of systems and take appropriate action depending on which ones are present. From this dictionary, you can extract the URL of each system and use it to make custom requests or prepend the URL to image join strings, etc.

Each entry in `CF.systems` exposes the following properties:

* `type` (_string_): the system type:
    * `"cf"` for control systems (talking the CommandFusion protocol)
    * `"tcp"` for TCP systems
    * `"udp"` for UDP systems
    * `"http"` for HTTP and HTTPS systems
* `enabled` (_boolean_): true if this system is enabled. See [CF.setSystemProperties()](#cF.setSystemProperties) to enable or disable systems programatically
* `address` (_string_): the address of the remote, as specified in GUI designer
* `port` (_number_): the remote port
* `localPort` (_number_): the local port (i.e. the one on which we accept connection for a server system)
* `connect` (_string_): the join that the system raises when it is connected, and lowers when it is disconnected. If you did not define the connect join in GUI Designer, this string is empty.
* `disconnect` (_string_): the join that the system raises when it is disconnected, and lowers when it is connected. If you did not define the disconnect join in GUI Designer, this string is empty.
* `connections` (_array_): current connections for this system, empty if not connected

iViewer permanently updates this information when systems connect and disconnect. The `connection` property is an array of strings of the form `"IP:port"`, each entry being the remote address and port of an active connection. The array can be:

* empty (system not connected)
* one entry with the current IP address and port to which the system is connected (for outgoing systems)
* a list of entries which are the clients currently connected to an incoming system.

Here is an example of `CF.systems` variable (this is the way we assign it, so you see what members you can get from it):

    {
        "": {                       // empty name is the main (Project) control system
            type: "cf",             // "cf" is a control system talking the CommandFusion protocol
            address: "192.168.0.7",
            port: 8020,
            connect: "d100",        // the join that reflects the connected status of this system,
                                    // an empty string if 0
            disconnect: "d200",     // the join that reflects the disconnected status of this system
            connections: []         // no 
        },
        "MY SYSTEM" {
            type: "tcp",            // This is an external TCP system (other types are "cf", "udp", "http")
            address: "192.168.0.1",
            port: 5050,
            connect: "",            // no connect join
            disconnect: ""          // no disconnect join
            connections: []
        },
        "WEB REQUEST" {
            type: "http",           // Both http and https systems carry a "http" type
            address: "http://www.mycompany.com",
            port: 80,
            connect: "",            // no connect join
            disconnect: ""          // no disconnect join
            connections: []         // At any given time, the active connections (incoming or outgoing) are listed here
        }
    }

For example, you may have a generic script that talks to an XBMC media center. In your `CF.userMain` function, you can check whether a system named _XBMC_ is defined in your GUI and call your generic XBMC script's initialization function:

    CF.userMain = function() {
        // Check whether a system named 'XBMC' is defined in the GUI
        if (CF.systems["XBMC"] != undefined) {
            // Call my generic function that will gather media info from XBMC
            startGettingDataFromXBMC();
        }
    };

#### Constants

The following constants are related to networking:

* `CF.UTF8` and `CF.BINARY` specify the output format for [CF.send()](#cF.send).


## Events

With the CF network API, you can be notified when outgoing systems connect or disconnect, when incoming systems accept or lose client connections, and when specific feedback items are matched.

#### CF.NetworkStatusChangeEvent

This event is sent when a change occurs in network conditions (connection goes up or down, or your device's IP address changes, for example when switching between connection types: 3G, WiFi, VPN). Your callback function receives an object whose properties contain the current network status, as well as the IPv4 and IPv6 addresses whenever applicable:

    var networkStatus = {
        hasNetwork: /* false or true */
        ipv4address: /* string with IPv4 address */
        ipv4netmask: /* string with the network mask for the IPv4 subnet, i.e. "255.255.255.0" */
        ipv6address: /* string with IPv6 address */
        ipv6netmask: /* string with the network mask for the IPv6 network */
        networkType: /* string with network connection type, i.e. "WiFi" */
    };

Here is example code that updates a serial join with the current IP address of the device:

    function onNetworkStatusChange(networkStatus) {
        if (networkStatus.hasNetwork) {
            // note that the device may not have an IPv6 address,
            // in which case the IPv6 address string will be empty
            CF.setJoin("s101", networkStatus.ipv4address);
            CF.setJoin("s102", networkStatus.ipv6address);
        } else {
            CF.setJoin("s101", "No network");
            CF.setJoin("s102", "No network");
        }
    }

    CF.userMain = function() {
        // Start watching network status changes. The last 'true'
        // parameter indicates that we want to receive an initial
        // event with the current status of the network, so as to
        // setup our UI
        CF.watch(CF.NetworkStatusChangeEvent, onNetworkStatusChange, true);
    };

Note that in addition to sending the `CF.NetworkStatusChangeEvent` event, iViewer keeps `CF.ipv4address` and `CF.ipv6address` properties up-to-date with the current device address on the network.

Finally, iViewer takes care of checking the network status when your application is resumed from the background (in multitasking mode, you left iViewer to use another application then come back to it later). If the network conditions has changed (new IP address or network down, etc) you script will get a `CF.NetworkStatusChangeEvent`.


#### CF.ConnectionStatusChangeEvent

This event is sent by a system when the following events occur (**only for TCP systems**):

* iViewer connects to a remote system
* iViewer disconnects from a remote system
* A remote client connects to one of the systems in your GUI that accepts connections
* A remote client disconnects from one of the systems in your GUI

**UDP systems** trigger a `CF.ConnectionStatusChangeEvent` when:

* They startup and are ready to send and receive data (`connected = true`)
* They stop and can't send or receive data anymore (`connected = false`)

Whenever one of the event listed above happens, iViewer will send an event to callback with three parameters: the name of the system that generated the event, a `boolean` indicating whether this is a connect or disconnect event, and a string in the format `"IP:port"` that identifies the remote. If the system is outgoing, you can detect simple connects or disconnects. If the system is incoming, you get notified when new clients connect or existing clients disconnect. You can further explore all the current connections for this system by looking at the `CF.system["SYSTEM_NAME"].connections` array.

Here is an example:

    function onConnectionChange(system, connected, remote) {
        // On connected==true, the remote is a string
        // for example: "192.168.0.16:5050"
        // When getting initial status, if the system is not connected, remote is null.
        if (connected) {
            CF.log("System " + system + " connected with " + remote);
        } else {
            if (remote == null)
                CF.log("Initial status: system " + system + " is not connected.");
            else
                CF.log("System " + system + " disconnected from " + remote);
        }
    }

    CF.userMain = function() {
        // Start watching connections with "CRESTRON" system The last 'true'
        // parameter indicates that we want to receive an initial
        // event with the current status of the system, so as to
        // setup our UI
        CF.watch(CF.ConnectionStatusChangeEvent, "CRESTRON", onConnectionChange, true);
    };

Note that if you ask for initial notification of the current connection status using the last `true` parameter, and the system is one that accepts incoming connections, you may receive several calls to your callbacks if multiple clients are connected to this system at the time. Instead, it may be wiser to look at the contents of the `CF.system["SYSTEM_NAME"].connections` array.

Also, the connection status is meaningful only for TCP systems. For UDP systems, it indicates that the system is ready to send and receive data. HTTP / HTTPS systems don't send connection or disconnection events at all.

#### CF.FeedbackMatchedEvent

This event is sent by iViewer when a specific feedback item is matched. You can get notified when this occurs, and receive the full matched string. There are several ways you can use this: you can just watch feedback matches and do some additional processing in javascript, or you can elect to define feedback items with no actions at all, and perform all the processing in your script.

When a match occurs, your callback receives two parameters:

* the name of the feedback item
* the full matched string

Then it's up to you to take any appropriate action, for example perform a regex match using Javascript to extract the components you need, fill data in your GUI, etc.

    var fanartURL = "";

    function onProcessListContent(feedbackItem, matchedString) {
        // Here you can process the full matched string
        // For example when a feedback item matches "ALBUM:.*\n", we add the album name to
        // a list and set the URL for album art image. Here we match two groups in the string,
        // we'll receive an array (sort of) which contains the full match as element 0,
        // the first capture group as element 1 and the second capture group as element 2.
        var matches = matchedString.match(/ALBUM:(.+)\|(.+)\n/);
        CF.listAdd("l1", [
            {
                title: true,
                "s1": matches[1],       // s1 is the album title
                "s2": fanartURL + "/" + matches[2]  // s2 is an image element, we set the image URL here
            }
        ]);
    }
    
    CF.userMain = function() {
        // Get the actual URL for "WEB REQUEST" system so we can prepend it when getting fanart
        // In this example, "WEB REQUEST" is the name of an http system
        fanartURL = CF.systems["WEB REQUEST"].address;

        // Start watching feedback items
        CF.watch(CF.FeedbackMatchedEvent, "MY SYSTEM", "Album name", onProcessListContent);
    };

## Functions

#### CF.setSystemProperties(systemName, changes)

Allows changing some properties of an external or the main control system on the fly. The `changes` object that you pass can replace the value of some of the properties defined in [CF.systems](#cF.systems). You don't have to pass all the properties of a system every time you call this function: you can pass only the values of the properties that you want to change:

* `enabled` (_boolean_): changes the `enabled` (active) state of this system. Set to `false` to turn the system off.
* `address` (_string_): changes the remote IP address or hostname of this remote system
* `port` (_number_): change the port of the system to connect to
* `localPort` (_number_): changes the originating port for connection to remote system. If the system is accepts connections, this changes the local port on the device on which connections (or messages for UDP systems) are accepted.
* `connect` (_number_): changes the digital join number that is triggered high when this system is connected
* `disconnect` (_number_): changes the digital join number that is triggered high when this system is not connected.

You identify the system whose properties must change using its name. To change the properties of the main control system defined in GUI designer, simply use an empty string as the system name.

Examples:

    // Update the IP address and port of system "MediaCenter"
    CF.setSystemProperties("MediaCenter", {
       address: "192.168.0.7",
       port: 12655 
    });
    
    // Disables the MediaCenter system (disconnects it if it is currently connected)
    CF.setSystemProperties("MediaCenter", {enabled: false});
    
    // Assuming we have a system named "MulticastListener" that listens to UDP multicast
    // packets, change the multicast address and port it is listening on and turn the system on
    CF.setSystemProperties("MulticastListener", {
        enabled: true,
        address: "226.7.8.9",
        port: 17000,
        localPort: 17000
    });
    
    // Change the IP address on the main control system
    CF.setSystemProperties("", {address: "192.168.0.32"});


Side note: when you change the network settings of a system that is currently enabled, it will shutdown first (possibly triggering connect / disconnect joins), update to new settings then restart if the `enabled` flag has not been changed to `false`.


#### CF.send(systemName, string [, outputFormat])

Send a string to the specified system, with a default output format of `CF.BINARY`. Each character of the string string you pass is converted to a byte (be careful if you have unicode characters over 0xFF in your string, they will be stripped to a single byte) and the resulting packet is sent to the specified system. Optionally, you can specify an output format of `CF.UTF8` which converts the string to an UTF-8 representation before sending it. Do not use this option when you have binary data in the string (in particular, characters over 0x7F) otherwise they won't be transmitted verbatim.

Note that regardless of any EOM string that may be specified for the system, iViewer will not try to add the EOM to the data you send. If you need to send an EOM string at the end of your message, add it to the string to send.

If you defined a Javascript Send Handler for the system you're sending data to, the data string **will not** go through the handler. It will be sent directly over the network.

For systems that accept incoming connections, the string will be sent to each connected system.

Examples:

    CF.send("System Name", "Data To Send");
    CF.send("System Name", "Data To Send in UTF-8 encoding", CF.UTF8);

#### CF.runCommand(systemName, commandName)

Run a named command. The system name is currently unused, as the command names are unique among all systems. You can safely pass an empty string or `null` as the system name.

Example:

    // All these commands will result in the same thing:
    CF.runCommand(null, "Command Name");
    CF.runCommand("", "Command Name");
    CF.runCommand("System Name", "Command Name");

#### CF.runMacro(macroName)

Run a predefined macro (defined in GUI Designer's System Manager).

Example:

    CF.runMacro("Macro Name");

#### CF.stopMacro(macroName)

Stop a named macro if it is currently being executed. You can stop **all** running macros by passing an empty string or `null`.

Example:

    CF.stopMacro("Macro Name");
    CF.stopMacro(null); // Stop all macros

#### CF.request()

This function lets you perform HTTP and HTTPS requests directly without the need or constraints of having an HTTP system defined. This can be particularly useful when you need to fetch from or post data to remote web servers. By using `CF.request`, you can make **GET**, **POST**, **PUT**, **HEAD** HTTP requests to retrieve or send data and customize the request headers. Additionally, for very specific needs, all other HTTP commands (**OPTIONS**, **DELETE**, **TRACE**, **CONNECT**) are supported.

The function takes a variable number of arguments, depending on what you want to do:

* `CF.request(url, callback)`
* `CF.request(url, headers, callback)`
* `CF.request(url, method, headers, callback)`
* `CF.request(url, method, headers, body, callback)`

##### Simple request
In the first form, a GET will be made to the specified URL. Your callback receives the status code, response headers and response body string as parameters.

Example:

    function testWebCallback(status, headers, body) {
    	if (status == 200) {
    		// extract information from body here
    		CF.log("Request succeeded (HTTP 200 OK). Body received: " + body);
    	} else {
    	    // an error occurred, display the returned status code
    	    CF.log("Error: returned status code " + status);
    	}
    }

    // Send a GET request to obtain the contents from Apple main page
    CF.request("http://www.apple.com", testWebCallback);


##### Simple request with custom headers
The second form can be used to customize the HTTP headers. Pass a dictionary (an object) that contains the headers you want to set as properties, and the values as each property's value. Your callback receives the status code, response headers and response body string as parameters.

Example:

    // Send a GET request with custom User-Agent string in request headers
    CF.request( "http://www.somewebsite.com",
                {"User-Agent": "my-user-agent-1.0"},    // customize the User-Agent field
                testWebCallback);



##### Request with custom HTTP command and optional custom headers
The third form can select the request method. By default, `CF.request` does an HTTP **GET**. You can specify any other HTTP command to suit your needs. If needed, you can pass an object that will augment the basic HTTP headers with additional headers. Properties of object will be used to fill the additional headers.

Example:

    // Obtain and display just the response headers (not the body)
    // Note that the callback function does not have to use all the parameters
    // passed to it (status, headers, body), that's why in this example we
    // only declare the first two parameters as we don't use the third one (response body)
    CF.request( "http://www.apple.com",
                "HEAD",
                null,                              // pass null if not customizing the request headers
                function(status, headers) {
                   if (status == 200) {
                       // Extract and show the HTTP server info
                       CF.log("HTTP server is: " + headers["Server"]);
                   } else {
                       CF.log("Error: returned status code " + status);
                   }
                });
    

##### Fully customized HTTP request
The final and most powerful form of `CF.request()` can completely customize the request. You can specify the HTTP command to use, pass additional headers if needed, and pass a custom body string. Any of the headers and body parameters can be `null`. For **POST** commands, you can pass an object instead of a body string. Properties of this object will be use to create an url-encoded body data, similar to what is generated by web forms.

Note that body contents strings will be sent as UTF8 data, and body objects (form data) will be sent as `application/x-www-form-urlencoded` standard mimetype.


Examples:

    // Create a POST request with data filled in as in a web form
    var postFields = {
        first_name: "John",
        last_name: "Doe",
        country: "USA",
        city: "New-York",
        email: "john@doe.net"
    };
    CF.request( "http://www.myserver.com/myform?register",
                "POST",                 // use POST method
                null,                   // pass null if not customizing the request headers
                postFields,
                function(status, headers, body) {
                    if (status == 200) {
                        CF.log("Form data sent");
                    } else {
                        CF.log("Error: request returned status " + status);
                    }
                });
                
    // Create a PUT request with a custom request body contents
    var requestBody = "<tree><object1>hello</object1><object2>world</object2></tree>";
    CF.request( "http://www.myserver.com/putdata",
                "PUT",
                {"User-Agent": "my-user-agent-1.0"},    // customize the User-Agent field
                requestBody,
                function(status, headers, body) {
                   if (status == 200) {
                       CF.log("PUT succeeded");
                   } else {
                       CF.log("PUT failed with status " + status);
                   }
                });
                


#### CF.startLookup(serviceType, serviceName, callback)

Perform network lookups. Currently, only Bonjour (aka ZeroConf) is supported. With `CF.startLookup()` you can find services on your local network without prior knowledge of their IP address. This can be used, for example, to locate iTunes or XBMC instances.

The `serviceType` parameter will filter on the requested Bonjour service type (for example `"_xbmc-jsonrpc._tcp."`). Make sure that the transport layer (in our example, TCP) is always present at the end of the service type and that it ends with a dot. In the previous example, the transport layer is "._tcp.".

 The `serviceName` parameter will select entries matching the requested service name (usually, the service name is the name of the computer on which the service runs, but it can be modified to avoid name conflicts). Leave the service name empty to look up all entries matching the requested service type.

When lookup finds changes on the network, your callback is called with three arguments:

* array of added services since last time your callback was called
* array of removed services since last time your callback was called
* a string that, if not null, contains an error message in case Bonjour lookup returned an error.

The structure of each returned service is as follow:

* `name` (_string_): the service name
* `type` (_string_): the service type
* `addresses` (_array_): an array of strings with IP addresses found for this service. IPv4 addresses are put first in the list.
* `hostname` (_string_): the hostname on which the service runs
* `port` (_number_): the port# at which the service listens for connections
* `data` (_object_): A dictionary containing the key/value list provided by the publishing service in its TXT record

When services disappear from the network, the service object contains only the `name` and `type` properties.

Note that for computers on your network, it is not uncommon for a service to have multiple IP addresses if the computer
is multihoming. iViewer filters out IPv4 addresses that are not on your subnet (the ones that you can't reach directly).

Here is an example of Bonjour lookup that searches iTunes. We start the lookup, get notified every time a change happens, then after 2 minutes just close the lookup. In a real application, you would either let a lookup run all the time, or start it when entering a specific page and stop it when leaving it.

    var iTunesInstances = [];

    function startITunesLookup() {
        CF.startLookup("_daap._tcp.", "", function(addedServices, removedServices, error) {
    		try {
    			// remove disappearing services
    			iTunesInstances.forEach(function(service, index) {
    				if (removedServices.some(function(item) { return (item.name == service.name); })) {
    					CF.log("Closed iTunes instance [" + index + "]: " + service.name);
    					iTunesInstances.splice(index, 1);
    				}
    			});
			
    			// add new services
    	        addedServices.forEach(function(service) {
    		 		CF.log("New iTunes instance [" + iTunesInstances.length + "]: " + service.name);
    				iTunesInstances.push(service);
    		 	});
    		}
    		catch (e) {
    			CF.log("Exception in iTunes services processing: " + e);
    		}
        });
    }

    function stopITunesLookup() {
    	CF.stopLookup("_daap._tcp.", "");
    	CF.log("Stop looking for iTunes")
    }

    CF.userMain = function() {
    	// Start looking for iTunes, kill the lookup 1mn later
		startITunesLookup();
		setTimeout(stopITunesLookup, 60000);
	};


As an example, the screenshot below shows the structure of the returned objects for services:

![Bonjour lookup services structure](/assets/bonjourlookup.png "Bonjour lookup services structure")

**Multitasking notes**

When you leave the iViewer application (put it to background) for extended periods of time, known services can be closed when you come back. iViewer takes care of checking that for you, and notifies you a few seconds after resuming the application. You have nothing to do, this is all automated for you by iViewer.


#### CF.stopLookup(serviceType, serviceName)

To cancel a previously started Bonjour lookup, use `CF.stopLookup`. Provide the same service type and service name parameters that you initially provided.

Note that it is good practice to start only one lookup for a particular service type. iViewer won't prevent you from starting several lookups at once for the same service type and name, but the behavior of `CF.stopLookup()` in this case is to close all the running lookups for the same service type / name pair.


#### CF.startPublishing(serviceType, serviceName, port, txtData, callback)

Publish (advertise) a service on Bonjour. You pass a service type, an optional service name (pass an empty string or `null` to let the OS create the name from the device name), a port number and an optional object whose properties will determine the contents of the TXT record data for your service.

Your callback will receive a notification indicating whether your publish was successful. Its parameters are:

* `serviceType` (_string_): the service type you published
* `serviceName` (_string_): the service name you published
* `port` (_number_): the port you publised
* `published` (_boolean_): whether the service was published or not
* `error` (_string_): the error that occurred if `published` is `false` and an error occured. For example, "Duplicate service" if you try publishing a service with the same type, name and port parameters multiple times. `error` will be `null` if you called `CF.stopPublishing()` for this service.

Typical use case for publishing services is to make a server system available to others on the network.

Bonjour allows you to pass small data that is made available on the network in what is known as a TXT data record. You can pass an object. Each property of this object will be the name of the TXT data entry, and its value will be made available as a string.

Here is an example extracted from the iTunes module:

    function onPublishServiceResult(name, type, port, published, error) {
    	CF.log("Service name="+name+", type="+type+", port="+port+", published="+published+", error="+error);
    }

    CF.userMain() = function() {
        // We have a system that accepts connections on port 10211. Publish it under the
        // service type "_touch-remote._tcp." so that we receive pairing requests from iTunes
        var txtData = {
			"DvNm": "iViewer Remote",
			"RemV": "10000",
			"DvTy": "iPod",
			"txtvers": "1",
			"Pair":"0000000000000001"	// pairing code: 1
		};
		CF.startPublishing("_touch-remote._tcp.", "", 10211, txtData, onPublishServiceResult);
    };

When iViewer is put to the background (user switches applications), the service appears no longer published on the network. It automatically comes back when the application resumes to the foreground. If at that time, publishing the service fails, your callback will receive an error.


#### CF.stopPublishing(serviceType, serviceName, port)

Stop publishing a previously published service. iViewer will call your publishing callback once with `published` set to `false`, and a `null` error. After this, there will be no more calls to your callback for this service, and the service is completely removed from the list of published services.

