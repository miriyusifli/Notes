## Interview Tips
To wrap up, we summarize a list of the Dos and Don’ts.
Dos
- Always ask for clarification. Do not assume your assumption is correct. - Understand the requirements of the problem.
- There is neither the right answer nor the best answer. A solution designed to solve the problems of a young startup is different from that of an established company with millions of users. Make sure you understand the requirements.
- Let the interviewer know what you are thinking. Communicate with your interview. - Suggest multiple approaches if possible.
- Once you agree with your interviewer on the blueprint, go into details on each component. Design the most critical components first.
- Bounce ideas off the interviewer. A good interviewer works with you as a teammate. - Never give up.
Don’ts
- Don't be unprepared for typical interview questions.
- Don’t jump into a solution without clarifying the requirements and assumptions.
- Don’t go into too much detail on a single component in the beginning. Give the high- level design first then drills down.
- If you get stuck, don't hesitate to ask for hints. - Again, communicate. Don't think in silence.
- Don’t think your interview is done once you give the design. You are not done until your interviewer says you are done. Ask for feedback early and often.

## SCALE FROM ZERO TO MILLIONS OF USERS
Vertical scaling, referred to as “scale up”, means the process of adding more power (CPU, RAM, etc.) to your servers. Horizontal scaling, referred to as “scale-out”, allows you to scale by adding more servers into your pool of resources.

## DESIGN A RATE LIMITER
### Where to put the rate limiter?
- Client-side implementation: Generally speaking, client is an unreliable place to enforce rate limiting because client requests can easily be forged by malicious actors. Moreover, we might not have control over the client implementation.
- Server-side implementation: A rate limiter that is placed on the server- side.
- Middleware implementation: which throttles requests to your APIs as shown below. API gateway is a fully managed service that supports rate limiting, SSL termination, authentication, IP whitelisting, servicing static content, etc. For now, we only need to know that the API gateway is a middleware that supports rate limiting.

![](img/ratelimiter_1.png)


### Algorithms for rate limiting
Here is a list of popular algorithms:
- Token bucket
- Leaking bucket
- Fixed window counter 
- Sliding window log
- Sliding window counter


### High-level architecture
The basic idea of rate limiting algorithms is simple. At the high-level, we need a counter to keep track of how many requests are sent from the same user, IP address, etc. If the counter is larger than the limit, the request is disallowed.
Where shall we store counters? Using the database is not a good idea due to slowness of disk access. In-memory cache is chosen because it is fast and supports time-based expiration strategy. For instance, Redis is a popular option to implement rate limiting. It is an in- memory store that offers two commands: INCR and EXPIRE.
- INCR: It increases the stored counter by 1.
- EXPIRE: It sets a timeout for the counter. If the timeout expires, the counter is automatically deleted.

- Rules are stored on the disk. Workers frequently pull rules from the disk and store them in the cache.
- When a client sends a request to the server, the request is sent to the rate limiter middleware first.
- Rate limiter middleware loads rules from the cache. It fetches counters and last request timestamp from Redis cache. Based on the response, the rate limiter decides:
- if the request is not rate limited, it is forwarded to API servers.
- if the request is rate limited, the rate limiter returns 429 too many requests error to the client. In the meantime, the request is either dropped or forwarded to the queue.

![](img/ratelimiter_2.png)


## DESIGN CONSISTENT HASHING
To achieve horizontal scaling, it is important to distribute requests/data efficiently and evenly across servers. Consistent hashing is a commonly used technique to achieve this goal. 

### The rehashing problem
If you have n cache servers, a common way to balance the load is to use the following hash method:
serverIndex = hash(key) % N, where N is the size of the server pool. This approach works well when the size of the server pool is fixed, and the data distribution is even. However, problems arise when new servers are added, or existing servers are removed. For example, if server 1 goes offline, the size of the server pool becomes 3. Using the same hash function, we get the same hash value for a key. But applying modular operation gives us different server indexes because the number of servers is reduced by 1.

### Solution: Consistent hashing
Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on average, where k is the number of keys, and n is the number of slots.

Now we understand the definition of consistent hashing, let us find out how it works. Assume SHA-1 is used as the hash function f, and the output range of the hash function is: x0, x1, x2, x3, ..., xn. In cryptography, SHA-1’s hash space goes from 0 to 2^160 - 1. That means x0 corresponds to 0, xn corresponds to 2^160 – 1, and all the other hash values in the middle fall between 0 and 2^160 - 1. Figure 5-3 shows the hash space.

![](img/consistenthashing_1.png)

