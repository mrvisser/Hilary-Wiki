# Elastic Search

We are using ElasticSearch for search. ElasticSearch was built from the ground up as a distributed full-text search technology that clusters and scales well. Like Solr, it uses Lucene under the hood, and is primarily used through an HTTP API.

To learn more about ElasticSearch, the presentations provided on their website offer a good overview on how to cluster an ElasticSearch server.

# Workflow

## Searching

All searches go through the REST endpoint `/api/search/<searchType>/...`. Search types are registered using the `SearchAPI.registerSearch` method, where you can provide the string label of the search (i.e. what is used in the URL path parameter) and the function that will generate the search query.

Additionally, you can provide **document transformer**s for a particular resource type (e.g., user, group, content) that will have an opportunity to convert a given set of search results into results that are safe to display to the user. Unless you are providing a new type of resource, this is not necessary to extend if you're just adding a new search.

The following is a workflow diagram of how general search would work:

[[resources/SearchProcess.png]]