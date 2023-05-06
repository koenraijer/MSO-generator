<!-- Moon.svelte -->
<script>
  export let size = 24;
  export let borderWidth = 4;
</script>
<style>
  .spinner {
      width: 100%;
      height: 100%;
      border: var(--border-width) solid rgba(0, 0, 0, 0.1);
      border-left-color: #6366f1;
      border-radius: 50%;
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
</style>

<div
  class="spinner-container"
  style="width: {size}px; height: {size}px;">
  <div
      class="spinner"
      style="--border-width: {borderWidth}px;">
  </div>
</div>
