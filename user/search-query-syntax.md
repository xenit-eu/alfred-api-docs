## Search Requests

### query
Object containing subcomponents that build the requested query.
All queries are translated to Alfresco Full Text Search ([AFTS]("https://community.alfresco.com/docs/DOC-5729-full-text-search-query-syntax")) by alfred-api when executed

#### Syntax

The query parameter takes a tree structure of searchnodes as its argument.
A searchnode is either an operator or a search term.

##### Operators

Operators currently include only the standard AND, OR & NOT logical operations.
An operator is structured in the json payload as a named list.

* And

Translating `cm:name:"Budget 2020.xls" AND example:customProperty:"#AA3BF5"`
```json
{
  "and": [
    {
      "property" : {
        "name": "cm:name",
        "value": "Budget 2020.xls"
      }
    },
    {
      "property" : {
        "name" : "example:customProperty",
        "value" : "#AA3BF5"
      }
    }
  ]
}
```

* Or

Translating `example:customProperty:"#FFF233" OR example:customProperty:"#AA3BF5"`
```json
{
  "or": [
    {
      "property" : {
          "name" : "example:customProperty",
          "value" : "#FFF233"
        }
    },
    {
      "property" : {
        "name" : "example:customProperty",
        "value" : "#AA3BF5"
      }
    }
  ]
}
```

* Not

Translating `NOT cm:name:"Budget 2020.xls"`
```json
{
  "not": [
    {
      "property" : {
          "name": "cm:name",
          "value": "Budget 2020.xls"
        }
    }
  ]
}
```

* Nesting

Operators can be nested to form complex queries:
```json
{
  "and" : [
    {
      "not": [
         {
            "property" : {
                "name" : "example:customProperty",
                "value" : "#FFF233"
             }
          }
      ]
    },
    {
      "or": [
          {
            "property" : {
                "name" : "example:customProperty",
                "value" : "#FFF233"
              }
          },
          {
            "property" : {
              "name" : "example:customProperty",
              "value" : "#AA3BF5"
            }
          }
      ]
    }
  ]
}
```
##### Search terms

Search terms can be split into two groups: special terms and generic terms.
Special terms are used to search for specific concepts, generic terms are used to search for property values at large.

###### Special terms

* type

`Supports transactional metadata queries`

Lookup for any nodes of the given contentmodel type.
```json
{ "type": "cm:folder" }
```


* aspect

`Supports transactional metadata queries`

Lookup for any nodes which have the give aspect.
```json
{ "aspect":  "sys:folder"}
```


* noderef

`Supports transactional metadata queries`

Lookup for a specific alfresco noderef.
```json
{ "noderef": "workspace://SpacesStore/c4ebd508-b9e3-4c48-9e93-cdd774af8bbc" }
```


* path

`Supports transactional metadata queries`

Lookup for nodes at the given XPath.
```json
{ "path": "/app:company_home/cm:Fred_x0020_Performance_x0020_Test/*" }
```


* parent

`Supports transactional metadata queries`

Lookup for any nodes with the given node as parent. Takes a noderef as argument.
```json
{ "parent": "workspace://SpacesStore/c4ebd508-b9e3-4c48-9e93-cdd774af8bbc" }
```

* text

Lookup for any nodes that have given text in their content. Requires content of nodes to be indexed by solr to be found.
```json
{ "text": "xenit solutions" }
```

* category

Lookup for any nodes are of the given category.
```json
{ "category": "/cm:generalclassifiable/cm:Software_x0020_Document_x0020_Classification/member" }
```

* all

Lookup for any nodes with a hit for the searchterm in any field or in the content.
```json
{ "all":  "Xenit solutions"}
```

* isunset

Lookup for any nodes where the value of given propety is not set.
```json
{ "isunset":  "cm:author"}
```

* exists

Lookup for any nodes where the given property is present.
```json
{ "exists": "cm:author" }
```

* isnull

Lookup for any nodes where the value of the given property is set to null.
```json
{ "isnull":  "cm:author"}
```

* isnotnull

Lookup for any nodes where the value of the given property is set and not null.
```json
{ "isnotnull": "cm:author" }
```

###### Note Unsupported terms

The following terms are available in alfresco, but are currently not supported by Alfred-api:
* isroot
* tx
* primaryparent
* class
* exactclass
* qname
* exactaspect
* tag

###### Generic terms

Generic terms are searchterms for any given property with any given value or range.
```json
{ "property": 
    {
      "name": "example:customProperty",
      "value": "som*value"
    }
}
```
```json
{ "property": 
  {
    "name":"cm:modified",
    "range":{ 
      "start":"2015-10-05T10:41:42+00:00",
      "end":"2020-02-25T09:36:01+00:00"
    }
  }
}
```
Terms using values can also take the `exact` boolean parameter (by default it is considered set to false). 
This signifies that the given value needs to be matched exactly, as opposed to the default fuzzy search. 
This also implies that wildcards are not compatible with the exact parameter. 
**For Transactional Metadata Queries, the `exact` parameter is mandatory.**
```json
{ "property": 
    {
      "name": "example:customProperty",
      "value": "som*value",
      "exact": true
    }
}
```

### paging

`Optional`

Options to page through the search results by setting a skipcount and a page size limit.
```json
{
  "paging": {
     "limit": 10,
     "skip": 0
   }
}
```

### facets

`Optional`

