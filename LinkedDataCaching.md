# Introduction #

The Linked Media Framework can optionally be extended with a Linked Data Caching Module that transparently retrieves resources from the Linked Data Cloud when they are needed (e.g. when querying a relation to a remote resource) and caches them transparently. Linked Data Caching is integrated at the triple store level and thus available to all services accessing the triple store, including the SPARQL endpoint and the SemanticSearch component.

# Functionality #

## Transparent Linked Data Access ##

The Linked Data Caching module is a powerful component for integrating with other Linked Data servers, either on the public Linked Data Cloud or from local severs deployed in an intranet. It provides transparent access to resources on Linked Data-aware servers. When enabled, it is triggered when the triples of a non-local resource are requested from the triple store. A typical case where this can happen is when a local resource links to a resource in the Linked Data Cloud using a triple and a query to the system requests information about this external resource.

For example, the FOAF description of a user in the LMF might contain a reference to the FOAF file of a user at some external location:

```
local:peter foaf:knows <http://example.com/john.rdf>
```

A query could then ask for all the names of persons that peter knows, e.g. in SPARQL:

```
SELECT ?name WHERE { local:peter foaf:knows ?p . ?p foaf:name ?name }
```

When evaluating the query, the Linked Data Caching module would then transparently retrieve the resource http://example.com/john.rdf and try to answer the query using the triples contained therein.

## Local Caching ##

When the triple data of a remote resource are retrieved, they are cached locally in the Linked Media Framework's triple store in a special named graph called "cache", together with provenance and cache expiry information. The cache graph is special in the sense that resources in this graph are by default not considered for direct indexing in the semantic search (i.e. a foaf:Person retrieved from the Linked Data Cloud will not be returned by the Semantic Search component), and triples and resources in this graph are not included in versioning.

A query to a resource is answered from the local cache as long as the entry is not expired, i.e. subsequent queries to a resource will be significantly faster than the first query as long as they are carried out within a certain time frame. The expiry date of a resource in the cache is determined in two ways:
  * if the Linked Data server providing the original resource sends an "Expires" header, the value of the header defines the expiry date of the local cache entry
  * otherwise, if there is an endpoint configuration for the URI prefix of the resource, the expiry time of the endpoint is used
  * otherwise, the default expiry time configured in the configuration variable ldcache.expiry is used

## Modes of Operation ##

Since the Linked Data Cloud is distributed over the Web and some services might not provide the reliability or availability necessary, or some servers are not yet Linked Data aware, the LMF LD Caching Module offers different modes of operation for accessing resources:
  * **Linked Data** (direct Linked Data access): makes use of the HTTP  content negotiation to directly retrieve a Linked Data resource from the Web as defined in the Linked Data Principles. This is the default behaviour if nothing else has been configured
  * **Cache** (access to a resource or entity cache endpoint): retrieves the triples of a resource from a defined server by accessing a cache endpoint. This is e.g. useful if there is a local cache of a public server (e.g. to improve reliability or performance) or if there is a "traditional" RDF server that does not offer Linked Data access
  * **SPARQL** (access to a resource via a SPARQL endpoint): retrieves the triples of a resource from a defined SPARQL endpoint by issuing a query for all triples with the resource as subject. This is e.g. useful for traditional RDF servers that are extended by a SPARQL endpoint using e.g. Joseki or other tools.
  * **NONE** (block access to the resource): does not try to retrieve the external resource. Useful for blocking access for known "bad" Linked Data servers. For example, many Linked Data sources contain references to Wikipedia, which does not offer its data as Linked Data directly.

The different modes of operation can be configured as described below. In addition to the RDF/XML format, the Linked Data Caching Module also supports additional RDF serialisation formats that can be configured for certain Linked Data servers.



# Configuration #

The Linked Data Caching Module can be configured using the LMF configuration mechanism. The following section describes the configuration options used.

## Configuration Options ##

The following options affect the general behaviour of the Linked Data Caching Module:

  * ldcache.enabled (true/false): if set to true, transparent Linked Data Caching is enabled; if set to false, remote resources will never be retrieved
  * ldcache.fallback (true/false): try to retrieve a resource using ordinary Linked Data call if no endpoint is  specified; when set to false, this option will not retrieve any remote resources except those defined with endpoints; can be used to provide transparent caching only to some specific Linked Data servers
  * ldcache.expiry (seconds): defines the default expiry time for a resource in the cache if the server does not provide an "Expires" header and the endpoint definition does not have an expiry time (default: 1 day)
  * ldcache.fetchall (true/false): if set to true, even a query for all resources will include the resources that are cached locally; the default is set to false because this might lead to unexpected behaviour and crawling of the whole Linked Data Cloud if not used with care


## Endpoint Configuration ##

The Linked Data Caching Module allows to define URI prefix to endpoint mappings. These can be used to either redirect Linked Data queries to cache or SPARQL endpoints for improving performance or accessing non-Linked Data resources, or for setting different parameters for certain server. An endpoint definition consists of the following parameters:

  * **name** (string): the public name of the endpoint, used for displaying
  * **prefix** (uri): the prefix of URIs that are redirected to the endpoint; if a resource requested from the cache has this prefix, it is handled by this endpoint
  * **kind** (SPARQL/LINKEDDATA/CACHE/NONE): the operation mode of the endpoint (see above)
  * **endpoint** (uri with parameters): the URI under which the endpoint can be accessed. Depending on the mode, certain parameters will be replaced by actual values in the URI:
    * CACHE: the parameter {uri} will be replaced by the actual URI of the requested resource, e.g. `http://api.sindice.com/v2/cache?url={uri}`
    * SPARQL: the parameter {query} will be replaced by the SPARQL query, the parameter {contenttype} will be replaced by the MIME type to be requested from the server, e.g. `http://sparql.sindice.com/sparql?default-graph-uri=&query={query}&format={contenttype}`
  * **mimetype** the MIME type to request/expect from the endpoint; depends on the mode of operation:
    * CACHE/LINKEDDATA: any RDF serialization supported by the LMF, e.g. application/rdf+xml, text/rdf+n3, ...
    * SPARQL: any SPARQL result format, e.g. application/sparql-results+xml
  * **expiry** (seconds): the default expiry time for this endpoint if not provided by the server itself

## Webservice ##

Endpoints are configured using the Linked Data Caching Webservice available under `<APPDIR>/cache/endpoint`; the Webservice provides the following operations for managing endpoints:
  * `POST <APPDIR>/cache/endpoint?name=...&prefix=...&endpoint=...&kind=...&mimetype=...&expiry=...` will create a new endpoint with the given parameters
  * `GET <APPDIR>/cache/endpoint/list` will return a list of configured endpoints in JSON
  * `GET <APPDIR>/cache/endpoint/{id}` will return the endpoint configuration for the endpoint with the given internal ID
  * `DELETE <APPDIR>/cache/endpoint/{id}` will delete the endpoint configuration for the endpoint with the given internal ID

In addition to managing endpoints, the webservice also offers two methods for retrieving resources:
  * `GET <APPDIR>/cache/live?uri=...` directly retrieves a resource from the endpoints or Linked Data Cloud without caching it locally; used for testing endpoints
  * `GET <APPDIR>/cache/cached?uri=...` refreshes the resource with the given URI and returns nothing; the resource can then be retrieved using the normal resource access webservices

For your convenience, the LMF administration interface offers a simple UI for adding, listing and removing
endpoint definitions.