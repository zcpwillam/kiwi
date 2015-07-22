# Introduction #

The Linked Media Framework includes an (optional) rule-based reasoner that is highly customizable and allows to evaluate user-defined rules over triples in the LMF Triple Store. Rules can be uploaded and stored in the LMF via an easy-to-use web service. The evaluation strategy is an incremental forward chaining reasoning with truth maintenance and is reasonably efficient even for big data sets. The truth maintenance can be used to provide explanations for inferred triples as well as for efficient updating of the reasoning information.

The sKWRL reasoner has originally been developed for the KiWi project, but has been re-implemented in context of the LMF using more efficient evaluation strategies but also a simplified language with no negation.

# Rules and Programs #

The LMF Reasoner can load one or more _programs_, which are themselves a collection of _rules_. Each rule consists of a rule body ("condition") and a rule head ("consequence"). For example, the following rule is part of the RDFS reasoning typically found in triple stores:

```
($p http://www.w3.org/2000/01/rdf-schema#domain $d), ($1 $p $2) 
            -> ($1 http://www.w3.org/1999/02/22-rdf-syntax-ns#type $d)
```

The rule can be read as "if there is a property $p with domain $d and there is a triple ($1 $p $2), then the subject $1 of this triple gets the inferred type $d".

The syntax of the reasoning rules is thus as follows:
  * $X denotes the variable X; variables are universally quantified over the rule, i.e. existentially quantified over the body, read: "if there exists a triple such that variable X can be bound"... Variables are bound using so-called unification, i.e. the system looks for bindings that are compatible with all occurrences of the variable in patterns
  * (S P O) denotes a triple pattern; S and P can be either URIs or variables, and O can be either a URI, a variable, or a literal. A triple pattern occurring in the body of a rule is considered a query pattern, a triple pattern occurring in the head of a rule is considered a construction pattern used for creating new triples
  * BODY -> HEAD denotes a rule; BODY is a comma-separated conjunctive (AND-connected) list of one or more triple patterns used for querying; the HEAD is a single triple pattern used for construction of inferred triples

# Program Web Service #

Programs can be uploaded and managed by the Program Webservice, which provides methods for uploading and deleting programs and triggers the necessary executions of the reasoning engine when the rule set is changed.

Note that the services described below are also available with a simple UI from the Admin interface (Tab "Reasoner").

## Uploading a Program ##

A program can be uploaded by posting to the service `reasoner/program/PROGRAM` as follows (where `PROGRAM` is the name of the program to upload, in the example `rdfs.kwrl`):

```
curl -i -H "Content-Type: text/plain" -X POST --data-binary @rdfs.kwrl http://localhost:8080/LMF/reasoner/program/rdfs.kwrl
```

When the program is uploaded, the web service will return 200 Ok and the parsed program for checking. It will also immediately trigger a full re-reasoning over the knowledge base.

## Listing Programs ##

The currently stored programs can be listed by calling the service `reasoner/program/list` using the GET method:

```
curl -i  -X GET http://localhost:8080/LMF/reasoner/program/list
```


## Deleting a Program ##

Programs can be deleted by calling the service for the respective program with the DELETE method:

```
curl -i -X DELETE http://localhost:8080/LMF/reasoner/program/rdfs.kwrl
```


# Reasoning Web Service #

The Reasoning Webservice provides access to the functionalities of the reasoner. In particular, it allows to provide justifications why a certain triple has been inferred. This information is e.g. used in the HTML resource view of the Linked Data Server.

## Listing Justifications ##

The reason maintenance component of the LMF Reasoner offers explanations to why a triple has been inferred. Each explanation consists of a set of base triples and rules that have been used in the reasoning process for inferring the triple. Each inferred triple might have one (if only one reasoning path leads to this triple) or more (if there are several different ways of inferring this triple) justification. When all justifications are removed (e.g. because a base triple or rule is removed), the inferred triple is also removed.

The Reasoning Webservice offers to retrieve the justifications for a triple using the `reasoner/engine/justify?id=TRIPLEID` service call. For example, the following call returns the justifications for the triple with ID 27:

```
curl -i -X GET http://localhost:8080/LMF/reasoner/engine/justify?id=27
```

The justifications are returned in JSON format so they can be displayed to the user. The justifications might look as follows:

