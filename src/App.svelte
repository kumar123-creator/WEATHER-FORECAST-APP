<script>
	let city = '';
	let weatherData = null;
	let errorMessage = '';
	let isLoading = false;
	let temperatureUnit = 'C';
  
	async function fetchWeatherData() {
	  isLoading = true;
	  try {
		const response = await fetch(
		  `https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=38326992ca5cb4961d8fd698ae59edfc`
		);
		if (!response.ok) {
		  throw new Error('Failed to fetch weather data');
		}
		const rawData = await response.json();
		weatherData = processWeatherData(rawData);
		errorMessage = '';
		weatherData.list.forEach((forecast) => {
		  forecast.temperature = getTemperature(forecast.temperature);
		});
	  } catch (error) {
		errorMessage = 'Failed to fetch weather data. Please try again later.';
		weatherData = null;
	  } finally {
		isLoading = false;
	  }
	}
  
	function processWeatherData(rawData) {
	  const dailyForecast = [];
	  const dates = new Set();
  
	  for (const forecast of rawData.list) {
		const date = new Date(forecast.dt * 1000).toDateString();
		if (!dates.has(date)) {
		  dates.add(date);
		  dailyForecast.push({
			date: date,
			temperature: (forecast.main.temp - 273.15).toFixed(2),
			weather: forecast.weather[0].main,
			humidity: forecast.main.humidity,
			windSpeed: forecast.wind.speed,
			icon: `https://openweathermap.org/img/wn/${forecast.weather[0].icon}.png`,
		  });
		}
	  }
  
	  return {
		city: rawData.city,
		list: dailyForecast,
	  };
	}
  
	function getTemperature(temperature) {
	  if (temperatureUnit === 'F') {
		return ((temperature * 9) / 5 + 32).toFixed(2);
	  } else {
		return temperature;
	  }
	}
  </script>
  
  <main>
	<h1>5-DAY WEATHER FORECAST</h1>
  
	<div class="input-container">
	  <input type="text" bind:value={city} placeholder="Enter city name" />
	  <button on:click={fetchWeatherData} disabled={isLoading}>
		{#if isLoading}
		  <span class="loader"></span> Loading...
		{:else}
		  Get Weather
		{/if}
	  </button>
	</div>
  
	<select bind:value={temperatureUnit}>
	  <option value="C">Celsius</option>
	  <option value="F">Fahrenheit</option>
	</select>
  
	{#if weatherData}
	  <div class="city-name">
		<h2>{weatherData.city.name}</h2>
	  </div>
	  <div class="weather-container">
		{#each weatherData.list as forecast}
		  <div class="forecast-item">
			<p>Date: {forecast.date}</p>
			<img src={forecast.icon} alt={forecast.weather} />
			<p>Temperature: {getTemperature(forecast.temperature)}{temperatureUnit === 'C' ? '°C' : '°F'}</p>
			<p>Weather: {forecast.weather}</p>
			<p>Humidity: {forecast.humidity}%</p>
			<p>Wind Speed: {forecast.windSpeed} m/s</p>
		  </div>
		{/each}
	  </div>
	{/if}
  
	{#if errorMessage}
	  <p class="error-message">{errorMessage}</p>
	{/if}
  </main>
  
  <style>
	main {
	  text-align: center;
	  padding: 20px;
	  font-family: Arial, sans-serif;
	  background-color: #f5f5f5;
	}
  
	h1 {
	  font-size: 28px;
	  margin-bottom: 20px;
	  color: #333;
	}
  
	.input-container {
	  display: flex;
	  align-items: center;
	  justify-content: center;
	  margin-bottom: 20px;
	}
  
	input {
	  padding: 10px;
	  margin-right: 10px;
	  border: 1px solid #ccc;
	  border-radius: 4px;
	  outline: none;
	}
  
	button {
	  padding: 10px 20px;
	  background-color: #4caf50;
	  color: white;
	  border: none;
	  border-radius: 4px;
	  cursor: pointer;
	  display: flex;
	  align-items: center;
	}
  
	.loader {
	  border: 2px solid rgba(255, 255, 255, 0.3);
	  border-top: 2px solid #fff;
	  border-radius: 50%;
	  width: 12px;
	  height: 12px;
	  margin-right: 5px;
	  animation: spin 1s linear infinite;
	}
  
	@keyframes spin {
	  0% {
		transform: rotate(0deg);
	  }
	  100% {
		transform: rotate(360deg);
	  }
	}
  
	.city-name {
	  margin-bottom: 20px;
	}
  
	.weather-container {
	  display: flex;
	  flex-wrap: wrap;
	  justify-content: center;
	}
  
	.forecast-item {
	  width: 200px;
	  padding: 10px;
	  margin: 10px;
	  border: 1px solid #ccc;
	  border-radius: 4px;
	  text-align: center;
	  background-color: #fff;
	  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
	}
  
	.forecast-item img {
	  width: 50px;
	  height: 50px;
	  margin-bottom: 10px;
	}
  
	.forecast-item p {
	  margin: 5px 0;
	}
  
	.error-message {
	  color: #ff0000;
	  margin-top: 10px;
	}
  </style>