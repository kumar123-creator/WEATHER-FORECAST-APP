<script>
    export let weather;
  
    function getIconUrl(icon) {
      return `https://openweathermap.org/img/wn/${icon}.png`;
    }
  </script>
  
  <div class="weather-card">
    <h2>{weather.city.name}, {weather.city.country}</h2>
    <div class="current-weather">
      <img src={getIconUrl(weather.list[0].weather[0].icon)} alt={weather.list[0].weather[0].description} />
      <p>{Math.round(weather.list[0].main.temp - 273.15)}°C</p>
      <p>{weather.list[0].weather[0].description}</p>
    </div>
    <div class="forecast">
      {#each weather.list.slice(1, 6) as item}
        <div class="forecast-item">
          <p>{new Date(item.dt_txt).toLocaleDateString()}</p>
          <img src={getIconUrl(item.weather[0].icon)} alt={item.weather[0].description} />
          <p>{Math.round(item.main.temp - 273.15)}°C</p>
          <p>{item.weather[0].description}</p>
        </div>
      {/each}
    </div>
  </div>
  
  <style>
    .weather-card {
      margin-top: 2rem;
    }
  
    .current-weather {
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 2rem;
    }
  
    .current-weather img {
      width: 100px;
      height: 100px;
    }
  
    .forecast {
      display: flex;
      justify-content: space-between;
    }
  
    .forecast-item {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 0 0.5rem;
    }
  </style>
  