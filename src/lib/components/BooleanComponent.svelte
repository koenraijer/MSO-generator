<script>
    import { createEventDispatcher } from "svelte";
  
    export let content = "";
  
    let textVisible = true;
    const dispatch = createEventDispatcher();
  
    function toggleText() {
      textVisible = !textVisible;
      dispatch("update", textVisible ? content : "");
    }
  </script>
  
  {#if textVisible}
    <button
      on:click={toggleText}
      class="btn variant-filled mx-2 bg-surface-400 py-1 rounded-md px-2"
    >
      <span>
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="32"
          height="32"
          fill="currentColor"
          class="w-5 h-5"
          viewBox="0 0 256 256"
        >
          <path
            d="M228,128a12,12,0,0,1-12,12H140v76a12,12,0,0,1-24,0V140H40a12,12,0,0,1,0-24h76V40a12,12,0,0,1,24,0v76h76A12,12,0,0,1,228,128Z"
          ></path>
        </svg>
      </span>
      <span>{content}</span>
    </button>
  {:else}
    <span>&nbsp;{content}</span>
    <button
      on:click={toggleText}
      class="btn variant-filled p-1 mx-1 !bg-surface-600"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="32"
        height="32"
        fill="currentColor"
        class="w-4 h-4"
        viewBox="0 0 256 256"
      >
        <path
          d="M228,128a12,12,0,0,1-12,12H40a12,12,0,0,1,0-24H216A12,12,0,0,1,228,128Z"
        ></path>
      </svg>
    </button>
  {/if}
  