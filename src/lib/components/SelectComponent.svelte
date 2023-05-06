<script>
    export let options = [];
    export let bindValue;
    export let current;
    export let disabled = false;
    import { createEventDispatcher } from 'svelte';
  
    const dispatch = createEventDispatcher();
    
    const handleSelectChange = (event) => {
      current = event.target.selectedIndex;
      bindValue = options[current].value;
      dispatch('change', current); // Dispatch custom 'change' event with the current index
    };
  
    $: {
      const selectedIndex = options.findIndex((option) => option.value === bindValue);
      if (selectedIndex !== -1 && current !== selectedIndex) {
        current = selectedIndex;
      }
    }
  </script>
  
  <select class="select" bind:value={bindValue} disabled={disabled} on:change={handleSelectChange}>
    {#each options as option, index}
      <option value={option.value} selected={index === current}>{option.label}</option>
    {/each}
  </select>
  