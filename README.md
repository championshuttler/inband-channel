## Inband channel occupancy events

## Contents

- [**Overview**](#overview)
- [**Why this required**](#why-this-required)
- [**How to enable Inband channel occupancy events**](#how-to-enable-inband-channel-occupancy-events)

### Overview

This feature allows a client to subscribe to metadata events relating to a single channel, and for those events to be delivered to the client inband - that is, as messages on the channel itself. 
The functionality supports [`occupancy`](https://www.ably.io/documentation/realtime/channel-metadata#occupancy) `meta-events` which contain the same information as is currently available in the `channel.lifecycle` [metachannel](https://www.ably.io/documentation/realtime/channel-metadata#metachannels), but only the channel client is subscribed to.

### Why this required

Currently, if you want to know the current state or current occupancy of a channel, it can be done via [`Channel Status API`](https://www.ably.io/documentation/rest/channel-status.). This feature was required because since the metadata of channels changes very frequently so requesting such information over the `REST` protocol is suboptimal, which means it is not efficient and data is still likely to become stale as soon as you have received it. Also for Ably, it is expensive to serve in comparison to sending the same information over the protocol.

### How to enable Inband channel occupancy events

You can enable Inband channel occupancy events by specifying `occupancy` param. [Channel Param](https://www.ably.io/documentation/realtime/channel-params) helps in customizing channel functionality means you can express properties of a channel or its attachment to a channel. It is a set of `key/value` pairs, where both keys and values are `strings`, `keys` correspond to specific features that we provide. Currently, these two param values are supported:

 #### `metrics`
 
 It is an `object`, an optional dictionary of membership categories for a real-time channel and their counts which enables events containing the full `Occupancy` details in `data` payload. There are different categories of Metrics:
 
Member | Type | Description |
|---|---|---|
| publishers | Integer | Number of connections attached to the channel that are authorized to publish |
| subscribers | Integer | Number of connections attached that are authorized to subscribe to messages |
| presenceSubscribers | Integer | Number of connections that are authorized to subscribe to presence messages |
| presenceConnections | Integer | Number of connections that are authorized to enter members into the presence channel |
| presenceMembers | Integer | Number of members currently entered into the presence channel |

 * Subscription example with `occupancy metrics` with SDK version <1.2.

    ```javascript
    const ably = new Ably.Realtime({ ... });
    const channelName = '[?occupancy=metrics]foobar';
    const channel = ably.channels.get(channelName);
    channel.subscribe('[meta]occupancy', (msg) => {
    console.log(msg.data);
    /* example msg data
        { "metrics":
        { "connections": 2,
            "publishers": 2,
            "subscribers": 2,
            "presenceConnections": 2,
            "presenceMembers": 0,
            "presenceSubscribers": 2
        }
        }
    */
    });
    ````
   Here we have an example of a channel with a `metrics` object. We added the `occupancy` param and subscribed to the channel. In last we are getting the value of every `metrics` category which is an `Occupancy` value. Events with metadata have the prefix `[meta]` added in the name. Event Name of `Occupancy` messages is **`[meta]occupancy`**.


#### `metrics.<category>`
  
  It enables events whose `data` payload contains an `Occupancy` value containing the occupancy for only the specified category. For Example, if you subscribe to a single category lets say `publishers`, then only that member will be present as you can see below:


```
{
  name: '[meta]occupancy',
  data: {
    metrics: {
      publishers: 2
    }
  }
}
```

**NOTE:  Events are sent when the count for one of the included categories changes with a maximum rate of 1 message/second.**
