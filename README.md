# 🚀 Secure Weather Data Pipeline & API Server

<img width="398" alt="curlapi" src="https://github.com/user-attachments/assets/676c87b8-c212-496c-9905-68ab13ae4116" />

🧠 What You'll Build
-
A secure and production-style Python system that:

Pulls real-time weather data from an external API.

Cleans & normalizes the data using pandas.

Stores it in a secure local SQLite DB and CSV file.

Serves selected data through a Flask API.

Implements security measures (API key storage, rate limiting, logging, input validation).

✅ Core Concepts You'll Practice
-
Python scripting & OOP modularization

Consuming and parsing external APIs

Data transformation with pandas

Flask REST API with routes & query params

Secure API development practices

Scheduled jobs with cron or APScheduler

SQLite for structured storage

Environment variable security (.env)

Logging & error handling

 # 🧪 Step-by-Step Guide
 
# 1. Setup Project & Install Packages

mkdir weather_pipeline_project && cd weather_pipeline_project
python -m venv venv
source venv/bin/activate
pip install requests pandas flask python-dotenv sqlite3 flask-limiter

Add to requirements.txt:

requests
pandas
flask
python-dotenv
flask-limiter

<img width="343" alt="requirem" src="https://github.com/user-attachments/assets/f590010f-e673-45bc-b7dd-19c0bd2cd48e" />


# 2. Setup .env for API Key Security
.env

WEATHER_API_KEY=your_openweathermap_key_here

🔐 Why? Never hardcode secrets — .env files are safer and environment-based.


# 3. Fetch Data from the Weather API (app/fetch.py)

import requests
import os
from dotenv import load_dotenv

load_dotenv()

def fetch_weather_data(city="Washington"):
    API_KEY = os.getenv("WEATHER_API_KEY")
    url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric"
    response = requests.get(url)

    if response.status_code != 200:
        raise Exception(f"API Error: {response.status_code}, {response.text}")


    return response.json()

    
<img width="449" alt="fetch" src="https://github.com/user-attachments/assets/17683a5d-6ec1-4bcc-9e9d-9c51f918bbc1" />


<img width="453" alt="pyfetch" src="https://github.com/user-attachments/assets/f8b0df60-f84e-4df4-98e2-689df7e60568" />

    
# 4. Process/Clean Data (app/process.py)

import pandas as pd

def normalize_weather(raw_json):
    data = {
        'city': raw_json['name'],
        'temp_c': raw_json['main']['temp'],
        'humidity': raw_json['main']['humidity'],
        'weather': raw_json['weather'][0]['main'],
        'description': raw_json['weather'][0]['description'],
        'wind_speed': raw_json['wind']['speed'],
        'timestamp': pd.Timestamp.now()
    }

    df = pd.DataFrame([data])
    return df

<img width="429" alt="processpy" src="https://github.com/user-attachments/assets/5fba255a-6255-4559-8eed-523ce34cf717" />

# 5. Save to CSV & SQLite (app/save_data.py)

import sqlite3
import os

def save_to_csv(df, path='data/weather.csv'):
    df.to_csv(path, mode='a', header=not os.path.exists(path), index=False)

def save_to_db(df, db='data/weather.db'):
    conn = sqlite3.connect(db)
    df.to_sql('weather', conn, if_exists='append', index=False)
    conn.close()

<img width="324" alt="savedata" src="https://github.com/user-attachments/assets/d28e0147-272a-40e1-9f90-c734468b3edf" />

<img width="432" alt="savpy" src="https://github.com/user-attachments/assets/a4981887-7f05-4289-b187-e9a7a4606cea" />
   
# 6. Build Flask API Server (app/api.py)

from flask import Flask, jsonify, request
import sqlite3
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)
limiter = Limiter(get_remote_address, app=app, default_limits=["5 per minute"])

def fetch_all():
    conn = sqlite3.connect("data/weather.db")
    data = conn.execute("SELECT * FROM weather ORDER BY timestamp DESC LIMIT 10").fetchall()
    conn.close()
    return data

@app.route("/weather", methods=["GET"])
def latest_weather():
    data = fetch_all()
    return jsonify(data)

@app.route("/weather/city", methods=["GET"])
@limiter.limit("2 per minute")
def weather_by_city():
    city = request.args.get("name")
    if not city:
        return {"error": "Missing city name"}, 400
    conn = sqlite3.connect("data/weather.db")
    cur = conn.cursor()
    cur.execute("SELECT * FROM weather WHERE city=? ORDER BY timestamp DESC", (city,))
    rows = cur.fetchall()
    conn.close()
    return jsonify(rows)

<img width="454" alt="apipy" src="https://github.com/user-attachments/assets/f22bb666-8090-4d1a-87dd-190c45878554" />


<img width="398" alt="curlapi" src="https://github.com/user-attachments/assets/d2bceeac-c5c7-4b6e-848d-f3886cba781e" />

    
# 7. Combine into Main Pipeline (main.py)
python
Copy code
from app.fetch import fetch_weather_data
from app.process import normalize_weather
from app.storage import save_to_csv, save_to_db
from app.api import app

if __name__ == "__main__":
    city = "Washington"
    raw = fetch_weather_data(city)
    df = normalize_weather(raw)
    save_to_csv(df)
    save_to_db(df)

    # Optional: Run Flask API
    app.run(debug=True)

<img width="454" alt="mainp" src="https://github.com/user-attachments/assets/89397063-bdfd-4379-acf8-53a828ec2889" />

<img width="287" alt="flask" src="https://github.com/user-attachments/assets/24a50204-8a59-4c22-942c-2ccfc56d4ae5" />


🟢 Set up a Scheduled Job (Cron)
-
Automatically fetch and save new weather data every 30 minutes, hour, or daily.

🔧 Example:

*/30 * * * * /root/weather_pipeline_project/venv/bin/python /root/weather_pipeline

<img width="452" alt="crond" src="https://github.com/user-attachments/assets/9076655c-e1df-4e1e-942e-e4dbcd68e6ab" />


<img width="454" alt="cront" src="https://github.com/user-attachments/assets/f50d1c9e-4395-40f4-a931-93f1127395f4" />


🔒 Security Best Practices
-
Feature	Why it Matters
.env & python-dotenv	Keep secrets out of code
Flask rate limiting	Prevents brute-force API abuse
Input validation	Prevents SQL injection and bad API calls
Logging (future step)	Audit trails & debugging
Role-based routes (extra)	Add JWT or API key auth for protected endpoints


🧠 What You'll Learn By Completing This
-
Pulling, cleaning, and storing real-time external API data

Organizing Python projects using modular code

Building and securing REST APIs with Flask

Using .env, rate limiting, and input validation for security

Deploying or containerizing real systems
