# Visionary_Infographica

## Project Overview

**Visionary Infographica** is an interactive web application developed using Streamlit and Plotly, designed to analyze and visualize sales data from a sample superstore dataset. The application enables users to explore data dynamically by applying various filters, generating insightful visualizations, and downloading data for further analysis.

## Features

1. **File Upload and Default Data Loading**:
    - Users can upload their own data file in CSV, TXT, XLSX, or XLS format.
    - If no file is uploaded, the application uses a default dataset (`Superstore.csv`).

2. **Date Range Filtering**:
    - Users can select a date range using date pickers to filter the data for specific periods.

3. **Data Filtering**:
    - Side panel filters allow users to select specific regions, states, and cities to narrow down the data scope.

4. **Sales Analysis and Visualizations**:
    - **Bar Chart**: Category-wise sales.
    - **Pie Charts**: Sales distribution by region and segment.
    - **Time Series Analysis**: Monthly sales trends.
    - **Treemap**: Hierarchical sales view by region, category, and sub-category.
    - **Scatter Plot**: Relationship between sales and profit.
    - **Summary Table**: Month-wise sub-category sales summary.

5. **Data Download**:
    - Users can download the filtered data and the visualized data in CSV format for offline analysis.

## How to Run the Application

1. **Clone the Repository**:
    ```bash
    git clone https://github.com/your-username/visionary-infographica.git
    ```

2. **Navigate to the Project Directory**:
    ```bash
    cd visionary-infographica
    ```

3. **Install the Required Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

4. **Run the Streamlit Application**:
    ```bash
    streamlit run app.py
    ```

## File Structure

- `app.py`: Main script to run the Streamlit application.
- `Superstore.csv`: Default dataset used if no file is uploaded.
- `requirements.txt`: List of required Python packages.
- `README.md`: Project overview and setup instructions.

## Detailed Code Explanation

### Importing Libraries
```python
import streamlit as st
import plotly.express as px
import pandas as pd
import os
import warnings
warnings.filterwarnings('ignore')
```

### Page Configuration
```python
st.set_page_config(page_title="superstore", page_icon=":bar_chart:", layout='wide')
st.title(":bar_chart: sample superstore")
st.markdown('<style> div.block-container{padding-top:1rem;} </style>', unsafe_allow_html=True)
```

### File Upload
```python
fl = st.file_uploader(" :file_folder: Upload a file", type=(['csv', 'txt', 'xlsx', 'xls']))
if fl is not None:
    filename = fl.name
    st.write(filename)
    df = pd.read_csv(filename, encoding="ISO-8859-1")
else:
    os.chdir(r"C:\Users\VARTIKA\Desktop\streamlit_app")
    df = pd.read_csv('Superstore.csv', encoding="ISO-8859-1")
```

### Date Range Filtering
```python
df["Order Date"] = pd.to_datetime(df["Order Date"], format='mixed')
startDate = pd.to_datetime(df["Order Date"]).min()
endDate = pd.to_datetime(df["Order Date"]).max()

col1, col2 = st.columns((2))
with col1:
    date1 = pd.to_datetime(st.date_input("Start Date", startDate))
with col2:
    date2 = pd.to_datetime(st.date_input("End Date", endDate))

df = df[(df["Order Date"] >= date1) & (df["Order Date"] <= date2)].copy()
```

### Sidebar Filters
```python
st.sidebar.header("Choose your filter: ")
region = st.sidebar.multiselect("Pick your Region", df["Region"].unique())
state = st.sidebar.multiselect("Pick the state", df["State"].unique())
city = st.sidebar.multiselect("Pick the City", df["City"].unique())

filtered_df = df[(df["Region"].isin(region) if region else True) & 
                 (df["State"].isin(state) if state else True) & 
                 (df["City"].isin(city) if city else True)]
```

### Visualizations and Data Display
```python
category_df = filtered_df.groupby(by=["Category"], as_index=False)["Sales"].sum()
col1, col2 = st.columns((2))
with col1:
    st.subheader("Category wise sales")
    fig = px.bar(category_df, x="Category", y="Sales", text=['${:,.2f}'.format(x) for x in category_df["Sales"]], template="seaborn")
    st.plotly_chart(fig, use_container_width=True)

with col2:
    st.subheader("Region wise Sales")
    fig = px.pie(filtered_df, values="Sales", names="Region", hole=.5)
    fig.update_traces(text=filtered_df["Region"], textposition="outside")
    st.plotly_chart(fig, use_container_width=True)
```

### Time Series Analysis
```python
filtered_df["month_year"] = filtered_df["Order Date"].dt.to_period("M")
linechart = pd.DataFrame(filtered_df.groupby(filtered_df["month_year"].dt.strftime("%Y : %b"))["Sales"].sum()).reset_index()
st.subheader('Time Series Analysis')
fig2 = px.line(linechart, x="month_year", y="Sales", labels={"Sales": "Amount"}, height=500, width=1000, template="gridon")
st.plotly_chart(fig2, use_container_width=True)
```

### Treemap Visualization
```python
st.subheader("Hierarchical view of Sales using TreeMap")
fig3 = px.treemap(filtered_df, path=["Region", "Category", "Sub-Category"], values="Sales", hover_data=["Sales"], color="Sub-Category")
fig3.update_layout(width=800, height=650)
st.plotly_chart(fig3, use_container_width=True)
```

### Segment and Category-wise Sales
```python
chart1, chart2 = st.columns((2))
with chart1:
    st.subheader('Segment wise Sales')
    fig = px.pie(filtered_df, values="Sales", names="Segment", template="plotly_dark")
    fig.update_traces(text=filtered_df["Segment"], textposition="inside")
    st.plotly_chart(fig, use_container_width=True)

with chart2:
    st.subheader('Category wise Sales')
    fig = px.pie(filtered_df, values="Sales", names="Category", template="gridon")
    fig.update_traces(text=filtered_df["Category"], textposition="inside")
    st.plotly_chart(fig, use_container_width=True)
```

### Scatter Plot
```python
data1 = px.scatter(filtered_df, x="Sales", y="Profit", size="Quantity")
data1['layout'].update(title="Relationship between Sales and Profits using Scatter Plot.", titlefont=dict(size=20), xaxis=dict(title="Sales", titlefont=dict(size=19)), yaxis=dict(title="Profit", titlefont=dict(size=19)))
st.plotly_chart(data1, use_container_width=True)
```

### Download Data
```python
csv = df.to_csv(index=False).encode('utf-8')
st.download_button('Download Data', data=csv, file_name="Data.csv", mime="text/csv")
```

## Conclusion

This project showcases how to build a comprehensive data analysis and visualization application using Streamlit and Plotly. Users can interact with the data, apply filters, and generate insightful visualizations effortlessly. The provided download options ensure that users can easily export data for further offline analysis.
