# Information Retrieval

Master degree in Computer Science, Università di Pisa.
Notes of the Information Retrieval course taught by prof. [Ferragina](http://pages.di.unipi.it/ferragina/).

The markdown files are thought to be converted via [Pandoc](https://pandoc.org/).

```sh
cat *.md | pandoc -o ir.pdf
```

An updated PDF copy of the notes is automatically generated as an artifact in the [actions](https://github.com/rmassidda/ir-2019/actions) tab.
A not-surely-updated PDF copy is by the way included in the repository as `ir-2019.pdf`.

## Contents
- [Crawling](01_crawling.md)
- [LSH](02_lsh.md)
- [Deduplication](03_deduplication.md)
- [Compressed Storage](04_compressed_storage.md)
- [Index Construction](05_index_construction.md)
- [Document Compression](06_document_compression.md)
- [Document Parsing](07_document_parsing.md)
- [Search](08_search.md)
- [Query Processing](09_query_processing.md)

## Todo

- [ ] LSH k-means comparison table kind of sucks.
- [ ] Cosine distance in 03
- [x] Copy block in 04
- [ ] MapReduce in distributed indexing
