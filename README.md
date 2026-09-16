# SheBuilds
'''html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Weather Dashboard</title>

<style>
* {
box-sizing: border-box;
margin: 0;
padding: 0;
font-family: Arial, sans-serif;
}

body {
min-height: 100vh;
display: flex;
justify-content: center;
align-items: center;
background: linear-gradient(135deg, #74ebd5, #9face6);
padding: 20px;
}

.container {
width: 100%;
max-width: 500px;
background: white;
border-radius: 20px;
padding: 30px;
box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

h1 {
text-align: center;
color: #333;
margin-bottom: 25px;
}

.search-box {
display: flex;
gap: 10px;
margin-bottom: 20px;
}

.search-box input {
flex: 1;
padding: 12px;
border: 2px solid #ddd;
border-radius: 10px;
font-size: 16px;
outline: none;
}

.search-box input:focus {
border-color: #667eea;
}

.search-box button {
padding: 12px 18px;
border: none;
border-radius: 10px;
background: #667eea;
color: white;
font-size: 16px;
cursor: pointer;
}

.search-box button:hover {
background: #5568d8;
}

#message {
text-align: center;
margin: 15px 0;
color: #d9534f;
font-weight: bold;
}

.weather-card {
display: none;
text-align: center;
}

.weather-card h2 {
color: #333;
margin-bottom: 10px;
}

.weather-icon {
font-size: 70px;
margin: 10px 0;
}

.temperature {
font-size: 50px;
font-weight: bold;
color: #667eea;
margin-bottom: 10px;
}

.condition {
font-size: 20px;
color: #555;
margin-bottom: 25px;
}

.weather-details {
display: grid;
grid-template-columns: 1fr 1fr;
gap: 15px;
}

.detail {
background: #f5f6fa;
padding: 18px;
border-radius: 12px;
}

.detail span {
display: block;
font-size: 14px;
color: #777;
margin-bottom: 8px;
}

.detail strong {
font-size: 18px;
color: #333;
}

.loading {
display: none;
text-align: center;
color: #667eea;
margin: 15px 0;
font-weight: bold;
}

.footer {
text-align: center;
margin-top: 20px;
font-size: 12px;
color: #888;
}

@media (max-width: 450px) {
.container {
padding: 20px;
}

.search-box {
flex-direction: column;
}

.search-box button {
width: 100%;
}
}
</style>
</head>

<body>

<div class="container">

<h1>🌤️ Weather Dashboard</h1>

<div class="search-box">
<input
type="text"
id="cityInput"
placeholder="Enter city name"
>

<button onclick="getWeather()">
Search
</button>
</div>

<div class="loading" id="loading">
Fetching weather data...
</div>

<div id="message"></div>

<div class="weather-card" id="weatherCard">

<h2 id="cityName">City</h2>

<div class="weather-icon" id="weatherIcon">
☀️
</div>

<div class="temperature" id="temperature">
0°C
</div>

<div class="condition" id="condition">
Clear Sky
</div>

<div class="weather-details">

<div class="detail">
<span>💧 Humidity</span>
<strong id="humidity">0%</strong>
</div>

<div class="detail">
<span>💨 Wind Speed</span>
<strong id="wind">0 km/h</strong>
</div>

<div class="detail">
<span>🌡️ Feels Like</span>
<strong id="feelsLike">0°C</strong>
</div>

<div class="detail">
<span>☁️ Weather</span>
<strong id="weatherCode">Clear</strong>
</div>

</div>

</div>

<div class="footer">
Weather data provided by Open-Meteo
</div>

</div>


<script>

/*
* WEATHER DASHBOARD
* Public API: Open-Meteo
*
* No API key required.
*/

async function getWeather() {

const cityInput = document.getElementById("cityInput");
const city = cityInput.value.trim();

const message = document.getElementById("message");
const loading = document.getElementById("loading");
const weatherCard = document.getElementById("weatherCard");

// Clear previous messages
message.textContent = "";
weatherCard.style.display = "none";

// -------------------------------
// 1. Validate user input
// -------------------------------

if (city === "") {
message.textContent = "Please enter a city name.";
return;
}

// Show loading message
loading.style.display = "block";

try {

// -------------------------------
// 2. Find city coordinates
// -------------------------------

const geoURL =
`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}&count=1&language=en&format=json`;

const geoResponse = await fetch(geoURL);

if (!geoResponse.ok) {
throw new Error("Unable to connect to location service.");
}

const geoData = await geoResponse.json();

// Invalid city
if (!geoData.results || geoData.results.length === 0) {
throw new Error("City not found. Please enter a valid city name.");
}

const location = geoData.results[0];

const latitude = location.latitude;
const longitude = location.longitude;

// -------------------------------
// 3. Get weather information
// -------------------------------

const weatherURL =
`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m&timezone=auto`;

const weatherResponse = await fetch(weatherURL);

if (!weatherResponse.ok) {
throw new Error("Unable to fetch weather data.");
}

const weatherData = await weatherResponse.json();

// -------------------------------
// 4. Get current weather data
// -------------------------------

const current = weatherData.current;

const temperature = current.temperature_2m;
const humidity = current.relative_humidity_2m;
const feelsLike = current.apparent_temperature;
const windSpeed = current.wind_speed_10m;
const code = current.weather_code;

// -------------------------------
// 5. Convert weather code
// -------------------------------

const weatherInfo = getWeatherInfo(code);

// -------------------------------
// 6. Display data
// -------------------------------

document.getElementById("cityName").textContent =
`${location.name}, ${location.country}`;

document.getElementById("temperature").textContent =
`${temperature}°C`;

document.getElementById("humidity").textContent =
`${humidity}%`;

document.getElementById("feelsLike").textContent =
`${feelsLike}°C`;

document.getElementById("wind").textContent =
`${windSpeed} km/h`;

document.getElementById("weatherIcon").textContent =
weatherInfo.icon;

document.getElementById("condition").textContent =
weatherInfo.description;

document.getElementById("weatherCode").textContent =
weatherInfo.description;

// Show weather card
weatherCard.style.display = "block";

}

catch (error) {

// -------------------------------
// 7. Error handling
// -------------------------------

message.textContent = error.message;

}

finally {

// Hide loading message
loading.style.display = "none";

}
}


// ----------------------------------------
// Weather code conversion function
// ----------------------------------------

function getWeatherInfo(code) {

if (code === 0) {
return {
description: "Clear Sky",
icon: "☀️"
};
}

if (code === 1 || code === 2) {
return {
description: "Partly Cloudy",
icon: "⛅"
};
}

if (code === 3) {
return {
description: "Overcast",
icon: "☁️"
};
}

if (code === 45 || code === 48) {
return {
description: "Foggy",
icon: "🌫️"
};
}

if (code >= 51 && code <= 57) {
return {
description: "Drizzle",
icon: "🌦️"
};
}

if (code >= 61 && code <= 67) {
return {
description: "Rain",
icon: "🌧️"
};
}

if (code >= 71 && code <= 77) {
return {
description: "Snow",
icon: "❄️"
};
}

if (code >= 80 && code <= 82) {
return {
description: "Rain Showers",
icon: "🌦️"
};
}

if (code === 85 || code === 86) {
return {
description: "Snow Showers",
icon: "🌨️"
};
}

if (code >= 95 && code <= 99) {
return {
description: "Thunderstorm",
icon: "⛈️"
};
}

return {
description: "Unknown Weather",
icon: "🌤️"
};
}


// ----------------------------------------
// Search when Enter key is pressed
// ----------------------------------------

document.getElementById("cityInput").addEventListener(
"keypress",
function(event) {

if (event.key === "Enter") {
getWeather();
}

}
);

</script>

</body>
</html>
