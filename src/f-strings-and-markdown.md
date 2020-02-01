# f-strings and Markdown

In building an interface around an analysis, much of it requires creating or manipulating strings in variable names, widget values, axis labels, widget labels, or narrative description.

If we want to display some analysis in narrative form and there’s a few particular variables we want to highlight, [f-strings](https://realpython.com/python-f-strings/) and markdown can help us out. Beyond an easy way to fill strings with specific variable values, it’s also an easy way to format them inline. For example, we might use something like this to display basic info about a column in a dataset and highlight them in a markdown string.

```python
mean = df["values"].mean()
n_rows = len(df)

md_results = f"The mean is **{mean:.2f}** and there are **{n_rows:,}**."

st.markdown(md_results)
```

We’ve used two formats here: `.2f` to round a float to two decimal places and `,` to use a comma as a thousands separator. We've also used markdown syntax to bold the values so that they're visually prominent in the text.

