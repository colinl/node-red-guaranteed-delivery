# A node-red node to facilitate guaranteed delivery of messages (upload, email, MQTT etc) across a network

This node facilitates the guaranteed delivery of data across a network or via some other route where it may not always be possible to send the data, due to network outage or server not available.  If a send fails then messages are automatically buffered in a queue and the send is retried later. Messages are guaranteed to be sent in the order they are received.

If persistent context is configured then the queue can be retained there so that the queue will survive a node-red restart or power down.

To use the node it should ideally possible to derive a success message when the data has been successfully delivered, and a fail message if it fails. In some circumstances it may not be possible to know that the send has failed, in which case a timeout setting can be used to tell the node that, if no success is received within a certain time, then it should assume that the send has failed.

To use this node feed it messages to be delivered.  Connect the output to the nodes that deliver it (to an email node for example).
