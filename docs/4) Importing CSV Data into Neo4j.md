# 4) Importing CSV Data into Neo4j

Link to [certificate](https://graphacademy.neo4j.com/u/fc0b0a0b-814c-48a8-ab4f-afd60a83e288/importing-data)

## 4.1 Preparing for importing Data into Neo4j

### 4.1.1 Overview of importing data into Neo4j

- Cypher can import `CSV` (out-of-the-box), `XML` (with APOC) and `JSON` (with APOC)
- Import requires mapping of:
  - Entities
  - Relationships
  - DataTypes

### 4.1.2 Understanding the source data

- Analyse CSV structure:
  - Column headers
  - Field delimiter
- Transform normalized data (exported from RDBMS):
  - Foreign keys have to be transformed into relationships
- Common practice: Use existing IDs as unique identifier for nodes

### 4.1.3 Inspecting the Data for Import

- Steps:
  1. acquire a local copy of the CSV file(s)
  2. Determine the field delimiter
  3. Determine if the column headers match the column data
  4. Determine if all fields are readable (i.e. check for quotes or special characters)
  5. Clean the data if required

### 4.1.4 Inspecting the Data

1. What is the delimiter? - Answer: Comma (,)
2. How many rows?

Query:

```
LOAD CSV WITH HEADERS FROM 'https://data.neo4j.com/importing/ratings.csv' as line RETURN count(line)
```

Answer:
3594

3. Is thie file readable? - Answer: No

### 4.1.5 Understanding the Data Model

- Prepared model contains:
  - Nodes:
    - Person
    - Actor
    - Director
    - User
    - Movie
    - Genre
  - Relationships:
    - ACTED_IN
    - DIRECTED
    - RATED
    - IN_GENRE

## 4.2 Using the Neo4j Data Importer

### 4.2.1 Overview of the Neo4j Data Importer

- Instance at: https://data-importer.neo4j.io/
- External tool supporting the data ingestion with a visual mapping

### 4.2.2 Importing CSV files with the Neo4j Data Importer

see [Model export](../data/neo4j_importer_model_2023-03-05.json)

### 4.2.3 Post-Import Steps

- List of strings have been imported as a single string with values separated by the pipe character

## 4.3 Refactoring Imported Data

### 4.3.1 Viewing Property Types Stored in the Graph

- Procedures:

  `apoc.meta.nodeTypeProperties()`

  `apoc.meta.relTypeProperties()`

### 4.3.2 Transform String Properties

1. Transform `born` field to date type:

```
MATCH (p:Person)
SET p.born = CASE p.born WHEN "" THEN null ELSE date(p.born) END
WITH p
SET p.died = CASE p.died WHEN "" THEN null ELSE date(p.died) END
```

2. View Node Types

```
CALL apoc.meta.nodeTypeProperties()
YIELD nodeType, propertyName, propertyTypes
```

3. View Rel. Types

```
CALL apoc.meta.relTypeProperties()
YIELD relType, propertyName, propertyTypes
```

### 4.3.3 Transforming Multi-value Properties

- Lists
- All values in the list must be of the same type

### 4.3.4 Transform Strings to Lists

```
MATCH (m:Movie)
SET m.countries = split(coalesce(m.countries,""), "|"),
    m.genres = split(coalesce(m.genres,""), "|"),
    m.languages = split(coalesce(m.languages,""), "|")
```

### 4.3.5 Adding labels

- Labels in the data model: Actor & Director
- Improves query performance by early filtering

```
MATCH (p: Person)-[:ACTED_IN]->()
WITH DISTINCT p SET p:Actor
```

### 4.3.6 Add labels to the graph

1. Add the Actor labels:

```
MATCH (p:Person)-[:ACTED_IN]->()
WITH DISTINCT p SET p:Actor
```

2. Add the Director labels:

```
MATCH (p:Person)-[:DIRECTED]->()
WITH DISTINCT p SET p:Director
```

### 4.3.7 Refactoring Properties as Nodes

- Improved transparency with dedicated nodes
- Reduced data duplication

### 4.3.8 Create the Genre Nodes

1. Create the constraint

```
CREATE CONSTRAINT Genre_name IF NOT EXISTS
FOR (x:Genre)
REQUIRE x.name IS UNIQUE
```

2. Create the Genre nodes

```
MATCH (m:Movie)
UNWIND m.genres AS genre
WITH m, genre
MERGE (g:Genre {name:genre})
MERGE (m)-[:IN_GENRE]->(g)
```

3. Remote the genres property

```
MATCH (m: Moview) SET m.genres = null
```

## 4.4 Importing CSV Data with Cypher

### 4.4.1 Importing Large Datasets with Cypher

- The Neo4j Data Importer is only capable of handling small to medium datasets with max. 1 Mio. rows.
- Cypher queries can be optimized to run large import procedures
  - Using Subqueries issued with `CALL`
  - Plus running it `IN TRANSACTIONS`

### 4.4.2 Import Using Cypher

1. Clear graph

```
MATCH (u:User) DETACH DELETE u;
MATCH (p:Person) DETACH DELETE p;
MATCH (m:Movie) DETACH DELETE m;
MATCH (n) DETACH DELETE n
```

2. View constraints

```
SHOW CONSTRAINTS
```

3. Import the Movie and Genre data

```
CALL {
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/2-movieData.csv'
AS row
//process only Movie rows
WITH row WHERE row.Entity = "Movie"
MERGE (m:Movie {movieId: toInteger(row.movieId)})
ON CREATE SET
m.tmdbId = toInteger(row.tmdbId),
m.imdbId = toInteger(row.imdbId),
m.imdbRating = toFloat(row.imdbRating),
m.released = datetime(row.released),
m.title = row.title,
m.year = toInteger(row.year),
m.poster = row.poster,
m.runtime = toInteger(row.runtime),
m.countries = split(coalesce(row.countries,""), "|"),
m.imdbVotes = toInteger(row.imdbVotes),
m.revenue = toInteger(row.revenue),
m.plot = row.plot,
m.url = row.url,
m.budget = toInteger(row.budget),
m.languages = split(coalesce(row.languages,""), "|")
WITH m,split(coalesce(row.genres,""), "|") AS genres
UNWIND genres AS genre
WITH m, genre
MERGE (g:Genre {name:genre})
MERGE (m)-[:IN_GENRE]->(g)
}
```

4. Import Person data

```
CALL {
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/2-movieData.csv'
AS row
WITH row WHERE row.Entity = "Person"
MERGE (p:Person {tmdbId: toInteger(row.tmdbId)})
ON CREATE SET
p.imdbId = toInteger(row.imdbId),
p.bornIn = row.bornIn,
p.name = row.name,
p.bio = row.bio,
p.poster = row.poster,
p.url = row.url,
p.born = CASE row.born WHEN "" THEN null ELSE date(row.born) END,
p.died = CASE row.died WHEN "" THEN null ELSE date(row.died) END
}
```

5. Import the ACTED_IN Relationship

```
CALL {
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/2-movieData.csv'
AS row
WITH row WHERE row.Entity = "Join" AND row.Work = "Acting"
MATCH (p:Person {tmdbId: toInteger(row.tmdbId)})
MATCH (m:Movie {movieId: toInteger(row.movieId)})
MERGE (p)-[r:ACTED_IN]->(m)
ON CREATE
SET r.role = row.role
SET p:Actor
}
```

6. Import the DIRECTED Relationship

```
CALL {
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/2-movieData.csv'
AS row
WITH row WHERE row.Entity = "Join" AND row.Work = "Directing"
MATCH (p:Person {tmdbId: toInteger(row.tmdbId)})
MATCH (m:Movie {movieId: toInteger(row.movieId)})
MERGE (p)-[r:DIRECTED]->(m)
ON CREATE
SET r.role = row.role
SET p:Director
}
```

7. Impoirt the User data

```
CALL {
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/2-ratingData.csv'
AS row
MATCH (m:Movie {movieId: toInteger(row.movieId)})
MERGE (u:User {userId: toInteger(row.userId)})
ON CREATE SET u.name = row.name
MERGE (u)-[r:RATED]->(m)
ON CREATE SET r.rating = toInteger(row.rating),
r.timestamp = toInteger(row.timestamp)
}
```
