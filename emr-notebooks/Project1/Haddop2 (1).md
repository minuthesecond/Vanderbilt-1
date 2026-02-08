**Steps Taken**

### 1. Initial Question - Understanding the Assignment
**What I did:**
- Asked Claude to explain what my professor meant by "explore multiple reducers" and "synchronization issues"

**Claude's Response:**
- Explained that multiple reducers create multiple output files (one per reducer)
- Described synchronization issues: reducers work independently, requiring manual aggregation for global statistics
- Explained combiners reduce network traffic by pre-aggregating data on mapper nodes

---

### 2.
**Discovery:**
- My notebook was already using 3 reducers! (Output showed `part-00000`, `part-00001`, `part-00002`)
- However, my code only read from `part-00000`, missing the synchronization issue demonstration

---

### 3. Received Implementation Guidance
**What Claude provided:**
- **IMPLEMENTATION_GUIDE.md** - Step-by-step instructions on what code to add

**Key changes needed:**
1. Update Section 7 to read from ALL reducer outputs (not just part-00000)
2. Add code to demonstrate the synchronization issue (manually combining results)
3. Add new Section 8 for combiner experiment
4. Add new Section 9 to verify combiner results

---

### 4. Updated My Notebook
**What I did:**
- Added new code to Section 7 to read all 3 reducer outputs
- Added loop to aggregate results from all reducers manually
- Added Section 8: Combiner experiment with `-combiner reducer.py` parameter
- Added Section 9: Verification that combiner produces identical results

**Code changes:**
```python
# Section 7: Read from ALL reducers
all_words = {}
for i in range(3):
    p = subprocess.run(['aws','s3','cp', f"{S3_OUTPUT}part-0000{i}", '-'], ...)
    # Aggregate results from all 3 files

# Section 8: Add combiner to Hadoop command
cmd = [
    'hadoop','jar', str(STREAMING_JAR),
    '-D','mapreduce.job.reduces=3',
    '-combiner','reducer.py',  # NEW LINE
    ...
]
```

---

### 5. Ran Updated Notebook on EMR
**What I did:**
- Opened notebook on EMR cluster
- Added the new code cells

**Results observed:**
- Section 7: Showed output from all 3 reducers and combined results
- Section 8: Ran job with combiner successfully
- Section 9: Verified combiner results matched non-combiner results

---

### 7. Troubleshooting - Outputs Not Showing on GitHub
**Problem:**
- When I first uploaded the notebook, outputs weren't visible on GitHub
- Orange dots appeared next to new code cells in EMR

**Solution:**
- Realized I needed to RUN all cells before downloading
- Ran all cells in EMR notebook
- Saved the notebook
- Downloaded again (this time with outputs included)
- Re-uploaded to GitHub

---

## Key Learnings

### Multiple Reducers
- Creates multiple output files (part-00000, part-00001, part-00002, etc.)
- Each reducer processes a hash-partitioned subset of keys
- **Synchronization Issue**: Must manually aggregate results from all output files to get global statistics
- Enables parallel processing for large datasets

### Combiner
- Acts as a "mini-reducer" on mapper nodes
- Reduces network traffic by pre-aggregating data locally
- Produces identical results to non-combiner jobs
- Only works for associative and commutative operations (sum, count, max, min)
- Significantly improves job performance

### Distributed Computing Challenges
- Reducers don't communicate with each other
- Global aggregation requires additional post-processing
- Trade-off between parallelization and synchronization complexity
