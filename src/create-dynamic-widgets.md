# Create Dynamic Widgets

Many examples focus on creating dynamic visualizations, but don’t forget you can also program dynamic widgets. The simplest example of this need is when two columns in a dataset have a nested relationship and there are two widgets to select values from those two columns. When building an app to filter data, the the dropdown for the first column should change the options available in the second dropdown.

Linking behavior of two dropdowns is a common use case. The example below builds a scatterplot with the `cars` dataset. We need a dynamic dropdown here because the variable we select for the x-axis doesn't need to be available for selection in the y-axis.

We can also go beyond this basic dynamic functionality: what if we sorted the available y-axis options by their correlation with the selected x variable? We can calculate the correlations and combining this with the widget’s `format_func` to display variables and their correlations in sorted order.

```python
import altair as alt
import streamlit as st
from vega_datasets import data

cars = data.cars()

quantitative_variables = [
    "Miles_per_Gallon",
    "Cylinders",
    "Displacement",
    "Horsepower",
    "Weight_in_lbs",
    "Acceleration",
]


@st.cache
def get_y_vars(dataset, x, variables):
    corrs = dataset.corr()[x]
    remaining_variables = [v for v in variables if v != x]
    sorted_remaining_variables = sorted(
        remaining_variables, key=lambda v: corrs[v], reverse=True
    )
    format_dict = {v: f"{v} ({corrs[v]:.2f})" for v in sorted_remaining_variables}
    return sorted_remaining_variables, format_dict


st.header("Cars Dataset - Correlation Dynamic Dropdown")
x = st.selectbox("x", quantitative_variables)
y_options, y_formats = get_y_vars(cars, x, quantitative_variables)
y = st.selectbox(
    f"y (sorted by correlation with {x})", y_options, format_func=y_formats.get
)

plot = alt.Chart(cars).mark_circle().encode(x=x, y=y)

st.altair_chart(plot)
```

[Link to Gist](https://gist.github.com/pmbaumgartner/23d919cb7badd3ce4f2807af68897a75/raw/72b71923cb4db876b64fbd6283f2380d4cce1020/dynamic_widget.py)