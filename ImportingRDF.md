Sometimes it is desirable to import larger datasets consisting of more than one resource. Individually adding resources would be cumbersome. Therefore, the LMF provides an import functionality that can be used for uploading:

Command:
```
curl -i -H "Content-Type: text/rdf+n3" -X POST -d @test.n3 http://localhost:8080/LMF/import/upload
```

Would import the file test.n3 into the system. The command
```
curl -X GET http://localhost:8080/LMF/import/types
```
returns a list of types supported by the LMF importer.