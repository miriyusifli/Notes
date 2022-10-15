# Chapter 1: Reliable, Scalable, and Maintainable Applications
We focus on three concerns that are important in most software systems:
- Reliability: The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).
- Scalability: As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.
- Maintainability: Over time, many different people will work on the system (engineering and oper‐ ations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.


## Reliability
For software, typical expectations include:
- The application performs the function that the user expected.
- It can tolerate the user making mistakes or using the software in unexpected ways.
- Its performance is good enough for the required use case, under the expected load and data volume.
- The system prevents any unauthorized access and abuse.

We can understand relia‐ bility as meaning, roughly, “continuing to work correctly, even when things go wrong.” 

Note that a fault is not the same as a failure. A fault is usually defined as one component of the system deviating from its spec, whereas a failure is when the system as a whole stops providing the required service to the user. It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures. 

## Scalability
First, we need to succinctly describe the current load on the system; only then can we discuss growth questions (what happens if our load doubles?). Load can be described with a few numbers which we call load parameters. The best choice of parameters depends on the architecture of your system: it may be requests per second to a web server, the ratio of reads to writes in a database, the number of simultaneously active users in a chat room, the hit rate on a cache, or something else.

It’s preferable to do more work at write time and less at read time.


The final twist of the Twitter anecdote: now that approach 2 is robustly implemented, Twitter is moving to a hybrid of both approaches. Most users’ tweets continue to be fanned out to home timelines at the time when they are posted, but a small number of users with a very large number of followers (i.e., celebrities) are excepted from this fan-out. Tweets from any celebrities that a user may follow are fetched separately and merged with that user’s home timeline when it is read, like in approach 1. This hybrid approach is able to deliver consistently good performance. We will revisit this exam‐ ple in Chapter 12 after we have covered some more technical ground.

Usually it is better to use percentiles to describe performance using response time. If you take your list of response times and sort it from fastest to slowest, then the median is the halfway point: for example, if your median response time is 200 ms, that means half your requests return in less than 200 ms, and half your requests take longer than that.

In order to figure out how bad your outliers are, you can look at higher percentiles: the 95th, 99th, and 99.9th percentiles are common (abbreviated p95, p99, and p999). They are the response time thresholds at which 95%, 99%, or 99.9% of requests are faster than that particular threshold. For example, if the 95th percentile response time is 1.5 seconds, that means 95 out of 100 requests take less than 1.5 seconds, and 5 out of 100 requests take 1.5 seconds or more. 

High percentiles of response times, also known as tail latencies, are important because they directly affect users’ experience of the service. For example, Amazon describes response time requirements for internal services in terms of the 99.9th per‐ centile, even though it only affects 1 in 1,000 requests. This is because the customers with the slowest requests are often those who have the most data on their accounts because they have made many purchases—that is, they’re the most valuable customers.

## Maintainability

However, we can and should design software in such a way that it will hopefully min‐ imize pain during maintenance, and thus avoid creating legacy software ourselves. To this end, we will pay particular attention to three design principles for software systems:
- Operability: Make it easy for operations teams to keep the system running smoothly.
- Simplicity: Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. (Note this is not the same as simplicity of the user interface.)
- Evolvability: Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change. Also known as extensibil‐ ity, modifiability, or plasticity.

# Chapter 2: Data Models and Query Languages
If you want to fetch a profile in the relational example, you need to either perform multiple queries (query each table by user_id) or perform a messy multi- way join between the users table and its subordinate tables. In the JSON representa‐ tion, all the relevant information is in one place, and one query is sufficient.

many-to-one relationships (many peo‐ ple live in one particular region, many people work in one particular industry), which don’t fit nicely into the document model. In relational databases, it’s normal to refer to rows in other tables by ID, because joins are easy. In document databases, joins are not needed for one-to-many tree structures, and support for joins is often weak.iii
If the database itself does not support joins, you have to emulate a join in application code by making multiple queries to the database.

While many-to-many relationships and joins are routinely used in relational data‐ bases, document databases and NoSQL reopened the debate on how best to represent such relationships in a database.


## Which data model leads to simpler application code?
If the data in your application has a document-like structure (i.e., a tree of one-to- many relationships, where typically the entire tree is loaded at once), then it’s probably a good idea to use a document model. The document model has limitations: for example, you cannot refer directly to a nes‐ ted item within a document, but instead you need to say something like “the second item in the list of positions for user 251”. However, if your application does use many-to-many relationships, the document model becomes less appealing. It’s possible to reduce the need for joins by denormal‐ izing, but then the application code needs to do additional work to keep the denor‐ malized data consistent. Joins can be emulated in application code by making multiple requests to the database, but that also moves complexity into the application and is usually slower than a join performed by specialized code inside the database. In such cases, using a document model can lead to significantly more complex appli‐ cation code and worse performance.


Document databases are sometimes called schemaless, but that’s misleading, as the code that reads the data usually assumes some kind of structure—i.e., there is an implicit schema, but it is not enforced by the database. A more accurate term is schema-on-read (the structure of the data is implicit, and only interpreted when the data is read), in contrast with schema-on-write (the traditional approach of relational databases, where the schema is explicit and the database ensures all written data con‐ forms to it. Schema-on-read is similar to dynamic (runtime) type checking in programming lan‐ guages, whereas schema-on-write is similar to static (compile-time) type checking.

Schema changes have a bad reputation of being slow and requiring downtime. This reputation is not entirely deserved: most relational database systems execute the ALTER TABLE statement in a few milliseconds. MySQL is a notable exception—it copies the entire table on ALTER TABLE, which can mean minutes or even hours of downtime when altering a large table—although various tools exist to work around this limitation.

The schema-on-read approach is advantageous if the items in the collection don’t all have the same structure for some reason. For example, because:
- There are many different types of objects, and it is not practical to put each type of object in its own table.
- The structure of the data is determined by external systems over which you have no control and which may change at any time.

PostgreSQL since version 9.3, MySQL since version 5.7, and IBM DB2 since ver‐ sion 10.5 also have a similar level of support for JSON documents. Given the popularity of JSON for web APIs, it is likely that other relational databases will follow in their footsteps and add JSON support.

On the document database side, RethinkDB supports relational-like joins in its query language, and some MongoDB drivers automatically resolve database references (effectively performing a client-side join, although this is likely to be slower than a join performed in the database since it requires additional network round-trips and is less optimized).
It seems that relational and document databases are becoming more similar over time, and that is a good thing: the data models complement each other.v If a database is able to handle document-like data and also perform relational queries on it, appli‐ cations can use the combination of features that best fits their needs.


## Query Languages for Data
When the relational model was introduced, it included a new way of querying data: SQL is a declarative query language, whereas IMS and CODASYL queried the database using imperative code. 
For example, if you have a list of animal species, you might write something like this to return only the sharks in the list:

```JavaScript

//imperative way
function getSharks() { 
    var sharks = [];
    for (var i = 0; i < animals.length; i++) { 
        if (animals[i].family === "Sharks") {
            sharks.push(animals[i]);
        }
    }
    return sharks; 
}
```

```SQL
-- declarative
SELECT * FROM animals WHERE family = 'Sharks';
```
- Imperative language: tells the computer to perform certain operations in a certain order. You can imagine stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time.

- Declarative language: like SQL or relational algebra, you just specify the pattern of the data you want—what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal. It is up to the database system’s query optimizer to decide which indexes and which join methods to use, and in which order to execute various parts of the query. 


A declarative query language is attractive because it is typically more concise and eas‐ ier to work with than an imperative API. But more importantly, it also hides imple‐ mentation details of the database engine, which makes it possible for the database system to introduce performance improvements without requiring any changes to queries. For example, in the imperative code shown at the beginning of this section, the list of animals appears in a particular order. If the database wants to reclaim unused disk space behind the scenes, it might need to move records around, changing the order in which the animals appear. Can the database do that safely, without breaking queries?
The SQL example doesn’t guarantee any particular ordering, and so it doesn’t mind if the order changes. But if the query is written as imperative code, the database can never be sure whether the code is relying on the ordering or not. The fact that SQL is more limited in functionality gives the database much more room for automatic opti‐ mizations.


Finally, declarative languages often lend themselves to parallel execution. Today, CPUs are getting faster by adding more cores, not by running at significantly higher clock speeds than before. Imperative code is very hard to parallelize across mul‐ tiple cores and multiple machines, because it specifies instructions that must be per‐ formed in a particular order. Declarative languages have a better chance of getting faster in parallel execution because they specify only the pattern of the results, not the algorithm that is used to determine the results. The database is free to use a parallel implementation of the query language, if appropriate.



## MapReduce Querying
MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularized by Google. A limited form of MapReduce is supported by some NoSQL datastores, including MongoDB and CouchDB. MapReduce is neither a declarative query language nor a fully imperative query API, but somewhere in between. To give an example, imagine you are a marine biologist, and you add an observation record to your database every time you see animals in the ocean. Now you want to generate a report saying how many sharks you have sighted per month. In PostgreSQL you might express that query like this:

```SQL
SELECT date_trunc('month', observation_timestamp) AS observation_month, sum(num_animals) AS total_animals
FROM observations
WHERE family = 'Sharks' 
GROUP BY observation_month;
```
This query first filters the observations to only show species in the Sharks family, then groups the observations by the calendar month in which they occurred, and finally adds up the number of animals seen in all observations in that month.

The same can be expressed with MongoDB’s MapReduce feature as follows:

```JavaScript
db.observations.mapReduce( 
    function map() {
        var year = this.observationTimestamp.getFullYear(); 
        var month = this.observationTimestamp.getMonth() + 1; 
        emit(year + "-" + month, this.numAnimals);
    },

    function reduce(key, values) {
        return Array.sum(values); 
    },
    
    {
            query: { family: "Sharks" },
            out: "monthlySharkReport"
    } 
);
```

1. The filter to consider only shark species can be specified declaratively (this is a MongoDB-specific extension to MapReduce).
2. The JavaScript function map is called once for every document that matches query, with this set to the document object.
3. The map function emits a key (a string consisting of year and month, such as "2013-12" or "2014-1") and a value (the number of animals in that observation).
4. The key-value pairs emitted by map are grouped by key. For all key-value pairs with the same key (i.e., the same month and year), the reduce function is called once.
5. The reduce function adds up the number of animals from all observations in a particular month.
6. The final output is written to the collection monthlySharkReport.

For example, say the observations collection contains these two documents:
```


{
    observationTimestamp: Date.parse("Mon, 25 Dec 1995 12:34:56 GMT"),
    family:     "Sharks",
    species:    "Carcharodon carcharias",
    numAnimals: 3
}
{
    observationTimestamp: Date.parse("Tue, 12 Dec 1995 16:17:18 GMT"),
    family:     "Sharks",
    species:    "Carcharias taurus",
    numAnimals: 4
}
```
The map function would be called once for each document, resulting in emit("1995-12", 3) and emit("1995-12", 4). Subsequently, the reduce function would be called with reduce("1995-12", [3, 4]), returning 7.

The map and reduce functions are somewhat restricted in what they are allowed to do. They must be pure functions, which means they only use the data that is passed to them as input, they cannot perform additional database queries, and they must not have any side effects. These restrictions allow the database to run the functions any‐ where, in any order, and rerun them on failure. However, they are nevertheless pow‐ erful: they can parse strings, call library functions, perform calculations, and more.

A usability problem with MapReduce is that you have to write two carefully coordi‐ nated JavaScript functions, which is often harder than writing a single query. More‐ over, a declarative query language offers more opportunities for a query optimizer to improve the performance of a query. For these reasons, MongoDB 2.2 added support for a declarative query language called the aggregation pipeline. In this language, the same shark-counting query looks like this:

```JavaScript
db.observations.aggregate([
    { $match: { family: "Sharks" } },
    { $group: {
        _id: {
            year:  { $year:  "$observationTimestamp" },
            month: { $month: "$observationTimestamp" }
        },
    totalAnimals: { $sum: "$numAnimals" } }}
    ]);
```
The aggregation pipeline language is similar in expressiveness to a subset of SQL, but it uses a JSON-based syntax rather than SQL’s English-sentence-style syntax;


## Graph-Like Data Models
But what if many-to-many relationships are very common in your data? The rela‐ tional model can handle simple cases of many-to-many relationships, but as the con‐ nections within your data become more complex, it becomes more natural to start modeling your data as a graph. There are several different, but related, ways of structuring and querying data in graphs. In this section we will discuss the property graph model (implemented by Neo4j, Titan, and InfiniteGraph) and the triple-store model (implemented by Datomic, AllegroGraph, and others).

### Property Graphs
In the property graph model, each vertex consists of:
- A unique identifier
- A set of outgoing edges
- A set of incoming edges
- A collection of properties (key-value pairs)

Each edge consists of:
- A unique identifier
- The vertex at which the edge starts (the tail vertex)
- The vertex at which the edge ends (the head vertex)
- A label to describe the kind of relationship between the two vertices
- A collection of properties (key-value pairs)


### Triple-Stores and SPARQL
In a triple-store, all information is stored in the form of very simple three-part state‐ ments: (subject, predicate, object). For example, in the triple (Jim, likes, bananas), Jim is the subject, likes is the predicate (verb), and bananas is the object. Examples show the same data as in Example 2-3, written as triples in a format called Turtle, a subset of Notation3
```Javascript
@prefix : <urn:example:>.
_:lucy     a       :Person.
_:lucy     :name   "Lucy".
_:lucy     :bornIn _:idaho.
_:idaho    a       :Location.
_:idaho    :name   "Idaho".
_:idaho    :type   "state".
_:usa      :type   "country".
_:usa      :within _:namerica.
_:namerica :name   "North America".
_:namerica :type   "continent"
```

Example:
```Javascript
@prefix : <urn:example:>.
_:lucy      a      :Person;      :name      "Lucy";               :bornIn      _:idaho.
_:idaho     a      :Location;    :name      "Idaho";              :type        "state";        :within      _:usa.
_:usa       a      :Location;    :name      "United States";      :type        "country";      :within      _:namerica
```


SPARQL is a query language for triple-stores using the RDF data model [43]. (It is an acronym for SPARQL Protocol and RDF Query Language.

```SQL
PREFIX : <urn:example:>
SELECT ?personName WHERE {
  ?person :name ?personName.
  ?person :bornIn  / :within* / :name "United States".
  ?person :livesIn / :within* / :name "Europe".
}
```


## Summary
New nonrelational “NoSQL” datastores have diverged in two main directions:
1. Document databases target use cases where data comes in self-contained documents and relationships between one document and another are rare.
2. Graph databases go in the opposite direction, targeting use cases where anything is potentially related to everything.


# Chapter 3: Storage and Retrieval
Any kind of index usually slows down writes, because the index also needs to be updated every time data is written.
This is an important trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes.

## Hash Indexes
Let’s say our data storage consists only of appending to a file, as in the preceding example. Then the simplest possible indexing strategy is this: keep an in-memory hash map where every key is mapped to a byte offset in the data file—the location at which the value can be found, as illustrated in Figure. Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys). When you want to look up a value, use the hash map to find the offset in the data file, seek to that location, and read the value.

![](img/chapter_3_1.png)

As described so far, we only ever append to a file—so how do we avoid eventually running out of disk space? A good solution is to break the log into segments of a cer‐ tain size by closing a segment file when it reaches a certain size, and making subse‐ quent writes to a new segment file. We can then perform compaction on these segments, as illustrated in Figure. Compaction means throwing away duplicate keys in the log, and keeping only the most recent update for each key.

![](img/chapter_3_2.png)


We can also merge several segments together at the same time as performing the compaction, as shown in Figure. Segments are never modified after they have been written, so the merged segment is written to a new file. The merging and compaction of frozen seg‐ ments can be done in a background thread, and while it is going on, we can still con‐ tinue to serve read and write requests as normal, using the old segment files. After the merging process is complete, we switch read requests to using the new merged seg‐ ment instead of the old segments—and then the old segment files can simply be deleted.

![](img/chapter_3_3.png)

In order to find the value for a key, we first check the most recent segment’s hash map; if the key is not present we check the second-most-recent segment, and so on. The merging process keeps the number of segments small, so lookups don’t need to check many hash maps.

- File format: CSV is not the best format for a log. It’s faster and simpler to use a binary format that first encodes the length of a string in bytes, followed by the raw string (without need for escaping).
- Deleting records: If you want to delete a key and its associated value, you have to append a special deletion record to the data file (sometimes called a tombstone). When log segments are merged, the tombstone tells the merging process to discard any previous values for the deleted key.
- Concurrency control: As writes are appended to the log in a strictly sequential order, a common implementation choice is to have only one writer thread. Data file segments are append-only and otherwise immutable, so they can be read concurrently by multiple threads.

Why don’t you update the file in place, overwriting the old value with the new value? But an append-only design turns out to be good for several reasons:
- Appending and segment merging are sequential write operations, which are gen‐ erally much faster than random writes, especially on magnetic spinning-disk hard drives. To some extent sequential writes are also preferable on flash-based solid state drives (SSDs) [4].
- Concurrency and crash recovery are much simpler if segment files are append- only or immutable. For example, you don’t have to worry about the case where a crash happened while a value was being overwritten, leaving you with a file con‐ taining part of the old and part of the new value spliced together.
- Merging old segments avoids the problem of data files getting fragmented over time.

However, the hash table index also has limitations. The hash table must fit in memory, so if you have a very large number of keys, you’re out of luck. In principle, you could maintain a hash map on disk, but unfortunately it is difficult to make an on-disk hash map perform well. It requires a lot of random access I/O, it is expensive to grow when it becomes full, and hash collisions require fiddly logic.


## SSTables and LSM-Trees
We require that the sequence of key-value pairs is sorted by key. We call this format Sorted String Table, or SSTable for short. We also require that each key only appears once within each merged segment file (the compaction process already ensures that). SSTables have several big advantages over log segments with hash indexes:

1. Merging segments is simple and efficient, even if the files are bigger than the available memory. The approach is like the one used in the mergesort algorithm and is illustrated in Figure 3-4: you start reading the input files side by side, look at the first key in each file, copy the lowest key (according to the sort order) to the output file, and repeat. This produces a new merged segment file, also sorted by key. When multiple segments contain the same key, we can keep the value from the most recent segment and discard the values in older segments.

![](img/chapter_3_4.png)

2. In order to find a particular key in the file, you no longer need to keep an index of all the keys in memory. For an example: say you’re looking for the key handiwork, but you don’t know the exact offset of that key in the segment file. However, you do know the offsets for the keys handbag and handsome, and because of the sorting you know that handiwork must appear between those two. This means you can jump to the offset for handbag and scan from there until you find handiwork (or not, if the key is not present in the file).

![](img/chapter_3_5.png)

You still need an in-memory index to tell you the offsets for some of the keys, but it can be sparse: one key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly.

### Constructing and maintaining SSTables
Maintaining a sorted structure on disk is possible (see “B-Trees” on page 79), but maintaining it in memory is much easier. There are plenty of well-known tree data structures that you can use, such as red-black trees or AVL trees. With these data structures, you can insert keys in any order and read them back in sorted order.

We can now make our storage engine work as follows:
- When a write comes in, add it to an in-memory balanced tree data structure (for
example, a red-black tree). This in-memory tree is sometimes called a memtable.
- When the memtable gets bigger than some threshold—typically a few megabytes —write it out to disk as an SSTable file. This can be done efficiently because the tree already maintains the key-value pairs sorted by key. The new SSTable file becomes the most recent segment of the database. While the SSTable is being written out to disk, writes can continue to a new memtable instance.
- In order to serve a read request, first try to find the key in the memtable, then in the most recent on-disk segment, then in the next-older segment, etc.
- From time to time, run a merging and compaction process in the background to combine segment files and to discard overwritten or deleted values.

This scheme works very well. It only suffers from one problem: if the database crashes, the most recent writes (which are in the memtable but not yet written out to disk) are lost. In order to avoid that problem, we can keep a separate log on disk to which every write is immediately appended.

Lucene, an indexing engine for full-text search used by Elasticsearch and Solr, uses a similar method for storing its term dictionary [12, 13]. A full-text index is much more complex than a key-value index but is based on a similar idea: given a word in a search query, find all the documents (web pages, product descriptions, etc.) that mention the word. This is implemented with a key-value structure where the key is a word (a term) and the value is the list of IDs of all the documents that contain the word (the postings list). In Lucene, this mapping from term to postings list is kept in SSTable-like sorted files, which are merged in the background as needed.

### Performance optimizations
 you have to check the memtable, then the segments all the way back to the oldest (possibly having to read from disk for each one) before you can be sure that the key does not exist. In order to optimize this kind of access, storage engines often use additional Bloom filters. (A Bloom filter is a memory-efficient data structure for approximating the contents of a set. It can tell you if a key does not appear in the database, and thus saves many unnecessary disk reads for nonexistent keys.)

 ## B-Trees
Like SSTables, B-trees keep key-value pairs sorted by key, which allows efficient key- value lookups and range queries. But that’s where the similarity ends: B-trees have a very different design philosophy.
The log-structured indexes we saw earlier break the database down into variable-size segments, typically several megabytes or more in size, and always write a segment sequentially. By contrast, B-trees break the database down into fixed-size blocks or pages, traditionally 4 KB in size (sometimes bigger), and read or write one page at a time.

![](img/chapter_3_6.png)

One page is designated as the root of the B-tree; whenever you want to look up a key in the index, you start here. The page contains several keys and references to child pages. Each child is responsible for a continuous range of keys, and the keys between the references indicate where the boundaries between those ranges lie.
In the example in Figure above, we are looking for the key 251, so we know that we need to follow the page reference between the boundaries 200 and 300. That takes us to a similar-looking page that further breaks down the 200–300 range into subranges. Eventually we get down to a page containing individual keys (a leaf page), which either contains the value for each key inline or contains references to the pages where the values can be found.
The number of references to child pages in one page of the B-tree is called the branching factor. For example, in Figure above the branching factor is six. In practice, the branching factor depends on the amount of space required to store the page refer‐ ences and the range boundaries, but typically it is several hundred.
If you want to update the value for an existing key in a B-tree, you search for the leaf page containing that key, change the value in that page, and write the page back to disk (any references to that page remain valid). If you want to add a new key, you need to find the page whose range encompasses the new key and add it to that page. If there isn’t enough free space in the page to accommodate the new key, it is split into two half-full pages, and the parent page is updated to account for the new subdi‐ vision of key ranges—see Figure below.

![](img/chapter_3_7.png)

This algorithm ensures that the tree remains balanced: a B-tree with n keys always has a depth of O(log n). Most databases can fit into a B-tree that is three or four levels deep, so you don’t need to follow many page references to find the page you are look‐ ing for. 

Moreover, some operations require several different pages to be overwritten. For example, if you split a page because an insertion caused it to be overfull, you need to write the two pages that were split, and also overwrite their parent page to update the references to the two child pages. This is a dangerous operation, because if the data‐ base crashes after only some of the pages have been written, you end up with a cor‐ rupted index (e.g., there may be an orphan page that is not a child of any parent).
In order to make the database resilient to crashes, it is common for B-tree implemen‐ tations to include an additional data structure on disk: a write-ahead log (WAL, also known as a redo log). This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself. When the data‐ base comes back up after a crash, this log is used to restore the B-tree back to a con‐ sistent state 

### B-tree optimizations
- Instead of overwriting pages and maintaining a WAL for crash recovery, some databases (like LMDB) use a copy-on-write scheme [21]. A modified page is written to a different location, and a new version of the parent pages in the tree is created, pointing at the new location. This approach is also useful for concurrency control.
- If a query needs to scan over a large part of the key range in sorted order, that page-by-page layout can be ineffi‐ cient, because a disk seek may be required for every page that is read. Many B- tree implementations therefore try to lay out the tree so that leaf pages appear in sequential order on disk. However, it’s difficult to maintain that order as the tree grows. By contrast, since LSM-trees rewrite large segments of the storage in one go during merging, it’s easier for them to keep sequential keys close to each other on disk.
- Additional pointers have been added to the tree. For example, each leaf page may have references to its sibling pages to the left and right, which allows scanning keys in order without jumping back to parent pages.

## Comparing B-Trees and LSM-Trees
Even though B-tree implementations are generally more mature than LSM-tree implementations, LSM-trees are also interesting due to their performance character‐ istics. As a rule of thumb, LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads. Reads are typically slower on LSM-trees because they have to check several different data structures and SSTables at different stages of compaction.

## Advantages of LSM-trees
A B-tree index must write every piece of data at least twice: once to the write-ahead log, and once to the tree page itself (and perhaps again as pages are split). There is also overhead from having to write an entire page at a time, even if only a few bytes in that page changed. Some storage engines even overwrite the same page twice in order to avoid ending up with a partially updated page in the event of a power failure 

Log-structured indexes also rewrite data multiple times due to repeated compaction and merging of SSTables. This effect—one write to the database resulting in multiple writes to the disk over the course of the database’s lifetime—is known as write ampli‐ fication. It is of particular concern on SSDs, which can only overwrite blocks a limi‐ ted number of times before wearing out.

In write-heavy applications, the performance bottleneck might be the rate at which the database can write to disk. In this case, write amplification has a direct perfor‐ mance cost: the more that a storage engine writes to disk, the fewer writes per second it can handle within the available disk bandwidth.

LSM-trees can be compressed better, and thus often produce smaller files on disk than B-trees. B-tree storage engines leave some disk space unused due to fragmenta‐ tion: when a page is split or when a row cannot fit into an existing page, some space in a page remains unused. Since LSM-trees are not page-oriented and periodically rewrite SSTables to remove fragmentation, they have lower storage overheads, espe‐ cially when using leveled compaction.

## Downsides of LSM-trees
- Another issue with compaction arises at high write throughput: the disk’s finite write bandwidth needs to be shared between the initial write (logging and flushing a memtable to disk) and the compaction threads running in the background. When writing to an empty database, the full disk bandwidth can be used for the initial write, but the bigger the database gets, the more disk bandwidth is required for compaction.
- If write throughput is high and compaction is not configured carefully, it can happen that compaction cannot keep up with the rate of incoming writes. In this case, the number of unmerged segments on disk keeps growing until you run out of disk space, and reads also slow down because they need to check more segment files. Typ‐ ically, SSTable-based storage engines do not throttle the rate of incoming writes, even if compaction cannot keep up, so you need explicit monitoring to detect this situa‐ tion [29, 30].
- An advantage of B-trees is that each key exists in exactly one place in the index, whereas a log-structured storage engine may have multiple copies of the same key in different segments. This aspect makes B-trees attractive in databases that want to offer strong transactional semantics: in many relational databases, transaction isola‐ tion is implemented using locks on ranges of keys, and in a B-tree index, those locks can be directly attached to the tree.
- B-trees are very ingrained in the architecture of databases and provide consistently good performance for many workloads, so it’s unlikely that they will go away anytime soon. In new datastores, log-structured indexes are becoming increasingly popular. There is no quick and easy rule for determining which type of storage engine is better for your use case, so it is worth testing empirically.

## Other Indexing Structures
So far we have only discussed key-value indexes, which are like a primary key index in the relational model. A primary key uniquely identifies one row in a relational table, or one document in a document database, or one vertex in a graph database. Other records in the database can refer to that row/document/vertex by its primary key (or ID), and the index is used to resolve such references.

It is also very common to have secondary indexes. In relational databases, you can create several secondary indexes on the same table using the CREATE INDEX command, and they are often crucial for performing joins efficiently.
A secondary index can easily be constructed from a key-value index. The main differ‐ ence is that keys are not unique; i.e., there might be many rows (documents, vertices) with the same key. This can be solved in two ways: either by making each value in the index a list of matching row identifiers (like a postings list in a full-text index) or by making each key unique by appending a row identifier to it. Either way, both B-trees and log-structured indexes can be used as secondary indexes.

## Data Warehousing
An application typically looks up a small number of records by some key, using an index. Records are inserted or updated based on the user’s input. Because these applications are interactive, the access pattern became known as online transaction processing (OLTP).

The following queries are often written by business analysts, and feed into reports that help the management of a company make better decisions (business intelligence). In order to differentiate this pattern of using databases from transaction processing, it has been called online analytic processing (OLAP).
- What was the total revenue of each of our stores in January?
- How many more bananas than usual did we sell during our latest promotion?
- Which brand of baby food is most often purchased together with brand X diapers?

These OLTP systems are usually expected to be highly available and to process trans‐ actions with low latency, since they are often critical to the operation of the business. Database administrators therefore closely guard their OLTP databases. They are usu‐ ally reluctant to let business analysts run ad hoc analytic queries on an OLTP data‐ base, since those queries are often expensive, scanning large parts of the dataset, which can harm the performance of concurrently executing transactions.

A data warehouse, by contrast, is a separate database that analysts can query to their hearts’ content, without affecting OLTP operations. The data warehouse con‐ tains a read-only copy of the data in all the various OLTP systems in the company. Data is extracted from OLTP databases (using either a periodic data dump or a con‐ tinuous stream of updates), transformed into an analysis-friendly schema, cleaned up, and then loaded into the data warehouse. This process of getting data into the warehouse is known as Extract–Transform–Load (ETL) and is illustrated in the figure.

![](img/chapter_3_8.png)

A big advantage of using a separate data warehouse, rather than querying OLTP sys‐ tems directly for analytics, is that the data warehouse can be optimized for analytic access patterns. It turns out that the indexing algorithms discussed in the first half of this chapter work well for OLTP, but are not very good at answering analytic queries.

## Summary
On a high level, we saw that storage engines fall into two broad categories: those opti‐ mized for transaction processing (OLTP), and those optimized for analytics (OLAP). There are big differences between the access patterns in those use cases:
- OLTP systems are typically user-facing, which means that they may see a huge volume of requests. In order to handle the load, applications usually only touch a small number of records in each query. The application requests records using some kind of key, and the storage engine uses an index to find the data for the requested key. Disk seek time is often the bottleneck here.
- Data warehouses and similar analytic systems are less well known, because they are primarily used by business analysts, not by end users. They handle a much lower volume of queries than OLTP systems, but each query is typically very demanding, requiring many millions of records to be scanned in a short time. Disk bandwidth (not seek time) is often the bottleneck here, and column- oriented storage is an increasingly popular solution for this kind of workload.