One thing worth mentioning is that hash function used here is different from the one in “the rehashing problem,” and there is no modular operation. As shown in the figure, 4 cache keys (key0, key1, key2, and key3) are hashed onto the hash ring. To determine which server a key is stored on, we go clockwise from the key position on the ring until a server is found. Figure 5-7 explains this process. Going clockwise, key0 is stored on server 0; key1 is stored on server 1; key2 is stored on server 2 and key3 is stored on server 3.

![](img/consistenthashing_2.png)


### Add a server
After a new server 4 is added, only key0 needs to be redistributed. k1, k2, and k3 remain on the same servers. Let us take a close look at the logic. Before server 4 is added, key0 is stored on server 0. Now, key0 will be stored on server 4 because server 4 is the first server it encounters by going clockwise from key0’s position on the ring. The other keys are not redistributed based on consistent hashing algorithm.

![](img/consistenthashing_3.png)

### Remove a server
When a server is removed, only a small fraction of keys require redistribution with consistent hashing. In Figure below, when server 1 is removed, only key1 must be remapped to server 2. The rest of the keys are unaffected.

![](img/consistenthashing_4.png)

### Two issues in the basic approach
The consistent hashing algorithm was introduced by Karger et al. at MIT [1]. The basic steps are:
- Map servers and keys on to the ring using a uniformly distributed hash function.
- To find out which server a key is mapped to, go clockwise from the key position until the first server on the ring is found.

Problems:
- First, it is impossible to keep the same size of partitions on the ring for all servers considering a server can be added or removed. It is possible that the size of the partitions on the ring assigned to each server is very small or fairly large. 
- Second, it is possible to have a non-uniform key distribution on the ring. For instance, if servers are mapped to positions, most of the keys are stored on server 2. However, server 1 and server 3 have no data.

A technique called virtual nodes or replicas is used to solve these problems.

### Virtual nodes
A virtual node refers to the real node, and each server is represented by multiple virtual nodes on the ring. In Figure 5-12, both server 0 and server 1 have 3 virtual nodes. The 3 is arbitrarily chosen; and in real-world systems, the number of virtual nodes is much larger. Instead of using s0, we have s0_0, s0_1, and s0_2 to represent server 0 on the ring. Similarly, s1_0, s1_1, and s1_2 represent server 1 on the ring. With virtual nodes, each server is responsible for multiple partitions. Partitions (edges) with label s0 are managed by server 0. On the other hand, partitions with label s1 are managed by server 1. To find which server a key is stored on, we go clockwise from the key’s location and find the first virtual node encountered on the ring. As the number of virtual nodes increases, the distribution of keys becomes more balanced. This is because the standard deviation gets smaller with more virtual nodes, leading to balanced data distribution.

![](img/consistenthashing_5.png)


### The benefits of consistent hashing:
- Minimized keys are redistributed when servers are added or removed.
- It is easy to scale horizontally because data are more evenly distributed.
- Mitigate hotspot key problem. Excessive access to a specific shard could cause server overload. Imagine data for Katy Perry, Justin Bieber, and Lady Gaga all end up on the same shard. Consistent hashing helps to mitigate the problem by distributing the data more evenly.


## DESIGN A UNIQUE ID GENERATOR IN DISTRIBUTED SYSTEMS

### Possible approaches
#### Approach 1: Multi-master replication
This approach uses the databases’ auto_increment feature. Instead of increasing the next ID by 1, we increase it by k, where k is the number of database servers in use. As illustrated in the figure below, next ID to be generated is equal to the previous ID in the same server plus 2. This solves some scalability issues because IDs can scale with the number of database servers. However, this strategy has some major drawbacks:
- Hard to scale with multiple data centers
- IDs do not go up with time across multiple servers.
- It does not scale well when a server is added or removed.

![](img/uniqueidgenerator_1.png)

#### Approach 2: UUID
A UUID is another easy way to obtain unique IDs. UUID is a 128-bit number used to identify information in computer systems. UUID has a very low probability of getting collusion. Quoted from Wikipedia, “after generating 1 billion UUIDs every second for approximately 100 years would the probability of creating a single duplicate reach 50%” 

Pros:
- Generating UUID is simple. No coordination between servers is needed so there will not be any synchronization issues.
- The system is easy to scale because each web server is responsible for generating IDs they consume. ID generator can easily scale with web servers.
Cons:
- IDs are 128 bits long, but our requirement is 64 bits.
- IDs could be non-numeric.

