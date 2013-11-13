# Elastic Search

We are using ElasticSearch for search. ElasticSearch was built from the ground up as a distributed full-text search technology that clusters and scales well. Like Solr, it uses Lucene under the hood, and is primarily used through an HTTP API.

To learn more about ElasticSearch, the presentations provided on their website offer a good overview on how to cluster an ElasticSearch server.

# Workflow

## Searching

All searches go through the REST endpoint `/api/search/<searchType>/...`. Search types are registered using the `SearchAPI.registerSearch` method, where you can provide the string label of the search (i.e. what is used in the URL path parameter) and the function that will generate the search query.

Additionally, you can provide **document transformer**s for a particular resource type (e.g., user, group, content) that will have an opportunity to convert a given set of search results into results that are safe to display to the user. Unless you are providing a new type of resource, this is not necessary to extend if you're just adding a new search.

The following is a workflow diagram of how general search would work:

[[resources/SearchProcess.png]]

## Indexing

An indexing operation can be triggered by firing an event through the MQ.submit method. The following tasks are handled by the search api:

* `SearchConstants.mq.TASK_INDEX_DOCUMENT`
* `SearchConstants.mq.TASK_DELETE_DOCUMENT`

The data for an indexing task looks like the following:

```javascript
MQ.submit(SearchConstants.mq.TASK_INDEX_DOCUMENT, {
    'resourceType': 'group',
    'resources': [{
        'id': groupId,
        'opts': {
            'indexResource': true,
            'indexMembers': false,
            'indexMemberships': false
        }
    }]
});
```

In the above example, a group document with the id groupId will be indexed. The following opts tell search worker to do the following:

* **indexResource:** When true, tells the indexer that you want to index the actual group data (e.g., title, name, etc...). In this case, the resource data will be sent to the registered group document producer.
* **indexMembers:** When true, the worker will update the indexed group members of the group
* **indexMemberships:** When true, the worker will update the indexed group memberships (e.g., the direct groups to which the provided group is a member)

You can find more examples in the `search.js` component files of the `oae-principals` and `oae-content` modules.

For each resource type, a single document producer can be registered using `SearchAPI.registerDocumentProducer`. This is responsible for taking the task resource data, and producing a set of search documents to be indexed:

[[resources/SearchIndexing.png]]
