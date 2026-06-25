app.py`
```python
from flask import Flask, send_from_directory, jsonify, request
import requests
import os

app = Flask(__name__, static_folder='site')

# Load API key from environment variable; fallback to placeholder for CI/CD
API_KEY = os.getenv('OPENWEATHER_API_KEY', 'cf581824457f24318f1e320f1da898d5')

@app.route('/')
def index():
    return send_from_directory(app.static_folder, 'index.html')

@app.route('/<path:filename>')
def static_files(filename):
    return send_from_directory(app.static_folder, filename)

@app.route('/weather')
def weather():
    city = request.args.get('city')
    if not city:
        return jsonify({'error': 'city parameter is required'}), 400
    url = (
        f"https://api.openweathermap.org/data/2.5/weather"
        f"?q={city}&appid={API_KEY}&units=metric"
    )
    try:
        resp = requests.get(url, timeout=10)
        resp.raise_for_status()
        data = resp.json()
        return jsonify(data)
    except requests.HTTPError as e:
        return jsonify({'error': str(e), 'status_code': resp.status_code}), resp.status_code
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=True)
```

### `weather_api.py`
```python
"""weather_api.py
A thin wrapper around the OpenWeatherMap API.
Provides a convenient `get_weather` function.
"""

import os
import requests
from typing import Dict, Optional

API_KEY = os.getenv('OPENWEATHER_API_KEY', 'YOUR_API_KEY')

def _build_url(city: str) -> str:
    return (
        f"https://api.openweathermap.org/data/2.5/weather"
        f"?q={city}&appid={API_KEY}&units=metric"
    )

def get_weather(city: str) -> Optional[Dict[str, any]]:
    """Fetch current weather for *city* and return a curated dictionary.
    Returns ``None`` on error.
    """
    url = _build_url(city)
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
    except requests.RequestException as exc:
        print(f"Error fetching weather for {city!r}: {exc}")
        return None
    data = response.json()
    return {
        "city": data.get('name'),
        "temperature": data.get('main', {}).get('temp'),
        "humidity": data.get('main', {}).get('humidity'),
        "description": data.get('weather', [{}])[0].get('description'),
        "wind_speed": data.get('wind', {}).get('speed'),
    }

if __name__ == '__main__':
    city_input = input('Enter city name: ')
    result = get_weather(city_input)
    if result:
        print('\nWeather Details')
        print('-' * 20)
        for k, v in result.items():
            print(f"{k.capitalize()}: {v}")
    else:
        print('City not found or request failed!')
```

### `site/index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Live Weather Dashboard</title>
    <link rel="stylesheet" href="style.css" />
    <meta name="description" content="A premium glass‑morphism weather dashboard that shows real‑time data from OpenWeatherMap." />
</head>
<body>
    <main class="container">
        <section class="card">
            <h1>Live Weather 🌤️</h1>
            <div class="input-group">
                <input type="text" id="city-input" placeholder="Enter city name" />
                <button id="search-btn">Get Weather</button>
            </div>
            <div id="weather-result" class="weather-result hidden">
                <h2 id="city-name"></h2>
                <p class="temp" id="temp"></p>
                <p id="description"></p>
                <p id="humidity"></p>
                <p id="wind"></p>
            </div>
            <p id="error-msg" class="error hidden"></p>
        </section>
    </main>
    <script src="script.js"></script>
</body>
</html>
```

### `site/script.js`
```javascript
// script.js – Fetches weather data from the Flask backend and updates the UI

const cityInput = document.getElementById("city-input");
const searchBtn = document.getElementById("search-btn");
const weatherResult = document.getElementById("weather-result");
const errorMsg = document.getElementById("error-msg");

const cityNameEl = document.getElementById("city-name");
const tempEl = document.getElementById("temp");
const descEl = document.getElementById("description");
const humidityEl = document.getElementById("humidity");
const windEl = document.getElementById("wind");

function showError(message) {
  errorMsg.textContent = message;
  errorMsg.classList.remove("hidden");
  weatherResult.classList.add("hidden");
}

function showWeather(data) {
  cityNameEl.textContent = data.name;
  tempEl.textContent = `${data.main.temp.toFixed(1)} \u00B0C`;
  descEl.textContent = data.weather[0].description;
  humidityEl.textContent = `Humidity: ${data.main.humidity}%`;
  windEl.textContent = `Wind: ${data.wind.speed} m/s`;
  errorMsg.classList.add("hidden");
  weatherResult.classList.remove("hidden");
}

async function fetchWeather(city) {
  const url = `/weather?city=${encodeURIComponent(city)}`;
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`Server responded ${response.status}`);
    }
    const data = await response.json();
    showWeather(data);
  } catch (err) {
    showError("Could not retrieve weather data. Please check the city name and API key.");
    console.error(err);
  }
}

searchBtn.addEventListener("click", () => {
  const city = cityInput.value.trim();
  if (!city) {
    showError("Please enter a city name.");
    return;
  }
  fetchWeather(city);
});

cityInput.addEventListener("keypress", (e) => {
  if (e.key === "Enter") {
    searchBtn.click();
  }
});
```

### `Dockerfile`
```dockerfile
# Use a lightweight Python base image
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy source code
COPY . .

# Environment variable – replace with your real key at runtime
ENV OPENWEATHER_API_KEY="YOUR_API_KEY"

EXPOSE 5000
CMD ["python", "app.py"]
```
# weather-information-
