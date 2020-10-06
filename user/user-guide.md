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

# Rest Api

For a full overview of the Rest api, please refer to the swagger specification

\include{search-query-syntax.md}

\include{rest-response-codes.md}

