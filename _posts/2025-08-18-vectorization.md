---
title: Why Python Loops Feel Slow (and How Vectorization Fixes It)
date: 2025-08-18 23:MM:SS +0300
author: hadar
categories: [code]
tags: [python, vectorization]
math: true
---

Python is slow, at least when you write plain `for` loops.  
The fix? **Vectorization.**

## What is Vectorization?

Vectorization means rewriting code so that operations are applied to whole arrays at once, instead of iterating in Python with `for` or `while`. This usually occurs in lower level code (e.g `C`)

Under the hood, libraries like NumPy, pandas, or PyTorch push the work down to C routines with contiguous memory and clear (non-dynamic) types. This avoids Python’s overhead (dynamic type checks, object boxing, interpreter, etc).

## But Isn’t Complexity the Same?

from a theoretical perspective - one might say that nothing is changed. If, for example, I need perform pairwise addition given two vectors $a,b$, I still _must_ go through each element $a_i,b_i$, one by one, and store the result $a_i+b_i$. so an $O(n)$ runtime must still be $O(n)$ runtime.

This is all true, but - the constant changes!
every CS grad remembers big $O$ notation:

$$f(x) = O(g(x)) \text{ if } \exists C > 0, x_0 \text{ such that } |f(x)| \le C |g(x)|, \forall x \ge x_0$$

so while the assymptotic properties remain the same ($O(T(n))$ stays $O(T(n))$), What _does_ change is the hidden constant $C$.  
In vectorized code, $C$ is dramatically smaller because all the per-element calculations Python normally does is gone.

In practice though, what matters is that vectorization reduces the constant in front of your runtime.

## Dummy Example

Here’s pairwise addition in plain Python:

```python
def pairwise_add_for_loop(a: list, b: list) -> list:
    ret = []
    for i in range(len(a)):
        ret.append(a[i] + b[i])
    return ret
```

and with numpy?

```python
import numpy as np

def pairwise_add_np(a:np.ndarray, b:np.ndarray)->list:
  return a+b
```

