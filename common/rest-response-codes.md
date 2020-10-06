## REST HTTP result codes
REST responses can return the following HTTP status codes:


### 2xx Success

Indicates request sent by client was understood and accepted.

**Code**                    | **Meaning**
----------------------      |-----------------------
200 OK                      | Generic success.
202 Accepted                | The request was successful and will be processed asynchronously.
207 Multi-Status            | A bulk request completed successfully. These responses should contain multi-status response that can be correlated to each individual request in the bulk request. Can be returned even if individual requests fail.


### 3xx Redirection

Indicates the client must take additional steps to complete the request.

**Code**                    | **Meaning**
----------------------      | -----------------------
301 Moved Permanently       | This and all future requests should be directed to the given URI.


### 4xx Client error

Indicates anticipated failures, such as requests for non-existant resources, 
requests with missing input and malformed requests.

A body *may* be provided in the response that clarifies the error.

**Code**                    | **Meaning**
----------------------      | -----------------------
400 Bad Request             | Generic client error.
401 Unauthorized            | User must log in.
403 Forbidden               | User not authorized to use this resource.
404 Not Found               | Requested resource not found. Returned also for e.g. requesting a node with an incorrect id, as well as unhandled URI's. For security reasons, a 404 can aso be returned when the requester has insufficient permissions.
405 Method Not Allowed      | A request method is not supported (e.g. PUT on an endpoint that only accepts GET).


### 5xx Server error

Indicates unexpected failures.

**Code**                    | **Meaning**
----------------------      | -----------------------
500 Internal Server Error   | Generic server error.
503 Service Unavailable     | Temporary server error. Retry later is sensible.