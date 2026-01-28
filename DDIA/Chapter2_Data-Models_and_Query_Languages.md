# Chapter 2: Data Models and Query Languages

## Overview

Chapter 2 explores the history and application of **data models**, which the author considers the most critical aspect of software development because they dictate not only how code is written but also how we perceive the problems we are solving. Most applications are built by layering models on top of one another, with each layer providing a clean abstraction that hides the complexity of the layer below. The chapter specifically compares the **relational model**, the **document model**, and various **graph-based models**, evaluating their query languages and suitability for different application requirements.

--------------------------------------------------------------------------------

## Chapter Structure

### Relational Model Versus Document Model

The relational model, proposed by Edgar Codd in 1970, remains the dominant paradigm, though it was originally doubted for its implementation efficiency. Its primary goal was to hide internal implementation details—such as the labyrinthine nested structures used by earlier models—behind a clean interface of tables (relations) and rows (tuples).

• **The Birth of NoSQL**: The rise of NoSQL databases in the 2010s `was driven by a need for greater scalability, a preference for open-source tools, and a desire for more expressive data models that the restrictive relational schema could not provide.`

• **The Object-Relational Mismatch**: Application development is primarily done in object-oriented languages, creating a disconnect with the relational model known as an **impedance mismatch**.

• **Many-to-One and Many-to-Many Relationships**: Relational databases use **normalization** to remove duplication, representing many-to-one relationships by referring to rows in other tables by ID (foreign keys). While document models handle one-to-many "tree" structures well by nesting data, they struggle with many-to-many relationships because joins are often poorly supported.

• **Schema Flexibility**: The author distinguishes between **schema-on-write** (traditional relational databases where the schema is explicit and enforced) and **schema-on-read** (where the structure is implicit and only interpreted when data is accessed).

• **Data Locality**: Document databases offer a performance advantage through storage locality if an application frequently needs to access an entire document at once, as the data is stored in one continuous string (JSON, XML, or BSON).

### Query Languages for Data

The chapter compares **declarative** languages, like SQL, with **imperative** code used in older systems like IMS and CODASYL.

• **Declarative Advantages**: In a declarative language, the developer specifies the pattern of data they want but not how to achieve it; this allows the **query optimizer** to introduce performance improvements without changing the queries. Declarative languages also lend themselves better to parallel execution across multiple cores and machines.

• **MapReduce Querying**: MapReduce is a programming model for bulk processing across many machines that sits between the declarative and imperative styles.

### Graph-Like Data Models

As data connections become more complex, modeling data as a graph—consisting of **vertices** (nodes) and **edges** (relationships)—becomes more natural.

• **Property Graphs**: In this model, each vertex and edge has a unique ID and a collection of key-value properties. They are highly flexible because any vertex can be connected to any other vertex regardless of kind, and both incoming and outgoing edges can be efficiently traversed.

• **Query Languages (Cypher, SPARQL, and Datalog)**:

    ◦ **Cypher** is a declarative query language for property graphs that uses an arrow notation (e.g., `(Idaho) -[:WITHIN]-> (USA)`) to describe patterns.

    ◦ **SPARQL** is the query language for the **triple-store model**, where information is stored as `(subject, predicate, object)` triples.

    ◦ **Datalog** is a much older language that provides the foundation for later systems; it defines rules that tell the database how to derive new predicates from existing data.

--------------------------------------------------------------------------------

Connection to the Book

Chapter 2 builds directly on the themes of **evolvability** and **maintainability** introduced in Chapter 1 by showing how different data models affect the ease of changing applications. It sets the stage for **Chapter 3 (Storage and Retrieval)**, which will explain how the data models discussed here are actually implemented at the physical storage level. By establishing that one model can be emulated in terms of another—though often awkwardly—the author prepares the reader for the later discussion in **Part III** on how to integrate disparate derived data systems.