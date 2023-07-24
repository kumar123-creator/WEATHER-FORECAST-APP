<script>
	let city = '';
	let weatherData = null;
	let errorMessage = '';
	let isLoading = false;
	let temperatureUnit = 'C';
	let savedCities = [];

	loadCitiesFromLocalStorage();
	
	$: {
    console.log('Temperature Unit Changed:', temperatureUnit);
    updateTemperatureDisplay();
  }

  function displayWeatherForCity(selectedCity) {
    city = selectedCity;

    // Add a class to the selected city for styling
    const cityElements = document.querySelectorAll('.city-saving-section li');
    cityElements.forEach((element) => {
      if (element.innerText === selectedCity) {
        element.classList.add('selected');
      } else {
        element.classList.remove('selected');
      }
    });

     fetchWeatherData();
    // The temperature display will be automatically updated by the reactive statement
  }

  function updateTemperatureDisplay() {
    if (weatherData) {
      // Update the current weather temperature in the UI
      const currentTempElement = document.querySelector('.city-name h2');
      currentTempElement.innerText = `${weatherData.city.name} (${getTemperature(
        weatherData.list[0].temperature
      )}${temperatureUnit === 'C' ? '°C' : '°F'})`;

      // Update the forecast temperature values in the UI
      const forecastItems = document.querySelectorAll('.forecast-item');
      forecastItems.forEach((item, index) => {
        const forecast = weatherData.list[index];
        const tempElement = item.querySelector('p:nth-child(3)');
        tempElement.innerText = `Temperature: ${getTemperature(
          forecast.temperature
        )}${temperatureUnit === 'C' ? '°C' : '°F'}`;
      });

      // Update the saved cities temperature values in the UI
      const savedCityItems = document.querySelectorAll('.city-saving-section li');
      savedCityItems.forEach((item) => {
        const cityName = item.querySelector('.city-name').textContent.split(' - ')[0];
        const cityData = savedCities.find((city) => city.name === cityName);
        if (cityData) {
          const tempElement = item.querySelector('span');
          tempElement.innerText = `${cityData.name} - ${getTemperature(cityData.temperature)}${
            temperatureUnit === 'C' ? '°C' : '°F'
          }`;
        }
      });
    }
  }
  

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

      const cityIndex = savedCities.findIndex((cityData) => cityData.name === weatherData.city.name);
      if (cityIndex !== -1) {
        savedCities.splice(cityIndex, 1); // Remove existing data for the selected city
      }

      const cityData = {
        name: weatherData.city.name,
        temperature: weatherData.list[0].temperature,
      };
      savedCities.push(cityData);

      // Limit the number of saved cities to 5
      if (savedCities.length > 5) {
        savedCities.shift(); // Remove the oldest city to maintain 5 saved cities
      }

      // Clear the input placeholder after successful fetch
      city = '';
	  saveCitiesToLocalStorage();
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

		  // Check if 5 days have been added and stop adding more data
		  if (dailyForecast.length === 5) {
          break;
        }
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
	
	function saveCitiesToLocalStorage() {
  localStorage.setItem('savedCities', JSON.stringify(savedCities));
}

function loadCitiesFromLocalStorage() {
  const savedCitiesData = localStorage.getItem('savedCities');
  if (savedCitiesData) {
    savedCities = JSON.parse(savedCitiesData);
  }
}

  
function removeCity(savedCityName) {
  savedCities = savedCities.filter((cityData) => cityData.name !== savedCityName);
  saveCitiesToLocalStorage(); // Save the updated list after removing the city
}


  </script>
  
  <main>
<!-- Saved Cities Column -->
<div class="saved-cities-section">
	<h2>Saved Cities</h2>
	<ul class="city-saving-section">
	  {#each savedCities as cityData}
	  <li on:click={() => displayWeatherForCity(cityData.name)} class:selected={cityData.name === (weatherData ? weatherData.city.name : '')}>
		<span class="city-name"> 
		  {cityData.name} - {getTemperature(cityData.temperature)}{temperatureUnit === 'C' ? '°C' : '°F'}
		</span>
		<button class="remove-button" on:click={() => removeCity(cityData.name)}>Remove</button>
	  </li>
	  {/each}
	</ul>
  </div>
  


	<div class="weather-forecast-section">
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
	</div>
  </main>
  
  <style>
	main {
    text-align: center;
    padding: 20px;
    font-family: Arial, sans-serif;
    background-image: url('https://static.vecteezy.com/system/resources/previews/006/893/924/large_2x/clear-blue-sky-with-plain-white-cloud-with-space-for-text-background-the-vast-blue-sky-and-clouds-blue-sky-background-with-tiny-clouds-nature-free-photo.jpg'); /* Replace 'path_to_your_blue_sky_image.jpg' with the actual path to your image */
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
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

	
	main {
    display: flex; 
    align-items: flex-start; 
    justify-content: center; 
    min-height: 100vh; 
  }

  .weather-forecast-section {
    flex: 2;
    text-align: center;
    padding: 20px;
    background-color: rgba(255, 255, 255, 0.8); 
    border-radius: 8px; 
	margin-left: 10px;

  }

  .saved-cities-section {
    flex: 1;
    text-align: left;
    padding: 20px;
    background-color: rgba(255, 255, 255, 0.8);
    border-radius: 8px;
	margin-right: 10px;
  }

  .city-saving-section {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .city-saving-section li {
    cursor: pointer;
    padding: 5px;
  }

  .city-saving-section li.selected {
    background-color: purple;
    color: white;
  }

  .selected {
  background-color: purple;
  color: white;
}

.city-saving-section li {
  cursor: pointer;
  padding: 5px;
  transition: background-color 0.3s; /* Add a transition for smooth color change */
}

.city-saving-section li:hover {
  background-color: wheat; /* Change background color on hover */
}

city-name {
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