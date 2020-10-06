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

\include{rest-response-codes.md}

