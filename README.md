# elastic_workshop


#Setup elastic with index and seed data
Download sense from chrome store. It's plugin that will maker our life easier during the workshop.

Create index for movies

PUT movies
PUT movies with mapping.json

curl -s --header "Content-Type:application/json"  -XPOST localhost:9200/_bulk --data-binary @movies.json


#Match all

Let's make the simplest possible query to our movies index. Query that returns all results, it's called match all query.

GET <name_of_index>/_search
{
    "query": {
        "match_all": {}
    }
}

You should get this type of result in response:

"hits": {
    "total": 306,
    "max_score": 1,


#string query

Still very simple query, we will only search for particular string.

GET /_search
{
    "query": {
        "query_string" : {
            "default_field" : "content",
            "query" : "this AND that OR thus"
        }
    }
}

Exercise.

Using this knowledge find movie Scarface in the elasticsearch. It should be returned as first result.


Let's build on this, we want to extend our search capabilities. Elasticsearch use operators like in programming, by default it uses 'OR' but we can use 'AND' to get exact match.

GET /_search
{
    "query": {
        "query_string" : {
            "default_field" : "content",
            "query" : "Strawberry pie with jello"
        }
    }
}

Now we will be sure that we will only get recipes we are interested.


Exercise.

Make a query to elasticsearch that will return only 1 result on query `Captain America first avenger`

"hits": {
     "total": 1,
     "max_score": 11.263437,
     "hits": [
        {
           "_index": "movies",
           "_type": "movie",
           "_id": "139",
           "_score": 11.263437,
           "_source": {
              "title": "Captain America: The First Avenger",
              "plot": "Predominantly set during World War II, Steve Rogers is a sickly man from Brooklyn who's transformed into super-soldier Captain America to aid in the war effort. Rogers must stop the Red Skull – Adolf Hitler's ruthless head of weaponry, and the leader of an organization that intends to use a mysterious device of untold powers for world domination.",
              "genres": null,


But what about case when users don't type correctly query. We should also handle this case. We could use match query with fuzz query here, it's a simpler cousin of string query.

"query": {
  "match": {
    "text": {
      "query": "jomped over me!",
      "fuzziness": "AUTO",
      "operator":  "and"
    }
  }
}

"fuzziness": "AUTO"
generates an edit distance based on the length of the term. For lengths:

0..2
must match exactly
3..5
one edit allowed
>5
two edits allowed

You could also use number values, like 0, 1, 2. Fuzziness is interpreted as  Levenshtein Edit Distance.

Exercise.

Write query that will return all Captain America movies based on query, which was mistyped: "Captaon America".



#Filtering
