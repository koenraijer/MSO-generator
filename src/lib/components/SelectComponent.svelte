<!-- SelectComponent.svelte -->
<script>
    export let options = [];
    export let bindValue;
    export let current;
  
    $: {
      const selectedIndex = options.findIndex(option => option.value === bindValue);
      if (selectedIndex !== -1) {
        current = selectedIndex;
      }
    }
  </script>
  
  <select class="select" bind:value={bindValue}>
    {#each options as option}
      <option value={option.value}>{option.label}</option>
    {/each}
  </select>
  