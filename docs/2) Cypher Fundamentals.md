# 2) Cypher Fundamentals
Link to [certificate](https://graphacademy.neo4j.com/u/fc0b0a0b-814c-48a8-ab4f-afd60a83e288/cypher-fundamentals)

Movie data schema:

![](https://graphacademy.neo4j.com/courses/cypher-fundamentals/1-reading/images/movie-schema.svg)

## Reading Data from Neo4j

### Introduction to Cypher

- Labels are introduces with `(:<Label>)` for example: `(:Person)`
- Relationships are specified wtih `[:<Relationship type>]` for example: `[:ACTED_IN]`
- Properties are specified with `{<key>: <value>}` for example: `{name: 'Max Mustermann'}`

Example Queries:

```
MATCH
(p: Person {name: 'Tom Hanks'}) -[:ACTED_IN]->(m: Movie {title: 'Cloud Atlas'})
RETURN p,m
```

Logical expressions such as `AND`, `OR`, `NOT` can only be used with `WHERE` statements

```
MATCH (p: Person)
WHERE p.name = 'Tom Hanks' OR p.name = 'Rita Wilson'
```

### Retrieving Nodes

Question: When was Kevin Bacon born

Query:

```
MATCH (p: Person {name: 'Kevin Bacon'}) RETURN p.born
```

Answer: 1958

### Finding Relationships

- Cypher pattern: `(<label>)-[<relationship>]->(<label>)`
  - Example: `MATCH (p: Person)-[:ACTED_IN]->(m: Movie) return p,m`

### Traversing Relationships

Question: How many people directed the movie Cloud Atlas?

Query:

```
MATCH (p: Person)-[:DIRECTED]->(m: Movie {title: 'Cloud Atlas'}) RETURN count(p)
```

Answer: 3

### Finding Emil

Question: Which movie has Emil Eifrem acted in?

Query:

```
MATCH (p: Person {name: 'Emil Eifrem'})-[:ACTED_IN]->(m) RETURN m.title
```

Answer: The Matrix

### Filtering Queries

- Label can also be tested in the WHERE clause
- To check if a property exists use `IS NOT NULL` in the WHERE clause
- String matches:
  - `STARTS WITH`
  - `ENDS WITH`
  - `CONTAINS`
- Functions:
  - `toLower`
- NOT EXISTS can be used to filter out entries basd on relationship:

  `MATCH (p)-[:ACTED_IN]->(m) WHERE NOT EXISTS ((p)-[:DIRECTED]->(m)) RETURN p, m`

- Range/List, `WHERE <attribute> IN [<value_one>, ...]`

### Finding Specific Actors

Question: How many actors in the movie The Matrix were born after 1960?

Query:

```
MATCH (p)-[:ACTED_IN]->(m {title: 'The Matrix'}) WHERE p.born > 1960 RETURN count(p)
```

Answer: 4

## Writing Data to Neo4j

### Creating Nodes

- Commands to add nodes:

  - `MERGE` - checks whether the entry already exists

    `MERGE (p: Person {name: 'Tom Hanks'})`

  - `CREATE` - simply adds the node even if it results in a duplicate; therefore `MERGE` is the recommended solution

### Creating a node

Query:

```
MERGE (p: Person {name: 'Daniel Kaluuya'})
```

### Creating Relationships

- Relationships can be added using `MERGE`
- References to the nodes have to be retrieved first

```
MATCH (p: Person {name: 'Tom Hanks'})
MATCH (m: Movie {title: 'Apollo 13'})
MERGE (p)-[:ACTED_IN]->(m)
```

### Creating a Relationship

Question:

    1. Find the Person node for Daniel Kaluuya.

    2. Create the Movie node, Get Out.

    3. Add the ACTED_IN relationship between Daniel Kaluuya and the movie, Get Out.

Query:

```
MATCH (p: Person {name: 'Daniel Kaluuya'})
MERGE (m: Movie {title: 'Get Out'})
MERGE (p)-[:ACTED_IN]->(m)
```

### Updating Properties

- Properties are added/updated with `SET`, e.g., `SET p.born = 1956`
- To add a relationship property, it has to be merged into the relationship, `MERGE (p)-[r: ACTED_IN {roles: ['ABC']}]->(m)`
  - using the reference to a relationship, the property can be updated with `SET`: `SET r.roles = ['ABC', 'XYZ']`
- Attributes can be removed using `REMOVE`, e.g., `REMOVE r.roles`
  - Alternative: Set the property to `null`
  - The identifying property should **never** be removed

### Adding Properties to a Movie

Question: Write the Cypher code to find this Movie node and add the tagline and released properties for the node with the values below.

tagline: Gripping, scary, witty and timely!

released: 2017

Query:

```
MATCH (m:Movie {title: 'Get Out'})
SET m.tagline = 'Gripping, scary, witty and timely!', m.released = 2017
RETURN m.title, m.tagline, m.released
```

### Merge Processing

- Conditions can be added to a merge clause to execute actions if the object can be found, for example:

  `MERGE (p: Person {name: 'Robin Williams'}) ON MATCH SET p.died = 2014 RETURN p`

- If an action should occur if the item is added `ON CREATE` can be used

### Adding or Updating a Movie

Query (run twice):

```
MERGE (m:Movie {title: 'Rocketman'})
ON MATCH
SET m.matchedAt = datetime()
ON CREATE
SET m.createdAt = datetime()
SET m.updatedAt = datetime()
RETURN m
```

### Deleting Data

- To delete a relationship, get a reference to it and call `DELETE <relationship>`
- If a node has no relationship to other nodes, simply call `DELETE <node>` using a reference to the node
- If a relationship still exists call `DETACH DELETE <node>`, the `DETACH` will dereference the node before deleting it

### Deleting Emil

Question: Remove Emil Eifrem

Query:

```
MATCH (p: Person {name: 'Emil Eifrem'})
DETACH DELETE p
```