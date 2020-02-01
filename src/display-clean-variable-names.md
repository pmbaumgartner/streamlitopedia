# Display Clean Variable Names

The variable names in a DataFrame might be snake cased or formatted in a way not appropriate for end users, e.g. `pointless_metric_my_boss_requested_and_i_reluctantly_included`. Most of the streamlit widgets contain a `format_func` parameter which takes function that applies formatting for display to the option values you provide the widget. As a simple example, you could title case each of the variable names.

You can also use this functionality, combined with a dictionary, to explicitly handle the formatting of your values. The example below cleans up the column names from the birdstrikes dataset for use as a dropdown to describe each column.

```python
import streamlit as st
from vega_datasets import data


@st.cache
def load_data():
    return data.birdstrikes()


cols = {
    "Airport__Name": "Airport Name",
    "Aircraft__Make_Model": "Aircraft Make & Model",
    "Effect__Amount_of_damage": "Effect: Amount of Damage",
    "Flight_Date": "Flight Date",
    "Aircraft__Airline_Operator": "Airline Operator",
    "Origin_State": "Origin State",
    "When__Phase_of_flight": "When (Phase of Flight)",
    "Wildlife__Size": "Wildlife Size",
    "Wildlife__Species": "Wildlife Species",
    "When__Time_of_day": "When (Time of Day)",
    "Cost__Other": "Cost (Other)",
    "Cost__Repair": "Cost (Repair)",
    "Cost__Total_$": "Cost (Total) ($)",
    "Speed_IAS_in_knots": "Speed (in Knots)",
}

dataset = load_data()

column = st.selectbox("Describe Column", list(dataset.columns), format_func=cols.get)

st.write(dataset[column].describe())
```

[Link to Gist](https://gist.github.com/pmbaumgartner/4558781c0dfe8a4539201eaeb575ba51/raw/7dedf6c4064af6586d2f136a35e4d2ac5814031f/format_func_with_dict.py)