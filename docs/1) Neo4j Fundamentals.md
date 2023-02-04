# 1) Neo4j Fundamentals
Link to [certificate](https://graphacademy.neo4j.com/u/fc0b0a0b-814c-48a8-ab4f-afd60a83e288/neo4j-fundamentals)

## Graph Thinking

### The Seven Bridges

- Graph theory was conceived by Leonhard Euler
- He modelled the seven bridges of Königsberg using nodes and edges to find out whether one could traverse the city visiting each bridge exactly once
- Today, graph theory is used to model use cases such as supply chain management in order to enable improved analytics

### Graph Elements

- Nodes (aka vertices)
    - e.g. people, locations, products
- Relationships (aka edges)
    - connnect nodes
    - e.g. Person x knows Person y

### Graph Structure

- Undirected Graph
    - relationships are symmetric
- Directed/Weighted Graph
    - adds semantic meaning to relationships
    - it can be used to model symmetric relationships but adding a weight to model the strength of the connection

Main action in Graphs: Traversal

### Graphs Are Everywhere
- Optimized for use cases with high relationship significance (e.g., product purchases, IT service operations)
    - SQL based DBs will have to hold a lot of data in memory to accomplish this, whereas graphs can quickly solve it

An overview of Neo4j implementations by Use Case or Industry can be found at [GraphGists](https://neo4j.com/graphgists/)

## Property Graphs

### What is a Property Graph?
- "Labels" can be added to a node to add it to a subset of nodes (basically creating classes)
    - a node can have `0..n` labels
- "Properties" are key-value pairs that are assigned to a node
    - can be edited/removed 
    - no schema, if a node does not have a property it defaults to null
    - can also be placed at relationships

Note: Relationships **must** have a `type` and `direction`

### Native Graph Advantage
- Storage as well as query manage are made for traversal
- Concept of index-free adjacency
    - Reference to relationship is stored at beginning and end of relationship
    - Nodes and relationships are stored as objects and connected using pointers
    - Benefits:
        - Fewer index lookups
        - No table scans
        - Reduced duplication of data

### Non-graph Databases to Graph
- Key-value model:
    - Optimized for lookups on large data on attribute level
    - Poor performance if keys are related to one another
- Document store:
    - Modelled as a tree, which is a kind of graph but not as optimized

## Your First Graph
### The Movie Graph
- Used for many graph academy courses
- contains:
    - 171 nodes overall
    - 38 movie nodes
    - 38 person nodes
- Movie node:
    - title: Identifier
    - released: Year of release
    - tagline: Phrase used to describe a movie
- Person node:
    - name: Identifier
    - born: Birthyear
- Relationships:
    - ACTED_IN with Role property: Person --> Movie
    - DIRECTED: Person --> Movie
    - WROTE: Person --> Movie
    - PRODUCED: Person --> Movie
    - REVIEWED with Rating and Summary property: Person --> Movie
    - FOLLOWS: Person --> Person

