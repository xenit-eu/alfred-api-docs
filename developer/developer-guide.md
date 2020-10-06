\include{metadata.md}

# Development

## Rules for pull requests
* Common sense trumps all rules.
* For every pull request please extend the [CHANGELOG.md](https://github.com/xenit-eu/alfred-api/blob/master/CHANGELOG.md).
* Do not make breaking changes since this is an API used by customers. Breaking changes include 
  adding, changing or removing endpoints or JSON objects used in requests and responses.
  * If you are forced to make a breaking change:
    * Notify maintainers
    * Add a note to the changelog with upgrade instructions
    * Notify all customers at the next release
* When working in REST code, please comply to **REST HTTP result codes** policy outlined in the
  [user guide](https://docs.xenit.eu/alfred-api/stable-user).
* Prefer unit tests over integration tests to keep builds fast
  
## Project structure
* *apix-interface* builds the interface of Alfred API. This part is agnostic of the 
Alfresco version used.
* *apix-rest-v1* builds the REST API of Alfred API. 
* *apix-impl* builds the AMP which is the main deliverable for Alfred API. The AMP contains the JARs of 
*apix-interface* and *apix-rest-v1*.
  * The top directory also contains code shared over different Alfresco versions.
  * *apix-impl/xx* contains all code per Alfresco version. It has a *src/java* folder
  for code specific to that Alfresco version and a *src/java-shared code* for the code shared between
  versions. This code is automatically symlinked from the *apix-impl* directory.   
* *apix-integrationtests* contains the integration tests for each Alfresco version.

## How to

### Run
If it is the first time you build Alfred API on your machine:
```bash
./setup.sh  # or ./setup.bat on Windows
```
Then:
```bash
./gradlew :apix-docker:docker-${VERSION}:composeUp --info
```
Where `VERSION` is e.g. `51`.
This starts up all docker containers required for an Alfresco running Alfred API.


### Run integration tests
```bash
./gradlew :apix-integrationtests:test-${VERSION}:integrationTest
```  
Again, where `VERSION` is e.g. `51`.

However, this starts (and afterwards stops) docker containers. This includes starting an Alfresco container,
 adding a startup time of several minutes. To circumvent this you also run the test on already running containers with
 for example:
 ```bash
./gradlew -x composeUp -x composeDown :apix-integrationtests:test-61:integrationTest -Pprotocol=http -Phost=localhost -Pport=8061
```


### Run integration tests under debugger
1. Debugging settings are already added by `apix-docker/${VERSION}/debug-extension.docker-compose.yml`, including a 
portmapping `8000:8000`. This file does not get loaded when running in Jenkins.
2. Prepare your remote debugger in IntelliJ and set breakpoints where you want in your tests
 (or Alfred API code).
3. Run the integration tests (see section above).
4. Wait until the container is started and healthy, then attach the debugger.

Again, where `VERSION` is e.g. `51`.

#### Deploy code changes for development
In a development scenario, it is possible to upload code changes to a running alfresco through dynamic extensions.
This requires the running alfresco to already have an older or equal version of alfred-api installed, and
the use of the jar artifact instead of the amp to do the new install. 
The JAR has the format `apix-impl-{ALFRESCO-VERSION}-{APIX-VERSION}.jar` and can be found under 
`apix-impl/{ALFRESCO-VERSION}/build/libs/`, where `ALFRESCO-VERSION` is one of *(50|51|52|60|61|62)*.
The new installation can be done either through the DE web interface, or with the following gradle task.
```bash
./gradlew :apix-impl:apix-impl-{ALFRESCO-VERSION}:installBundle -Phost={ALFRESCO-HOST} -Pport={ALFRESCO-PORT}
```

*Protip:* If you get tired of changing the port after every `docker-compose up`, you can temporarily put a
fixed port in the *docker-compose.yml* of the version you are working with. (The rationale behind using 
variable ephemeral ports is that during parallel builds on Jenkins port clashes must be avoided.)

For example for version 5.1, change in *apix-docker/51/docker-compose.yml* 
the ports line from:
```yaml
services:
  alfresco-core:
    ports:
      - ${DOCKER_IP}:8080
``` 
to: 
```yaml
services:
  alfresco-core:
    ports:
      - ${DOCKER_IP}:9051:8080
```
and then restart the containers with:

```bash
./gradlew :apix-docker:docker-51:composeUp --info
```

# Alfred api Concepts
Alfred API is composed of two logical layers.

* The base layer is a *Java API* built on top of the Alfresco.
* A *REST API* is built on top of the Java abstraction layer, exposing a stable HTTP API.


## Data objects
Alfred API has data objects that mirror the Alfresco concepts of QName, NodeRef, StoreRef, Path, 
ContentData and ContentInputStream. These data objects are used to communicate with the 
Alfred Java API without being dependent on Alfresco data types.

Conversion between Alfresco and Alfred API data objects is the responsibility of the 
`ApixToAlfrescoConversion` service. It is also possible to construct an Alfred API data object by
passing its string representation to the constructor.

## REST API
The Alfred REST API is a thin wrapper around the Java abstraction layer. It converts its received
parameters to the corresponding Alfred API data objects, then calls the corresponding service and
serializes its return value to JSON.

### Developer notes on search syntax
#### Unimplemented search options:
These options have some code towards handling, but are not implemented such that they are used in the search.
* `facets.mincount`
* `facets.limit`
* `orderBy.expression`

# Services
Only the most important services are described here. Full documentation is available in 
[the generated JavaDoc](#viewing-javadoc).

## NodeService
The `NodeService` provides operations on nodes.

* Fetch and modify the metadata for a Node
* Fetch the root node of a Store
* Fetch, create and remove child, parent and target associations for a Node
* Copy or move a Node to another parent
* Create and delete Nodes
* Checkout, checkin and fetch working copies for a Node

## SearchService
The `SearchService` allows searching for nodes based on an object tree.

The `SearchService.query()` method takes a `SearchQuery` object which contains the search query to execute,
as well as pagination, faceting and ordering options.

\define{EXAMPLE_IMPORTS}
\define{EXAMPLE_SEARCH_QUERY_OPTS}
```java
\include{examples/src/main/java/searchQuery.java}
```
\undef{EXAMPLE_IMPORTS}
\undef{EXAMPLE_SEARCH_QUERY_OPTS}

The query itself can be constructed using the `QueryBuilder`, which provides a fluent interface to 
build search queries.

\define{EXAMPLE_SEARCH_QUERY_QUERY}
```java
\include{examples/src/main/java/searchQuery.java}
```
\undef{EXAMPLE_SEARCH_QUERY_QUERY}

When using the REST API, a JSON payload describing the search query has to be POST'ed to the 
`apix/v1/search` endpoint. 
This JSON document reflects the node structure created by the query builder, and is shown below:

```json
\include{examples/src/main/resources/jsonsearchquery.json}
```

## DictionaryService
The `DictionaryService` provides meta-information about the metadata model.
It allows to fetch information about registered types, aspect and properties.

## Viewing JavaDoc
Full JavaDoc documentation of the Alfred API Java interface is available in the JavaDoc. You can
view the JavaDoc by browsing to `/alfresco/s/apix/javadocs/index.html` on your Alfresco host that
has Alfred API installed.