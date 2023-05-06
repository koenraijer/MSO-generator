<script>
      import data from '$lib/data.json';
      import StandardText from '$lib/components/StandardText.svelte';
</script>

<div class="container mx-auto p-8 space-y-8 max-w-3xl">
    <StandardText paragraphs={data} />
</div>
