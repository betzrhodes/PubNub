# PubNub

The [PubNub class](./PubNub.class.nut) wraps [PubNub’s API](http://www.pubnub.com/) for real-time messaging.

The [MessageBus class](./MessageBus) uses the PubNub API to create seamless device to device communication (through their agents and PubNub).

**To add this library to your project, add** `#require "PubNub.class.nut:1.0.0"` **to the top of your agent code**

## Contributors

- Matt Haines
- Tom Byrne

## Class Usage

### Constructor

To instantiate a new object, you’ll need your PubNub Publish-Key, Subscribe-Key and Secret-Key.

```squirrel
pubNub <- PubNub(publishKey, subscribeKey, secretKey)
```

You may pass an optional fourth parameter, *uuid*. If you leave this blank, the library will automatically use the last part of your agent URL.

### Class Methods

### Publish Data: publish(*channel*, *data*, [*callback*])

To publish data, you need to specify the name of the channel and the data. The channel name is a string. The data can be a basic type (string, integer, etc), an array, or an object:

```squirrel
// Sending a string

pubNub.publish(channel, "test data")

// Sending an object

pubNub.publish(channel, { foo = "bar" })
```

You can specify an optional third parameter, a callback function which takes two parameters: *err* and *data*. If you do not specify the callback function, a default callback function that logs the results will be used. Here’s an example where we specify a callback:

```squirrel
pubNub.publish(channel, { foo = "bar" }, function(err, data) {
	if (err != null) 
	{
		server.log("ERROR: " + err)
		return
	}

	// You should do something interesting with data; we're just going to log it
	
	if (data[0] == 1 && data[1] == "Send") 
	{
		server.log("Success!")
	else 
	{
		server.log(data[1])
	}
})
```

### Subscribing to Data: subscribe(*channels*, *callback*)

To subscribe to the channel, you need to specify the channel or channels you are subscribing to and provide a callback function to execute whenever there is more data. However many channels you provide, pass them as an array of one or more strings. The callback function takes three parameters: *err*, *result* and *timetoken*. The *result* parameter is a table containing a channel/value pair for each channel/message received:

```squirrel
pubNub.subscribe(["foo", "demo"], function(err, result, timetoken) {
    if (err != null) 
    {
        server.log(err)
        return
    }

    local logstr = "Received at " + timetoken + ": "
    local idx = 1
    foreach (channel, value in result)
    {
        logstr += (channel + ": " + value)
        if (idx++ < result.len())
        {
            logstr += ", "
        }
    }
    server.log(logstr)
});
```

The subscribe endpoint will automatically reconnect after each datapoint. However, if there is an error, you are responsible for reconnecting.

### Getting Channel History: history(*channel*, *maxValues*, *callback*)

To get historical values published on a given channel, specify the channel by name as a string; indicate the maximum number of values you want ot be returned; and provide a callback to execute when the data arrives. The callback takes two parameters: *err* and *data*. The *err* parameter is `null` on success. The *data* parameter is an array of messages.

```squirrel
// Get up to 50 historical values from the demo channel

pubNub.history("demo", 50, function(err, data) {
    if (err != null) 
    {
        server.error(err)
        return
    } 
    else 
    {
        server.log("History: " + http.jsonencode(data))
    }
})
```

### Presence Detection: whereNow(*callback*)

The *whereNow()* function returns a list of channels on which this client’s UUID is currently ‘present’. A UUID is marked present when it publishes or subscribes to a channel, and is removed when that client leaves a channel with the *leave()* method. The *whereNow()* function takes one parameter: a callback function to execute with the list of channels is returned. The callback must take two parameters: *err* and *channels*. The *err* parameter is `null` on success, and the *channels* parameter is an array of channels for which the UUID is present.

```squirrel
// list the channels that this UUID is currently present on

pubNub.whereNow(function(err, channels) {
    if (err != null) 
    {
        server.log(err)
        return
    }
    
    server.log("Currently watching channels: " + http.jsonencode(channels))
})
```

### Presence Detection: hereNow(*channel*, *callback*)

The *hereNow()* function provides the current occupancy of a given channel. It takes two parameters: the channel name as a string and a callback function executed when the data arrives. The callback takes two parameters: *err* and *result*. The *err* parameter is `null` on success. The *result* parameter is a table with two members: *occupancy* (an integer) and *uuids* (an array).

```squirrel
// List the UUIDs that are currently watching the temp_c channel

pubNub.hereNow("temp_c", function(err, result) {
    if (err != null) 
    {
        server.log(err)
        return
    }
    
    server.log(result.occupancy + " Total UUIDs watching temp_c: " + http.jsonencode(result.uuids))
})
```

### Presence Detection: globalHereNow(*callback*)

The *globalHereNow()* function provides the current occupancy of a given subscribe key. It takes one parameter: a callback function. The callback takes two parameters: *err* and *result*. The *err* parameter is `null` on success. The *result* parameter contains a key/value pair for each channel on the requested subscribe key; the key is the channel name, and each value is a table with two members: *occupancy* (an integer) and *uuids* (an array).

```squirrel
// List all channels and UUIDs that are currently using the same subscribe key as us

pubNub.globalHereNow(function(err,result) {
    if (err != null) 
    {
        server.log(err)
        return
    }
    
    server.log("Other Channels Using this Subscribe Key:")
    foreach (chname, channel in result) 
    {
        server.log(chname + " (Occupancy: " + channel.occupancy + "): " + http.jsonencode(channel.uuids))
    }
})
```

### Leaving a Channel: leave(*channel*)

The *leave()* function informs the PubNub Presence Server that this UUID is leaving the specified channel. The UUID will no longer be returned in results provided by the presence functions described above. It takes a single parameter: the name of the channel being exited as a string.

```squirrel
// Exit the 'foo' channel

pubNub.leave("foo")

// Check we have left. This should log an error

pubNub.hereNow("foo", function(err, result) {
    if (err != null)
    {
    	server.log(err)
    	return
    }
    
    server.log(result.occupancy + " Total UUIDs watching temp_c: " + http.jsonencode(result.uuids))
})
```

## License

The PubNub library is licensed under the [MIT License](./LICENSE).
