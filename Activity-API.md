# Example Activities

First, natural language activity exploration:

* User posted comment to content
  * User posted comment to "Content Name"
* User updated comment on content
* User deleted comment on content
* User created content
    * User created link
    * User created file
        * User "uploaded" file
        * User created image
        * User uploaded a video
    * User created link titled "Google Website"
    * User created Document titled "My Document"
* User updated content
* User updated link
* User updated file
    * User "uploaded new" file
    * User updated image
    * User updated a video
* User changed visibility
* User deleted content
    * User deleted link
    * User deleted file
        * User deleted an image
        * User deleted a document
* User shared content with User
    * User shared a link with User
    * User shared a file with User
        * User shared a video with User
* User added User to Group
* User added Group to Group

# Modelling

## Activity Representations

### Activity Task "Seed" Model

The cheapest representation possible of an activity that has just occurred. This model is created by the application server and fired as a RabbitMQ task to be processed by an activity server. The minimum data required on this model is all data that should not require any resources from the application server. In some cases, additional data may be attached to the seed, but should only be done so if the cost for doing it is substantially cheaper for the application to do it than it is for the activity server (e.g., if the activity is that a content item was created, attach the content item to the seed since the application server already has it. If you don't, then the activity server will have to re-fetch it, which is wasteful).

**Example "Post Comment" Seed Model:**

```javascript
{
    "activityType": "comment-post",            // Required
    "verb": "post",                            // Required
    "actorResource": {                         // Required
        "resourceType": "user",                // Required
        "resourceId": "u:oae:mrvisser",        // Required
        "data": {}                             // Optional. Just stores any information we had "on hand" to avoid unnecessary fetches (e.g., the user object)
    },
    "objectResource": {                        // Optional
        "resourceType": "comment",             // Required
        "resourceId": "c:oae:cDFjw9/fjw87dF",  // Required
        "data": {}                             // Optional
    },
    "targetResource": {                        // Optional
        "resourceType": "content",             // Required
        "resourceId": "c:oae:cDFjw9",          // Required
        "data": {}                             // Optional
    }
}
```

### Activity "Persistent Model

This is the data that is persisted into the activity routes. It is a highly read-optimized denormalization of the activity data, so that additional resources are not required to fill in activity data when they are read. The "Produced Entity" referenced in the schema is the entity that is created for an objectType as regstered through ActivityAPI.registerActivityEntityProducer. The actual schema of the produced entity is not important, the important thing is that the "Entity Transformer" (which creates the "UI" model below) knows how to read it it.

**Example "Posted Comment" Persistence Model:**

```javascript
{
    "oae:activityType": "comment-post",         // Required
    "published": "2011-02-10T15:04:55Z",        // Required
    "verb": "create"                            // Required
    "actor": { <ProducedEntity> },
    "object": { <ProducedEntity> },
    "target": { <ProducedEntity> }
}
```

### Activity "UI Model ("Transformed")

Intended to be consistent with the [ActivityStrea.ms](http://http://activitystrea.ms/) model specification. The Activity "UI" model is created from the persistent model data by handlers registered through ActivityAPI.registerActivityEntityTransformer. Here is a sample model:

**Example "Share Content" Persistence Model:**

```javascript
{
    "oae:activityType": "content-share",
    "published": "2011-02-10T15:04:55Z",
    "verb": "share"
    "actor": {
        "objectType": "user",
        "id": "http://my.oae.org/api/user/u:oae:mrvisser",
        "displayName": "Branden Visser",
        "url": "http://some.oae.org/~u:oae:mrvisser",
        "image": {                                            // Optional, specifies a profile thumbnail url for the user
            "url": "https://.../api/download?...",
            "width": 30,
            "height": 30
        }
    },
    "object": {
        "objectType": "content",
        "oae:contentType": "file",
        "oae:mimeType": "image/png"
        "id": "http://my.oae.org/content/contentId",
        "url": "http://my.oae.org/content/contentId",
        "displayName": "Super cool image",
        "image": {                                            // Optional, specifies a preview for the content item
            "url": "https://.../api/download?...",            // Required
            "width": 350,                                     // Required
            "width": 350                                      // Required
        }
    },
    "target": {
        "objectType": "user",
        "id": "http://my.oae.org/user/u:cam:bert",
        "url": "http://my.oae.org/~u:cam:bert",
        "image": {                                            // Optional, specifies a profile thumbnail url for the user
            "url": "https://.../api/download?...",
            "width": 30,
            "height": 30
        }
    }
}
```

## 3akai-ux Model Interpretation

Here are some thoughts as to how the 3akai-ux could interpret the UI model, reacting based on the information provided (e.g., how it could be an OAE ActivityStrea.ms consumer).

### Activity Interpretation

The OAE back-end will guarantee a **oae:activityType**, **published**, **verb** and an **actor**. Where there is an appropriate **object** and **target**, they will be returned. Given the oae:activityType, 3akai-ux can make more advanced assumptions about how the data is related, and can potentially improve the language, template and UX of the activity.

### Verb Interpretation

The verbs are going to be from a core predefined set of verbs that are rather generic (create, update, delete, share, add, like). For a consumer, more advanced assumptions can be made about the language of these given the activity type and related actor, object and target objectTypes. For example, if someone comments on content, you may have verb "create", as the user created a comment. If there was no special UI handling, you would have a generic "User a 'created' 'objectType' on 'targetDisplayName'" message on the UI. With the activity type, you can key on custom functionality to make more advanced assumptions about the context, and instead change the language to "User commented on...", you could template the activity such that a comment-bubble appears, and you can take the "summary" property of the comment and drop it into the speech bubble.

In the case of creating content, you could detect the oae:mimeType of the content item and use the "image" (MediaLink) of the **object** to embed an image or video of the content item.

### Actor Interpretation

The actor is the user who performed the activity. They have the following attributes:

* **objectType:** (required) Probably always "user"
* **displayName:** (required) Plain-Text to display on the UI that describes the user
* **url:** (optional) The URL to the user's profile. If not supplied, UI could just render plain-text name.
* **image:** (optional) The URL to the user's profile photo. If not supplied, could have nothing, or a placeholder image

### Object Interpretation

The object is the resource that was created / updated / deleted / shared etc... Has the following attributes:

* **objectType:** (required) The type of the resource (content, user, group, comment, etc..)
* One or more of the following will be available:
    * **image:** e.g., User thumbnail, content preview
    * **summary:** e.g. (abbreviated) comment content
    * **displayName** e.g., group name, content name, user displayName
* **url:** (optional) Specifies a link to the item
* Extension properties
    * **oae:contentType:** A content item can put something like file, link here
    * **oae:mimeType:** If a file, it could image/png for example.

### Target Interpretation

The target is the target of the activity. The target is largely dependent on the verb, but is usually what would be considered to have the preposition "to" in the activity language (e.g., John added a content item **to a group** – the group would be the target).

* **objectType:** (required) The type of the resource (content, user, group, comment, etc...)
* **image:** (optional) A thumbnail representation of the target. No large previews for content will likely be displayed here
* **displayName:** (optional) A display name for the target (e.g., name of group, name of content..)
* **url:** (optional) Specifies a link to the item

# Interfaces

**TODO:** Update docs to indicate recent changes with registering associations, which can be used both for propagation and routing

## Activity Entity Producer

The producer is registered into the ActivityAPI along with a unique "ResourceType" string. The producer will accept an id (as specified in its own seed object) and will expand that to a Persistent Entity, as per the representation above. Producing the activity objects is not done on the application servers, it is offloaded to an activity slave.

## Activity Entity Router

The router is registered into the ActivityAPI with a unique resource type string. It will accept a produced activity entity (e.g., the output from a produced content item, or user), and it will return the following information:

* Activity Routes: The User and Group IDs to which this activity should be routed
* Notification Routes: The User IDs for the users who should receive a notification of this activity
* Propagation: Since the routes specified here are not the only routes where this data is routed (e.g., 3 sets of routes are produced, for actor, object and target), it is important for a router to define how the entity is allowed to be propagated to other routes. If there are routes to which this entity is not allowed to be propagated, then this activity is not delivered to that route. This is the essence of permission/privacy in activities. The possible propagation values are:
    * ANY: The produced entity data can be routed to any route. i.e., The entity is public or loggedin
    * ROUTES (default): The entity can only be propagated to the activity routes specified by the router
    * SPECIFY: This entity can only be propagated to streams that the router has specified. An additional "specify" property is available for a list of routes to which the entity may be propagated

It should be noted that for every Activity Entity Producer, there should be an Activity Entity Transformer that knows how to convert that model into a UI model when the activity stream is read.

## Activity Entity Transformer

The transformer is registered into the ActivityAPI with a unique resource type string. It will accept a produced activity entity (e.g., the output from a produced content item, or user), and it will return a UI entity model as per the ActivityStrea.ms specification, described previously. The activity transformer is invoked at read-time for every entity (actor, object, target) in every activity that is read, which should be quite often. For this reason, any data that you need to output in the activity stream that you need to fetch from storage or compute, if it can be computed/fetched and persisted by the entity **producer**, it should be done that way. In some cases this will not be possible (e.g., when creating a content item, the activity is probably produced and persisted before the preview URL is read), in which case you cannot get away from using some resources.

# Routing

**TODO:** Documentation was not migrated as it is out of date. Document configuration-based activity routing when registering new activities. Also add note about privacy VIA the propagation configuration. Can reference the ActivityEntityRouter propagation description under interfaces.

# Aggregation

Aggregation is the process of taking activities that have happened, matching them with activities that have recently occurred, and rolling them up ("aggregating") them into one single activity. How an activity is aggregated is configurable based for each "activity type", based on a "groupBy" option which specifies which ways an activity can be matched with one another. This activity type customization is configured by registering an activity via ActivityAPI.registerActivityType.

Examples:

**Content Creation:** Content create activities are registered with a group by specification of "actor". This means that content-create activities will aggregate on "actor":

1. **Activity #1:** Branden created "Syllabus"
2. **Activity #2:** Branden created "Introduction to Basket Weaving"

In this case, the actor of these 2 activities are the same, so they are aggregated together and the "object"s are collected as an array of objects in the activity. The result would say something like "Branden created 2 content items", and the activity entity for each is available in the model.

**Content Sharing (multiple groupings):** Content sharing is an example of an activity that aggregates based on 2 combinations of groups: actor+object and actor+target.

1. **Activity #1:** Branden shared Syllabus with GroupA
2. **Activity #2:** Branden shared Syllabus with GroupB
3. **Activity #3:** Branden shared Introduction to Basket Weaving with Group A

In this case, it is possible for these 3 activities to aggregate into two separate activities. We have:

1. **Aggregated Activity #1:** Branden shared Syllabus with 2 groups
2. **Aggregated Activity #2:** Branden shared 2 content items with Group A

Another example of needing 2 aggregate groupings would be to aggregate the "following" activity into either: "Branden followed 7 users" or "7 users followed Branden".

## Aggregate Buckets

When activities are routed, they are immediately placed into buckets. Aggregation Buckets hold routed activities that are waiting to be aggregated by activity slaves. There are a couple gotchas in activity aggregation that lead to these buckets being necessary:

1. Activity Aggregation can be full of race conditions. The process of aggregating one activity into another results in the need to insert a new activity and delete an old one. If you have a cluster of activity nodes and 2 activities get aggregated into an older one at the same time, which will probably happen often, then you can end up with all sorts of duplicate activities in a stream, how annoying!
2. The solution to #1 is to serialize the aggregation so 2 activities aren't aggregated into the same activity at one time. However, serializing down to 1 activity node will result in simply not being able to aggregate activities fast enough. It needs to scale better than that.

Therefore, activities are allocated into a configurable number of buckets, each bucket is processed by only one activity node at a time. The bucket to which an activity is allocated is determined by the modulus of a hash. The hash is computed based on the activity type and the route, which ensures that any activity for the same type and route will always be processed by one node, and race conditions are no longer an issue.

This does imply that if the throughput of a particular activity type for a particular route is faster than a single node can process, then we have reached an upper limit to how far this technique can scale. If that becomes an issue, I believe the hash can be further discriminated based on a computation of the "groupBy" specification of an activity. This *may* be a necessary optimization if an "anonymous public route" is implemented, as it will incur very many activities.

## Aggregation Process

Now that we know why buckets exist, lets talk about how aggregation happens. A more detailed explanation of how this works will require reading through the [very well-documented source code](https://github.com/sakaiproject/Hilary/blob/master/node_modules/oae-activity/lib/internal/aggregator.js). Here is an overview:

1. All activity nodes poll Redis to find buckets that have routed activities sitting in them
2. Once an activity node finds a bucket with routed activities to be collected, it locks the bucket (using Redis for locking)
3. The node grabs a batch of routed activities and determines which ones match previous activities that occurred, by extracting an "aggregate key" which is used as a redis key
4. For each matching aggregate key, it will merge (aggregate) the contents of the routed activity, persisting the new aggregated data. For any routed activity that didn't match an aggregate key, new aggregate keys will be created with that activity data
5. It will then take the full data associated with the aggregate and persist it into the route ("delivery")
6. The activity node will then delete the collected activities from the bucket and then query for the next batch
7. When there are no more routed activities in the bucket, the activity node will unlock the bucket and wait to poll again

The in-memory aggregate keys expire after a configurable amount of time (see config.js for more details). Also, "volatile-ttl" expiration should be configured into the Redis server so if you experience an unexpectedly high volume of data, the redis memory foot print will start short-circuiting the expiry of aggregate keys. The impact of short-circuiting the expiration is simply a shorter idle time for activities, not data-loss.

For performance analysis, see [this github issue](https://github.com/sakaiproject/Hilary/pull/351). After performance improvements were made, the result of pushing in 4 batches of data (4000 users, 8000 goups and 20,000 content items), with 3 xsmall activity nodes and one 8Gb Redis server, we had a max throughput of 1500 routed activities per second, and Redis hit an upper memory consumption of 4Gb. The same test with 6 activity nodes was 2500+ routed activities per second, same memory footprint.

# Activity Gloassary

Here is a glossary of known activities. This will probably become out-dated in no time:

<table>
  <thead>
    <tr>
      <th>Action
      <th>Activity Type
      <th>Actor type
      <th>Object type
      <th>Target type
      <th>Aggregate Groupings
      <th>Notification?
      <th>Email?
  <tbody>
    <tr>
      <td>User creates a content item
      <td>content-create
      <td>user
      <td>content
      <td>:x:
      <td>actor
      <td>Content Members
      <td>:white_check_mark:
    <tr>
      <td>User uploads a new version of a file
      <td>content-revision
      <td>user
      <td>content
      <td>:x:
      <td>object
      <td>:x:
      <td>:x:
    <tr>
      <td>User updates content details
      <td>content-update
      <td>user
      <td>content
      <td>:x:
      <td>object
      <td>:x:
      <td>:x:
    <tr>
      <td>User updates the visibility of content
      <td>content-update-visibility
      <td>user
      <td>contnet
      <td>:x:
      <td>:x:
      <td>:x:
      <td>:x:
    <tr>
      <td>User shares content item
      <td>content-share
      <td>user
      <td>content
      <td>user
      <td>actor+object, actor+target
      <td>Target user
      <td>:white_check_mark:
    <tr>
      <td>User adds content to their library
      <td>content-add-to-library
      <td>user
      <td>content
      <td>:x:
      <td>actor
      <td>:x:
      <td>:x:
    <tr>
      <td>User comments on a content item
      <td>content-comment
      <td>user
      <td>content-comment
      <td>content
      <td>target
      <td>content managers, Recent commenters
      <td>:white_check_mark:
    <tr>
      <td>User creates a group
      <td>group-create
      <td>user
      <td>group
      <td>:x:
      <td>actor
      <td>Group members
      <td>:white_check_mark:
    <tr>
      <td>User updates a group's details
      <td>group-update
      <td>user
      <td>group
      <td>:x:
      <td>object
      <td>:x:
      <td>:x:
    <tr>
      <td>User updates the visibility of a group
      <td>group-update-visibility
      <td>user
      <td>group
      <td>:x:
      <td>:x:
      <td>:x:
      <td>:x:
    <tr>
      <td>User adds a member (or changes role) to a group
      <td>group-add-member
      <td>user
      <td>user
      <td>group
      <td>target
      <td>Object user
      <td>:white_check_mark:
    <tr>
      <td>User joins a group
      <td>group-join
      <td>user
      <td>group
      <td>:x:
      <td>object
      <td>:x:
      <td>:x:
</table>