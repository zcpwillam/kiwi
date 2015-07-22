# Introduction #


All resources stored in the Linked Media Framework are indexed in the semantic search component that uses Apache SOLR as its backend and creates a search index over selected properties of the resources. The semantic search component extends SOLR by some semantic capabilities, but it is fully API compatible and you can use any SOLR client library to access the LMF Search (e.g. AjaxSOLR).

The LMF semantic search component is capable of handling several different search configurations at the same time using the SOLR multi-core functionality. In this way, a LMF installation can be used for offering different perspectives on the data managed by it.


# Configuring Semantic Search #

Search over resources will always be domain-specific and will need to take into account the schema of the data. Therefore, the semantic search component provides only very simple and straightforward search functionality by default for three typical generic cases of metadata:
  * **rdf**: takes into account rdfs:label and rdfs:comment properties of resources and stores them in the search index
  * **dc**: takes into account dc:title and dc:description properties from the Dublin Core schema and stores them in the search index
  * **skos**: takes into account skos:prefLabel, skos:altLabel, and skos:definition properties from the SKOS schema and stores them in the search index.

To adapt the search component to your specific domain, the LMF Admin Interface offers  the possibility to define so-called "RDF Path Programs". An RDF Path Program is a set of rules that map index fields to RDF properties or paths of RDF properties. For example, the following program (rdf) defines four fields (title, summary, tag, type):

```
  @prefix hg : <http://www.holygoat.co.uk/owl/redwood/0.1/tags/> ;
  title   = rdfs:label :: xsd:string ;
  summary = rdfs:comment :: lmf:text ;
  tag     = hg:taggedWithTag / hg:name :: xsd:string ;
  type    = rdf:type :: xsd:anyURI ;
```

The complete RDF Path Language is documented under RdfPathLanguage .

In the most simple case (e.g. title), the rule maps an index field to exactly one RDF property. In more complex cases, the rule allows to follow a path of RDF properties; e.g. in the "tag" field above, the rule would start at the current resource and follow the hg:taggedWithTag property, and from there it will follow the hg:name property and store it in the index.

RDF Path Programs can be uploaded by the SOLR Cores Webservice available under `<APPDIR>/solr/cores`. It supports the following operations:
  * `POST <APPDIR>/solr/cores/{name}` uploads an RDF path program and creates a new search index (SOLR core) with the name given as path argument; all resources will automatically be reindexed in the new core. If the core already exists, returns a `302 exists` and does nothing.
  * `DELETE <APPDIR>/solr/cores/{name}` remove the search index (SOLR core) with the name given as path argument; will delete all index files and the path program from the system. Returns `404 Not Found` if the core does not exist
  * `POST <APPDIR>/solr/cores/{name}/enable` enables the (disabled) search index/core in the system; will trigger reindexing like creating a new core, because otherwise the consistency of the index cannot be guaranteed
  * `POST <APPDIR>/solr/cores/{name}/disable` disables the (enabled) search index/core in the system without deleting its index files or configuration



# Testing Search #

The "Query & Search" tab in the admin interface offers you to immediately try out the semantic search component by entering the search into the "Query Explorer" and pressing the "Run" button.

A good start is to enter the query `*:*` and let it execute. This search will list you all indexed resources together with their field values. Try adding a simple rule to the shortcuts, upload and reindex and then run the search again. You will see how the new field has been added to the search results.


# Using and Implementing Search #

The semantic search component can be used with any SOLR client library. A list of SOLR client libraries is available at http://wiki.apache.org/solr/IntegratingSolr . Note that the LMF only offers the search functionality but not the indexing functionality of existing clients.

Bundled with the Linked Media Framework comes an exemplary implementation of a Semantic Search interface in pure HTML and Javascript using the AjaxSOLR library. The interface is accessible via http://localhost:8080/LMF/solr/ui/ . Given the right kind of data, it also demonstrates how facetting over metadata properties can be used to display additional information in widgets, like a map or a timeline.

In case you want to offer a customised search interface for your application scenario, you can copy the bundled Search UI implementation and quickly adapt it to your requirements. In this way we have already implemented several search demonstrators.

A typical approach to using the LMF as a Semantic Search server is to
  * write an importer for the data you intend to index; this importer should map content and metadata fields to RDF properties of resources, preferrably conforming to a certain schema or ontology; an example can be found in the SNImporterImpl.java file, which imports a specific XML syntax of Salzburger Nachrichten
  * write a shortcut program that maps certain paths to index fields
  * adapt the Search UI according to the data you have imported by modifying / adding facets to display, configuring widgets, and adding query parameters to the search string (e.g. the type)