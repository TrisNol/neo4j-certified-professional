# 3) Graph Data Modelling Fundamentals
Link to [certificate](https://graphacademy.neo4j.com/u/fc0b0a0b-814c-48a8-ab4f-afd60a83e288/modeling-fundamentals)

## Getting Started

### What is Graph Data Modeling?

- Steps:
  - Understand the domain and define specific use cases (questions)
  - Develop an initial data model (nodes + relationships)
  - Test the use cases against the model
  - Create a test graph instance using Cypher
  - Test the use cases (incl. performance) against the graph
  - Refactor the model if required
  - Implement the changes

### The Domain

- Steps:
  - Identify the stakeholders and developers of the application
  - In co-operation with stakeholders and devs:
    - Describe the application in detail
    - Identify the users of the application (people, technical systems)
    - Agree upon use cases for the application
    - Rank the importance of the use cases

### Purpose of the Model

- Data Model
  - Describes labels, properties and relationships in the graph
  - There are **no values** only schema
- Instance Model
  - Example of the model with actual instances incl. their properties and relationships
  - Used to test the model against use cases
- Guidelines:
  - Label is a single identifier starting with a capital letter
  - Relationship is a single identifier in all capital letters
  - Properties should start with a lower character

## Modeling Modes

### Modeling Nodes

- Defining Labels
  - Dominant entities in use cases
  - Often derived from **nouns** used to describe the use case
- Node properties
  - Uniquely identifying property
  - Additional properties use to describe details of the nodes

### Creating Nodes

Query:

```
MATCH (n) DETACH DELETE n;

MERGE (:Movie {title: 'Apollo 13', tmdbId: 568, released: '1995-06-30', imdbRating: 7.6, genres: ['Drama', 'Adventure', 'IMAX']})
MERGE (:Person {name: 'Tom Hanks', tmdbId: 31, born: '1956-07-09'})
MERGE (:Person {name: 'Meg Ryan', tmdbId: 5344, born: '1961-11-19'})
MERGE (:Person {name: 'Danny DeVito', tmdbId: 518, born: '1944-11-17'})
MERGE (:Person {name: 'Jack Nicholson', tmdbId: 514, born: '1937-04-22'})
MERGE (:Movie {title: 'Sleepless in Seattle', tmdbId: 858, released: '1993-06-25', imdbRating: 6.8, genres: ['Comedy', 'Drama', 'Romance']})
MERGE (:Movie {title: 'Hoffa', tmdbId: 10410, released: '1992-12-25', imdbRating: 6.6, genres: ['Crime', 'Drama']})

```

### Identifying a New Label

Add a new node called `User`

### Creating Mode Nodes

Task: Create two User nodes for:

    'Sandy Jones' with the userId of 534
    'Clinton Spencer' with the userId of 105

Query:
MERGE (s: User {userId: 534})
SET s.name = 'Sandy Jones'
MERGE (c: User {userId: 105, name: 'Clinton Spencer'})

## Modeling Relationships

### Modeling Relationships

- Extracted from verbs used in the use case
- Fanout:
  - Extracting properties as separate nodes
  - Can help answer specific question
  - Risk of "super nodes" and high number of relationships (chaos)

### Creating Initial Relationships

Query:

```
MATCH (apollo:Movie {title: 'Apollo 13'})
MATCH (tom:Person {name: 'Tom Hanks'})
MATCH (meg:Person {name: 'Meg Ryan'})
MATCH (danny:Person {name: 'Danny DeVito'})
MATCH (sleep:Movie {title: 'Sleepless in Seattle'})
MATCH (hoffa:Movie {title: 'Hoffa'})
MATCH (jack:Person {name: 'Jack Nicholson'})

// create the relationships between nodes
MERGE (tom)-[:ACTED_IN {role: 'Jim Lovell'}]->(apollo)
MERGE (tom)-[:ACTED_IN {role: 'Sam Baldwin'}]->(sleep)
MERGE (meg)-[:ACTED_IN {role: 'Annie Reed'}]->(sleep)
MERGE (danny)-[:ACTED_IN {role: 'Bobby Ciaro'}]->(hoffa)
MERGE (danny)-[:DIRECTED]->(hoffa)
MERGE (jack)-[:ACTED_IN {role: 'Jimmy Hoffa'}]->(hoffa)
```

### Identifying a New Relationship

Answer: Create a relationship called `RATED` with a property `rating`

### Creating More Relationships

Query:

```
MATCH (sandy:User {name: 'Sandy Jones'})
MATCH (clinton:User {name: 'Clinton Spencer'})
MATCH (apollo:Movie {title: 'Apollo 13'})
MATCH (sleep:Movie {title: 'Sleepless in Seattle'})
MATCH (hoffa:Movie {title: 'Hoffa'})

MERGE (sandy)-[:RATED {rating:5}]->(apollo)
MERGE (sandy)-[:RATED {rating:4}]->(sleep)

MERGE (clinton)-[:RATED {rating:3}]->(apollo)
MERGE (clinton)-[:RATED {rating:3}]->(sleep)
MERGE (clinton)-[:RATED {rating:3}]->(hoffa)
```

## Testing the Model

### Testing

### Testing with Instance Model

Prep:

```
MERGE (casino:Movie {title: 'Casino', tmdbId: 524, released: '1995-11-22', imdbRating: 8.2, genres: ['Drama','Crime']})
MERGE (martin:Person {name: 'Martin Scorsese', tmdbId: 1032})
MERGE (martin)-[:DIRECTED]->(casino)
```

1. How many Person nodes are in the graph?

   Query:
   `   MATCH (p: Person)
RETURN count(p) AS num_person`

   Answer: 5

2. How many Movie nodes are in the graph?

   Query:
   `   MATCH (m: Movie)
RETURN count(m) AS num_movie`

   Answer: 4

3. How many User nodes are in the graph?

   Query:
   `   MATCH (u: User)
RETURN count(u) as num_user`

   Answer: 2

4. How many ACTED_IN relationships are in the graph?

   Query:
   `   MATCH (p: Person)-[r: ACTED_IN]->(m: Movie)
RETURN count(r) as num_acted_in`

   Answer: 5

5. How many DIRECTED relationships are in the graph?

   Query:
   `   MATCH (p: Person)-[r: DIRECTED]->(m: Movie)
RETURN count(r) as num_acted_in`

   Answer: 2

6. How many RATED relationships are in the graph?

   Query:
   `   MATCH (u: User)-[r: RATED]->(m: Movie)
RETURN count(r) as num_rated`

   Answer: 5

## Refactoring the Graph

### Refactoring

- Required when:
  - The graph does not cover all use cases
  - The model does not scale well
- Steps:
  - Design new model
  - Implement Cypher queries to transform the model
  - Retest use cases and update Cypher

### Labels in the Graph

- Labels are an anchor in the query
- They reduce the number of nodes retrieved, reducing number of nodes to be filtered and processed
- Best practice: Limit number of labels per node to **4**
- Changing the label: `MATCH (p: Person) HERE exists ((p)-[ACTED_IN]-()) SET p: Actor`

### Adding the Actor Label

Query:

```
MATCH (p:Person)
WHERE exists ((p)-[:ACTED_IN]-())
SET p:Actor
```

### Retest After Refactoring

- Identify use cases affected by the refactoring
- Rewrite all queries that can be improved
- Test all queries affected by the refactoring and ensure they return the expected output

### Retest with Actor Label

Question: How many Actor nodes are now in the graph?

Query:

```
MATCH (a: Actor)
RETURN count(a) as num_actors
```

Answer: 4

### Adding the Director Label

Query:

```
MATCH (p:Person) WHERE exists ((p)-[:DIRECTED]-()) SET p:Director

MATCH (p:Director)-[:DIRECTED]-(m:Movie)
WHERE m.title = 'Hoffa'
RETURN  p.name AS Director
```

### Avoid These Labels

- Class hierarchies
  - Do not use multiple labels, instead model the relationships
- Synonyms

## Eliminiating Duplicate Data

### Eliminating Duplicate Data

- Duplicate data increases size of the graph and slows down queries

### Adding Language Data

Prep:

```
MATCH (apollo:Movie {title: 'Apollo 13', tmdbId: 568, released: '1995-06-30', imdbRating: 7.6, genres: ['Drama', 'Adventure', 'IMAX']})
MATCH (sleep:Movie {title: 'Sleepless in Seattle', tmdbId: 858, released: '1993-06-25', imdbRating: 6.8, genres: ['Comedy', 'Drama', 'Romance']})
MATCH (hoffa:Movie {title: 'Hoffa', tmdbId: 10410, released: '1992-12-25', imdbRating: 6.6, genres: ['Crime', 'Drama']})
MATCH (casino:Movie {title: 'Casino', tmdbId: 524, released: '1995-11-22', imdbRating: 8.2, genres: ['Drama','Crime']})

SET apollo.languages = ['English']
SET sleep.languages =  ['English']
SET hoffa.languages =  ['English', 'Italian', 'Latin']
SET casino.languages =  ['English']
```

Question: Which movie features dialogue spoken in Italian?

Query:

```
MATCH (m: Movie)
WHERE 'Italian' in m.languages
RETURN m.title
```

Answer: Hoffa

### Refactoring Duplicate Data

- Extracting unique property values as nodes
- Connect to original node using a relationship

### Adding Language nodes

Prep:

```
MATCH (m:Movie)
UNWIND m.languages AS language
WITH  language, collect(m) AS movies
    MERGE (l:Language {name:language})
    WITH l, movies
        UNWIND movies AS m
        WITH l,m
            MERGE (m)-[:IN_LANGUAGE]->(l);
            MATCH (m:Movie)
            SET m.languages = null
```

### Adding Genre nodes

Query:

```
MATCH (m:Movie)
UNWIND m.genres AS genre
WITH genre, collect(m) AS movies
MERGE (g:Genre {name:genre})
WITH g, movies
UNWIND movies AS m
WITH g,m
MERGE (m)-[:IN_GENRE]->(g);
MATCH (m:Movie)
SET m.genres = null

MATCH (p:Actor)-[:ACTED_IN]-(m:Movie)-[:IN_GENRE]->(g: Genre)
WHERE p.name = 'Tom Hanks' AND
g.name = 'Drama'
RETURN m.title AS Movie
```

### Eliminating Complex Data in Nodes

- Depends on scalability of the graph model

## Using Specific Relationships

### Specific Relationships

- Using relationships instead of properties to filter entries scales better since Neo4j is made for graph traversal

### Specializing ACTED_IN and DIRECTED Relationships

Query:
```
MATCH (n:Actor)-[:ACTED_IN]->(m:Movie)
CALL apoc.merge.relationship(n,
  'ACTED_IN_' + left(m.released,4),
  {},
  {},
  m ,
  {}
) YIELD rel
RETURN count(*) AS `Number of relationships merged`;

MATCH (p:Actor)-[:ACTED_IN_1995]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN m.title AS Movie

MATCH (n:Director)-[:DIRECTED]->(m:Movie)
CALL apoc.merge.relationship(n,
  'DIRECTED_' + left(m.released,4),
  {},
  {},
  m ,
  {}
) YIELD rel
RETURN count(*) AS `Number of relationships merged`;
```

### Specializing RATED Relationships

Query:
```
MATCH (n:User)-[r:RATED]->(m:Movie)
CALL apoc.merge.relationship(n,
  'RATED_' + r.rating,
  {},
  {},
  m ,
  {}
) YIELD rel
RETURN count(*) AS `Number of relationships merged`
```

## Adding Intermediate Nodes
### Intermediate Nodes

- "n:m" table like scenario in SQL
- Instead of placing all properties on the relationship, a node is added in between

### Adding a Role Node

Query:
```
MATCH (a: Actor)-[r: ACTED_IN]->(m: Movie)
MERGE (n: Role {name: r.role})
MERGE (a)-[:PLAYED]->(n)
MERGE (n)-[:IN_MOVIE]->(m)
```