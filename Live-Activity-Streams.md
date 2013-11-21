The following changes to activity configuration (and appropriate changes to the router) should allow for using Activity for all currently known push notifications use-cases:


## "Live Streams"

For this we can introduce the concept of "Subscribing to a live stream", which will deliver activities directly to open websocket connections.

## Configuration Change Examples

The key to this change is to promote the concept of a "Stream". Before this was a hard-coded concept in activity, but with this change we will make streams pluggable and configurable entities. The configuration / registration of streams can be out-scoped as custom configuration per stream is likely not necessary to cover push notifications, therefore at this time new streams can be added in an ad-hoc manner VIA activity configuration rather than an explicit registration pattern.

Once we have promoted the concept of a stream, we can subscribe to a "live feed" of any stream that has been introduced -- this is where push can be used to deliver activity in real-time to the web-socket connections.

This requires a change in the structure of the activity configuration to allow for specifying (in an adhoc manner for now since we don't need explicit stream registration) how each activity is routed to different streams.

Here is the direct conversion from the old activity configuration schema to the new one, by using content-create as an example, which doesn't need any logical configuration change:

```javascript
ActivityAPI.registerActivityType(ContentConstants.activity.ACTIVITY_CONTENT_CREATE, {
    'groupBy': [{'actor': true}],
    'streams': {
        'activity': {
            'router': {
                'actor': ['self', 'followers'],
                'object': ['members']
            }
        },
        'notification': {
            'router': {
                'object': ['members']
            },

            /**
             * The inclusion of email here is fine for now but may change later. Having
             * this here suggests that we could send email based on any stream, but that
             * may be way too complex to implement. If it becomes difficult to support,
             * we can consider "email" as something that is specific to the "notification"
             * stream, with the idea of providing custom stream functionality down the
             * road that an "oae-notification" module (for example) could utilize to
             * to handle notifications and sending emails.
             *
             * In summary, don't put any effort into supporting the ability to plug email
             * into any arbitrary stream.
             */
            'email': {
                'emailTemplateModule': 'oae-content',
                'emailTemplateId': 'notify-content-share'
            }
        }
    }
});
```

In this configuration, a new top-level configuration element "streams" exists to hold the names of streams as keys. Then you can define the router of the stream and other aspects such as email, or in the future stream-specific `groupBy` rules or other custom stream properties.

This then allows us to introduce new streams. Using the use-case of "subscribe to content comment push notifications" as an example, here is how this could be configured:

```javascript
ActivityAPI.registerActivityType(ContentConstants.activity.ACTIVITY_CONTENT_COMMENT, {
    'groupBy': [{'target': true}],
    'streams': {
        'activity': {
            'router': {
                'actor': ['self'],
                'target': ['message-contributors', 'members']
            }
        },
        'notification': {
            'router': {
                'target': ['message-contributors', 'managers']
            },
            'email': {
                'emailTemplateModule': 'oae-content',
                'emailTemplateId': 'notify-content-comment'
            }
        },

        'message': {
            'router': {
                'target': ['self']
            }
        }
    }
});
```

There are 2 fundamental changes in activity here:

1. Now the activity is routed to a new stream called "message". This stream would receive only the following activities based on the routers that are provided:
  * content-comment
  * content-comment-delete
  * discussion-message
  * discussion-message-delete

2. The activity is now routed to a *content item* (i.e., the "self" association of the `target` content item). Note that in implementation, the "self" association will need to be registered as a new association applicable to content and discussion resource types. There may also be some easter-eggs deep within activity (look in `router.js`) that make an assumption that routes are only users and groups. I might actually strip out any non-principal items from the routes to avoid bugs that extenders could accidentally introduce. That logic can be ripped out to support this. Also, association-based propagation (i.e., propagation of type `ActivityConstants.entityPropagation.ASSOCIATION`) will currently only allow for items to be routed to user and group feeds (the "members" association). It is probably fine to update `ActivityUtil.getStandardResourcePropagation` to simply always allow propagation to the 'self' association in addition to 'members'. This will ensure that content propagation won't restrict activities from being routed to its own "message" route.

Then, content-comment-delete could be introduced as a new activity as such:

```javascript
ActivityAPI.registerActivityType(ContentConstants.activity.ACTIVITY_CONTENT_COMMENT_DELETE, {
    'streams': {
        'message': {
            'router': {
                'target': ['self']
            }
        }
    }
});
```

Since we simply don't have an `activity` or `notification` router, we are no longer concerned with this being delivered to activity streams.

## Subscribing to a "Live Stream"

The [SockJS](https://github.com/sockjs/sockjs-node) endpoint is exposed at `/api/push`. Only websocket connections should be attempted to be made.
The websocket is full-duplex, but clients will typically only send `messages` when authenticating/subscribing.
A `message` is a stringified JSON object which looks something like:
```javascript
var msg = {
    "id": <id>,
    "name": <authentication|subscribe>,
    "payload": <payload>
};
```
The properties are:
 * `id` - A unique string. When the server sends it reply, it will include this ID. This allows the client to connect a reply to a previously sent message
 * `name` - One of `authentication` or `subscribe`. See below
 * `payload`- An object that depends on what kind of message is being sent. See below

Once the socket has been set up, the very first thing the client should do is try to authenticate themselves. The server will close the socket if anything besides an authentication message gets sent.

An authentication message could look like this:
```javascript
var msg = {
    "id": 42,
    "name": "authentication",
    "payload": {
        "userId": oae.data.me.id,
        "tenantAlias": oae.data.me.tenant.alias,
        "signature": oae.data.me.signature
    }
};
```

We need to authenticate the client on the socket via a signature as the websocket doesn't support headers/cookies. Once authenticated, the server will send a very basic response back:
```javascript
{
    "id": 42
}
```
In case the authentication failed, the server will send and error response and **close** the socket:
```javascript
{
    "id": 42,
    "error": {
        "code": 400,
        "msg": ".."
    }
}
```

Once authenticated, the client can start subscribing for activities. This can be done by sending the following message:
```javascript
{
    "id": 19,
    "name": "subscribe",
    "payload": {
        "resourceId": <userId | groupId | contentId | discussionId>,
        "activityStreamId": <activity | notification | message>,
        "token": { .. }
    }
}
```

Possible activity stream ids:
 * `activity` - all resources have this
 * `notification` - only user resources have this
 * `message` - only content and discussion resources have this (related to new comments)

The token (=signature) can usually be retrieved from the resource's profile endpoint (`/api/group/groupId`, `/api/content/:contentId`, ..)
It's not necessary to pass in a token for a user feed.

A client can subscribe on multiple feeds in parallel.

The same response mechanisme as with authentication is used here.

## NGINX config
Add the following to the tenant server
```
        location /api/push/ {
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_pass http://tenantworkers;
            proxy_redirect off;
            proxy_buffering off;
        }
```