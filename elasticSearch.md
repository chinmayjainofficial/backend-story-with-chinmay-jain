# Full-Text Search & Elasticsearch

> Source: video transcript on why SQL `LIKE` search breaks down, how inverted indexes work, and why tools like Elasticsearch exist.

---

## Table of Contents

1. [The SQL Search Problem](#1-the-sql-search-problem)
2. [Requirements for Better Search](#2-requirements-for-better-search)
3. [Database as a Librarian (Analogy)](#3-database-as-a-librarian-analogy)
4. [The Information Explosion](#4-the-information-explosion)
5. [Inverted Index: The Key Idea](#5-inverted-index-the-key-idea)
6. [Apache Lucene and Elasticsearch](#6-apache-lucene-and-elasticsearch)
7. [Relevance Scoring (BM25)](#7-relevance-scoring-bm25)
8. [Factors That Affect Relevance](#8-factors-that-affect-relevance)
9. [Elasticsearch Use Cases](#9-elasticsearch-use-cases)
10. [Demo: Postgres vs Elasticsearch](#10-demo-postgres-vs-elasticsearch)
11. [Benchmark Results](#11-benchmark-results)
12. [Final Takeaways](#12-final-takeaways)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. The SQL Search Problem

### The scenario

- Imagine it is **2005**. You are a software engineer at a fast-growing **e-commerce company**.
- Your task: build an API that lets users **search ~5,000 products**.

### The basic solution

```sql
SELECT * FROM products
WHERE name ILIKE '%laptop%'
   OR description ILIKE '%laptop%';
```

**What this means:**

- `%` is a wildcard. It matches **any characters** before or after the word.
- `LIKE` / `ILIKE` = pattern matching (`ILIKE` is **case-insensitive**).
- If "laptop" appears in the **name OR description**, that row is returned.

### Why it works at first

- With only 5,000 products, results come back quickly.
- Customers search "laptop", get a few results, and **life is simple**.

---

## 2. Requirements for Better Search

### What changed

- The company grew rapidly, and now has **millions of products**.
- The simple `LIKE` query that took **~50 ms** now takes **~30 seconds**.
- Customers, managers, and everyone else are frustrated. You are asked to fix it.

### New requirements (not just "make it faster")

| Requirement        | Meaning                              | Example                                                                  |
| ------------------ | ------------------------------------ | ------------------------------------------------------------------------ |
| **Speed**          | Results in milliseconds              | 50 ms, not 30 s                                                          |
| **Relevance**      | Show the _most useful_ results first | Searching "laptop" should show a **MacBook Pro** before a **laptop bag** |
| **Typo tolerance** | Handle spelling mistakes             | A user types "lapotp" and still gets laptop results                      |

> **Why typos matter:** During sales, customers are in a hurry and type fast, so typos are frequent. The search should not break.

### The result

These requirements are the **reason search engines like Elasticsearch were created**.

---

## 3. Database as a Librarian (Analogy)

Think of your relational database (e.g. **PostgreSQL**) as a **librarian**.

- The librarian knows where every book is.
- But it has **one fatal flaw**: to find a book by _content/title search_, it must **check every single book on every shelf, one by one**.

### Example: searching for "machine learning"

1. Check book 1: _Harry Potter and the Philosopher's Stone_. No match, move on.
2. Check book 2: _Game of Thrones_. No match, move on.
3. Check book 3: _Introduction to Machine Learning_. **Match!**
4. ...keep going through **every other book**.

For a library with 10 million or 1 billion books, this could take **minutes, hours, or even days**.

### Two big problems

#### Problem 1: Speed

- It scans **everything**, so time grows with data size.
- This is called a **full scan**.

#### Problem 2: No concept of relevance

Suppose the librarian finds two books:

- **Book A:** title is _Introduction to Machine Learning_. Highly relevant.
- **Book B:** a different topic, with "machine learning" mentioned once on the last page. Barely relevant.

The librarian can return them **in any order**. Book B could even come first. You'd actually prefer to **skip Book B entirely**.

---

## 4. The Information Explosion

### How `ILIKE '%term%'` works internally

- The database **scans every row**.
- It examines **every text field**.
- It does **character-by-character pattern matching**.

**Result:**

- Thorough: it finds all matches.
- Painfully slow.
- No ranking. Results come back in effectively **random order**, so the best result might be at position 1,000 or 10,000.

### Historical context (~2005)

- Websites, servers, and databases were **exploding in number**.
- **Google** was crawling and indexing **billions of web pages**.
- **Amazon** was cataloging **millions of products**.
- **LinkedIn** was indexing **millions of profiles**.

### Why speed is critical

- Waiting 30 seconds means **customers leave**.
- Even **2 seconds** of delay is considered huge today.
- Search must respond in **milliseconds**.
- Slow search directly hurts **conversions and user base** (and revenue).

### The solution came from research

- Text search and **information retrieval** were not new problems.
- Computer scientists had studied them **since the 1960s**.
- The answer came from decades of research.

---

## 5. Inverted Index: The Key Idea

### The core idea

Instead of **going through every book to look for words**, what if we **flip the problem**?

> **Normal way:** Book → words inside it (scan to find a term)
> **Inverted way:** Word → list of books (and locations) where it appears

This is the **inverted index**, the key invention that changed text search forever.

### How it works (simple version)

When a book **first arrives on the shelf** (i.e., when data is stored):

1. Take **all the words** in the book.
2. Build an **index** that maps each word to **where it appears**: which book, and which pages.

### Example inverted index

**Word: `machine`**

| Book                             | Pages where "machine" appears |
| -------------------------------- | ----------------------------- |
| Introduction to Machine Learning | 1, 15, 23                     |
| The Machine Age                  | 5, 89                         |
| Coffee Machine Manual            | 1                             |

**Word: `learning`**

| Book                             | Pages     |
| -------------------------------- | --------- |
| Introduction to Machine Learning | 1, 16, 24 |
| Learning to Cook                 | 2 places  |
| Deep Learning Fundamentals       | 3 places  |

> _Note:_ In real life, words like "machine" and "learning" would appear **hundreds of times** in a book called _Introduction to Machine Learning_. The small numbers are just for the example.

### Why it is called "inverted"

- Before: we go through **content** to find **terms**.
- Now: we have the **terms**, and through the terms we find the **content**.
- We **inverted** the search.

### Why this is fast

- To search "machine learning", the librarian just **looks up "machine" and "learning" in the index**.
- No need to scan every book.
- Think of it like the **index at the back of a textbook**.

---

## 6. Apache Lucene and Elasticsearch

- The inverted index technique **powers Elasticsearch**.
- Elasticsearch is built on **Apache Lucene**, the core **inverted-index-based text search technology**.
- Elasticsearch is **not a brand-new invention**. It builds on Lucene.

### Other tools

- Elasticsearch is **not the only** full-text search tool.
- Many other tools also use **Apache Lucene** underneath.
- Even relational databases like **PostgreSQL** now support **full-text search**.

### Two advantages of an inverted index

1. **Speed:** direct lookup instead of full scan.
2. **Relevance:** the index stores _where_ and _how often_ a word appears, so results can be **ranked**.

---

## 7. Relevance Scoring (BM25)

### What is relevance scoring?

Elasticsearch has a built-in feature called **relevance scoring**. It decides **which results are most meaningful** and shows them first.

### Example: searching "machine"

| Book                                 | Why it ranks where it does                                                                     |
| ------------------------------------ | ---------------------------------------------------------------------------------------------- |
| **Introduction to Machine Learning** | Term is **in the title** (big boost) + appears **many times** (another big boost) → **Rank 1** |
| **The Machine Age**                  | Term is **in the title**, but appears **less frequently** → **Rank 2**                         |
| Others                               | In the title but **very rarely used** → lower ranks                                            |

> **Key point:** We don't want _any_ result that matches. We want the **most meaningful** result, and Elasticsearch gives you that out of the box.

### The BM25 algorithm

- Elasticsearch uses an algorithm called **BM25** for scoring.
- There is a lot of **math/theory** behind it.

### Practical advice from the video

- You **don't need to understand the theory** unless you are building a search library or an Elasticsearch alternative.
- Treat it as a **tool**:
  1. Know the use case ("I need full-text search / relevance").
  2. Pick Elasticsearch (or Postgres full-text search).
  3. Read the docs, implement, and move on.
- If you're **curious**, the Elasticsearch docs are a great place to learn more.

---

## 8. Factors That Affect Relevance

BM25 uses several parameters to sort results.

### 1. Term Frequency (TF)

- **How often** a term (e.g., "machine") appears **in a single document**.
- More appearances usually means more relevant.

### 2. Document Frequency (DF)

- **How common** the term is **across all documents**.
- Very common words are less useful for distinguishing documents.

> **Easy way to remember:**
>
> - **Term frequency** = in **one** document
> - **Document frequency** = across **all** documents

### 3. Document Length

- How **long** the document is (short vs. long, e.g. a short note vs. a big book).
- Length is taken into account so long documents don't unfairly dominate.

### 4. Field Boosting

Some **fields** matter more than others.

```
Title match  >  Description match  >  Content match
```

- A term in the **title** is more relevant than in the **description**, which is more relevant than in the **content**.
- You can **customize** these boosts in your query. It's **not fixed**. If you want content to matter more, you can change it.

### Elasticsearch basics

- A single entity (like one book) is called a **document**.
- A document is a **JSON document**, similar to MongoDB.
- Queries are written in the **Elasticsearch DSL**, a **JSON-based query language** with many parameters for different kinds of search.

---

## 9. Elasticsearch Use Cases

### 1. Typo tolerance

- Example: Google search. Type **"what is treading today"** (typo!).
- Google understands the **likely intended query**: _"what is trending today"_.
- Full-text search tools handle this using **context**.
- _(The video says it's unclear whether Google uses Elasticsearch or its own tech built on Lucene, but it's a technology **similar to Elasticsearch**.)_

### 2. Type-ahead / autocomplete

- As you type "what is...", suggestions appear instantly.
- Seen on sites like **Amazon** and **Google**.
- Can be built with Elasticsearch.

### 3. Log management: the ELK Stack

- Elasticsearch is also very famous in **log management**.
- **ELK Stack** =
  - **E**lasticsearch (storage + search)
  - **L**ogstash (log collection/processing)
  - **K**ibana (visualization)
- Elasticsearch is **very fast** at searching logs, deriving **statistics**, and visualizing data.

### Postgres or Elasticsearch? Which to choose?

| Situation                                                             | Choice                                               |
| --------------------------------------------------------------------- | ---------------------------------------------------- |
| Company already uses **Elasticsearch** (e.g., for ELK log management) | Using Elasticsearch for full-text search makes sense |
| Simple search needs, already on Postgres                              | **Postgres full-text search** is a good option       |

---

## 10. Demo: Postgres vs Elasticsearch

### Goal

Compare a **traditional database search** (`ILIKE`) vs an **Elasticsearch search** on the same data.

### Setup

- Built with **Next.js** (only because LLMs are good at generating prototype apps with it; not a requirement).
- Compares both databases **side by side** in the UI.

### Fair-test conditions

- **PostgreSQL:** hosted on **Neon** (serverless cloud Postgres).
- **Elasticsearch:** hosted on **Elastic Cloud**.
- Both are in the **US-West region**, so **network distance isn't a big factor**.

### Data

- Postgres table `reviews` with 3 fields:
  - `id`
  - `review` (text)
  - `sentiment` (positive/negative)
- Source: a **CSV file with ~50,000 reviews** (review text + sentiment).

---

## 11. Benchmark Results

### How the demo's code works

#### Populate script (Node.js)

1. Read the database URL, Elasticsearch address, and API key from **environment variables** (throws an error if any are missing).
2. Initialize the **Neon client** and **Elasticsearch client**.
3. Read the CSV file (`readFileSync`).

**For Postgres:**

- Create the table if it doesn't exist.
- Reset the ID field to start fresh.
- Filter out rows missing **review** or **sentiment**.
- Insert in **batches of 1,000** (inserting everything at once hits a Neon limitation).

**For Elasticsearch:**

- Check whether the `reviews` index exists, and **delete it** if so (to start fresh).
- Create the index with field **mappings**:
  - `review` → type **`text`** (analyzed and split into words, used for full-text search)
  - `sentiment` → type **`keyword`** (matched **exactly**, no word splitting)
- Filter out invalid records.
- **Bulk insert** all 50,000 records.

**Verification:** 50,000 rows in Postgres (`SELECT COUNT(*) FROM reviews`) and 50,000 documents in Elasticsearch.

#### The search API route

- Receives the search term from JSON (returns **bad request** if invalid).
- Uses **streaming** so each database's result is sent to the frontend **as soon as it's ready**. This way, one doesn't wait for the other, and you can see which is faster.

**Postgres query:**

```sql
SELECT id, review, sentiment FROM reviews
WHERE review ILIKE '%searchTerm%';
```

**Elasticsearch query:**

- Runs a **query string** search on the `reviews` index.
- Converts the search term to **lowercase** for a fair comparison.
- Uses wildcard-style matching on both sides.

> Search criteria were kept **equal** for fairness: lowercase + match anything before/after.

### Results

| Search term                  | Elasticsearch | Postgres (`ILIKE`)  |
| ---------------------------- | ------------- | ------------------- |
| `laptop`                     | ~**1 second** | ~**3 to 4 seconds** |
| `something` (~8,000 results) | ~**500 ms**   | ~**7.5 seconds**    |

**Observations:**

- Both returned the **same number of results** (~8,000), so the comparison is fair.
- Elasticsearch was **significantly faster** (~500 ms vs ~7.5 s).
- Even on a small dataset (50K rows), the gap is large. At millions of rows, it gets far worse.

---

## 12. Final Takeaways

1. **For search / type-ahead features, use full-text search.**
   - Options: **Postgres full-text search** or a tool like **Elasticsearch**.
2. **Elasticsearch is something to have in your arsenal as a backend engineer**, but you **don't need to master it**.
   - Copy-pasting snippets from docs or LLMs covers **most use cases**.
   - Read more only when you need to **optimize**.
3. **Mastering databases is far more important.**
   - Learn how to **work with** and **optimize** databases, and **understand indexes** deeply.
   - Databases are involved in almost **99% of a backend engineer's codebase**.
4. **Don't over-invest in theory** (like BM25 math) unless you are building search tools yourself. Know the use case, use the tool, follow best practices.

---

## 13. Quick Revision Cheat Sheet

| Concept            | One-line summary                                                                        |
| ------------------ | --------------------------------------------------------------------------------------- |
| `LIKE '%x%'`       | Scans every row, slow, no ranking                                                       |
| Librarian analogy  | DB checks every book one by one (full scan)                                             |
| Relevance          | Showing the _most useful_ results first                                                 |
| Typo tolerance     | Finding the right result even with spelling mistakes                                    |
| **Inverted index** | Map each **word → documents/locations** instead of scanning documents                   |
| Apache Lucene      | Core library behind inverted-index-based search (used by Elasticsearch)                 |
| Elasticsearch      | Fast search engine built on Lucene; stores JSON documents                               |
| BM25               | The relevance-scoring algorithm Elasticsearch uses                                      |
| Term Frequency     | How often a word appears in **one** document                                            |
| Document Frequency | How common a word is across **all** documents                                           |
| Document Length    | Length of the document is factored into scoring                                         |
| Field Boosting     | Title match counts more than description, which counts more than content (customizable) |
| Elasticsearch DSL  | JSON-based query language for Elasticsearch                                             |
| `text` type        | Analyzed field, used for full-text search                                               |
| `keyword` type     | Exact-match field                                                                       |
| ELK Stack          | Elasticsearch + Logstash + Kibana, for log management                                   |
| Postgres FTS       | Built-in full-text search alternative to Elasticsearch                                  |
| Demo result        | Elasticsearch ~500 ms vs Postgres `ILIKE` ~7.5 s on 50K rows                            |

### Mini decision guide

```
Need search / autocomplete / typo tolerance / relevance?
│
├── Simple needs, already on Postgres  →  Postgres Full-Text Search
│
└── Heavy search, or already using ELK →  Elasticsearch
```
