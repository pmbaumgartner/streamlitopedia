# Altair for Visualizations

If you’ve been prototyping visualizations with another library, consider switching to Altair to build your visualizations. In my experience, I think there are three key reasons a switch could be beneficial:

1. Altair is probably faster (unless we’re plotting a lot of data)
2. It operates directly on pandas DataFrames
3. Interactive visualizations are easy to create

On the first point about speed, we can see a drastic speedup if we prototyped using matplotlib. Most of that speedup is just the fact that it takes more time to render a static image and place it in the app compared to rendering a javascript visualization. This is demonstrated in the example app below, which generates a scatterplot for some generated data and outputs the timing for the creation and rendering for each part of the visualization.

```python
from time import time

import altair as alt
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import streamlit as st


def mpl_scatter(dataset, x, y):
    fig, ax = plt.subplots()
    dataset.plot.scatter(x=x, y=y, alpha=0.8, ax=ax)
    return fig


def altair_scatter(dataset, x, y):
    plot = (
        alt.Chart(dataset, height=400, width=400)
        .mark_point(filled=True, opacity=0.8)
        .encode(x=x, y=y)
    )
    return plot


size = st.slider("Size", min_value=1000, max_value=100_000, step=10_000)
dataset = pd.DataFrame(
    {"x": np.random.normal(size=size), "y": np.random.normal(size=size)}
)

mpl_start = time()
mpl_plot = mpl_scatter(dataset, "x", "y")
mpl_finish = time()

st.pyplot(mpl_plot)
mpl_render = time()
st.subheader("Matplotlib")
st.write(f"Create: {mpl_finish - mpl_start:.3f}s")
st.write(f"Render: {mpl_render - mpl_finish:.3f}s")
st.write(f"Total: {mpl_render - mpl_start:.3f}s")

alt_start = time()
alt_plot = altair_scatter(dataset, "x", "y")
alt_finish = time()

st.altair_chart(alt_plot)
alt_render = time()
st.subheader("Altair")
st.write(f"Create: {alt_finish - alt_start:.3f}s")
st.write(f"Render: {alt_render - alt_finish:.3f}s")
st.write(f"Total: {alt_render - alt_start:.3f}s")

speedup = (mpl_render - mpl_start) / (alt_render - alt_start)
st.write(f"MPL / Altair Ratio: {speedup:.1f}x")
```

[Link to Gist](https://gist.github.com/pmbaumgartner/e8cf0cbdf85817aa24d51c8a8e008d7d/raw/3357054a0caaa1f4bf27277c6aaab34e0d0e747d/mpl_altair_streamlit_timing.py)

Working directly with DataFrames provides another benefit. It can ease the debugging process: if there’s an issue with the input data, we can use `st.write(df)` to display the DataFrame in a streamlit app and inspect it. This makes the feedback loop for debugging data issues much shorter. The second benefit is that it reduces the amount of transformational glue code sometimes required to create specific visualizations. For basic plots, we could use a DataFrame's plotting methods, but more complex visualizations might require us to restructure our dataset in a way that makes sense with the visualization API. This additional code between the dataset and visualization can be the source of additional complexity and can be a pain point as the app grows. Since Altair uses the Vega-Lite visualization grammar, the functions available in the [transforms API](https://vega.github.io/vega-lite/docs/transform.html) can be used to make any visualization appropriate transformations.

Finally, interactive visualizations with Altair are easy. While an app might start by using streamlit widgets to filter and select data, an app could also use a visualization could as the selection mechanism. Rather than communicating information as a string in a widget or narrative, (interactive visualizations)[https://altair-viz.github.io/gallery/index.html#interactive-charts] allow visual communication of aspects of the data within a visualization.