#### Approach 3: Ticket Server
The idea is to use a centralized auto_increment feature in a single database server (Ticket Server). To learn more about this, refer to flicker’s engineering blog article.

Pros:
- Numeric IDs.
- It is easy to implement, and it works for small to medium-scale applications.
Cons:
- Single point of failure. Single ticket server means if the ticket server goes down, all systems that depend on it will face issues. To avoid a single point of failure, we can set up multiple ticket servers. However, this will introduce new challenges such as data synchronization.

![](img/uniqueidgenerator_2.png)

#### Approach 4: Twitter snowflake
Divide and conquer is our friend. Instead of generating an ID directly, we divide an ID into different sections. Figure shows the layout of a 64-bit ID.

![](img/uniqueidgenerator_3.png)

Each section is explained below.
- Sign bit: 1 bit. It will always be 0. This is reserved for future uses. It can potentially be used to distinguish between signed and unsigned numbers.
- Timestamp: 41 bits. Milliseconds since the epoch or custom epoch. We use Twitter snowflake default epoch 1288834974657, equivalent to Nov 04, 2010, 01:42:54 UTC.
- Datacenter ID: 5 bits, which gives us 2 ^ 5 = 32 datacenters.
- Machine ID: 5 bits, which gives us 2 ^ 5 = 32 machines per datacenter.
- Sequence number: 12 bits. For every ID generated on that machine/process, the sequence number is incremented by 1. The number is reset to 0 every millisecond.


The most important 41 bits make up the timestamp section. As timestamps grow with time, IDs are sortable by time. 41 bits means maximum ~69 years. 

Sequence number is 12 bits, which give us 2 ^ 12 = 4096 combinations. This field is 0 unless more than one ID is generated in a millisecond on the same server. In theory, a machine can support a maximum of 4096 new IDs per millisecond.

## DESIGN A URL SHORTENER

A URL shortener primary needs two API endpoints:
1. URL shortening. To create a new short URL, a client sends a POST request, which contains one parameter: the original long URL
2. URL redirecting. To redirect a short URL to the corresponding long URL, a client sends a GET request.

The hash function must satisfy the following requirements: 
- Each longURL must be hashed to one hashValue.
- Each hashValue can be mapped back to the longURL.


We will explore two types of hash functions for a URL shortener. The first one is “hash + collision resolution”, and the second one is “base 62 conversion.” Let us look at them one by one.

### hash + collision resolution
To shorten a long URL, we should implement a hash function that hashes a long URL to a 7- character string. A straightforward solution is to use well-known hash functions like CRC32, MD5, or SHA-1.
But, even the shortest hash value (from CRC32) is too long (more than 7 characters). How can we make it shorter? The first approach is to collect the first 7 characters of a hash value; however, this method can lead to hash collisions. To resolve hash collisions, we can recursively append a new predefined string until no more collision is discovered. This method can eliminate collision; however, it is expensive to query the database to check if a shortURL exists for every request.

![](img/urlshortener_1.png)

### Base 62 conversion
Base conversion is another approach commonly used for URL shorteners. Base conversion helps to convert the same number between its different number representation systems. Base 62 conversion is used as there are 62 possible characters for hashValue. For example, 11157 is converted to XT2.

![](img/urlshortener_2.png)

### Comparison of the two approaches
![](img/urlshortener_3.png)

### Big picture
![](img/urlshortener_4.png)

1. longURL is the input.
2. The system checks if the longURL is in the database.
3. If it is, it means the longURL was converted to shortURL before. In this case, fetch the shortURL from the database and return it to the client.
4. If not, the longURL is new. A new unique ID (primary key) Is generated by the unique ID generator.
5. Convert the ID to shortURL with base 62 conversion.
6. Create a new database row with the ID, shortURL, and longURL. To make the flow easier to understand, let us look at a concrete example.
- Assuming the input longURL is: https://en.wikipedia.org/wiki/Systems_design • Unique ID generator returns ID: 2009215674938.
- Convert the ID to shortURL using the base 62 conversion. ID (2009215674938) is converted to “zn9edcu”.
- Save ID, shortURL, and longURL to the database as shown in Table 8-4.

![](img/urlshortener_5.png)

The flow of URL redirecting is summarized as follows:
1. A user clicks a short URL link: https://tinyurl.com/zn9edcu
2. The load balancer forwards the request to web servers.
3. If a shortURL is already in the cache, return the longURL directly.
4. If a shortURL is not in the cache, fetch the longURL from the database. If it is not in the database, it is likely a user entered an invalid shortURL.
5. The longURL is returned to the user.