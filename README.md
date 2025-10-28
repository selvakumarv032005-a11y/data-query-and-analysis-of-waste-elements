# data-query-and-analysis-of-waste-elements
This project presents a Waste Data Query &amp; Analysis App developed using Python and Streamlit. The app enables users to filter and analyze waste generation data based on multiple parameters such as Ward/Area, Waste Type, Source, and Date Range. It also integrates a threshold-based analysis.
[app.txt](https://github.com/user-attachments/files/23195030/app.txt)
import streamlit as st
import pandas as pd
import numpy as np

st.title("Data Query for Waste Elements & Analysis App")

# Load dataset safely
try:
    df = pd.read_csv("waste_data.csv")
except FileNotFoundError:
    st.error("Error: 'waste_data.csv' not found. Please ensure the file is in the correct directory.")
    st.stop()
except Exception as e:
    st.error(f"Error loading data: {e}")
    st.stop()

df["Date"] = pd.to_datetime(df["Date"])

st.markdown("---")
st.subheader("📅 Monthly Waste Prediction and Analysis")

# --- Month selection (clean container) ---
month_option = st.selectbox(
    "",
    ["November", "December"],
    index=None,
    placeholder="Select Month"
)

# --- Predict button ---
if st.button("🔍 Predict"):
    if not month_option:
        st.warning("Please select a month before predicting.")
        st.stop()

    # --- Prepare monthly data ---
    df["Month"] = df["Date"].dt.month
    df["Year"] = df["Date"].dt.year

    # Group by month to get average waste
    monthly_avg = df.groupby("Month")["Quantity_kg"].mean()

    # Predict November & December based on first 10 months
    base_mean = monthly_avg.loc[1:10].mean()
    if month_option == "November":
        predicted_avg = base_mean * 1.05
        month_num = 11
    else:
        predicted_avg = base_mean * 1.10
        month_num = 12

    avg_quantity = predicted_avg

    # --- Environmental status ---
    if avg_quantity < 1000:
        status = "🟢 Safe Zone"
        st.success(f"{status} — Predicted Average Waste: {avg_quantity:.2f} kg (Below 1000 kg limit)")
    elif 1000 <= avg_quantity <= 1150:
        status = "🟠 Risk Zone"
        st.warning(f"{status} — Predicted Average Waste: {avg_quantity:.2f} kg (Approaching hazardous level)")
    else:
        status = "🔴 High Risk / Disease Spread Zone"
        st.error(f"{status} — Predicted Average Waste: {avg_quantity:.2f} kg (Above 1150 kg limit)")

    # --- Truck requirement prediction ---
    st.markdown("---")
    st.subheader("🚛 Predicted Truck Requirement")

    truck_capacity = 1000
    total_quantity = avg_quantity * 30
    trucks_needed = int(total_quantity / truck_capacity) + (1 if total_quantity % truck_capacity != 0 else 0)

    st.info(f"Predicted total waste for {month_option}: {total_quantity:.2f} kg (approx. for 30 days)")
    st.success(f"🚛 Trucks Required for {month_option}: {trucks_needed}")

    # --- Generate predicted table for each ward & source ---
    st.markdown("---")
    st.subheader(f"📋 Predicted Waste Data Table for {month_option}")

    # Unique wards and sources from dataset
    wards = df["Ward/Area"].dropna().unique()
    sources = df["Source"].dropna().unique()

    # Create simulated daily data
    date_range = pd.date_range(f"2024-{month_num:02d}-01", periods=30, freq="D")

    simulated_data = []
    for date in date_range:
        for ward in wards:
            source = np.random.choice(sources)
            # Random quantity around predicted average ±20%
            quantity = np.random.uniform(avg_quantity * 0.8, avg_quantity * 1.2)
            simulated_data.append({
                "Date": date.date(),
                "Ward/Area": ward,
                "Source": source,
                "Predicted_Quantity_kg": round(quantity, 2)
            })

    predicted_df = pd.DataFrame(simulated_data)

    # Display predicted dataset
    st.dataframe(predicted_df)


else:
    st.info("Select a month and click **Predict** to view predicted results.")
