<script>
    import { createEventDispatcher } from 'svelte';
  
    let location = '';
    const dispatch = createEventDispatcher();
  
    function handleSubmit(event) {
      event.preventDefault();
      dispatch('submit', location);
    }
  </script>
  
  <form on:submit={handleSubmit}>
    <input type="text" bind:value={location} placeholder="Enter location" required />
    <button type="submit">Get Weather</button>
  </form>
  
  <style>
    form {
      display: flex;
      justify-content: center;
      margin-bottom: 1rem;
    }
  
    input {
      padding: 0.5rem;
      margin-right: 0.5rem;
    }
  
    button {
      padding: 0.5rem 1rem;
    }
  </style>
  