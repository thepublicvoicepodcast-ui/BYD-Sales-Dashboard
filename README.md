# BYD-Sales-Dashboard
Interactive BYD Sales Analysis Dashboard built using Python, Pandas, Streamlit, Plotly, Matplotlib, and Seaborn.
import streamlit as st
import pandas as pd
import plotly.express as px
import matplotlib.pyplot as plt
import seaborn as sns
# -------------------------------------------------
# PAGE CONFIGURATION
# -------------------------------------------------

st.set_page_config(
    page_title="BYD Sales Analysis Dashboard",
    page_icon="🚗",
    layout="wide"
)

st.title("🚗 BYD Sales Analysis Dashboard")
st.write("Python | Pandas | Plotly | Matplotlib | Seaborn")

# -------------------------------------------------
# LOAD DATA
# -------------------------------------------------

@st.cache_data
def load_data():
    df = pd.read_excel(
        "BYD_Realistic_Python_Practice_Dataset.xlsx",
        sheet_name="Raw_Data"
    )

    return df


df = load_data()

# -------------------------------------------------
# DATA CLEANING
# -------------------------------------------------

df = df.drop_duplicates()

# Fill missing values

df["Customer_Rating"] = df["Customer_Rating"].fillna(
    df["Customer_Rating"].mean()
)

df["Discount_Rate"] = df["Discount_Rate"].fillna(
    df["Discount_Rate"].median()
)

df["Sales_Channel"] = df["Sales_Channel"].fillna(
    "Unknown"
)

# Convert date column

df["Order_Date"] = pd.to_datetime(df["Order_Date"])

# -------------------------------------------------
# SIDEBAR FILTERS
# -------------------------------------------------

st.sidebar.header("🔎 Filter Data")

country = st.sidebar.multiselect(
    "Select Country",
    options=df["Country"].unique(),
    default=df["Country"].unique()
)

model = st.sidebar.multiselect(
    "Select BYD Model",
    options=df["Model"].unique(),
    default=df["Model"].unique()
)

vehicle_type = st.sidebar.multiselect(
    "Select Vehicle Type",
    options=df["Vehicle_Type"].unique(),
    default=df["Vehicle_Type"].unique()
)

sales_channel = st.sidebar.multiselect(
    "Select Sales Channel",
    options=df["Sales_Channel"].unique(),
    default=df["Sales_Channel"].unique()
)

# -------------------------------------------------
# APPLY FILTERS
# -------------------------------------------------

filtered_df = df[
    (df["Country"].isin(country)) &
    (df["Model"].isin(model)) &
    (df["Vehicle_Type"].isin(vehicle_type)) &
    (df["Sales_Channel"].isin(sales_channel))
]

# -------------------------------------------------
# KPI METRICS
# -------------------------------------------------

total_revenue = filtered_df["Revenue_USD"].sum()

total_profit = filtered_df["Estimated_Profit_USD"].sum()

total_quantity = filtered_df["Quantity"].sum()

total_orders = filtered_df["Order_ID"].nunique()

col1, col2, col3, col4 = st.columns(4)

col1.metric(
    "💰 Total Revenue",
    f"${total_revenue:,.0f}"
)

col2.metric(
    "📈 Total Profit",
    f"${total_profit:,.0f}"
)

col3.metric(
    "🚗 Vehicles Sold",
    f"{total_quantity:,}"
)

col4.metric(
    "📦 Total Orders",
    f"{total_orders:,}"
)

st.divider()

# -------------------------------------------------
# REVENUE BY MODEL
# -------------------------------------------------

st.subheader("📊 Revenue by BYD Model")

model_sales = (
    filtered_df
    .groupby("Model")["Revenue_USD"]
    .sum()
    .reset_index()
    .sort_values("Revenue_USD", ascending=False)
)

fig1 = px.bar(
    model_sales,
    x="Model",
    y="Revenue_USD",
    title="Total Revenue by BYD Model",
    text_auto=".2s"
)

st.plotly_chart(
    fig1,
    use_container_width=True
)

# -------------------------------------------------
# PROFIT BY MODEL
# -------------------------------------------------

st.subheader("💵 Profit by BYD Model")

model_profit = (
    filtered_df
    .groupby("Model")["Estimated_Profit_USD"]
    .sum()
    .reset_index()
    .sort_values(
        "Estimated_Profit_USD",
        ascending=False
    )
)

fig2 = px.bar(
    model_profit,
    x="Model",
    y="Estimated_Profit_USD",
    title="Estimated Profit by BYD Model"
)

