# Weather-pattern-predictor
# It is a data analytics project that generates synthetic weather data for 30 days using Python.

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import mysql.connector

# generating data
np.random.seed(42)

dates = pd.date_range(start="2026-01-01", periods=30)

df = pd.DataFrame({
    "Date": dates,
    "Temperature": np.random.normal(30, 5, 30).round(1),
    "Humidity": np.random.randint(40, 90, 30),
    "Rainfall": np.random.exponential(5, 30).round(1),
    "WindSpeed": np.random.uniform(5, 20, 30).round(1)
})

df["WeatherCondition"] = np.where(
    df["Rainfall"] > 5, "Rainy",
    np.where(df["Humidity"] > 70, "Cloudy", "Sunny")
)

print("Synthetic Data Generated:")
print(df.head())

# data analysis
print("\nData Statistics:")
print(df.describe())

# 
plt.figure(figsize=(10,5))
plt.plot(df["Date"], df["Temperature"], marker='o')
plt.title("30-Day Temperature Trend")
plt.xlabel("Date")
plt.ylabel("Temperature (°C)")
plt.xticks(rotation=45)
plt.show()

# temperature predictions
df["Temp_Prediction"] = df["Temperature"].rolling(window=3).mean()

# data storing in db
try:
    conn = mysql.connector.connect(
        host="localhost",
        user="root",
        password="Kritika@mysql",
        database="weather_db"
    )
    cursor = conn.cursor()

    for _, row in df.iterrows():
        cursor.execute("""
        INSERT INTO weather_data 
        (date, temperature, humidity, rainfall, windspeed, weather_condition)
        VALUES (%s, %s, %s, %s, %s, %s)
        """, (
            row["Date"].date(),
            row["Temperature"],
            row["Humidity"],
            row["Rainfall"],
            row["WindSpeed"],
            row["WeatherCondition"]
        ))

    conn.commit()
    print("Data successfully stored in MySQL")

except Exception as e:
    print("Database error:", e)

finally:
    if 'conn' in locals():
        conn.close()

