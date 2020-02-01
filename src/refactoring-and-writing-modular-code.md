# Refactoring & Writing Modular Code

It’s easy to spend a few hours with streamlit and have a 500 line `app.py` file that nobody but you understands. If you're handing off your code, deploying your app, or adding a some new functionality it's now possible that you'll be spending a significant amount of time trying to remember how your code works because you've neglected good code hygiene.

If an app is beyond 100 lines of code, it can probably benefit from a refactor. A good first step is to create functions from the code and put those functions in a separate `helpers.py` file. This also makes it easier to test and benchmark caching on these functions.

There’s no specific right way on how exactly to refactor code, but I’ve developed an exercise that can help when starting an app refactor.

**Refactoring Exercise**

In the `app.py`, try to:
- only import streamlit and helper functions (don’t forget to [benchmark](./use-caching.md) `@st.cache` on these helper functions)
- never create a variable that isn’t input into a streamlit object, i.e. visualization or widget, in the next line of code (with the exception of the data loading function)

These aren’t hard and fast rules to always abide by: you could follow them specifically and have a poorly organized app because you’ve got large, complex functions that do too much. However, they are good objectives to start with when moving from everything in app.py to a more modular structure.
The example below highlights an app before and after going through this exercise.


**⛔️ BAD EXAMPLE: PRE-REFACTOR**

```python
# app.py
import streamlit as st
import pandas as pd

data = pd.read_csv("data.csv")  # no function → no cache, requires pandas import: 👎,👎
sample = data.head(100)  # not input into streamlit object: 👎
described_sample = sample.describe()  # input into streamlit object: ✅
st.write(described_sample)

```
**✅ GOOD EXAMPLE**

```python
# app.py
import streamlit as st
from helpers import load_data, describe_sample

data = load_data()  # data import: ✅
described_sample = describe_sample(data, 100)  # input into streamlit object: ✅
st.write(described_sample)

# helpers.py
import streamlit as st
import pandas as pd


@st.cache
def load_data():
    return pd.read_csv("data.csv")


@st.cache
def describe_sample(dataset, nrows):
    sample = dataset.head(nrows)
    return sample.describe()
```

Another benefit of reorganizing code in this way is that the functions in the helpers file are now easier to write tests for. Sometimes I struggle with coming up with ideas of what to test, but I’ve found that now it’s really easy to come up with tests for my apps because I’m more quickly discovering bugs and edge cases now that I’m interacting more closely with the data and code. Now, any time my app displays a traceback, I fix the function that caused it and write a test to make sure the new behavior is what I expect.