st.plotly_chart(
    fig2,
    use_container_width=True
)

# -------------------------------------------------
# MONTHLY SALES TREND
# -------------------------------------------------

st.subheader("📈 Monthly Revenue Trend")

filtered_df = filtered_df.copy()

filtered_df["Month"] = (
    filtered_df["Order_Date"]
    .dt.to_period("M")
    .astype(str)
)

monthly_sales = (
    filtered_df
    .groupby("Month")["Revenue_USD"]
    .sum()
    .reset_index()
)

fig3 = px.line(
    monthly_sales,
    x="Month",
    y="Revenue_USD",
    markers=True,
    title="Monthly Revenue Trend"
)

st.plotly_chart(
    fig3,
    use_container_width=True
)

# -------------------------------------------------
# SALES BY COUNTRY
# -------------------------------------------------

st.subheader("🌍 Revenue by Country")

country_sales = (
    filtered_df
    .groupby("Country")["Revenue_USD"]
    .sum()
    .reset_index()
    .sort_values(
        "Revenue_USD",
        ascending=False
    )
)

fig4 = px.bar(
    country_sales,
    x="Country",
    y="Revenue_USD",
    title="Revenue by Country"
)

st.plotly_chart(
    fig4,
    use_container_width=True
)

# -------------------------------------------------
# CUSTOMER SEGMENT ANALYSIS
# -------------------------------------------------

st.subheader("👥 Customer Segment Analysis")

customer_sales = (
    filtered_df
    .groupby("Customer_Segment")["Revenue_USD"]
    .sum()
    .reset_index()
)

fig5 = px.pie(
    customer_sales,
    names="Customer_Segment",
    values="Revenue_USD",
    title="Revenue by Customer Segment"
)

st.plotly_chart(
    fig5,
    use_container_width=True
)

# -------------------------------------------------
# VEHICLE TYPE ANALYSIS
# -------------------------------------------------

st.subheader("🚙 Vehicle Type Performance")

vehicle_sales = (
    filtered_df
    .groupby("Vehicle_Type")
    .agg({
        "Revenue_USD": "sum",
        "Estimated_Profit_USD": "sum",
        "Quantity": "sum"
    })
    .reset_index()
)

st.dataframe(
    vehicle_sales,
    use_container_width=True
)

# -------------------------------------------------
# CORRELATION HEATMAP
# -------------------------------------------------

st.subheader("🔥 Correlation Heatmap")

numeric_columns = filtered_df.select_dtypes(
    include="number"
)

correlation = numeric_columns.corr()

fig, ax = plt.subplots(figsize=(12, 7))

sns.heatmap(
    correlation,
    annot=True,
    cmap="coolwarm",
    fmt=".2f",
    ax=ax
)

ax.set_title("Correlation Between Numerical Variables")

st.pyplot(fig)

# -------------------------------------------------
# MATPLOTLIB PRODUCT PERFORMANCE
# -------------------------------------------------

st.subheader("📊 Product Performance - Matplotlib")

top_models = (
    filtered_df
    .groupby("Model")["Quantity"]
    .sum()
    .sort_values(
        ascending=False
    )
)

fig2, ax2 = plt.subplots(figsize=(10, 5))

ax2.bar(
    top_models.index,
    top_models.values
)

ax2.set_title("Quantity Sold by BYD Model")

ax2.set_xlabel("BYD Model")

ax2.set_ylabel("Quantity Sold")

plt.xticks(rotation=45)

st.pyplot(fig2)

# -------------------------------------------------
# CUSTOMER BEHAVIOR
# -------------------------------------------------

st.subheader("👤 Customer Behavior Analysis")

customer_behavior = (
    filtered_df
    .groupby("Customer_Segment")
    .agg(
        Total_Revenue=("Revenue_USD", "sum"),
        Average_Order_Value=("Revenue_USD", "mean"),
        Total_Quantity=("Quantity", "sum"),
        Average_Rating=("Customer_Rating", "mean")
    )
    .reset_index()
)

st.dataframe(
    customer_behavior,
    use_container_width=True
)

# -------------------------------------------------
# RAW DATA
# -------------------------------------------------

st.subheader("📄 Filtered Raw Data")

st.dataframe(
    filtered_df,
    use_container_width=True
)

# -------------------------------------------------
# DOWNLOAD CLEANED DATA
# -------------------------------------------------

csv = filtered_df.to_csv(
    index=False
).encode("utf-8")

st.download_button(
    label="⬇ Download Filtered Data",
    data=csv,
    file_name="BYD_Filtered_Data.csv",
    mime="text/csv"
)
