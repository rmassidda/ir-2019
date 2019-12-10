# Information Retrieval

Master degree in Computer Science, Università di Pisa.
Notes of the Information Retrieval course taught by prof. [Ferragina](http://pages.di.unipi.it/ferragina/).

The markdown files are thought to be converted via [Pandoc](https://pandoc.org/).

```sh
pandoc ??_*.md -o ir-2019.pdf
```

An updated PDF copy of the notes is automatically generated as an artifact in the [actions](https://github.com/rmassidda/ir-2019/actions) tab.
A not-surely-updated PDF copy is by the way included in the repository as `ir-2019.pdf`.

## Contents
- [Introduction](00_introduction.md)
- [Crawling](01_crawling.md)
- [LSH](02_lsh.md)
- [Deduplication](03_deduplication.md)
- [Compressed Storage](04_compressed_storage.md)
- [Index Construction](05_index_construction.md)
- [Document Compression](06_document_compression.md)
- [Document Parsing](07_document_parsing.md)
- [Search](08_search.md)
- [Posting Compression](09_posting_compression.md)
- [Query Processing](10_query_processing.md)
- [Document Ranking](11_document_ranking.md)
- [Web ranking](12_web_ranking.md)
- [Packing to fewer dimensions](13_packing.md)
- [Semantic annotation](14_semantic_annotation.md)

## Todo

- [x] LSH k-means comparison table kind of sucks.
- [x] Cosine distance in 03 (now draft)
- [x] Copy block in 04
- [ ] MapReduce in distributed indexing
- [x] Fancy-hits heuristic in 11
- [ ] Skip Pointers with known distribution in 10
- [ ] Confusing WAND description
- [ ] Personalized PageRank in 12
- [ ] Eigenvalues ↔ SingularValues conditions in 13
- [ ] Proof of the cosine-distance bound in 13
- [ ] Wiser presentation in 14
