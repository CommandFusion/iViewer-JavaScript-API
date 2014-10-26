# Video API

Starting with version 4.0, iViewer now supports video objects that can play H.264, MPEG-4 part 2 and MJPEG video. While few network cameras already support the more powerful H.264 standard, nearly all cameras on the market at least support MJPEG live feeds.

Your script can receive a host of information about a video feed: several events are being generated at various points in time to help your code determine both the feed properties and status.

## Video feed information

When you receive an event related to a video feed (either static or live), an object detailing the feed status and known features is send to your callback. The properties of this object are:

* `url` (_string_): the video URL as you defined it
* `dataURL` (_string_): the actual URL from which the data is playing. Most of the time it will be equal to `url` except when the video has been cached and is playing from a local file.
* `width` (_number_): the native width of the video
* `height` (_number_): the native height of the video
* `duration` (_number_): the duration of the video (in seconds), if this is a static feed. Live streams have a 0 duration.
* `hasAudio` (_boolean_): whether the feed provides audio
* `hasVideo` (_boolean_): whether the feed provides video
* `playable` (_boolean_): whether iViewer buffered enough data to start playing the feed (but also check the `buffered` property to know whether the whole video has been buffered)
* `buffered` (_boolean_): whether the video has been fully buffered
* `streaming` (_boolean_): whether the video is a static of live (streaming) feed
* `playing` (_boolean_): whether the feed is playing (but may be paused or seeking)
* `stopped` (_boolean_): whether the feed is stopped or has not started playing yet
* `paused` (_boolean_): whether the feed is paused (in this case, `playing` is also `true`)
* `finished` (_boolean_): whether the movie has finished playing completely (in this case, `stopped` is also `true`)
* `error` (_boolean_): whether an error occured while trying to access the video

## Events

#### CF.MovieInfoReceivedEvent

This event can be sent several times when the feed is starting (typically once for MJPEG feeds, and multiple times for H.264/MPEG-4 feeds). Setup your callback using a `CF.watch(CF.MovieInfoReceivedEvent, join, function)` call. Your function receives two parameters:

* the join identifier of the video object that received feed information
* an object detailing the status and known features of the feed

Please refer to [Feed information](#video_feed_information) above for details about the feed features object.

Example:
    
    CF.userMain = function() {
        CF.watch(CF.MovieInfoReceivedEvent, "s100", videoInfoReceived);
    };
    
    function videoInfoReceived(join, info) {
        CF.log("Video info updated for " + join + " (URL: " + info.url + ")");
        if (info.width != 0)
            CF.log("- Native image size: " + info.width + "x" + info.height);
        if (info.duration != 0)
            CF.log("- Video duration: " + info.duration + " seconds");
        else if (info.streaming)
            CF.log("- This is a live feed");
        CF.log("- Video: " + info.hasVideo.toString() + ", Audio: " + info.hasAudio.toString());
    }


#### CF.MoviePlaybackStateChangedEvent

This event is sent whenever the feed playback status changes: the feed starts or stops playing, is paused or resume, or user is seeking backward or forward through the video.

Setup your callback using a `CF.watch(CF.MoviePlaybackStateChangedEvent, join, function)` call. Your function receives two parameters:

* the join identifier of the video object that received feed information
* an object detailing the status and known features of the feed

Please refer to [Video feed information](#video_feed_information) above for details about the feed features object.

Example:

    CF.userMain = function() {
        CF.watch(CF.MoviePlaybackStateChangedEvent, "s100", videoPlaybackStateChanged);
    };
    
    function videoPlaybackStateChanged(join, info) {
        CF.log("Video playback state changed for " + join + " (URL: " + info.url + ")");
        if (info.playing) {
            if (info.paused)
                CF.log("- Paused");
            else
                CF.log("- Playing");
        } else if (info.stopped)
            CF.log("- Stopped");
    }
    

#### CF.MovieLoadStateChangedEvent

This event is sent when the _load state_ of a movie changes. This happens to let you know when the movie has been at least partially buffered (enough to start playing), when it has been fully buffered, or when a load error happened.

Setup your callback using a `CF.watch(CF.MovieLoadStateChangedEvent, join, function)` call. Your function receives two parameters:

* the join identifier of the video object that received feed information
* an object detailing the status and known features of the feed

Please refer to [Video feed information](#video_feed_information) above for details about the feed features object.

Example:
    
    CF.userMain = function() {
        CF.watch(CF.MovieLoadStateChangedEvent, "s100", videoLoadStateChanged);
    };
    
    function videoLoadStateChanged(join, info) {
        CF.log("Video load state changed for " + join + " (URL: " + info.url + ")");
        if (info.buffered)
            CF.log("- Video is fully buffered");
        else if (info.playable)
            CF.log("- Video is partially buffered");
        else
            CF.log("- Video is not buffered yet");
    }