![for-loop-vs-numpy](https://raw.githubusercontent.com/Hadar933/hadar933/refs/heads/main/assets/img/blog-images/pairwise-add-runtime.png)

thats order(s) of magnitude difference!

## A case-study

let's examine a simplified version of recommendation systems retrieve/similarity search in work:
lets say we have a dataset (a `pd.DataFrame` denoted `movies`) of $N$ movies, each has some known properties, eg genre, length, director,...


| MovieID | Title              | Genre          | Length (min) | Director        | Year |
| ------- | ------------------ | -------------- | ------------ | --------------- | ---- |
| 1       | Kpop Demon Hunters | Action/Fantasy | 100          | Maggie Kang     | 2025 |
| 2       | Space Odyssey      | Sci-Fi         | 149          | Stanley Kubrick | 1968 |
| ...     | ...                | ...            | ...          | ...             | ...  |
| N       | Spirited Away      | Animation      | 125          | Hayao Miyazaki  | 2001 |


for some movie $n\in[N]$, we want to find all other similar movies. in our example - we assume that "similar" means same genre and same director (and for computation-sake, we encode genre and director to integer value)

### Loop

to better understand the problem at hand, lets begin with a simple nested loop - for each movie in our database, go over every other movie. if a movie has same director and genre - its a match

```python

def nested_loop_matching(movies: pd.DataFrame)->list[list[int]]:
  similar_movies = []
  for i, movie_i in movies.iterrows():
      similar_to_i = []
      for j, movie_j in movies.iterrows():
          if i != j:
              if movie_i["Genre"] == movie_j["Genre"] and \
                 movie_i["Director"] == movie_j["Director"]:
                  similar_to_i.append(j)
      similar_movies.append(similar_to_i)

  return similar_movies

```

now, while this solution will get you kicked out of a FAANG interview, it does examplifies the simlicity of the problem, and is very readable. of course, we can do better

### Partially vectorized

Instead of looping over every pair of movies explicitly, we can leverage **pandas boolean masking** to compare columns in bulk. This avoids the inner loop and speeds up computation for moderately sized datasets:

```python
def partially_vectorized_matching(movies: pd.DataFrame) -> list[list[int]]:
    similar_movies = []
    for i, movie in movies.iterrows():
        mask = (movies["Genre"] == movie["Genre"]) & \
               (movies["Director"] == movie["Director"]) & \
               (movies.index != i)
        similar_to_i = movies.index[mask].tolist()
        similar_movies.append(similar_to_i)
    return similar_movies
```

**What's happening here?**

The key insight is that we still iterate over each movie (outer loop), but we eliminate the inner loop by using pandas' vectorized operations:

1. **Boolean masking**: `(movies["Genre"] == movie["Genre"])` (notice the `s` there) creates a boolean array comparing the current movie's genre against _all_ movies at once. This single operation replaces what would be $N$ individual comparisons in the nested loop.

2. **Vectorized AND operations**: The `&` operator combines multiple boolean masks efficiently at the C level, rather than doing element-by-element Python comparisons.

3. **Index filtering**: `movies.index[mask]` uses the boolean mask to extract matching indices in one operation, instead of manually building a list through iterations.

**Why is this faster?**

- **Reduced Python overhead**: Instead of $N^2$ Python iterations, we have $N$ iterations with vectorized operations inside each.
- **Memory locality**: pandas operations work on contiguous arrays, leading to better cache performance.
- **Compiled operations**: The comparison and masking operations are handled by optimized C code under the hood.

This approach gives us a complexity reduction from $O(N^2)$ Python operations to $O(N)$ Python operations with $O(N)$ vectorized operations per iteration - a significant practical speedup even though the theoretical complexity remains $O(N^2)$.

### Fully vectorized (with memory issues)

The first approach I thought of that is fully vectorized utilizes **broadcast comparison**.  
The idea is to generate a pairwise tensor $(N, N, 2)$ (where `2` represents our two columns we want to check for equality).

```python
def broadcast_exact_match(movies: pd.DataFrame) -> list[np.ndarray]:
    vals =  movies[['Genre', 'Director']].to_numpy()
    L = vals[:, None, :]  #(N,1,2)
    R = vals[None, :, :]  #(1,N,2)
    eq = (L == R) #(N,N,2)
    exact = eq.all(axis=2) #(N,N) -> requirting all cols match
    np.fill_diagonal(exact, False)  # Remove self-matches
    i_idx, j_idx = np.where(exact) # if exact[2,5]==True (5 is similar to 2) -> i_idx=2,j_idx=5
    counts = np.bincount(i_idx, minlength=exact.shape[0]) # counts[i] = how many movies are similar to movie i
    splits = np.cumsum(counts[:-1]) #  splits[i] tells us where movie i's matches end in the flattened j_idx
    neighbors_pos = np.split(j_idx, splits) # split the flattened j_idx array at the split points to get separate arrays of similar movies for each movie i
    return neighbors_pos
```

We're using np's broadcasting to eliminate all Python loops:

1. **Shape manipulation**: We reshape our data into `L` (N,1,2) and `R` (1,N,2). When these arrays are compared, numpy automatically broadcasts them to (N,N,2), creating every possible pairwise comparison. notice this means were allocating (At least) $N\cdot N \cdot 2$ floats in memory.

2. **Vectorized comparison**: `(L == R)` performs all $N^2 \times 2$ comparisons in a single C-level operation, resulting in a boolean tensor where `eq[i,j,k]` tells us if movie `i` and movie `j` match on feature `f`. we then simply use the `all` operator on the features axis, `eq.all(axis=2)`, meaning we expect all features to match
3. **Result extraction (aka the boring part)**: The remaining operations convert the 2D boolean matrix into the desired list format using NumPy's optimized indexing and splitting functions. This is more of a utility than anything else.

**The Memory Problem**

so no loops, but at what price? a fatal memory bottleneck: **memory usage scales as $O(N^2)$**.

For a dataset with 100K movies, we'd need to store a (100K, 100K, 2) tensor - that's roughly 160GB of memory just for the comparison results! This makes the approach impractical for real-world datasets, despite being theoretically the fastest in terms of computational complexity.

### Fully vectorized (without memory issues)

```python
def hashing_exact_match(df: pd.DataFrame) -> list[np.ndarray]:
    key = pd.util.hash_pandas_object(df, index=False) # same key for a row where df is the same
    left = pd.DataFrame({'i': df.index, 'k': key}) # df where i is an alternative index and k is the hash key
    right = pd.DataFrame({'j': df.index, 'k': key}) # repeat the same for j
    pairs = left.merge(right, on='k', how='inner') # merge the two dfs on the hash key so we get all rows with the same hash key
    pairs = pairs.loc[pairs.i.ne(pairs.j), ['i', 'j']] # drop self-matches
    same = pairs.groupby('i', sort=False)['j'].agg(list) # group by i and get the list of j's
    idx = df.index
    neighbors_pos = same.reindex(idx, fill_value=[]).map(lambda L: np.asarray(L, dtype=idx.dtype)).tolist()
    return neighbors_pos
```

This final approach solves the memory problem through **hashing to group identical rows**.

1. **Hash generation**: `hash_pandas_object` creates a hash for each row based on its content. Crucially, rows with identical values get identical hashes.

2. **Self-join setup**: We create two identical dfs (`left` and `right`) containing movie indices and their corresponding hashes. This prepares us for a self-join operation.

3. **Hash-based grouping**: `left.merge(right, on='k', how='inner')` performs an inner join on the hash keys. This automatically groups all movies with identical feature combinations

4. **Cleanup and formatting (aka the boring part)**: again, we remove self-matches and reshape the results into the desired format using pandas groupby op.

**time/memory analysis**

- **Memory efficiency**: Instead of storing an $O(N^2)$ comparison matrix, we only store $O(N)$ hashes and $O(M)$ matching pairs (where $M$ is the number of actual matches).
- **Computational efficiency**: The merge operation is highly optimized in pandas and runs in approximately $O(N \log N)$ time (might need to verify that, be I believe it involves sorting as the core computation).
- **Scalability**: This approach can handle datasets with millions of rows without running into memory constraints.

**The trade-off:**

We've moved from $O(N^2)$ space complexity to $O(N + M)$, where $M$ is typically much smaller than $N^2$ in real datasets. The time complexity becomes $O(N \log N)$ for the merge operation, which is often faster in practice than the $O(N^2)$ approaches due to better cache locality and optimized implementations.

**A note on correctness**: One have to adress amortized concerns when it comes to hashing. concretely - this approach relies on pandas' hash function having low collision rates - in practice this works well, but theoretically false matches could occur. The runtime is amortized $O(N \log N)$ but could degrade to $O(N^2)$ (but this is extremely unlikely with real data).

## Wrapping Up

The journey from nested loops to fully vectorized solutions shows how different optimization strategies can dramatically impact both performance and resource usage. While the naive $O(N^2)$ approach is readable, the partially vectorized version gives us practical speedups for moderate datasets. The broadcast approach achieves perfect vectorization but hits memory walls, while the hashing solution elegantly solves this limitation entirely, albeit with some theoretical concerns on collisions.

If there is a single thing you can take from this blog, it'll be this: modern Python is rarely slow, but only if we, the developers, know how to use it properly.
I hope you learned something new, like I did when writing this blog. See you next time!
