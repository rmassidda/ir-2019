# Crawling

Web crawling is the process by which we gather pages from the Web graph to index them and support a search engine.
It is possibile to recognize different features that a web crawler should implement:

- Quality, it should be biased toward fetching "useful" pages first.
- Efficiency, it should avoid duplication, or near duplication, of the content crawled.
- Netiquette, there exist implicit and explicit policies regulating the access to a server by a crawler.
- Freshness, the content crawled should ideally reflect an updated image of the web.

## Architecture
The crawler begins with one or more URLs, that constitute a seed set, inside its URLs frontier.
Continuously it picks a URL from the frontier, then fetches the corresponding web page.
The fetched page is then parsed, to extract both the text and the links from the page.
The extracted text is fed to a text-indexer, while the extracted links are then added to the URL frontier, which at all times consists of URLs whose corresponding pages have yet to be fetched by the crawler.

In the architecture of a crawler we can distinguish many different modules, all of these are executed by different, and possibly multiple, threads in a parallel and distributed computing fashion.

- Link extractor
```
while ( PageRepository not empty )
  pop page from PageRepository
  check for near duplication
  extract links
  push links in PriorityQueue
```

- Downloader
```
while ( AssignedRepository not empty )
  pop URL from AssignedRepository
  download and archive page
  push page in PageRepository
```

- Crawler manager
```
while ( PriorityQueue not empty )
  pop URLs from PriorityQueue
  foreach URL
    check if recently crawled
    preprocess URL (DNS, robots, etc)
    push URL in AssignedRepository
```

## Mercator
To guarantee the desired features of quality and politeness, it is mandatory for a crawler to implement some priority policies in the extraction of the URLs to be parsed.
In the Mercator architecture this is done by using two set of FIFO queues: the front queues $F$ and the back queues $B$.

After being extracted, a new URL is analyzed with some heuristic able to determine its priority value $i$, that is an integer between $1$ and $|F|$.
After this the value is assigned to the $i$-th front queue.

The URLs are then extracted from the front queues in a way biased by the priority of the queue, after this each one of the extracted URLs is pushed into one back queue according to its host.
Each of the back queues B contains URLs from a single host, the mapping from hosts to back queues is maintained by an auxiliary table T.
If one of the back queues gets empty a new host is immediately reassigned.

The URLs from the back queues are then extracted and inserted in a min-heap.
This data structure contains one and only one element for each queue and it's ordered by the timestamps $t_e$, that represent the first eligible moment to repeat a request to a certain host.
The crawler main loop extracts the root of the min-heap, waits for the indicated time $t_e$ to expire and then adds the URL to the `AssignedRepository`.


## Bloom Filter
To check if a page has been parsed or downloaded before a first control can be done on the URL, obviously, given the size of the web graph, storing all the crawled URLs in a dictionary is not a option.

A Bloom Filter (1970) is a probabilistic data structure used to check the elements of a set, it ensures a positive result if an element is in the set, but could also provide false positives.
A binary array of size $m$ is created, and then a family of $k$ hash functions is used to map an object to a position in the array.
To check if a certain element $u$ is in the array is equivalent to the proposition $\land^{k}_{i=1} h_i (u)$.

Under the assumption of simple uniform hashing, the probability that a fixed position in the array contains zero is $p = (1-\frac{1}{m})^{kn} \approx e^{-kn/m}$, and so the probability of a false positive $\epsilon$ is equal to: 

$$
\begin{aligned}
P ( \forall i .B[ h_i ( u )] \neq 0 ) = \\
P (B[ h_i ( u )] \neq 0 ) ^ k = \\
( 1 - P ( B[h_i ( u )] = 0 ) ) ^ k = \\
( 1 - e^{-kn/m} ) ^ k \quad
\end{aligned}
$$
Fixed the size of the array $m$ and the number of elements in the set $n$, from this formula we can derive an optimal value for the number of hash functions $k_* = \ln 2 \frac{m}{n}$, that implies an error rate $\epsilon_* = (0.6185)^{\frac{m}{n}}$.

