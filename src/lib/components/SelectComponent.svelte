<!-- SelectComponent.svelte -->
<script>
    export let options = [];
    export let bindValue;
  
  </script>
  
  <select class="select" bind:value={bindValue}>
    {#each options as option}
      <option value={option.value}>{option.label}</option>
    {/each}
  </select>
  