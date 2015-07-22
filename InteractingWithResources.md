The LMF resource service allows to create, retrieve, update and delete individual resources and the metadata associated with them (see also LinkedMediaPrinciples).

To interact with resources, you need a REST client that is capable of issuing POST, PUT, GET and DELETE commands to remote resources. A very basic command line tool for this is cURL (http://curl.haxx.se/download.html).

The following series of commands creates, retrieves, updates and deletes the resource http://localhost:8080/LMF/resource/1234:

## Create Resource ##

Command:
```
curl -i -X POST http://localhost:8080/LMF/resource/1234
```

Result:
```
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/LMF/resource/1234
Content-Length: 0
Date: Wed, 18 May 2011 12:17:12 GMT
```

The result code of 201 indicates that the resource has been created. If the resource already exists, a "302 Found" will be returned.

## Upload Metadata ##

The server decides based on the "Content-Type" header how to process data sent with the request. To upload metadata, the Content-Type should be of the form "type/subtype; **rel=meta**". Examples:
  * application/rdf+xml; rel=meta
  * text/rdf+n3; rel=meta
  * ...

Command:
```
curl -X PUT -i -H "Content-Type: text/rdf+n3; rel=meta"  \
     -d '<http://localhost:8080/LMF/resource/1234> <http://www.w3.org/2000/01/rdf-schema#label> "Test Resource".' \
     http://localhost:8080/LMF/resource/1234
```

Result:
```
HTTP/1.1 303 See Other
Server: Apache-Coyote/1.1
Location: http://localhost:8080/LMF/meta/text/rdf+n3/1234
Content-Length: 0
Date: Wed, 18 May 2011 12:33:32 GMT
```

The result indicates that we have to redirect the request to the location http://localhost:8080/LMF/meta/text/rdf+n3/1234 . Note that many clients will not perform an automatic redirect properly, since they rewrite the method from PUT to GET. So it is preferrable to manually execute the redirect:

```
curl -X PUT -i -H "Content-Type: text/rdf+n3; rel=meta" \
     -d '<http://localhost:8080/LMF/resource/1234> <http://www.w3.org/2000/01/rdf-schema#label> "Test Resource".' \
     http://localhost:8080/LMF/meta/text/rdf+n3/1234
```

Result:
```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Length: 0
Date: Wed, 18 May 2011 12:38:30 GMT
```


## Upload Content ##

The server decides based on the "Content-Type" header how to process data sent with the request. To upload content, the Content-Type should be of the form "type/subtype; **rel=content**". Examples:
  * text/html; rel=content
  * text/plain; rel=content
  * image/jpeg; rel=content
  * ...


Command:
```
curl -X PUT -i -H "Content-Type: text/plain; rel=content" \
     -d 'This is textual content of the resource' \
     http://localhost:8080/LMF/resource/1234
```

Result:
```
HTTP/1.1 303 See Other
Server: Apache-Coyote/1.1
Location: http://localhost:8080/LMF/content/text/plain/1234
Content-Length: 0
Date: Wed, 18 May 2011 12:47:16 GMT
```

Again, we manually perform the redirect, this time to http://localhost:8080/LMF/content/text/plain/1234:

```
curl -X PUT -i -H "Content-Type: text/plain; rel=content" \
     -d 'This is textual content of the resource' \
     http://localhost:8080/LMF/content/text/plain/1234
```

Result:
```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Length: 0
Date: Wed, 18 May 2011 12:47:48 GMT
```

## Retrieving Metadata ##

The server decides based on the "Accept" header how to process data sent with the request. To retrieve metadata, the Accept should be of the form "type/subtype; **rel=meta**". Examples:
  * application/rdf+xml; rel=meta
  * text/rdf+n3; rel=meta
  * ...

When issuing the GET request, the server again returns a redirect. Since the request is a GET, it is safe to let curl automatically follow the redirect (-L):

Command:
```
curl -X GET  -L -i -H "Accept: application/rdf+xml; rel=meta" \
     http://localhost:8080/LMF/resource/1234
```

Result:
```
HTTP/1.1 303 See Other
Server: Apache-Coyote/1.1
Location: http://localhost:8080/LMF/meta/application/rdf+xml/1234
Content-Length: 0
Date: Wed, 18 May 2011 12:51:21 GMT

HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Link: <http://localhost:8080/LMF/content/text/plain/1234>;type=text/plain;rel=content
Content-Type: application/rdf+xml
Content-Length: 298
Date: Wed, 18 May 2011 12:51:21 GMT

<?xml version="1.0" encoding="UTF-8"?>
<rdf:RDF
	xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#"
	xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
<rdf:Description rdf:about="http://localhost:8080/LMF/resource/1234">
	<rdfs:label>Test Resource</rdfs:label>
</rdf:Description>

</rdf:RDF>
```


## Retrieving Content ##

The server decides based on the "Accept" header how to process the data requested. To retrieve content, the Accept should be of the form "type/subtype; **rel=content**". Examples:
  * text/html; rel=content
  * text/plain; rel=content
  * image/jpeg; rel=content
  * ...

When issuing the GET request, the server again returns a redirect. Since the request is a GET, it is safe to let curl automatically follow the redirect (-L):

Command:
```
curl -X GET  -L -i -H "Accept: text/plain; rel=content" \
     http://localhost:8080/LMF/resource/1234
```

Result:
```
HTTP/1.1 303 See Other
Server: Apache-Coyote/1.1
Location: http://localhost:8080/LMF/content/text/plain/1234
Content-Length: 0
Date: Wed, 18 May 2011 12:58:07 GMT

HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Links: ...
Content-Type: text/plain
Content-Length: 39
Date: Wed, 18 May 2011 12:58:07 GMT

This is textual content of the resource
```

The "Links:" header gives pointers to available alternative formats for the resource.

## Deleting a Resource ##

A resource is deleted using the DELETE method:

Command:
```
curl -i -X DELETE http://localhost:8080/LMF/resource/1234
```

Result:
```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Length: 0
Date: Wed, 18 May 2011 12:38:30 GMT
```