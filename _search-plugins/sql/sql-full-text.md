---
layout: default
title: Full-Text Search
parent: SQL
nav_order: 8
---

# Full-text search

Use SQL commands for full-text search. The SQL plugin supports a subset of the full-text queries available in OpenSearch.

To learn about full-text queries in OpenSearch, see [Full-text queries]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/).

## Match

Use the `match` command to search documents that match a `string`, `number`, `date`, or `boolean` value for a given field.

### Syntax

```sql
match(field_expression, query_expression[, option=<option_value>]*)
```

You can specify the following options:

- `analyzer`
- `auto_generate_synonyms_phrase`
- `fuzziness`
- `max_expansions`
- `prefix_length`
- `fuzzy_transpositions`
- `fuzzy_rewrite`
- `lenient`
- `operator`
- `minimum_should_match`
- `zero_terms_query`
- `boost`

*Example 1*: Search the `message` field:

```json
GET my_index/_search
{
  "query": {
    "match": {
      "message": "this is a test"
    }
  }
}
```

SQL query:

```sql
SELECT message FROM my_index WHERE match(message, "this is a test")
```

*Example 2*: Search the `message` field with the `operator` parameter:

```json
GET my_index/_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test",
        "operator": "and"
      }
    }
  }
}
```

SQL query:

```sql
SELECT message FROM my_index WHERE match(message, "this is a test", operator=and)
```

*Example 3*: Search the `message` field with the `operator` and `zero_terms_query` parameters:

```json
GET my_index/_search
{
  "query": {
    "match": {
      "message": {
        "query": "to be or not to be",
        "operator": "and",
        "zero_terms_query": "all"
      }
    }
  }
}
```

SQL query:

```sql
SELECT message FROM my_index WHERE match(message, "this is a test", operator=and, zero_terms_query=all)
```

To search for text in a single field, use `MATCHQUERY` or `MATCH_QUERY` functions.

Pass in your search query and the field name that you want to search against.

```sql
SELECT account_number, address
FROM accounts
WHERE MATCH_QUERY(address, 'Holmes')
```

Alternate syntax:

```sql
SELECT account_number, address
FROM accounts
WHERE address = MATCH_QUERY('Holmes')
```


| account_number | address
:--- | :---
1 | 880 Holmes Lane


## Multi match

To search for text in multiple fields, use `MULTI_MATCH`, `MULTIMATCH`, or `MULTIMATCHQUERY` functions.

For example, search for `Dale` in either the `firstname` or `lastname` fields:


```sql
SELECT firstname, lastname
FROM accounts
WHERE MULTI_MATCH('query'='Dale', 'fields'='*name')
```


| firstname | lastname
:--- | :---
Dale | Adams


## Query string

To split text based on operators, use the `QUERY` function.


```sql
SELECT account_number, address
FROM accounts
WHERE QUERY('address:Lane OR address:Street')
```


| account_number | address
:--- | :---
1 | 880 Holmes Lane
6 | 671 Bristol Street
13 | 789 Madison Street


The `QUERY` function supports logical connectives, wildcard, regex, and proximity search.


## Match phrase

To search for exact phrases, use `MATCHPHRASE`, `MATCH_PHRASE`, or `MATCHPHRASEQUERY` functions.


```sql
SELECT account_number, address
FROM accounts
WHERE MATCH_PHRASE(address, '880 Holmes Lane')
```


| account_number | address
:--- | :---
1 | 880 Holmes Lane


## Score query

To return a relevance score along with every matching document, use `SCORE`, `SCOREQUERY`, or `SCORE_QUERY` functions.

You need to pass in two arguments. The first is the `MATCH_QUERY` expression. The second is an optional floating point number to boost the score (default value is 1.0).


```sql
SELECT account_number, address, _score
FROM accounts
WHERE SCORE(MATCH_QUERY(address, 'Lane'), 0.5) OR
  SCORE(MATCH_QUERY(address, 'Street'), 100)
ORDER BY _score
```


| account_number | address | score
:--- | :--- | :---
1 | 880 Holmes Lane | 0.5
6 | 671 Bristol Street | 100
13 | 789 Madison Street | 100
