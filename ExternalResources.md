In certain setups, it is necessary to even store resources with URIs that do not resolve to the installation of the Linked Media server. For example,
  * you might want to work with existing ontologies that you import into the LMF system and then extend
  * you might want to import data from a source like DBPedia or GeoNames and hold it locally for efficiency or trust reasons
  * you might want to use the Linked Media server to create metadata for arbitrary web sites, e.g. to create a service like delicious.com
  * you might want to use the Linked Media Server for integrating data from different sources and you want to preserve the existing URIs
  * ...

To address this issue, the Linked Media Framework offers the same API that is described in InteractingWithResources to work with arbitrary URIs. The only difference is that instead of directly calling the resource URI, it is passed as a ?uri= query parameter to the webservice call.

For example, the following call will create an external resource with the URI http://www.example.com/1234:
```
curl -i -X POST http://localhost:8080/LMF/resource?uri=http://www.example.com/1234
```

Result:
```
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/LMF/resource?uri=http%3A%2F%2Fwww.example.com%2F1234
Content-Length: 0
Date: Thu, 19 May 2011 15:02:13 GMT
```

In the same manner, you can also upload content and metadata and delete the resource:

```
curl -X PUT -i -H "Content-Type: text/rdf+n3; rel=meta"  \
     -d '<http://www.example.com/1234> <http://www.w3.org/2000/01/rdf-schema#label> "Example Resource".' \
     http://localhost:8080/LMF/resource?uri=http://www.example.com/1234
```

will send a redirect to http://localhost:8080/LMF/meta/text/rdf+n3?uri=http://www.example.com/1234, and if followed will upload the metadata to the LMF:

```
curl -X PUT -i -H "Content-Type: text/rdf+n3; rel=meta"  \
     -d '<http://www.example.com/1234> <http://www.w3.org/2000/01/rdf-schema#label> "Example Resource".' \
     http://localhost:8080/LMF/meta/text/rdf+n3?uri=http://www.example.com/1234
```

Updating content, retrieving metadata and content, and deleting external resources works analogously (as described in InteractingWithResources).