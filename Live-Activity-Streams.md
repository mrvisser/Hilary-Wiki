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

In this configuration, a new top-level configuration element "streams" exists to hold the names of streams as keys. Then you can define the router of the stream, and other aspects such as email, or perhaps stream-specific configuration properties in the future.

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

Implementation gotchas:

* Currently, the "activity" stream keys currently look like `<routeId>`, whereas "notification" stream keys look like: `<routeId>#notification`. When making this change, we will need to generalize the routes to have key `<routeId>#<streamName>`, which indicates we'll need migration to copy all routes `<routeId>` to `<routeId>#activity`