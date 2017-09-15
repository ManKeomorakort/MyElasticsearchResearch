# MyElasticsearchResearch

# Partial update on field that is not indexed

what if we do particial update of not indexed field - will elasticsearch reindex the entire document?

Yes, whilst the views field is not indexed individually it is part of the _source field. The _source field contains the original JSON you sent to Elasticsearch when you indexed the document and is returned in the results if there is a match on the document during a search. The _source field is indexed with the document in Lucene. In your update script you are changing the _source field so the whole document will be re-indexed.

Could you then evaluate the following strategy. Every time someone reads the article I send update to elastic. However refresh_interval I set to 30 seconds. Will this strategy be normal if during 30 second interval about 1000 users have read one article?

You are still indexing the 1000 documents, 1 document will be indexed as the current document, 999 documents will be indexed marked as deleted and removed from the index during the next Lucene merge.
