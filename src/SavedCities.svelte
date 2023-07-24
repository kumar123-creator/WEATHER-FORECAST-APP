<script>
  export let savedCities;
  export let weatherData;
  export let city;
  export let temperatureUnit;

  async function displayWeatherForCity(selectedCity) {
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

await fetchWeatherData();
}
  

  function removeCity(savedCityName) {
    savedCities = savedCities.filter((cityData) => cityData.name !== savedCityName);
  }
</script>

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

<style>
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
