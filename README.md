# A node-red node to facilitate guaranteed delivery of messages (upload, email, MQTT etc) across a network

This node facilitates the guaranteed delivery of data across a network, or via some other route, where it may not always be possible to send the data, due to network outage or server not available.  If a send fails then messages are automatically buffered in a queue and the send is retried later. Messages are guaranteed to be sent in the order they are received.

If persistent context is configured then the queue will be retained in context so that the queue will survive a node-red restart or power down.

To use the node it should ideally be possible to derive a success message when the data has been successfully delivered, and a fail message if it fails. In some circumstances it may not be possible to know that the send has failed, in which case a timeout setting can be used to tell the node that, if no success is received within a certain time, then it should assume that the send has failed.

In use, feed messages to be delivered into the node.  Connect the output to the nodes that deliver it (to an email node for example).  Then arrange for an OK message to be fed back into the node when the send succeeds, or a FAIL message to be fed back if the send fails. See below for how to build OK/FAIL messages.

### Installing
Generally the node should be installed using Manage Palette and searching for @colinl/node-red-guaranteed-delivery.  Alternatively it may be installed from the node-red user directory (usually ~/.node-red) using
`npm install @colinl/node-red-guaranteed-delivery`

### Configuration

##### Property for OK/FAIL
This defines which message property will be used to indicate that a message is a Control message, not one to be sent on for delivery.

##### Value for success
This defines the contents of the control property that will indicate that a message has been successfully sent.

##### Value for failure
This defines the contents of the control property that will indicate that an attempt to send a message has failed.

##### Retry period (secs)
When a message fails to be sent then after this time the node will pass it on again for another attempt

##### Fail timeout (secs)
If the node does not receive OK or FAIL after passing on a message, then after this time it will assume that it has failed.  This can be used on occasions where a timeout is the only way of knowing that it has failed, but it is also a good idea to set a value here longer than it will normally take for success or failure, in case a message gets lost.  This prevents a permanent lockup situation.  Setting this to 0 disables the timeout.

##### Max messages in queue
If this is non-zero then it will limit the number of messages held in the queue, once the queue is full then further messages received will be discarded.  If this is set to zero then no limit will be applied.

##### Context store to use
This determines which context store will be used to hold the queue.  See the docs on [node-red-context-stores](https://nodered.org/docs/user-guide/context#context-stores). If a persistent context store is selected then the queue will be retained over a node-red restart or a power failure.

### Inputs

If `msg.control` (or whatever property has been configured as the Control property as above) is not present in the message then the message will be passed on to the next node in the flow (such as an email node) for delivery to its destination.  If there are already messages queued then it will be added to the queue rather than being passed on.

If `msg.control` is present then its value should be either that configured as Value for Success or Value for Failure as described above.  
If it is the Success value then that indicates that the message has been successfully delivered, it will be removed from the queue of waiting messages and the next one (if any) will be sent.
If it is the Failure value than the message will remain in the queue and sending will be retried after the configured Retry Period, or if the node is passed another message to be sent.  No message will be sent until all preceding messages have been successfully sent.

### Examples

There are two example flows that can be imported by selecting Import from the dropdown menu, then Examples, and @colinl/node-red-guaranteed-delivery.

##### guaranteed-email
This demonstrates how the node can be used to guarantee that messages are successfully delivered to email server. This does not, of course, guarantee that they have been delivered to the recipient as there is no way of knowing that.
To try this, edit the `Send test email` node and enter an appropriate email recipient in `msg.to`. Edit the email node and enter your email server and credentials. If the email delivery (to the server) is successful then it will wait for more messages. If the delivery fails then it will retry approximately every 60 seconds until it succeeds. In the meantime any additional requests will be queued and will be sent when possible.

To modify the flow to deliver to something other than email then replace the email node with a node or nodes to perform the required delivery action.  Remove, or re-configure the Complete and Catch nodes and replace them with nodes to send an OK control message back to the node when the delivery works, and a FAIL message back in case it fails, in the same way as is done in the sample flow. It is imperative that either an OK or a FAIL message is sent, and not both.

##### guaranteed-mqtt-publish
This example shows how the node can be used to guarantee the data to be published to MQTT is delivered to the MQTT broker.  
This requirement is more complex than the email example, if the network fails, there may be no indication of this, other than the Complete node is not triggered.  The node's internal timer is used here to determine that 5 seconds has elapsed since the message was passed to the MQTT node and the complete node has not triggered.  It is then assumed that it has failed.  
In addition the MQTT Out node may buffer messages that cannot be sent, and send them when the broker becomes available.  This means that it is necessary to actively check the output from the Complete node to see which message has been sent.
Note that when using this flow the client subscribing to the topic must allow for the fact that messages may be published more than once.
To try this example, import it, configure the MQTT node as required, and pass messages to be published to the Link In node labelled `Send to MQTT`.
Note that this flow only guarantees that the data has been published, not that it has been received by any subscribed clients.

### Issues
Any issues with the node can be reported at https://github.com/colinl/node-red-guaranteed-delivery/issues