### Various applications
The distributed computation of set intersection between two sets $A$ and $B$ can be done by computing the Bloom Filter of the first set, than computing $Q$ as the set of all the $b \in B$ elements that are present according to the bloom filter and then by intersecting $A$ and $Q$, where the advantage is given by the fact $|Q|<|B|$.
The bit cost of the transmission of the Bloom Filter of $A$ and $Q$ is $\Theta (m_A) + (|A \cap B| + \epsilon|B|)\log |U|$, so it less expensive than sending the whole $A$ set at a cost of $|A|\log{|U|}$.

The distributed computation of approximate set difference can be derived by the previous algorithm, when $A$ can correctly compute the difference via $A-B=A-Q$, and $B$ can obtain an approximation via $B-A \approx B - Q$.

Another possibile approach to compute the set difference is using Patricia Trees derived from $A$ and $B$.
Comparing each node in a top-down fashion, if the node are equals the visit backtracks, otherwise it proceeds to all children, when a leaf is reached then the corresponding element of $B$ is declared to be in $B-A$.
This solution is obviously unfeasible in practice because of the cost of replicating the subsets in a node, but this could be avoided by using the same algorithm on a Merkle Tree, that stores instead the hash of each subset in the corresponding node.
It's possible to improve again the space occupancy by computing a Bloom Filter of the nodes of the Merkle Tree $MT_A$, and then visiting top-down the $MT_B$ tree each node is checked against $BF(MT_A)$.
This improvement comes at the cost of possible false positives during the matching of the nodes between the two trees.
Assuming $m_A = \Theta (|A| \log\log |B|)$ and the optimal $k_A = \Theta ( \log\log |B| )$, the bit cost of the transmission is $O(|A| \log \log |B|)$, that is the bit size of the Bloom Filter.

### Spectral Bloom Filter
A Spectral Bloom Filter is a variation of a standard BF that makes use of an integer array instead of a binary one, where each position of the array counts the number of occurrences of an object $s \in S$ in a multiset $M$.
The space usage is slightly larger, but in a constant order, and opens to new possible use cases like aggregate and iceberg queries.
The error probability is the same of the standard bloom filter, so $\epsilon \approx (1-p)^k$.

The insertion and deletion operations are implemented by simply incrementing, or decrementing, each counter derived by the application of each hash function to the element.
The query result is instead given by the minimum of all the counters, this minimum selection allows to select the counter where the minimum number of collision have occurred.

It can be noticed that if the minimum of all the counters relative to an object is repeated multiple times, it is less likely that the same item is subject to a false positive error.
This situation, called recurring minimum, can be used to improve the overall performances of the SBF.
We can operate then with two separate SBF, where the second one is smaller and is used to store single minimum elements.

```
INSERTION
insert x in SBF1
if x single minimum in SBF1:
  if x in SBF2:
    insert x in SBF2
  else:
    set counters of x in SBF2 as the min value of x in SBF1

DELETION
delete x from SBF1
if x single minimum in SBF1:
  if x in SBF2:
    delete x from SBF2

LOOKUP
if x recurring minimum in SBF1: 
  return min x in SBF1
else if x in SBF2:
  return min x in SBF2
else
  return min x in SBF1
```

## Parallel Crawling
The web is to big to be crawled by a single crawler, so the work should be divided avoiding duplication of work.
Using static assignment is difficult to load balance the URLs assigned to a crawler, also the situation of fault-tolerance where one downloader could be removed or created in a dynamic way makes the static assignment prone to errors.

A possibile solution is the use of the consistent hashing technique.
Given two[^1] hashing functions $h_s : \textrm{Server} \rightarrow {x}$ and $h_u : \textrm{URL} \rightarrow {x}$, we use an orientated circular mapping where the items are dynamically partitioned in arcs between the servers and, assuming clockwise orientation, each server needs to communicate only with its successor in case of mutation in the topography.

In average each server of the $m$ server has assigned $\frac{n}{m}$ URLs, it's possibile to prove that this happens with hight probability.

[^1]: The slides of the course mention only one function, also in some of the exercises the same function is used for both objects.
