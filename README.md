# About

This is the Factual supported Python driver for [Factual's public API](http://developer.factual.com/display/docs/Factual+Developer+APIs+Version+3).


This API supports queries to Factual's Read, Schema, Crosswalk, and Resolve APIs. Full documentation is available on the Factual website:

*   [Read](http://developer.factual.com/display/docs/Factual+Developer+APIs+Version+3): Search the data
*   [Schema](http://developer.factual.com/display/docs/Core+API+-+Schema): Get table metadata
*   [Crosswalk](http://developer.factual.com/display/docs/Places+API+-+Crosswalk): Get third-party IDs
*   [Resolve](http://developer.factual.com/display/docs/Places+API+-+Resolve): Enrich your data and match it against Factual's
*   [Facets](http://developer.factual.com/display/docs/Core+API+-+Facets): Get counts of data by facet

Full documentation is available at http://developer.factual.com

If you need additional support, please visit http://support.factual.com

### Warning
The driver also contains support for Submit and Flag requests which are not yet available through the Factual public API.  These API features will be enabled in the near future.

# Overview


## Setup
The easiest way to get started with the driver is to install it from the Python Package Index.

```shell
pip install factual-api
```

Obtain an OAuth key and secret from [Factual](http://www.factual.com/devtools/beta).

To use the driver in a python program, just create a Factual object using your OAuth key and secret.

```python
from factual import Factual
factual = Factual(KEY, SECRET)
```

example.py is provided with the driver as a reference.

## Dependencies
[Requests](http://docs.python-requests.org/en/v0.10.7/index.html)

[requests-oauth](https://github.com/maraujop/requests-oauth)


## Basic Design

The driver allows you to create an authenticated handle to Factual. With a Factual handle, you can send queries and get results back.

Queries are created using the Factual handle, which provides a fluent interface to constructing your queries.  One thing to be aware of is the behavior of the query modifier functions.  These return new query instances, so base queries can be set up and then modified in different ways to produce new queries.

```python
# Create a base search query
q = factual.table("places").search("sushi")

# Use this query with a filter
filter_query = q.filters({"website":{"$blank":False}})

# Use the same base query with select parameters (will not have website filter applied)
select_name = q.select("name")
```

## Tables
The Factual API is a generic API that sits over all tables available via the Factual v3 API. Some popular ones:

*   Table <tt>global</tt> for international places
*   Table <tt>restaurants-us</tt> for US restaurants only
*   Table <tt>places</tt> for US places only
*   Table <tt>products-cpg</tt> for consumer products

## Unit Tests
Unit Tests are provided to ensure the driver and OAuth are functioning as expected.  
Add your Oauth credentials to tests/test_settings.py
From the command line, run: python -m tests.api_test


## Simple Read Examples

```python
# Return entities from the Places dataset where name equals "starbucks"
factual.table("places").filters({"name":"starbucks"}).data()
```

```python
# Full text search for "sushi santa monica"
factual.table("places").search("sushi santa monica").data()
```

```python
# Return entity names and non-blank websites from the Global dataset, for entities located in Thailand
factual.table("global").select("name,website").filters(
	{"country":"TH","website":{"$blank":False}}).data()
```

```python
# Return highly rated U.S. restaurants in Los Angeles with WiFi
factual.table("restaurants-us").filters(
  {"$and":[{"locality":"los angeles"},{"rating":{"$gte":4}},{"wifi":"true"}]}).data()
```


## Simple Crosswalk Example

```python
# Get Crosswalk data using a Factual ID
FACTUAL_ID = "110ace9f-80a7-47d3-9170-e9317624ebd9"
query = factual.crosswalk().factual_id(FACTUAL_ID)
query.data()
```

```python
# Get Crosswalk data using a third party namespace and namespace_id
SIMPLEGEO_ID = "SG_6XIEi3qehN44LH8m8i86v0"
query = factual.crosswalk().namespace('simplegeo',SIMPLEGEO_ID)
query.data()
```

## Simple Resolve Example

```python
# Returns resolved entities
query = factual.resolve({"name":"McDonalds","address":"10451 Santa Monica Blvd","region":"CA","postcode":"90025"})
query.data()
query.data()[1]["resolved"]  # true or false
```

## Simple Facets Example

```python
# Count the number of Starbucks per country
query = factual.facets("global").search("starbucks").select("country")
query.data()
```


## More Read Examples

```python
# 1. Specify the table Global
query = factual.table("global")
```

```python
# 2. Filter results in country US
query = query.filters("country":"US")
```

```python
# 3. Search for "sushi" or "sashimi"
query = query.search("sushi", "sashimi")
```

```python
# 4. Filter by geolocation
query = query.geo({"$circle":{"$center":[34.06021, -118.41828], "$meters":5000}})
```

```python
# 5. Sorting
query = query.sort("name:asc")       # ascending 
query = query.sort("name:desc")      # descending
```

```python
# 6. Paging
query = query.offset("20")
```


# Read API

## All Top Level Query Parameters

<table>
  <col width="33%"/>
  <col width="33%"/>
  <col width="33%"/>
  <tr>
    <th>Parameter</th>
    <th>Description</th>
    <th>Example</th>
  </tr>
  <tr>
    <td>filters</td>
    <td>Restrict the data returned to conform to specific conditions.</td>
    <td><tt>query = query.filters("name":{"$bw":"starbucks"})</tt></td>
  </tr>
  <tr>
    <td>include_count</td>
    <td>returns the total count of the number of rows in the dataset that conform to the query.</td>
    <td><tt>query = query.include_count(True)</tt><br><tt>count = query.data().total_row_count()</tt></td>
  </tr>
  <tr>
    <td>geo</td>
    <td>Restrict data to be returned to be within a geographical range based.</td>
    <td><tt>query.geo({"$circle":{"$center":[34.06021, -118.41828], "$meters":5000}})</tt></td>
  </tr>
  <tr>
    <td>limit</td>
    <td>Limit the results</td>
    <td><tt>query = query.limit(12)</tt></td>
  </tr>
  <tr>
    <td>page</td>
    <td>Limit the results to a specific "page".</td>
    <td><tt>query = query.page(2, :per:10)</tt></td>
  </tr>
  <tr>
    <td>search (across entity)</td>
    <td>Full text search across entity</td>
    <td>
      Find "sushi":<br><tt>query = query.search("sushi")</tt><p>
      Find "sushi" or "sashimi":<br><tt>query = query.search("sushi", "sashimi")</tt><p>
      Find "sushi" and "santa" and "monica":<br><tt>query.search("sushi santa monica")</tt>
    </td>
  </tr>
  <tr>
    <td>search (across field)</td>
    <td>Full text search on specific field</td>
    <td><tt>query = query.filters({"name":{"$search":"cafe"}})</tt></td>
  </tr>
  <tr>
    <td>select</td>
    <td>Specifiy which fields to include in the query results.  Note that the order of fields will not necessarily be preserved in the resulting response due to the nature Hashes.</td>
    <td><tt>query = query.select("name,address,locality,region")</tt></td>
  </tr>
  <tr>
    <td>sort</td>
    <td>The field (or fields) to sort data on, as well as the direction of sort.<p>
        Sorts ascending by default, but supports both explicitly sorting ascending and descending, by using <tt>sort_asc</tt> or <tt>sort_desc</tt>.
        Supports $distance as a sort option if a geo-filter is specified.<p>
        Supports $relevance as a sort option if a full text search is specified either using the q parameter or using the $search operator in the filter parameter.<p>
        By default, any query with a full text search will be sorted by relevance.<p>
        Any query with a geo filter will be sorted by distance from the reference point.  If both a geo filter and full text search are present, the default will be relevance followed by distance.</td>
    <td><tt>query = query.sort("name:asc")</tt><br>
    <tt>query = query.sort("$distance:asc")</tt>
    <tt>query = query.sort("$distance:asc,name:desc")</tt></td>
  </tr>
</table>

## Row Filters

The driver supports various row filter logic. For example:

```python
# Returns records from the Places dataset with names beginning with "starbucks"
factual.table("places").filters("name":{"$bw":"starbucks"}).data()
```

### Supported row filter logic

<table>
  <tr>
    <th>Predicate</th>
    <th width="25%">Description</th>
    <th>Example</th>
  </tr>
  <tr>
    <td>$eq</td>
    <td>equal to</td>
    <td><tt>query = query.filters("region":{"$eq":"CA"})</tt></td>
  </tr>
  <tr>
    <td>$neq</td>
    <td>not equal to</td>
    <td><tt>query = query.filters("region":{"$neq":"CA"})</tt></td>
  </tr>
  <tr>
    <td>search</td>
    <td>full text search</td>
    <td><tt>query = query.search("sushi")</tt></td>
  </tr>
  <tr>
    <td>$in</td>
    <td>equals any of</td>
    <td><tt>query = query.filters("region":{"$in":["CA", "NM", "NY"]})</tt></td>
  </tr>
  <tr>
    <td>$nin</td>
    <td>does not equal any of</td>
    <td><tt>query = query.filters("region":{"$nin":["CA", "NM", "NY"]})</tt></td>
  </tr>
  <tr>
    <td>$bw</td>
    <td>begins with</td>
    <td><tt>query = query.filters("name":{"$bw":"starbucks"})</tt></td>
  </tr>
  <tr>
    <td>$nbw</td>
    <td>does not begin with</td>
    <td><tt>query = query.filters("name":{"$nbw":"starbucks"})</tt></td>
  </tr>
  <tr>
    <td>$bwin</td>
    <td>begins with any of</td>
    <td><tt>query = query.filters("name":{"$bwin":["starbucks", "coffee", "tea"]})</tt></td>
  </tr>
  <tr>
    <td>$nbwin</td>
    <td>does not begin with any of</td>
    <td><tt>query = query.filters("name":{"$nbwin":["starbucks", "coffee", "tea"]})</tt></td>
  </tr>
  <tr>
    <td>$blank</td>
    <td>test to see if a value is (or is not) blank or null</td>
    <td><tt>query = query.filters("tel":{"$blank":true})</tt><br>
        <tt>query = query.filters("website":{"$blank":false})</tt></td>
  </tr>
  <tr>
    <td>$gt</td>
    <td>greater than</td>
    <td><tt>query = query.filters("rating":{"$gt":7.5})</tt></td>
  </tr>
  <tr>
    <td>$gte</td>
    <td>greater than or equal</td>
    <td><tt>query = query.filters("rating":{"$gte":7.5})</tt></td>
  </tr>
  <tr>
    <td>$lt</td>
    <td>less than</td>
    <td><tt>query = query.filters("rating":{"$lt":7.5})</tt></td>
  </tr>
  <tr>
    <td>$lte</td>
    <td>less than or equal</td>
    <td><tt>query = query.filters("rating":{"$lte":7.5})</tt></td>
  </tr>
</table>

### AND

Filters can be logically AND'd together. For example:

```python
# name begins with "coffee" AND tel is not blank
query = query.filters({ "$and":[{"name":{"$bw":"coffee"}}, {"tel":{"$blank":false}}] })
```

### OR

Filters can be logically OR'd. For example:

```python
# name begins with "coffee" OR tel is not blank
query = query.filters({ "$or":[{"name":{"$bw":"coffee"}}, {"tel":{"$blank":false}}] })
```

### Combined ANDs and ORs

You can nest AND and OR logic to whatever level of complexity you need. For example:

```python
# (name begins with "Starbucks") OR (name begins with "Coffee")
# OR
# (name full text search matches on "tea" AND tel is not blank)
query = query.filters({ "$or":[ {"$or":[ {"name":{"$bw":"starbucks"}},
                                               {"name":{"$bw":"coffee"}}]},
                                   {"$and":[ {"name":{"$search":"tea"}},
                                                {"tel":{"$blank":false}} ]} ]})
```

## Full Documentation
Full documentation is available at http://developer.factual.com
