---
title: TDB Architecture
---

This page gives an overview of the TDB architecture. Specific
details refer to TDB 0.8.

## Contents

-   [Terminology](#terminology)
-   [Design](#design)
    -   [The Node Table](#the-node-table)
    -   [Triple and Quad indexes](#triple-and-quad-indexes)
    -   [Prefixes Table](#prefixes-table)
    -   [TDB B+Trees](#tdb-btrees)
-   [Inline values](#inline-values)
-   [Query Processing](#query-processing)
-   [Caching on 32 and 64 bit Java systems](#caching-on-32-and-64-bit-java-systems)

## Terminology

Terms like "table" and "index" are used in this description. They
don't directly correspond to concepts in SQL, For example, in SQL
terms, there is no triple table; that can be seen as just having
indexes for the table or, alternatively, there are 3 tables, each
of which has a primary key and TDB manages the relationship between
them.

## Design

A dataset backed by TDB is stored in a single directory in the
filing system. A dataset consists of

-   The node table
-   Triple and Quad indexes
-   The prefixes table

### The Node Table

The node table stores the representation of RDF terms (except for
inlined value - see below). It provides two mappings from Node to
NodeId and from NodeId to Node. This is sometimes called a
dictionary.

The Node to NodeId mapping is used during data loading and when
converting constant terms in queries from their Jena Node
representation to the TDB-specific internal ids.

The NodeId to Node mapping is used to turn query results expressed
as TDB NodeIds into the Jena Node representation and also during
query processing when filters are applied if the whole node
representation is needed for testing (e.g. regex).

Node table implementations usually provide a large cache - the
NodeId to Node mapping is heavily used in query processing yet the
same NodeId can appear in many query results.

NodeIds are 8 byte quantities. The Node to NodeId mapping is based
on hash of the Node (a 128 bit MD5 hash - the length was found not
to major performance factor).

The default storage of the node table is a sequential access file
for the NodeId to Node mapping and a B+Tree for the Node to NodeId
mapping.

### Triple and Quad indexes

Quads are used for named graphs, triples for the default graph.
Triples are held as 3-tuples of NodeIds in triple indexes - quads
as 4-tuples. Otherwise they are handled in the same manner.

The triple table is 3 indexes - there is no distinguished triple
table with secondary indexes. Instead, each index has all the
information about a triple.

The default storage of each indexes

### Prefixes Table

The prefixes table uses a node table and a index for GPU
(Graph-\>Prefix-\>URI). It is usually small. It does not take part
in query processing. It provides support for Jena's PrefixMappings
used mainly for presentation and for serialisation of triples in
[RDF/XML](http://www.w3.org/TR/REC-rdf-syntax/ "http://www.w3.org/TR/REC-rdf-syntax/")
or
[Turtle](http://www.w3.org/TeamSubmission/turtle/ "http://www.w3.org/TeamSubmission/turtle/").

### TDB B+Trees

Many of the persistent data structures in TDB use a custom
implementation of threaded
[B+Trees](http://en.wikipedia.org/wiki/B+_tree "http://en.wikipedia.org/wiki/B%2B_tree").
The TDB implementation only provides for fixed length key and fixed
length value. There is no use of the value part in triple indexes.

The threaded nature means that long scans of indexes proceeds
without needing to traverse the branches of the tree.

See the description of index caching below.

## Inline values

Values of certain datatypes are held as part of the NodeId in the
bottom 56 bits. The top 8 bits indicates the type - external NodeId
or the value space.

The value spaces handled are (TDB 0.8): xsd:decimal, xsd:integer,
xsd:dateTime, xsd:date and xsd:boolean. Each has its own encoding
to fit in 56 bits. If a node falls outside of the range of values
that can be represented in the 56 bit encoding.

The xsd:dateTime and xsd:date ranges cover about 8000 years from
year zero with a precision down to 1 millisecond. Timezone
information is retained to an accuracy of 15 minutes with special
timezones for Z and for no explicit timezone.

By storing the value, the exact lexical form is not recorded. The
integers 01 and 1 will both be treated as the value 1.

Derived XSD datatypes are held as their base type. The exact
datatype is not retained; the value of the RDF term is.

## Query Processing

TDB uses the
[OpExecutor extension point of ARQ](TODO).
TDB provides low level optimization of basic graph patterns using a
[statistics based optimizer](optimizer.html "TDB/Optimizer").

## Caching on 32 and 64 bit Java systems

TDB runs on both 32-bit and 64-bit Java Virtual Machines. The same
file formats are used on both systems and database files can be
transferred between architectures (no TDB system should be running
for the database at the time of copy). What differs is the file
access mechanism used.

TDB is faster on a 64 bit JVM because more memory is available for
file caching.

The node table caches are always in the Java heap.

The file access mechanism can be set explicitly, but this is not a
good idea for production usage, only for experimentation - see the
[File Access mode option](configuration.html#File_Access_Mode "TDB/Configuration").

On 64-bit Java, TDB uses memory mapped files, accessed 8M segments,
and the operating system handles caching between RAM and disk. The
amount of RAM used for file caching increases and decreases as
other application run on the machine. The fewer other programs
running on the machine, the more RAM will be available for file
caching. The mapped address space counts as part of the application
processes memory usage but this space is not part of the Java
heap.

On a 32 bit JVM, this approach does not work because Java
addressing is limited to about 1.5Gbytes (the exact figure is JVM
specific and includes any memory mapped file usage) and this would
limit the size of TDB datasets. Instead, TDB provides an in-heap
LRU cache of B+Tree blocks. Applications should set the JVM heap to
1G or above (within the JVM specific limit).

On 32-bit Java, TDB uses its own file caching to enable large
databases. 32-bit Java limits the address space of the JVM to about
1.5Gbytes (the exact size is JVM-dependent), and this includes
memory mapped files, even though they are not in the Java heap. The
JVM heap size may need to be increased to make space for the disk
caches used by TDB.



