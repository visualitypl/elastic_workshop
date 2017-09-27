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

Using range query.

This how the queries for finding ppl with particular range of age would look like.
GET _search
{
    "query": {
        "range" : {
            "age" : {
                "gte" : 10,
                "lte" : 20,
                "boost" : 2.0
            }
        }
    }
}

gte = Greater-than or equal to

gt = Greater-than

lte = Less-than or equal to

lt = Less-than

Exercise.

Create query that would return movies with running time between 60 and 90 minutes.

It should return 57 results.

*Bool query*

The bool query takes a more-matches-is-better approach, so the score from each matching must or should clause will be added together to provide the final _score for each document.

must - The clause (query) must appear in matching documents and will contribute to the score.

filter - Filter clauses are executed in filter context, meaning that scoring is ignored and clauses are considered for caching.

should - The clause (query) should appear in the matching document.

Example query:

POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}

Exercise.

Create query that will find superhero movies (keywords field:  superhero) that are no longer than 120 minutes and not shorter than 60 minutes (field runtime) and must not have Robert Downey Jr. as starring actor (actors field).

You should get 12 results for this query


#Aggregations

Let's get some interesting stats for analytics, we want to get overall view how some value occurs through the documents. The stats aggregation would give us general insight.

{
    "aggs" : {
        "grades_stats" : { "stats" : { "field" : "grade" } }
    }
}

{
    ...

    "aggregations": {
        "grades_stats": {
            "count": 6,
            "min": 60,
            "max": 98,
            "avg": 78.5,
            "sum": 471
        }
    }
}

Exercise.

Get overall data for rating in movies: min, max, average. Do that using stats query.


Range Aggregation

A multi-bucket value source based aggregation that enables the user to define a set of ranges - each representing a bucket.


GET products/_search?size=0

{
  "aggs": {
    "weight_ranges": {
      "range": {
        "field": "weight",
        "ranges": [
          {
            "to": 500
          },
          {
            "from": 500,
            "to": 1000
          },
          {
            "from": 1000,
            "to": 1500
          }
        ]
      }
    }
  }
}

and this will return aggregated data:

    ...

    "aggregations": {
        "weight_ranges" : {
            "buckets": [
                {
                    "to": 500,
                    "doc_count": 20
                },
                {
                    "from": 500,
                    "to": 1000,
                    "doc_count": 4
                },
                {
                    "from": 1000,
                    "doc_count": 4
                }
            ]
        }
    }
}

Exercise.

Using range queries, count how many movies were in mentioned run times: below 60 minutes, between 60 and 75 minutes, between 90 and 120 minutes.

#Histogram aggregation

We can also use histogram to bucket data instead of ranges. It's useful for prices in shops, so we can see how prices fall between different ranges 0$-10$, 10$-20$

POST /sales/_search?size=0
{
    "aggs" : {
        "prices" : {
            "histogram" : {
                "field" : "price",
                "interval" : 10
            }
        }
    }
}

Would return:
{
    ...
    "aggregations": {
        "prices" : {
            "buckets": [
                {
                    "key": 0.0,
                    "doc_count": 1
                },
                {
                    "key": 50.0,
                    "doc_count": 1
                },
                {
                    "key": 100.0,
                    "doc_count": 0
                },
                {
                    "key": 150.0,
                    "doc_count": 2
                },
                {
                    "key": 200.0,
                    "doc_count": 3
                }
            ]
        }
    }
}

Exercise.

Create histogram aggregation for rating in movies with interval equal 1.


#Sorting

Allows to add one or more sort on specific fields. Each sort can be reversed as well. The sort is defined on a per field level, with special field name for _score to sort by score, and _doc to sort by index order.

GET /my_index/my_type/_search
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}

Exercise.

Sort Captain America movies by release date in ascending order, oldest movie first. You should display only Captain America movies here. Keep results relevant


#Highlighting

Allows to highlight search results on one or more fields. It's useful for seeing in results page, where did your query appear in searched field.

GET /_search
{
    "query" : {
        "match": { "content": "kimchy" }
    },
    "highlight" : {
      "pre_tags" : ["<tag1>"],
      "post_tags" : ["</tag1>"],
        "fields" : {
            "content" : {}
        }
    }
}

Exercise.

Create highlight for your query to search plot in movies 'terrorist attack'. It should return with highlighted fields with tags <highlight> </highlight> like this:

"highlight": {
             "plot": [
                "Jack Ryan, as a young covert CIA analyst, uncovers a Russian plot to crash the U.S. economy with a <highlight>terrorist</highlight> <highlight>attack</highlight>."
             ]
          }


#Pagination

You can create pagination by passing parameters size and from to query. Size will dictate number of elements on page and from will work as offset.

For pages 1 to 3.

GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10

also could be passed to body

{
  "query": {
    "match_all": {}
  },
  "size": 5
}


Exercise.

Create pagination for movies with genre action. 
