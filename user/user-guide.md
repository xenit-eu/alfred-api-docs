\include{metadata.md}

\include{about.md}

Instead of mirroring the Alfresco API one-to-one, Alfred API groups frequently used operations
together in a single method call. This allows you to focus on business concerns instead of focusing
on fetching all required data from Alfresco.

For example, the `NodeService.getMetadata()` method returns an object with all metadata about 
a node: its type, aspects and properties in a single function call. It groups together the 
information that would otherwise have to be obtained by combining requests for type, aspects 
and properties separately.

\include{naming.md}
# Java Api

The alfred-api java api the core to exposing alfresco functionality and normalizing operations across version.
Any extensions written while depending on alfred api can be easily ported to a new alfresco version.

When the api is installed, all of its service are available as beans and can be wired into your own classes.

# Rest Api

For a full overview of the Rest api, please refer to [the swagger specification](https://demo.xenit.eu/alfresco/s/apix/v1/docs/ui.html).

\include{search-query-syntax.md}

## Usage Example

In this example a new node will be created and its metadata set:

**1.** Find the default alfresco folder named "Shared" to use as parent folder for the new node.
```bash
curl -X POST \
--header "Authorization: Basic a8d46fw84649" \
--header "Accepts: application/json" \
--header "Content-Type: application/json" \
--data '{ "query": { "property": { "name":"cm:name", "value":"Shared", "exact": true } } }' \
https://www.alfresco.example/alfresco/s/apix/v1/search 
```
Response:
```json
{
  "noderefs": [ "workspace://SpacesStore/df18bcde-531b-4b39-9698-e460cbff2bb5" ],
  "totalResultCount": 1
}
```
**2.** Create the new node
```bash
curl -X POST \
--header "Authorization: Basic a8d46fw84649" \
--header "Content-Type: application/json" \
--data '{ "parent": "workspace://SpacesStore/df18bcde-531b-4b39-9698-e460cbff2bb5", "name": "Red test node", "type": "{http://www.alfresco.org/model/content/1.0}content" }' \
https://www.alfresco.example/alfresco/s/apix/v1/nodes
```
Response:
```json
{
  "noderef":"workspace://SpacesStore/d26176d6-11d9-4381-a327-cccb7600efc4",
  "metadata": {
      "id":"workspace://SpacesStore/d26176d6-11d9-4381-a327-cccb7600efc4",
      "type":"{http://www.alfresco.org/model/content/1.0}content",
      "baseType":"{http://www.alfresco.org/model/content/1.0}content",
      "transactionId":66261,
      "properties": {
          "{http://www.alfresco.org/model/system/1.0}store-protocol":["workspace"],
          "{http://www.alfresco.org/model/system/1.0}node-dbid":["128993"],
          "{http://www.alfresco.org/model/content/1.0}name":["Red test node"],
          "{http://www.alfresco.org/model/content/1.0}modified":["2020-10-06T12:31:12.356Z"],
          "{http://www.alfresco.org/model/content/1.0}creator":["admin"],
          "{http://www.alfresco.org/model/system/1.0}locale":["en_US"],
          "{http://www.alfresco.org/model/content/1.0}created":["2020-10-06T12:31:12.356Z"],
          "{http://www.alfresco.org/model/system/1.0}store-identifier":["SpacesStore"],
          "{http://www.alfresco.org/model/content/1.0}modifier":["admin"],
          "{http://www.alfresco.org/model/system/1.0}node-uuid":["d26176d6-11d9-4381-a327-cccb7600efc4"],
      },
      "aspects":[
        "{http://www.alfresco.org/model/content/1.0}auditable",
        "{http://www.alfresco.org/model/system/1.0}referenceable",
        "{http://www.alfresco.org/model/system/1.0}localized"
        ]
    },
  "permissions": {
      "Read":"ALLOW",
      "Write":"ALLOW",
      "Delete":"ALLOW",
      "AddChildren":"ALLOW",
      "ReadPermissions":"ALLOW",
      "ReadRecords":"DENY",
      "Filing":"DENY",
      "CreateChildren":"ALLOW",
      "ChangePermissions":"ALLOW",
  },
  "associations":{
      "children":[],
      "parents":[
          {
            "source":"workspace://SpacesStore/d26176d6-11d9-4381-a327-cccb7600efc4",
            "target":"workspace://SpacesStore/df18bcde-531b-4b39-9698-e460cbff2bb5",
            "type":"{http://www.alfresco.org/model/content/1.0}contains",
            "primary":true
          }
      ],
      "targets":[]
  },
  "path":{
      "displayPath":"/Company Home/Shared",
      "qnamePath":"/app:company_home/app:shared/cm:Red_x0020_test_x0020_node"
  }
}
```

\include{rest-response-codes.md}

