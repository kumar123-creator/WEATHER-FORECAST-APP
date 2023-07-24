<script>
  export let weatherData = null;
  export let temperatureUnit = 'C';
  export let city = '';
  export let isLoading = false;
  export let errorMessage = '';
</script>

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
  <h2>{weatherData.city.name} ({getTemperature(weatherData.list[0].temperature)}{temperatureUnit === 'C' ? '°C' : '°F'})</h2>
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

<style>
  h1 {
    font-size: 28px;
    margin-bottom: 20px;
    color: #333;
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

  .city-name {
    display: inline-block;
    margin-right: 10px;
  }

  .remove-button {
    padding: 5px 8px;
    font-size: 12px;
    background-color: #f44336;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
</style>
