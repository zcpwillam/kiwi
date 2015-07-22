# Background #


The idea is a kind of combination of concepts from Linked Data, Media Management and Enterprise Knowledge Management (from KiWi). Up till now, the Linked Data world is read-only and primarily concerned with the structured data associated with a resource (regardless of whether this data is represented in RDF or visualised in HTML). However, in order to build more interactive mashups, it would make sense to also allow updates to the data in Linked Data servers. And in enterprise settings, it makes sense to have a unified means to manage both structured data and human-readable content for a resource. For example, a resource might represent a video on the internet, and depending on how I access the video I want to get either the video itself or the structured metadata about the video (e.g. a list of RDF links to DBPedia for all persons depicted in the video).

Our Linked Media idea tries to address both issues:
  * it extends the Linked Data principles with RESTful principles for addition, modification, and deletion of resources
  * it extends the Linked Data principles by means to manage content and meta-data alike using MIME to URL mapping


# Linked Media Idea #

## 1. extending the Linked Data principles for updates using REST ##

Linked Data is currently "read-only" and depending on Accept headers in the HTTP request, it redirects a request to the appropriate representation (RDF or HTML). For supporting updates in Linked Data, a consequent extension of Linked Data is to apply REST and otherwise use the same or analgous principles. This means that GET is used to retrieve a resource, POST is used to create a resource, PUT is used to update a resource, and DELETE is used to remove a resource. In case of GET, the Accept header determines what to retrieve and redirects to the appropriate URL; in case of PUT, the Content-Type header determines what to update and also redirects to the appropriate URL. This extension is therefore fully backwards compatible to Linked Data, i.e. each Linked Media server is a Linked Data server.

## 2. extending the Linked Data principles for arbitrary content using MIME mapping and "rel" Content Type ##

Linked Data currently distinguishes between an RDF representation and a human readable representation in the GET request. The GET request then redirects either to the URL of the RDF representation or to the URL of the human readable (HTML) representation. We extended this principle so that it can handle arbitrary formats based on the MIME type in Accept/Content-Type headers and so that it can still distinguish between content and metadata based on the "rel" extension for Accept/Content-Type headers.

The basic idea is to rewrite resource URLs of the form http://localhost/resource/1234 depending on the MIME type as follows:
  * if the `Accept/Content-Type` header is of the form "Accept: type/subtype; **rel=content**", then the redirect URL is `http://localhost/content/type/subtype/1234`; when this URL is requested, the Linked Media Server will deliver the CONTENT of the resource in the content type passed, or it will return "415 Unsupported Media Format" in case the content is not available in this MIME type
  * if the `Accept/Content-Type` header is of the form "Accept: type/subtype; **rel=meta**", then the redirect URL is `http://localhost/meta/type/subtype/1234`; when the URL is requested, the Linked Media Server will delivered METADATA associated with the resource and tries to seralise it in the content type passed (or again returns 415); in this way, different RDF serialisations can be supported, e.g. RDF/JSON

The differentiation between content and metadata using "rel" is necessary for a "kind-of" reification, because we need to be able to distinguish between an RDF/XML document stored in the server (i.e. content) and the metadata associated with it (i.e. metadata). The same holds for human-readable content: "Accept: text/html; rel=content" will return the HTML content, while "Accept: text/html; rel=meta" will return an HTML visualisation of the metadata (e.g. as a table).

This extension is also fully backwards-compatible to Linked Data, because the default behaviour (if no "rel" is given) is "rel=meta". So in case a Linked Data client accesses the Linked Media server, it will behave as expected.




# Implementation / Principles #

What we implemented in our Linked Media Server is therefore the following extensions (to simplify the presentation, I am always using a concrete resource URI, but it can be more or less arbitrary):

## GET http://localhost/resource/1234 ##

in case the resource does not exist, returns a "404 Not Found", otherwise, a "303 See Other" as follows:
  1. header: "Accept: type/subtype; rel=content" will redirect to http://localhost/content/type/subtype/1234; requesting the URL will return the content in the format requested by the MIME type if available; the HTTP response will then contain a "Link:" header linking to all metadata representations of the URL in all metadata serialisation formats supported; in case the resource is not available in the format requested, returns "415 Unsupported Media Type"
  1. header: "Accept: type/subtype; rel=meta" will redirect to http://localhost/meta/type/subtype/1234; requesting the URL will return the RDF metadata about the resource in the format requested by the MIME type; the HTTP response will then contain a "Link:" header linking to all content representations of the URL in all content formats available; in case the resource metadata is not available in the format requested, returns "415 Unsupported Media Type"

## POST http://localhost/resource/1234 ##

will create the resource with the URI given and return "201 Created"; the response will contain a "Location:" header pointing to the URL

## POST http://localhost/resource ##

will create a resource with a random URI and return "201 Created"; the response will contain a "Location:" header pointing to the generated URL

## PUT http://localhost/resource/1234 ##

in case the resource does not exist, returns a "404 Not Found", otherwise a "303 See Other" analoguous to GET, but instead of the "Accept" header, the "Content-Type" header is used:
  1. header: "Content-Type: type/subtype; rel=content" will redirect to the content location as in GET; a subsequent PUT to the redirected URL will upload the content in the given MIME type and stored on the server; returns a "200 Ok" in case of successful upload or different error codes in case of errors
  1. header:  "Content-Type: type/subtype; rel=meta" will redirect to the metadata location as in GET; a subsequent PUT to the redirected URL will upload the metadata in the given MIME type, which will then be parsed by the server and stored in the triple store; returns a "200 Ok" in case of successful upload or a "415 Unsupported Media Type" in case the parser does not exist, or also other error codes

## DELETE http://localhost/resource/1234 ##

in case the resource does not exist, returns a "404 Not Found", otherwise removes the resource and returns a "200 Ok"; removing the resource removes all content and all metadata (currently all triples where the resource is subject, object or predicate) from the triple store