# Knowledge Graphs for RAG

This repository contains my personal notes, code, and supplementary materials for the course **[Knowledge Graphs for RAG](https://www.deeplearning.ai/short-courses/knowledge-graphs-rag/)**, provided by [DeepLearning.AI](https://www.deeplearning.ai/) in collaboration with [Neo4j](https://neo4j.com/).

The materials herein are for personal learning and demonstration purposes only.

-----

## Introduction

Knowledge graphs are a powerful tool used in development to structure complex data relationships, drive intelligent search functionality, and build robust AI applications that can reason over diverse data types. They excel at connecting data from both structured and unstructured sources (like databases and text documents), offering an
intuitive and flexible method to model complex, real-world scenarios.

Unlike traditional tables or simple lists, knowledge graphs capture the intrinsic meaning and context behind the data. This allows for the discovery of insights and connections that are often obscured in conventional databases. This rich, structured context is ideal for enhancing the output of Large Language Models (LLMs). By using a knowledge graph, you can build a far more relevant and factually-grounded context for the model than is possible with semantic search alone.

This course teaches how to leverage knowledge graphs within Retrieval Augmented Generation (RAG) applications.

**In this course, you will learn to:**

  * Understand the basics of how knowledge graphs store data using nodes (entities) and edges (relationships).
  * Use Neo4j’s query language, **Cypher**, to retrieve information from a sample graph of movie and actor data.
  * Add a vector index to a knowledge graph to represent unstructured text data and perform vector similarity searches.
  * Build a knowledge graph from scratch using unstructured text, based on publicly available financial (SEC) documents.
  * Explore advanced techniques for connecting multiple knowledge graphs and using complex queries for comprehensive data retrieval.
  * Write advanced Cypher queries to retrieve relevant information from the graph and format it for inclusion in an LLM prompt.

-----

## Course Topics

Below is a detailed breakdown of each module covered in the course.

<details>
<summary><h3>1. Knowledge Graph Fundamentals</h3></summary>

This foundational module introduces the core concepts of graph databases and knowledge graphs (KGs). Before one can build or query a graph, it's essential to understand the "why" and "how" of this data modeling paradigm. This section moves beyond the familiar world of relational databases (tables, rows, and columns) and into a more flexible and intuitive way of representing data that mirrors the real world.

The central components, as outlined in the course description, are **nodes** and **edges** (also called relationships).

  * **Nodes** are used to represent **entities**. These are the "nouns" of your dataset. In the context of the course, an entity could be a `Person` (like an actor or a CEO), a `Movie`, a `Company` (like Apple Inc.), or a `Document` (like a 10-K filing). Nodes are typically assigned **Labels** to categorize them (e.g., `:Person`, `:Company`).
  * **Edges** (or **Relationships**) represent the **connections** between nodes. These are the "verbs" that link your entities. An edge has a direction and a **Type**. For example, a `(:Person)` node might have an `[:ACTED_IN]` relationship pointing to a `(:Movie)` node. A `(:Company)` node might have a `[:LED_BY]` relationship pointing to a `(:Person)` node.

This module will almost certainly delve into **Properties**, which are key-value pairs of data stored on *both* nodes and edges. A `:Person` node could have properties like `name: "Tom Hanks"` or `born: 1956`. A `:Company` node could have `name: "Apple Inc."` and `stock_ticker: "AAPL"`. Even relationships can have properties; for instance, the `[:ACTED_IN]` relationship could have a `role: "Forrest Gump"` property.

The real power of this model, and a key focus of this lesson, is its departure from rigid SQL schemas. In a relational database, modeling complex, many-to-many relationships (like actors, movies, and directors) requires multiple tables and expensive `JOIN` operations. In a graph, these relationships are "first-class citizens." They are stored directly, making queries about connections—like "find all actors who worked with the same director as Tom Hanks"—incredibly fast and intuitive.

For RAG, this is the entire point. An LLM's "context" is often shallow. A typical semantic search might find a document *mentioning* "Apple Inc." A graph-based retrieval, which this module sets the foundation for, can retrieve "Apple Inc., which is headquartered in Cupertino, led by CEO Tim Cook, and recently filed its Q4 10-K report." This is the "rich, structured context" the course description highlights. This module teaches the "alphabet" of this new data language—the nodes, labels, relationships, and properties—that allows you to build such a powerful context. It establishes the "property graph model" used by Neo4j and provides the mental model needed for all subsequent lessons.

</details>

<details>
<summary><h3>2. Querying Knowledge Graphs</h3></summary>

Once you understand *what* a knowledge graph is, the next logical step is to learn how to *ask it questions*. This module transitions from theory to practice by introducing **Cypher**, the declarative query language for property graphs, most notably used by Neo4j. Cypher is to graph databases what SQL is to relational databases.

The course description explicitly states this module will "Use Neo4j’s query language, Cypher, to retrieve information from a fun graph of movie and actor data." This "movie graph" is a classic teaching dataset because its connections are intuitive (actors act in movies, directors direct movies, actors may know each other).

The defining feature of Cypher, which will be the core of this lesson, is its "ASCII-art" style. It allows you to *visually describe* the data patterns you're looking for.

  * Nodes are represented by parentheses: `(n)`
  * Relationships are represented by arrows: `-[:REL_TYPE]->`

A complete pattern to find all movies Tom Hanks acted in would look like this:
`MATCH (actor:Person {name: "Tom Hanks"})-[:ACTED_IN]->(movie:Movie)`
`RETURN movie.title, movie.released`

This query is highly readable: "Match a node labeled 'Person' with the name 'Tom Hanks', which has an 'ACTED\_IN' relationship to any node labeled 'Movie'. Return the title and release year of that movie."

This module will likely cover the primary Cypher clauses:

  * **`MATCH`**: The `SELECT` of the graph world. Used to specify the *pattern* of nodes and relationships to find.
  * **`WHERE`**: Used to filter results, just like in SQL (e.g., `WHERE movie.released > 2000`).
  * **`RETURN`**: Specifies what data to return (e.g., node properties, entire nodes, or counts).
  * **`CREATE` / `MERGE`**: Used to write data. `CREATE` always makes new data, while `MERGE` is an "upsert"—it finds a pattern or creates it if it doesn't exist. `MERGE` is critical for building KGs from documents, as you want to avoid creating duplicate nodes for "Apple" every time it's mentioned.
  * **`SET` / `REMOVE`**: Used to add, update, or remove properties and labels.
  * **Aggregation**: Clauses like `COUNT()` and `collect()` will be introduced to answer questions like "How many movies has each actor been in?"

The real power, especially for RAG, comes from "pathfinding" queries. This module will likely demonstrate "variable-length paths." A query like `MATCH (kevin:Person {name: "Kevin Bacon"})-[:ACTED_IN*1..6]-(other:Person)` can find all actors within "six degrees" of Kevin Bacon. This ability to traverse complex, multi-hop relationships is something graph databases excel at and is nearly impossible for relational databases or vector search alone. This skill is the "R" (Retrieval) in RAG. Before you can "augment" an LLM, you must retrieve the data. This module provides the essential tool—Cypher—to perform that precise, relationship-aware retrieval.

</details>

<details>
<summary><h3>3. Preparing Text for RAG</h3></summary>

This module marks a critical pivot. Modules 1 and 2 dealt with clean, *structured* graph data (like the movie database). The real world, however, is dominated by *unstructured* data: PDFs, emails, web pages, and, in this course's example, financial documents. This lesson tackles how to bridge the gap between messy, unstructured text and the structured/semi-structured world of RAG.

The course description highlights the two-pronged approach: "Add a **vector index** to a knowledge graph" and use "vector **similarity search**." This module introduces the concept that a modern RAG system needs *both* structured graph queries (Cypher) and unstructured semantic search (vectors).

The first step in "preparing text" is **Text Chunking**. You cannot simply take a 200-page 10-K filing and treat it as one "thing." To be useful for RAG, it must be broken down into smaller, semantically coherent pieces, such as paragraphs, sections, or even sentences. This is a non-trivial process, as the way you chunk your text (e.g., by fixed size, by sentence, or using recursive splitters) has a huge impact on retrieval quality.

The second step is **Embedding Generation**. Once you have these text chunks, this module will show how to feed them into an embedding model (e.g., from OpenAI, Cohere, or a self-hosted Hugging Face model). This model converts each text chunk into a "vector embedding"—a long list of numbers (e.g., 1536 dimensions) that mathematically represents the chunk's *semantic meaning*. Chunks with similar meanings will have vectors that are "close" to each other in this high-dimensional space.

The third step, and the key to this module, is **Vector Indexing**. These vectors must be stored in a specialized database that can perform *fast* similarity searches. The course description states, "Add a vector index to a knowledge graph." This is a key feature of modern databases like Neo4j. Instead of having a separate graph database *and* a separate vector database, you can store the vector embedding *as a property on a node* in the graph.

For example, you might create a node:
`(:Chunk {text: "...", vector: [0.12, 0.45, ..., -0.8]})`

You can then create a **vector index** on the `vector` property. When a user asks a question, "What were the company's main risks last quarter?", that *question* is also embedded into a vector. The system can then perform a **vector similarity search** (like cosine similarity) against the index to instantly find the `Chunk` nodes whose `vector` property is semantically closest to the question's vector.

This module provides the "semantic search" half of the RAG equation. It's how you find relevant *prose* and *passages* from your documents. The RAG application will later combine this retrieved unstructured text with the structured data retrieved via Cypher (from Module 2) to build a comprehensive, fact-rich prompt for the LLM.

</details>

<details>
<summary><h3>4. Constructing a Knowledge Graph from Text Documents</h3></summary>

This module is arguably the most complex and powerful part of the course. It addresses the fundamental challenge: "How do I get a structured graph from a pile of unstructured text?" Module 3 showed how to *index* text chunks for semantic search. This module shows how to *extract* structured nodes and relationships *from* that text to build the graph itself.

The course description specifies the use case: "Build a knowledge graph of text documents from scratch, using publicly available **financial and investment documents** as the demo use case." These "SEC documents" (like 10-K annual reports or 10-Q quarterly reports) are ideal because they are dense with entities (companies, people, products) and relationships (executives *work for* companies, companies *report* financial results, companies *list* competitors).

The core process taught here is **Information Extraction**, which is supercharged by modern LLMs. The pipeline will likely look like this:

1.  **Input:** A text chunk (from Module 3) e.g., "Apple Inc., headquartered in Cupertino and led by CEO Tim Cook, reported record revenue of $394 billion in 2022."
2.  **LLM Prompting:** This is the magic. You don't write complex regular expressions. Instead, you use an LLM (like GPT-4) with a carefully engineered prompt. This prompt instructs the LLM to act as a data extractor. For example: "You are an expert financial analyst. From the following text, extract all entities of type `Company`, `Person`, `Location`, or `Metric`. Then, identify all relationships between them, such as `IS_HEADQUARTERED_IN`, `IS_CEO_OF`, or `REPORTED_REVENUE`. Output your answer in a structured JSON format with a list of 'nodes' and 'edges'."
3.  **LLM Output (JSON):** The LLM would return a JSON object like:
    ```json
    {
      "nodes": [
        {"id": "Apple Inc.", "label": "Company"},
        {"id": "Cupertino", "label": "Location"},
        {"id": "Tim Cook", "label": "Person"},
        {"id": "$394 billion", "label": "Metric", "type": "Revenue", "year": 2022}
      ],
      "edges": [
        {"from": "Apple Inc.", "to": "Cupertino", "type": "IS_HEADQUARTERED_IN"},
        {"from": "Apple Inc.", "to": "Tim Cook", "type": "LED_BY", "role": "CEO"},
        {"from": "Apple Inc.", "to": "$394 billion", "type": "REPORTED"}
      ]
    }
    ```
4.  **Graph Ingestion:** This module will then show how to write a script (likely in Python) that parses this JSON output. This script will connect to the Neo4j database and execute a series of Cypher **`MERGE`** commands. `MERGE` is essential here (as learned in Module 2) because it prevents duplicate data. It means "find this node or create it."
      * `MERGE (c:Company {name: "Apple Inc."})`
      * `MERGE (p:Person {name: "Tim Cook"})`
      * `MERGE (c)-[:LED_BY {role: "CEO"}]->(p)`

This module is the "construction" phase. It's a repeatable, automated pipeline: `Document -> Chunk -> LLM (Extract) -> JSON -> Cypher (Ingest) -> Graph`. By the end of this lesson, you will have learned how to transform a directory of flat, text-based SEC filings into a rich, interconnected, and queryable knowledge graph, forming the "brain" of your RAG application.

</details>

<details>
<summary><h3>5. Adding Relationships to the SEC Knowledge Graph</h3></summary>

At first glance, this topic might seem redundant with Module 4, which was also about construction. However, this module addresses a more advanced and subtle set of problems related to **graph enrichment**, **entity resolution**, and **inference**. Building a graph is not a "one-pass" task. The initial graph from Module 4 will be good, but it will also be "sparse" and contain disconnected pockets of information. This module is about making that graph *denser* and *smarter*.

The first challenge this module will likely address is **Entity Resolution (or Disambiguation)**. Your LLM extraction (from Module 4) might create a `(:Company {name: "Apple Inc."})` node from one document and a `(:Company {name: "Apple"})` node from another. From a human perspective, these are the same, but in the graph, they are two separate nodes. This module will teach techniques to *resolve* these. For example, you might run a post-processing query that merges nodes with similar names or that share a unique identifier (like a stock ticker, if one was extracted).

The second, and more complex, topic is **Relationship Inference**. Module 4 focused on *explicit* relationships found in a single sentence ("Tim Cook is CEO of Apple"). But what about *implicit* relationships that span multiple paragraphs or documents?

  * **Coreference Resolution:** A document might say, "Apple launched the iPhone. *The company's* flagship product..." The LLM from Module 4 might not realize that "*The company*" refers to "Apple." This module could introduce techniques (perhaps another LLM prompt) specifically designed to find these "coreferences" and add the missing relationships, like `(iPhone)-[:IS_PRODUCT_OF]->(Apple)`.
  * **Inferring New Connections:** Imagine your graph has `(Tim Cook)-[:IS_CEO_OF]->(Apple)` and `(Jeff Williams)-[:IS_COO_OF]->(Apple)`. From this, you can *infer* a new relationship: `(Tim Cook)-[:WORKS_WITH]->(Jeff Williams)`. This module might teach how to run Cypher queries that find these "triangular" patterns and write the new, inferred relationship back into the graph.

This is particularly critical for the SEC use case. One filing (a 10-K) might list a person as a board member. A different filing (a Form 4) might show that person selling stock. A *third* filing (a proxy statement) might detail their committee assignments. The initial extraction might create three disconnected "fact clusters." This module is about the "glue." It teaches the advanced Cypher queries and data integration strategies needed to *link* these clusters together, ensuring the `(:Person)` node from the 10-K is the *same* `(:Person)` node as the one in the Form 4.

This process transforms the graph from a simple collection of extracted facts into a true *knowledge* base, where the *connections* between facts generate new, emergent insights. For RAG, this means your context will be far richer, allowing you to answer complex, multi-hop questions like, "Which board members, who also sit on the audit committee, sold stock last quarter?"

</details>

<details>
<summary><h3>6. Expanding the SEC Knowledge Graph</h3></summary>

This module takes the graph built in Modules 4 and 5 and scales it *outward*. So far, the knowledge graph has been built from a single, homogenous data source: SEC filings. But a truly powerful RAG system needs to reason over *all* available data. This lesson, as the course description says, is about "advanced techniques for **connecting multiple knowledge graphs**" and integrating heterogeneous data sources.

This is the **data integration** and **federation** phase. The module will likely explore several scenarios:

1.  **Integrating Structured Data:** This is the most straightforward. Your `SEC_Graph` has a `(:Company {name: "Apple Inc."})` node. You also have a simple CSV file or a SQL database table that maps company names to stock tickers. This module will show how to write a script (e.g., Python with `pandas` and the `neo4j` driver) that reads this structured file and *enriches* the existing graph. It would run a Cypher query like: `UNWIND $csv_rows AS row MERGE (c:Company {name: row.company_name}) SET c.ticker = row.ticker`. This *adds properties* to your existing nodes, making them more complete.

2.  **Integrating Other Unstructured Data:** What about news articles, press releases, or even social media posts about these companies? This module would show how to run the *same* extraction pipeline (from Module 4) on this *new* data source. The `MERGE` command becomes absolutely critical here. When the extractor finds "Apple" in a news article, the `MERGE (c:Company {name: "Apple"})` query will *find* the existing node (created from the SEC filing) and add the new relationships (e.g., `(c)-[:MENTIONED_IN]->(article:NewsArticle {url: "..."})`) without creating a duplicate company. This connects your formal SEC data to real-time market news.

3.  **Connecting Multiple Knowledge Graphs:** This is the most advanced concept. What if you have *two separate databases*? Your `SEC_Graph` (private, internal) and a large, *public* knowledge graph like Wikidata (which has data on millions of companies, people, and products).

      * **Physical Merge:** You could extract data *from* Wikidata and ingest it into your Neo4j graph, linking your "Apple" node to the Wikidata "Apple" node.
      * **Virtual/Federated Query:** This is a more complex but powerful idea. Neo4j has a tool called Fabric that allows a *single* Cypher query to run *across multiple, independent databases*. This module might introduce this concept, where you can "join" your internal `SEC_Graph` with an external `Market_Data_Graph` on the fly, without having to physically copy all the data.

For a RAG application, this is the key to answering *truly* complex questions. A user might ask, "How did the market react to the risks mentioned in Apple's last 10-K?"
To answer this, the RAG system needs to:

1.  Query the `SEC_Graph` for "risks" (from Module 4/5).
2.  Query the `News_Graph` for "market reaction" (from this module).
3.  Query the `Market_Data_Graph` for "stock price" (from this module).

This module teaches you how to build this comprehensive, multi-source "knowledge-base-of-knowledge-bases" that provides the holistic context needed for a state-of-the-art AI assistant.

</details>

<details>
<summary><h3>7. Chatting with the Knowledge Graph</h3></summary>

This is the final "payoff" module where all the previous concepts—graph fundamentals, Cypher querying, vector search, and graph construction—are assembled into a single, working **RAG application**. This is the "chat" interface that interacts with the knowledge graph.

The course description highlights the two key challenges: "Write **advanced Cypher queries** to retrieve relevant information" and "**format it for inclusion in your prompt to an LLM**." This module is all about building the *application logic* that sits between the user and the LLM.

The core workflow of a "KG-RAG" application, which this module will teach, looks like this:

1.  **User Input:** A user types a natural language question: "Who is the CEO of Apple, and what was their compensation last year?"
2.  **Step 1: NL-to-Cypher (Query Generation):** This is the first "magic" step. The application *cannot* just vector-search this question. It needs to *understand* the user's *intent* and **translate the question into a Cypher query**. This is a "Text-to-Cypher" task, and it's typically solved using... another LLM call\!
      * **Meta-Prompt:** The application will have a "meta-prompt" that it sends to an LLM (like GPT-4). This prompt contains:
          * The **graph schema** (e.g., "I have nodes `(:Person)`, `(:Company)`, `(:CompensationPackage)`. I have relationships `[:IS_CEO_OF]` and `[:RECEIVES]`. The `CompensationPackage` node has a `year` property...").
          * The **user's question**.
      * **LLM Task:** "Given the schema above, generate the precise Cypher query needed to answer this question: 'Who is the CEO of Apple, and what was their compensation last year?'"
      * **LLM Response:** The LLM would *return* a string of Cypher code:
        `MATCH (p:Person)-[:IS_CEO_OF]->(c:Company {name: "Apple Inc."})`
        `MATCH (p)-[:RECEIVES]->(comp:CompensationPackage {year: 2023})`
        `RETURN p.name, comp.total_value`
3.  **Step 2: Retrieval (The "R"):** The application (e.g., a Python, LangChain, or LlamaIndex app) takes this LLM-generated Cypher query and executes it against the Neo4j database (built in Modules 4-6).
      * **DB Result (Structured Context):** The database returns a structured result, like `[{"name": "Tim Cook", "total_value": "$49,000,000"}]`.
4.  **Step 3: Augmentation (The "A"):** The application now *augments* a *final* prompt for the *chat* LLM. This is the "formatting for inclusion" step.
      * **Final Prompt:**
        "You are a helpful financial assistant. Answer the user's question based *only* on the following context.
          * User Question: 'Who is the CEO of Apple, and what was their compensation last year?'
          * Retrieved Context from Knowledge Graph: 'The CEO's name is Tim Cook. The compensation for 2023 was $49,000,000.'
          * Answer:"
5.  **Step 4: Generation (The "G"):** This final, augmented prompt is sent to the LLM.
      * **LLM's Final Answer:** The LLM, now "grounded" by the facts from the graph, generates a safe and accurate response: "The CEO of Apple is Tim Cook, and his total compensation for 2023 was $49 million."

This module teaches you how to build this entire end-to-end "Text-to-Cypher-to-Text" pipeline. It combines LLMs-as-reasoners (Step 2) with LLMs-as-synthesizers (Step 5), using the Knowledge Graph as the verifiable, factual "brain" in the middle.

</details>

-----

## Acknowledgement

This repository is for personal, educational use only. All course materials, content, and intellectual property are owned by **DeepLearning.AI** and **Neo4j**. The rights and licenses for the original course content are held exclusively by them.

This repository is intended as a personal learning log and to demonstrate the skills acquired from the course, not to replace or distribute the official course material.
