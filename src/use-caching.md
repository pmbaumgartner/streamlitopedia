# Use Caching (and Benchmark It)

It can be tempting to throw that handy `@st.cache` decorator on everything and hope for the best. However, mindlessly applying caching means that we're missing a great opportunity to get meta and use streamlit to understand where caching helps the most.

Rather than decorating every function, create two versions of each function: one with the decorator and one without. Then do some basic benchmarking of how long it takes to execute both the cached and uncached versions of that function.

In the example below, we simulate loading a large dataset by concatenating 100 copies of the `airports` dataset, then dynamically selecting the first n rows and describing them.

```python
import streamlit as st
from vega_datasets import data
from time import time
import pandas as pd


@st.cache
def load_data():
    return pd.concat((data.airports() for _ in range(100)))


@st.cache
def select_rows(dataset, nrows):
    return dataset.head(nrows)


@st.cache
def describe(dataset):
    return dataset.describe()


rows = st.slider("Rows", min_value=100, max_value=3300 * 100, step=10000)

start_uncached = time()
dataset_uncached = pd.concat((data.airports() for _ in range(100)))
load_uncached = time()
dataset_sample_uncached = dataset_uncached.head(rows)
select_uncached = time()
describe_uncached_dataset = dataset_sample_uncached.describe()
finish_uncached = time()
benchmark_uncached = (
    f"Cached. Total: {finish_uncached - start_uncached:.2f}s"
    f" Load: {load_uncached - start_uncached:.2f}"
    f" Select: {select_uncached - load_uncached:.2f}"
    f" Describe: {finish_uncached - select_uncached:.2f}"
)

st.text(benchmark_uncached)
st.write(describe_uncached_dataset)

start_cached = time()
dataset_cached = load_data()
load_cached = time()
dataset_sample_cached = select_rows(dataset_cached, rows)
select_cached = time()
describe_cached_dataset = describe(dataset_sample_cached)
finish_cached = time()

benchmark_cached = (
    f"Cached. Total: {finish_cached - start_cached:.2f}s"
    f" Load: {load_cached - start_cached:.2f}"
    f" Select: {select_cached - load_cached:.2f}"
    f" Describe: {finish_cached - select_cached:.2f}"
)
st.text(benchmark_cached)
st.write(describe_cached_dataset)
```

[Link to Gist](https://gist.github.com/pmbaumgartner/8097c08313cea57f02c54f23d4836621/raw/e5f89d6ade3aca306b38a3814ef550afa4992a80/cache_benchmarking.py)

Since each step (data load, select rows, describe selection) of this is timed, we can see where caching provides a speedup. From my experience with this example, my heuristics for caching are:

- Always cache loading the dataset
- Probably cache functions that take longer than a half second
- Benchmark everything else

I think caching is one of streamlit’s killer features and I know they’re focusing on it and improving it. Caching intelligently is also complex problem, so it’s a good idea to lean more towards benchmarking and validating that the caching functionality is acting as expected.