<img src="https://guides.neo4j.com/rdf/n10s.png" height="150"><img src="https://raw.githubusercontent.com/RDFLib/rdflib/master/docs/_static/RDFlib.png" height="150">

# rdflib-neo4j
RDFLib Store backed by neo4j + n10s

With rdflib-neo4j you will be able to make your RDFLib code interact (read/write) with a Neo4j Graph Database through the [neosemantics(n10s) plugin](https://github.com/neo4j-labs/neosemantics/)

If you're not familiar with RDFLib you can [learn more here](https://github.com/RDFLib/rdflib/#getting-started). 

If you're not familiar with n10s you can [learn more here](https://neo4j.com/labs/neosemantics/). 


## Getting Started

Here are the steps you need to follow on your Neo4j database and on your python code:


### On the Neo4j side
In order to set up your Neo4j Graph DB to work with RDF data you need to do two things:

#### 1. Install the n10s plugin

You can do it in a couple of clicks from the neo4j desktop (plugins section of your DB) 

<img src="https://raw.githubusercontent.com/neo4j-labs/rdflib-neo4j/master/img/install-n10s.png" height="400">


Or you can do it manually following the [instructions in the manual](https://neo4j.com/labs/neosemantics/4.0/install/)

#### 2. Initialise the Database

Connect to your database and either from the browser or the console, create a uniqueness constraint on Resources' uri by running the following cypher fragment:
```cypher
CREATE CONSTRAINT n10s_unique_uri ON (r:Resource)
ASSERT r.uri IS UNIQUE;
```

* Set the configuration of the graph:
The simplest way of doing it is by running the following cypher expression: (all default options) 
```cypher
CALL n10s.graphconfig.init();
```
If you want to know more about additional config options, have a look at the [n10s reference](https://neo4j.com/labs/neosemantics/4.0/reference/#_rdf_config).


### On the Python side
rdflib-neo4j can be installed with Python's package management tool *pip*:

    $ pip install -i https://test.pypi.org/simple/ rdflib-neo4j


### You're ready to go!
You can now create an RDFLib graph and load RDF data in it. Every single triple will be transparently persisted in your Neo4j DB. Here's how:

You can import the data from an RDF document (for example [this one serialised using N-Triples](https://github.com/jbarrasa/datasets/blob/master/rdf/music.nt)):

```python
import rdflib

# create a neo4j backed Graph
g = rdflib.Graph(store='Neo4j')

# set the configuration to connect to your Neo4j DB 
theconfig = {'uri': "neo4j://localhost:7687", 'database': 'rdfstore', 'auth': {'user': "neo4j", 'pwd': "neo"}}

g.open(theconfig, create = False)

g.load("https://raw.githubusercontent.com/jbarrasa/datasets/master/rdf/music.nt", format="nt")

# For each foaf:Person in the store, print out their mbox property's value.
print("--- printing band's names ---")
for band in g.subjects(rdflib.RDF.type, rdflib.URIRef("http://neo4j.com/voc/music#Band")):
    for bandName in g.objects(band, rdflib.URIRef("http://neo4j.com/voc/music#name")):
        print(bandName)
```
This fragment retunrs a listing with the names of the bands in the dataset and at the same time, has populated your Neo4j DB with a graph like this one:

<img src="https://raw.githubusercontent.com/neo4j-labs/rdflib-neo4j/master/img/graph-view.png" height="400">

You can also write to the graph triple by triple like this:

```python
MUSIC = rdflib.Namespace("http://neo4j.com/voc/music#")
fm = rdflib.URIRef("http://neo4j.com/indiv#Fleetwood_Mac")

g.add((fm, rdflib.RDF.type, MUSIC.Band))
g.add((fm, MUSIC.name, rdflib.Literal("Fleetwood Mac")))
```
Where `rdflib.RDFS` is the RDFS Namespace, `MUSIC` is a custom namespace for the vocabulary used in our dataset, and `rdflib.URIRef` and `rdflib.Literal` are used to construct URI objects and Literal objects from their string representations. 

The previous fragment would add another node to the graph representing the band Fleetwood Mac.

It is also possible to serialise your graph like this:
 ```python
print(g.serialize(format="turtle").decode("utf-8"))
```

Producing the following result (truncated):
```turtle
<http://neo4j.com/indiv#The_Beatles> a <http://neo4j.com/voc/music#Artist>,
        <http://neo4j.com/voc/music#Band> ;
    <http://neo4j.com/voc/music#member> <http://neo4j.com/indiv#George_Harrison>,
        <http://neo4j.com/indiv#John_Lennon>,
        <http://neo4j.com/indiv#Paul_McCartney>,
        <http://neo4j.com/indiv#Ringo_Starr> ;
    <http://neo4j.com/voc/music#name> "The Beatles"^^<http://www.w3.org/2001/XMLSchema#string> .
    
    [...]
```