```
[{
 "triple": { "http://www.kiwi-project.eu/kiwi/core/E" : { "http://www.w3.org/2000/01/rdf-schema#subClassOf" : [{  "type" : "uri", "value" : "http://www.kiwi-project.eu/kiwi/core/A" }] } },
"justifications": [
  {
    "triples": [
      { "http://www.kiwi-project.eu/kiwi/core/E" : { "http://www.w3.org/2000/01/rdf-schema#subClassOf" : [{  "type" : "uri", "value" : "http://www.kiwi-project.eu/kiwi/core/D" }] } },
      { "http://www.kiwi-project.eu/kiwi/core/C" : { "http://www.w3.org/2000/01/rdf-schema#subClassOf" : [{  "type" : "uri", "value" : "http://www.kiwi-project.eu/kiwi/core/B" }] } },
      { "http://www.kiwi-project.eu/kiwi/core/D" : { "http://www.w3.org/2000/01/rdf-schema#subClassOf" : [{  "type" : "uri", "value" : "http://www.kiwi-project.eu/kiwi/core/C" }] } },
      { "http://www.kiwi-project.eu/kiwi/core/B" : { "http://www.w3.org/2000/01/rdf-schema#subClassOf" : [{  "type" : "uri", "value" : "http://www.kiwi-project.eu/kiwi/core/A" }] } }
    ],    "rules": [
      "($1 http://www.w3.org/2000/01/rdf-schema#subClassOf $2), ($2 http://www.w3.org/2000/01/rdf-schema#subClassOf $3) -> ($1 http://www.w3.org/2000/01/rdf-schema#subClassOf $3)"
    ]  }
]}
]
```

The triple (E subClassOf A) is inferred because of the subClassOf rule (see below) and because of the base triples (E subClassOf D), (D subClassOf C), (C subClassOf B) and (B subClassOf A).

## Full Reasoning ##

The reasoner also provides a method to completely restart all reasoning and carry out a full reasoning from scratch. This might be useful in case a running reasoning process has been interrupted. The service is called as follows:

```
curl -i -X POST http://localhost:8080/LMF/reasoner/engine/run
```



# Example Programs #

In the following, we give a collection of sample reasoning programs that you can directly use in the Linked Media Framework.

## RDFS Reasoning ##

The following program implements RDFS subclass, type, domain and range reasoning:

```
($1 http://www.w3.org/2000/01/rdf-schema#subClassOf $2), ($2 http://www.w3.org/2000/01/rdf-schema#subClassOf $3) -> ($1 http://www.w3.org/2000/01/rdf-schema#subClassOf $3)
($1 http://www.w3.org/2000/01/rdf-schema#subPropertyOf $2), ($2 http://www.w3.org/2000/01/rdf-schema#subPropertyOf $3) -> ($1 http://www.w3.org/2000/01/rdf-schema#subPropertyOf $3)
($1 http://www.w3.org/1999/02/22-rdf-syntax-ns#type $2), ($2 http://www.w3.org/2000/01/rdf-schema#subClassOf $3) -> ($1 http://www.w3.org/1999/02/22-rdf-syntax-ns#type $3)
($p http://www.w3.org/2000/01/rdf-schema#range $r), ($1 $p $2) -> ($2 http://www.w3.org/1999/02/22-rdf-syntax-ns#type $r)
($p http://www.w3.org/2000/01/rdf-schema#domain $d), ($1 $p $2) -> ($1 http://www.w3.org/1999/02/22-rdf-syntax-ns#type $d)
```


## SKOS Reasoning ##

The following SKOS program is very exhaustive; in most scenarios it is sufficient to only use the first rule or the first three rules.

```
($1 http://www.w3.org/2004/02/skos/core#broader $2), ($2 http://www.w3.org/2004/02/skos/core#broader $3) -> ($1 http://www.w3.org/2004/02/skos/core#broader $3)
($1 http://www.w3.org/2004/02/skos/core#broader $2) -> ($2 http://www.w3.org/2004/02/skos/core#narrower $1)
($1 http://www.w3.org/2004/02/skos/core#narrower $2) -> ($2 http://www.w3.org/2004/02/skos/core#broader $1)
($1 http://www.w3.org/2004/02/skos/core#broader $2) -> ($1 http://www.w3.org/2004/02/skos/core#related $2)
($1 http://www.w3.org/2004/02/skos/core#narrower $2) -> ($1 http://www.w3.org/2004/02/skos/core#related $2)
($1 http://www.w3.org/2004/02/skos/core#related $2) -> ($2 http://www.w3.org/2004/02/skos/core#related $1)
```