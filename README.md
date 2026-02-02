# EMR MapReduce Word Frequency Counter

A hands-on exploration of MapReduce concepts using Amazon EMR (Elastic MapReduce) and Hadoop Streaming.

## Project Overview

This project demonstrates core distributed computing concepts through word frequency counting using MapReduce on Amazon EMR. It explores how data processing scales across multiple nodes and the tradeoffs involved in parallel computation.

## What's Inside

### Basic Implementation
- `EMR_Word_Frequency_Hadoop_Streaming.ipynb` - Original notebook demonstrating basic MapReduce word counting
- Simple mapper that emits `(word, 1)` pairs
- Reducer that aggregates counts per word
- End-to-end pipeline from data upload to results

### Advanced Exploration
- `EMR_Word_Frequency_Advanced.ipynb` - Extended experiments with:
  - **Multiple Reducers**: Parallel reduce operations with output partitioning
  - **Combiners**: Local pre-aggregation to reduce network traffic
  - **Synchronization Challenges**: Demonstrates issues with distributed aggregation

## Key Concepts Demonstrated

### MapReduce Fundamentals
- **Map Phase**: Tokenizes text and emits key-value pairs
- **Shuffle & Sort**: Hadoop groups data by key across the cluster
- **Reduce Phase**: Aggregates values for each key

### Multiple Reducers
When using multiple reducers (e.g., 3 reducers):
- Output splits into multiple files: `part-00000`, `part-00001`, `part-00002`
- Each reducer processes a hash-partitioned subset of keys
- **Synchronization Issue**: Global aggregation requires post-processing all outputs

**Example Problem**: If you want a total word count across the entire dataset, you must sum counts from all reducer output files—the reducers don't communicate with each other.

### Combiners
Combiners act as "mini-reducers" on mapper nodes:
- Pre-aggregate data before network shuffle
- Reduce bandwidth and improve performance
- Only work for associative/commutative operations (sum, count, max, min)

**Impact**: Instead of sending thousands of `(word, 1)` pairs across the network, send `(word, local_count)`.

## Technical Setup

### Prerequisites
- Amazon EMR cluster with Hadoop installed
- S3 bucket for input/output storage
- Python 3 with AWS CLI configured

### Configuration
```python
S3_BUCKET = 'your-bucket-name'
PREFIX = 'mapreduce/wordcount'
```

### Running the Job
1. Upload input data and scripts to S3
2. Submit Hadoop Streaming job with mapper and reducer
3. Download and analyze results

## Files

- **mapper.py**: Reads input, tokenizes, emits `word\t1`
- **reducer.py**: Aggregates counts per word
- **logs.txt** / **alice.txt**: Sample input data

## Learning Outcomes

1. **Distributed Computing**: Understanding how work distributes across cluster nodes
2. **MapReduce Paradigm**: Hands-on experience with map/reduce phases
3. **Performance Optimization**: Impact of combiners on network efficiency
4. **Synchronization Challenges**: Tradeoffs of parallelization in distributed systems

## Experiments You Can Try

1. **Vary reducer count**: Try 1, 3, 5, or 10 reducers and observe output partitioning
2. **Compare with/without combiner**: Monitor job metrics to see network savings
3. **Use larger datasets**: Test with alice.txt or other books from Project Gutenberg
4. **Global statistics**: Implement post-processing to compute corpus-wide metrics

## References

- [Amazon EMR Documentation](https://docs.aws.amazon.com/emr/)
- [Hadoop Streaming Guide](https://hadoop.apache.org/docs/stable/hadoop-streaming/HadoopStreaming.html)
- [MapReduce Paper (Google)](https://research.google/pubs/pub62/)

## Author

Justin - Vanderbilt University

---

*This project was completed as part of exploring distributed computing concepts with MapReduce and Amazon EMR.*
