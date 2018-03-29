# MyElasticsearchResearch

# Partial update on field that is not indexed

what if we do particial update of not indexed field - will elasticsearch reindex the entire document?

Yes, whilst the views field is not indexed individually it is part of the _source field. The _source field contains the original JSON you sent to Elasticsearch when you indexed the document and is returned in the results if there is a match on the document during a search. The _source field is indexed with the document in Lucene. In your update script you are changing the _source field so the whole document will be re-indexed.

Could you then evaluate the following strategy. Every time someone reads the article I send update to elastic. However refresh_interval I set to 30 seconds. Will this strategy be normal if during 30 second interval about 1000 users have read one article?

You are still indexing the 1000 documents, 1 document will be indexed as the current document, 999 documents will be indexed marked as deleted and removed from the index during the next Lucene merge.

# Problem(s)

**Elasticsearch indexed data sudently removed**

 Solution:
 
  1- Not allow port 9200/9300 to access from public.
  
  2- set **script.engine.groovy.inline.search: off** (By default is off). This will avoid hacker to inject code via search query. 
  
  For example, i checked in log file and i found that hacker try to inject code via search query as below : 
  
    [failed to parse search source[{
       "size": 1,
       "query": {
        "filtered": {
         "query": {
          "match_all": {}
         }
        }
       },
       "script_fields": {
        "exp": {
         "script": "import java.util.*;\nimport java.io.*;\nString str = \"\";BufferedReader br = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec(\"rm *\").getInputStream()));StringBuilder sb = new StringBuilder();while((str=br.readLine())!=null){sb.append(str);}sb.toString();"
        }
       }
      }]];
      nested: ScriptException[scripts of type[inline], operation[search] and lang[groovy] are disabled];
      Caused by: SearchParseException[failed to parse search source[{
       "size": 1,
       "query": {
        "filtered": {
         "query": {
          "match_all": {}
         }
        }
       },
       "script_fields": {
        "exp": {
         "script": "import java.util.*;\nimport java.io.*;\nString str = \"\";BufferedReader br = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec(\"rm *\").getInputStream()));StringBuilder sb = new StringBuilder();while((str=br.readLine())!=null){sb.append(str);}sb.toString();"
        }
       }
      }]];
      nested: ScriptException[scripts of type[inline], operation[search] and lang[groovy] are disabled];
      
      