Options to enable facets. This will return facet results as configured on the server.
The alfresco limit and minCount parameters are not implemented.

**Note:** facets with 0 hits are not returned in the result
```json
{
  "facets": {
    "enabled": true
  }
}
```


### orderBy

`Optional`

Options to define a list of properties to order by and the direction of ordering per property.
Expression based sorting is not implemented.
```json
{ 
  "orderBy": [
    {
      "property": "cm:name",
      "order": "descending"
    },
    {
      "property": "cm:modified",
      "order": "ascending"
    }
  ]
}
```

### consistency

`Optional`

Option to request specific consistency. Options are:
* `EVENTUAL`
* `TRANSACTIONAL`
* `TRANSACTIONAL_IF_POSSIBLE` (default)
```json
{
  "consistency": "TRANSACTIONAL"
}
```
#### Note on search, consistency and fuzzyness

Alfresco internally supports 2 types of searchqueries: database-backed and solr-backed. Based on the server configuration
and the search query, it will determine which of these 2 to use. Alfresco will attempt to use the database, but for any query 
on indexed properties, content or for fuzzy matching, solr is required.

In the documentation database-backed queries are known as `Transactional Metadata Queries` or `TDMQ`'s.
To enable TDMQ's for alfred-api the `exact` parameter is required for generic search terms to be searched in the database,
and from the special search terms, only a subset is available for use. See the searchterm section 

The other search, solr-backed, allows the usage of wild cards and other forms of fuzziness, but requires the solr component
to build indexes against the alfresco repository. This means that some time and resources are spent to bring the search index
up to date with the repository. As a consequence, after any given change, a window of time exists where a searchrequest against
solr will return a result inconsistent with the repo. This effect will pass with time, hence the name eventual consistency.

### locale

`Optional`

Options to request specific locale and encoding options.
```json
{
  "locale": "BE"
}
```

### workspace

`Optional`

Options to change the target alfresco workspace. This would allow searches on the archive or on custom workspaces.
A workspace is in this context defined by the `workspace name` + the protocol delimiter `://` + `the target store name`.

**Note**: if the target store is not indexed by solr, eventually consistent queries will result in errors.
```json
{
  "workspace": "archive://SpacesStore"
}
```

### highlight

`Requires alfresco version 5.2 or higher`

`Optional`

Options to change the highlight configuration.
Minimal requirement is the `fields` array, which takes object containing at least the `field` property. 
Each list element specifies a property on which higlighting needs to be applied. 
When no fields are specified, the call defaults to `cm:content` as field.
Full documentation can be found on the alfresco documentation page: 
* [5.2](https://docs.alfresco.com/5.2/concepts/search-api-highlight.html)
* [6.0](https://docs.alfresco.com/6.0/concepts/search-api-highlight.html)
* [6.1](https://docs.alfresco.com/6.1/concepts/search-api-highlight.html)
* [6.2](https://docs.alfresco.com/6.2/concepts/search-api-highlight.html)

```json
{
  "highlights": 
    {
      "prefix": "<highlight>",
      "postfix": "</highlight>",
      "snippetCount": 2,
      "fields": [
        {
          "field": "cm:content"
        }
      ]
    }
}
```

### Examples

Search for the first 10 nodes in the `cm:content` namespace with facets:
```json
{
  "query": {"type":"{http://www.alfresco.org/model/content/1.0}content"},
  "paging": {
    "limit": 10,
    "skip": 0
  },
  "facets": {
    "enabled": false
  }
}
```

Search for all nodes with the term 'budget' in the `cm:content` property (fulltext), and show two highlighted hits with delimiter \\<highlight>\\</highlight>:
```json
{
    "query": {
        "property": {
            "exact": false,
            "name": "cm:content",
            "value": "budget"
        }
    },
    "highlight":{
        "prefix":"<highlight>",
        "postfix":"</highlight>",
        "snippetCount":2,
        "fields":[{"field":"cm:content"}]
    }
}
```

### Json response

The search rest call returns a JSON object of the following form:
```json
{
  "noderefs": [ // List of noderefs
    "workspace://SpacesStore/c4ebd508-b9e3-4c48-9e93-cdd774af8bbc",
    "workspace://SpacesStore/ff5874ad-b9e3-4c48-9e93-cddaaa9746ec"
  ],
  "facets": [ // list of facets
    {
      "name": "TYPE", // facet field name
      "values": [ // Facet values
        {
          "value": "Factuur",
          "label": "Factuur",
          "count": 16
        }
      ]
    },
    {
      "name": "cm:modified", // facet field name
      "values": [ // Facet values
        {
          "range":
          {
            "start":"2020-05-01T00:00:00+00:00",
            "end":"2020-05-31T00:00:00+00:00"
          },
          "label": "One month ago",
          "count": 16 // value and count
        }
      ]
    }
  ],
  "totalResultCount": 2, //Sum of all results
  "highlights": {
      "noderefs": { //A set of objects with an entry for each noderef for which a highligth can be returned
          "workspace://SpacesStore/e818fd1c-262e-43f1-8751-461cd8de9293": [ //list of highlights per specified field for the given node
              {
                  "field": "cm:name",
                  "snippets": [
                      "Stretch your research <highlight>budget</highlight>-20150813-155144.msg"
                  ]
              }
          ]
      }
  }
}

```

### Additional notes

#### Date format

Dates and times must be specified int the ISO-8601 format
