# Notes on writing an SDK for Emitter

From the point of view of a client application, Emitter behaves exaclty like any MQTT brokers. Therefore, Emitter SDKs are merely light wrappers around some pre-existing MQTT lib. The role of the SDK is to simplify as much as possible the communication with the Emitter broker, so that - for example - the following code...

```
var emitter = emitter.connect(); 
var key = 'LyOL3fhQDfOKyB6PWNkv6vemLSPjP8hg';

emitter.publish({
  key: key,
  channel: "testChannel,
  message: JSON.stringify({
    id: 1,
    text: "message"
    })
});
```

...is really all it takes for a Javascript app to start a connection with Emitter and send a message.

## The choice of the MQTT library

Most SDKs developped by our team are based on MQTT libs distributed by the [Eclipse Paho project](http://www.eclipse.org/paho/). This project provide an implementation for most mainstream language, like C#, Java, Python, C, C++, ... But you'll easily find MQTT libs for almost any languages, and you are free to choose any implementations of MQTT in accordance with you requirements. For example, although the Paho project provide an embedded C implementation, you might be happy with something as light as [knolleary's pubsubclient implementation](https://github.com/krohling/ArduinoPusherClient). I successfully communicated with Emitter from my recently acquired Arduino Yun, using this implementation.


Just be sure to check whether all the features you need (for example, the SSL support) are implemented in the flavour you covet.

## The channel

### The channel format

Because of Emitter's security model, you have to embed a key to the channel in the MQTT channel itself.


A typical channel after formatting will look like this :

```
LyOL3fhQDfOKyB6PWNkv6vemLSPjP8hf/article1/518ad0ed566d5507338c81a5e405829b/?ttl=1200
```

More formally :

```
KEY/CHANNEL[/ANY NUMBER OF SUB-CHANNELS][/?OPTIONS]
```

### The options



### An implementation of the formatting

```
Emitter.prototype._formatChannel = function (key, channel, options) {
    // Prefix with the key
    var formatted = this._endsWith(key, "/")
        ? key + channel
        : key + "/" + channel;
    // Add trailing slash
    if (!this._endsWith(formatted, "/"))
        formatted += "/";
    // Add options
    if (options != null && options.length > 0) {
        formatted += "?";
        for (var i = 0; i < options.length; ++i) {
            formatted += options[i].key + "=" + options[i].value;
            if (i + 1 < options.length)
                formatted += "&";
        }
    }
    // We're done compiling the channel name
    return formatted;
};
```

