<script>
  export let savedCities = [];
  export let weatherData = null;
  export let city = '';
  export let temperatureUnit = 'C';

  async function displayWeatherForCity(selectedCity) {
    city = selectedCity;
    const event = new CustomEvent('citySelected', {
      detail: { city },
    });
    dispatchEvent(event);
  }

  function getTemperature(temperature) {
    if (temperatureUnit === 'F') {
      return ((temperature * 9) / 5 + 32).toFixed(2);
    } else {
      return temperature;
    }
  }
</script>

<h2>Saved Cities</h2>
<ul class="city-saving-section">
  {#each savedCities as cityData}
  <li
    on:click={() => displayWeatherForCity(cityData.name)}
    class:selected={cityData.name === (weatherData ? weatherData.city.name : '')}
  >
    <span class="city-name">
      {cityData.name} - {getTemperature(cityData.temperature)}{temperatureUnit === 'C' ? '°C' : '°F'}
    </span>
    <button class="remove-button" on:click={() => removeCity(cityData.name)}>Remove</button>
  </li>
  {/each}
</ul>

<style>
  h2 {
    font-size: 20px;
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
</style>
