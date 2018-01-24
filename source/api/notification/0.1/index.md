# IIIF Notifications

This information is extracted from the [Crawling](/api/harvest/) work, as it applies to notifications using the [ActivityStreams](https://www.w3.org/TR/activitystreams-core/) pattern.

## Notifications of Change

[Linked Data Notifications](https://www.w3.org/TR/ldn/) allow for these notifications to be sent to and made available from an "inbox", and thus we would be consistent with section 3 of the charter around Notification.  Further, the [ActivityPub](https://www.w3.org/TR/activitypub/) specification introduces the additional notion of an "outbox".

Sending Manifests to an Inbox is not a "notification", it's just moving data around. Instead we send a description of the activity, which the reciever can process or not as it chooses.
 
LDN provides a storage and publication mechanism, but clients are still expected to come and retrieve the set of changes in the same way as above.  Assuming that either (a) systems register their interest in a resource, and the publisher notifies them directly or (b) the publisher writes to the inbox, and the inbox then auto-forwards the notification to the subscribers ... then there needs to be a method of subscribing to have the changes pushed to a remote inbox.

### Linked Data Notifications: Subscription

While there is [WebSub](https://www.w3.org/TR/websub/), this does not interoperate with LDN and is from a service oriented mindset. WebSub makes various requirements that are not appropriate for Notifications (you MUST send the entire contents of the resource that you have subscribed to) which prevents subscribing either to binary resources (you don't want the entire gigabyte TIFF) or or aggregation-level containers (you don't want every change, every time, just the new one).

ActivityPub is a method for transferring the notifications and subscribing, but does not follow REST patterns. Instead it uses side effects associated with particular activities. For example, the side effect of recieving a "Follow" activity is to add the sender to the "followers" of the resource. A REST based approach would be to create the subscription resource within the followers container.

A more LOD / LDP / LDN appropriate pattern would be to use a REST pattern with Containers, that allow subscription to the inbox, manage filtering and the callback endpoint.  One way to do this could be as [described here](https://docs.google.com/document/d/1JsQS1LVFt8wuJSYo_XsOzyP28pS8hfSLEhAC3BFuN6o/edit).

For the purposes of IIIF discovery, the above decision is firmly in the Notification section of the charter and does not impact the publication of the ActivityStreams Collections as resources.
