# List API

The list API lets you populate, modify, manipulate and explore your list joins.

## Events

Lists fire JavaScript events when scrolling occurs. You can watch these events if you want to synchronize GUI information with the current list scroll position. When scrolling occurs (either the user dragging the list, JavaScript [CF.listScroll()](#cF.listScroll) or a list scroll command received), the events fire in the following order:

* one [CF.ListWillStartScrollingEvent](#cF.ListWillStartScrollingEvent) before scroll starts
* zero or more [CF.ListDidScrollEvent](#cF.ListDidScrollEvent). If the list just scrolls a few pixels and the first visible item does not change, this event won't fire at all.
* one [CF.ListDidEndScrollingEvent](#cF.ListDidEndScrollingEvent) when deceleration is complete.


#### CF.ListWillStartScrollingEvent

This event is fired when a list is about to start scrolling, whether on user action (drag) or progammatically (using [CF.listScroll()](#cF.listScroll) or if list received a scroll command through value setting). You can take advantage of this event to manage specific UI elements surrounding lists.

Note that this event is sent only for lists that are currently on screen. If you send a scroll command to an offscreen list, the scroll position is memorized but not updated until the list comes on screen. If you issued one or more scroll commands for a list that is not on screen, you may get at most one `CF.ListWillStartScrollingEvent` / `CF.ListDidEndScrollingEvent` pair of events fired when the list appears.

Your callback should be of the form `function(listJoin, count, firstItem, numVisible, topPixelPosition)` and receives the following parameters:

* `listJoin` (_string_): the list join
* `count` (_number_): the number of items in the list
* `firstItem` (_number_): the index of the item currently visible at top (or left) of the list, at the time scroll starts
* `numVisible` (_number_): the current number of items visible on screen in the list
* `topPixelPosition` (_number_): the current scroll position of the list in pixels

See below [CF.ListDidEndScrollingEvent](#cF.ListDidEndScrollingEvent) for an example of use.

#### CF.ListDidScrollEvent

This event is fired while a list is scrolling, and only when the topmost (or leftmost for horizontal lists) item changes. Therefore, you will get at most one callback per change of first visible item. You use this event if you need to synchronize display / position of other GUI elements while a list is scrolling.

Note: on iOS, the system runs JavaScript and list updates on the same thread, both competing for CPU use. Try to limit your JavaScript processing as much as possible (and optimize the code that you run) so as to not make list scrolling appear choppy.

Your callback should be of the form `function(listJoin, count, firstItem, numVisible, topPixelPosition)` and receives the following parameters:

* `listJoin` (_string_): the list join
* `count` (_number_): the number of items in the list
* `firstItem` (_number_): the index of the item currently visible at top (or left) of the list
* `numVisible` (_number_): the current number of items visible on screen in the list
* `topPixelPosition` (_number_): the current scroll position of the list in pixels

See below [CF.ListDidEndScrollingEvent](#cF.ListDidEndScrollingEvent) for an example of use.


#### CF.ListDidEndScrollingEvent

This event is fired when a list is done scrolling,  whether on user action (drag) or progammatically (using [CF.listScroll()](#cF.listScroll) or if list received a scroll command through value setting). You can take advantage of this event to manage specific UI elements surrounding your lists.

Note that this event is sent only for lists that are currently on screen. If you send a scroll command to an offscreen list, the scroll position is memorized but not updated until the list comes on screen. If you issued one or more scroll commands for a list that is not on screen, you may get at most one `CF.ListWillStartScrollingEvent` / `CF.ListDidEndScrollingEvent` pair of events fired when the list appears.

Your callback should be of the form `function(listJoin, count, firstItem, numVisible, topPixelPosition)` and receives the following parameters:

* `listJoin` (_string_): the list join
* `count` (_number_): the number of items in the list
* `firstItem` (_number_): the index of the item visible at top (or left) of the list at end of scroll
* `numVisible` (_number_): the current number of items visible on screen in the list
* `topPixelPosition` (_number_): the scroll position of the list in pixels at end of scroll

Example:

    CF.userMain = function() {
        // watch scroll events for list l1
        CF.watch(CF.ListWillStartScrollingEvent, "l1", onListWillStartScrolling);
        CF.watch(CF.ListDidScrollEvent, "l1", onListDidScroll);
        CF.watch(CF.ListDidEndScrollingEvent, "l1", onListDidEndScrolling);
        
        // you can also watch multiple lists at once
        CF.watch(CF.ListWillStartScrollingEvent, ["l2", "l3"], onListWillStartScrolling);
        CF.watch(CF.ListDidScrollEvent, ["l2", "l3"], onListDidScroll);
        CF.watch(CF.ListDidEndScrollingEvent, ["l2", "l3"], onListDidEndScrolling);
    };
    
    function onListWillStartScrolling(join, count, first, numItems, topPixel) {
        CF.log("List " + join + " will start scrolling. Current position: first item=" + first + ", pixel position=" + topPixel);
    }

    function onListDidScroll(join, count, first, numItems, topPixel) {
        CF.log("List " + join + " did scroll. Current position: first item=" + first + ", pixel position=" + topPixel);
    }
    
    function onListDidEndScrolling(join, count, first, numItems, topPixel) {
        CF.log("List " + join + " did end scrolling. Final position: first item=" + first + ", pixel position=" + topPixel);
    }
    


## Item index constants

* `CF.LastItem` Constant for [CF.listUpdate()](#cF.listUpdate) to target the last item in a list
* `CF.AllItems` Constant for [CF.listUpdate()](#cF.listUpdate) to target **all** items in a list

## List scroll position constants

These constants are to be used with the [CF.listScroll()](#cF.listScroll) API:

* `CF.TopPosition`: make item visible at top or left of the list
* `CF.LeftPosition`: synonym for `CF.TopPosition`
* `CF.MiddlePosition`: make item visible at middle of the list
* `CF.BottomPosition`: make item visible at bottom or right of the list
* `CF.RightPosition`: synonym for `CF.BottomPosition`
* `CF.VisiblePosition`: make item visible with a minimal amount of scrolling (either at top/left or bottom/right of the list)
* `CF.RelativePosition`: scroll backwards or forward by N items
* `CF.PixelPosition`: scroll to a specific pixel position in the list

## Functions

#### CF.listAdd(list, array [, position])

Add items to a list. You pass an array of objects whose properties are the joins to set in list elements. Each object will add one item to the list. The object's property names are the joins for set, the property's value is the join value.

In addition to this, each object can carry:

* an optional `subpage` property (_string_) which is the name of the subpage to use to display the item,
* or an optional `title` property (_boolean_) which indicates whether the item to be created is a Title item, or a Regular item (in this case, the subpage used to display the item will be the title or content subpage defined in the list)
* if none of the above is present, the item will use the list's defined content subpage.

You can also perform more complex inserts by inserting items that also carry tokens. To achieve this, instead of passing a single string for a join, you can pass a full-fledged object that carries the following properties:

* `value` (_string_): the join value
* `tokens` (_object_): an object whose properties are token names, and the value of each property is the token value

An optional last parameter specifies the position in the list at which to insert the created elements. If this parameter is ommitted or outside of the current range of items in the list, the new items will be appended to the end of the list.

Insertions will be animated using the list's defined Insert transition.

To summarize, a join you want to set in a list item can have two forms:

    {
        title: /* true or false, will be false if omitted */,
        <join>: <string>,       /* a simple join and value */
        <join>: {               /* a join with a value and one or more tokens */
            value: <string>,
            tokens: {
                <token name>: <string>,
                // ... add as many tokens as needed
            }
        }
    }

Example:

    // Basic list population: Add a title and three items, all at once
    CF.listAdd("l1", [
        {
            // First item: a title item, with a title string (s1) and subtitle (s2)
            title: true,
            s1: "Title Item",
            s2: "This is the title item in the list", 
        },
        {
            // Second item: a regular item with a text (s1) and a button title set (s5)
            // The button will also carry two tokens. This uses the default content subpage
            // defined in the list
            s1: "First regular item",
            s5: {
                value: "Press Me 1",
                tokens: {
                    "[something]":"some token value",
                    "[somethingElse]":"some other token value"
                }
            }
        },
        {
            // Third item: an item using a custom subpage
            // (neither the default Title or Content subpage defined in the list)
            subpage: "custom_item_subpage",
            s1: "Second regular item",
            s5: "Press Me 2"
        }
    ]);


Now although the example above is pretty simple, you can also populate lists by building your objects array programmatically (for example, using data you grab from a remote website) then call `CF.listAdd` to add the items to the list.


#### CF.listUpdate(list, array)

Update items in a list. With this API you can perform disjoint updates (updates on non-consecutive list items), update only select joins of the list items of your choosing, all this with one simple call.

Note that any change you make using `CF.listUpdate` will **not** trigger a `CF.JoinChangeEvent`.

Each object in the array that you provide must at least have one property: `index` (_number_) which is the list item index on which the updates apply. Other properties are considered join strings, and the value of each property is the join value.

The `index` property can take one of the two special values:

* [CF.LastItem](#item_index_constants): represents the last item in the list
* [CF.AllItems](#item_index_constants): will update ALL items in the list with the provided values

It is perfectly acceptable to mix real with special item indexes to update specific values at specific item indexes, while globally setting another join for all items.

Examples:

    // Update items we created on the previous example
    CF.listUpdate("l1", [
        {
            // Update the title item in the list
            index: 0,
            s1: "Modified title"
        },
        {
            // Change the text in third item, as well as the button title
            index: 2,
            s1: "Modified second item",
            s5: {
                value: "Don't press me 2",
                tokens: {
                    "newtoken":"new token for item 3"
                }
            }
        }
    ]);
    
    
    // Update some joins at specific items (0 and 7), and globally reset d1 for all items
    CF.listUpdate("l1", [
        { index:0, s1:"Some text" },
        { index:7, s5:"Another text" },
        { index:CF.AllItems, d1:0 }
    ]);

Note that you may update as many joins as you want in each item. You don't have to update all the joins, just the ones you want to change.


#### CF.listRemove()

Remove one, several or all items in a list. This API takes variable parameters depending on your intent:

* `CF.listRemove(list)` completely clears the list contents (all items are removed)
* `CF.listRemove(list, index)` removes all items from the given position to the end of the list
* `CF.listRemove(list, index, count)` removes one or several items from the list

Trying to remove items at non-existing indices, or more items than exist starting from the provided index will just process what exists and ignore the rest.

The remove operation will be animated using the list's Item Delete transition.


#### CF.listScroll(list, index, position, animated [, visibleOnly])

Scroll a list to make the item at the specified `index` visible at table position `position` with optional animation. If position is `CF.Relative`, index is a positive or negative displacement in number of rows, that will scroll the list up or down. Otherwise, index is an absolute index in the list. You are allowed to use `CF.LastItem` instead of an actual index for the absolute positioning modes. Position can be one of:

* `CF.TopPosition`, `CF.LeftPosition`: item at `index` will to stand at top (for vertical lists) or left (for horizontal lists). Both parameters are equivalent and can be used interchangeably.
* `CF.MiddlePosition`: item at `index` will stand in the middle of the list area
* `CF.BottomPosition`, `CF.RightPosition`: item at `index` will stand at bottom (for vertical lists) or right (for horizontal lists). Both parameters are equivalent and can be used interchangeably.
* `CF.VisiblePosition`: item at `index` will be made fully visible with a minimal amount of scrolling. If the item is already on screen, list won't move. If the item is partially or not visible, the list with scroll as few as possible either way to make it visible (the item will then be visible at one of the edges of the list).
* `CF.RelativePosition`: will scroll N items (positive or negative) relatively to current position
* `CF.PixelPosition`: top of the list will scroll at the provided pixel position (in pixels from top if list is vertical, from left if list is horizontal)

The `animated` parameter can be `true` or `false`. When `true`, scrolling is animated.

By default, all lists on the same join scroll simultaneously. If the list is not currently on screen (on a page that is not being displayed), its scroll position is updated and the next time the least appears on screen, it will be at the selected scroll position.

If you have multiple lists on the same join but only want to programmatically scroll the onscreen list(s), pass an additional `false` flag for the `visibleOnly` parameter.

Note: if you issue multiple CF.listScroll() commands in a row, the first one being executed will take precedence, others may be ignored until the first scroll completes. Make sure you take this into account if you need to make items visible in a speedy fashion - Carefully craft your scroll commands (ideally, only issue one and allow 0.5 second for it to complete).

Examples:

    // Scroll row 30 to middle position, with animation
    CF.listScroll("l1", 30, CF.MiddlePosition, true);
    
    // Scroll backwards by 10 rows, animated
    CF.listScroll("l1", -10, CF.RelativePosition, true);
    
    // Scroll only the onscreen list, placing item 19 at bottom of the list with animation
    CF.listScroll("l1", 19, CF.BottomPosition, true, true);

    // Scroll only the onscreen list, ensuring item 24 is fully visible, scrolling the list as few as possible
    CF.listScroll("l1", 24, CF.VisiblePosition, true, true);

    // Scroll only the onscreen list 100 pixel frop top/left position
    CF.listScroll("l1", 100, CF.PixelPosition, true, true);


#### CF.listInfo(list, callback)

Calls your callback with information about a list. The callback parameters are:

* list join
* number of items
* current index of first visible item (or 0 if list empty or in a page that never appeared)
* number of visible items (or 0 if list empty or in a page that never appeared)
* current scroll position, in pixels from the beginning of list (new in v4.0.6)

You can use this API to retrieve the current number of items in a list, or to remember at which position the list currently is.

If you have multiple lists on the same join, iViewer will return the current scroll position of the list that is onscreen. If you have two lists on the same join at the same time on screen, both may have different scroll positions. In this special case, the list from which you get scroll information is undefined (it may be either one).

If the list is not visible, you will get the last displayed position, regardless of any call to `CF.listScroll()` you may have made since the last time the list was displayed. Again, if several lists are on the same join and all are offscreen, the one from which you get the current scroll position is undefined.

For these reasons, when you need to get the current scroll position of a list, it is generally advisable to make sure that you only have one list on this join.

Example:

    // Get information about a list
    CF.listInfo("l1", function(list, count, first, numVisible, scrollPosition) {
        CF.log("List " + list + " has " + count + " items, showing " + numVisible + " items starting from " + first + " (scroll position: " + scrollPosition + "px)");
    });


#### CF.listContents(list, index, count, callback)

Extract the contents from a list, returning an array of `count` items starting from `index`. Each returned item is an object whose properties are the joins present in the list item, as well as a `subpage` property (_string_) which is the name of the subpage used to display this item.

Each join property contains an object with `value` (_string_) and `tokens` (_object_) properties. To summarize, what you can is something like:

    // Structure of the object returned by CF.listContents. This is an example of the returned
    // object's structure, not something to include in your code!
    var contents = [
        // first item
        {
            // this item is displayed using the "list_title" subpage
            subpage: "list_title",
            
            // other properties are the joins contained in the subpage
            s100: {
                value: "This is a title item",
                tokens: {
                    "[join]": "s100"
                }
            }
        },
    
        // second item
        {
            // this item is displayed using the "list_item" subpage
            subpage: "list_item",
            
            // other properties are the joins contained in the subpage
            s0: {
                value: "A label",
                tokens: {
                    "[join]": "s0"
                }
            },
            s1: {
                value: "Another label",
                tokens: {
                    "[join]": "s1"
                }
            }
        }
    ];


You can set `count` to `0` to get all the elements from `index` to the end of the list.

Example:

    // Get and log the first three items in the list
    CF.listContents("l1", 0, 3, function(items) {
        for (var i = 0; i < items.length; i++) {
            // log a line with the item index
            var s = "[" + i + "] ";
            var item = items[i];
            for (var join in item) {
                // add the join and value
                s += join + "=" + item[join].value;
                
                // add the tokens if they exist
                var tokens = "";
                for (var token in item[join].tokens) {
                    tokens += token + ": " + item[join].tokens[token];
                }
                if (tokens.length)
                    s += " (" + tokens + ") ";
            }
            CF.log(s);
        }
    });
    
    // Get all the elements from index 5 and onwards
    CF.listContents("l1", 5, 0, function(items) {
        // ... use the returned items here ...
    });

