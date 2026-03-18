✈️ Flight Price Prediction – Feature Engineering & EDA

This project focuses on feature engineering and exploratory data analysis (EDA) on flight booking data to understand pricing behavior and prepare data for predictive modeling.

📁 Project Overview

The goal of this project is to:

Extract meaningful features from raw flight schedule data

Analyze factors affecting flight prices

Transform categorical and time-based variables into model-ready features

The dataset includes details such as airline, source, destination, route, journey date, departure/arrival time, stops, and additional services.

🛠️ Tools & Technologies

Python

Pandas & NumPy

Matplotlib & Seaborn

Jupyter Notebook

🔧 Feature Engineering
📅 Date Transformation

Split Date_of_Journey into:

Day

Month

Year (page 5–6)

👉 Helps capture seasonal trends in pricing

⏱️ Time Feature Extraction

Extracted:

Departure Hour & Minute

Arrival Hour & Minute (page 8–9)

👉 Enables analysis of peak travel hours

✈️ Stops Conversion

Converted categorical stops:

Non-stop → 0

1 stop → 1

2 stops → 2

etc. (page 7)

👉 Makes it usable for numerical analysis and modeling

🧾 Additional Info Encoding

One-hot encoded features like:

In-flight meals

Business class

Layovers (page 11)

👉 Captures service-level differences affecting price

🧹 Data Cleaning

Removed unnecessary columns:

Route

Additional Info (after encoding)

Converted date to proper datetime format

📊 Exploratory Data Analysis
💸 Price vs Number of Stops

Prices increase significantly with number of stops (page 7)

👉 Key business insight:

Direct flights are cheaper in this dataset

Multi-stop flights often indicate premium/long-haul routes

🛫 Airline Pricing Analysis

Jet Airways Business:

Highest average price (~₹60,000)

Trujet:

Lowest (~₹4,000) (page 14)

👉 Strong airline-based pricing differentiation

🏆 Most Used Airlines

Jet Airways (~50%)

IndiGo (~27%)

Air India (~23%) (page 15)

👉 Market dominance clearly visible

📌 Key Insights

Number of stops is a major price driver

Airline brand significantly impacts pricing

Time and date features are critical for modeling

Encoded service features improve predictive power

🚀 Business Implications

Airlines can optimize pricing based on:

Route complexity

Demand patterns

Customers can:

Choose optimal routes based on price vs stops trade-off

Travel platforms can:

Improve recommendation engines

📎 Project File

👉 View Full Project PDF

⭐ Why This Project Stands Out

Strong focus on feature engineering (very important for ML roles)

Converts raw data into model-ready format

Combines:

Data cleaning

Feature extraction

Business insights
