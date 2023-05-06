<script>
    import { page } from '$app/stores'
    import { supabase } from '$lib/supabaseClient.js'
    import StandardText from '$lib/components/StandardText.svelte';
    import Moon from '$lib/components/Moon.svelte'
    import { onMount } from "svelte";
    let fetchedData
    let loading = true
    onMount(async () => {
      try {
        const { data, error } = await supabase
            .from("Standardtexts")
            .select("json")
            .eq("uuid", $page.params.slug);
  
        if (error) {
          console.error("Error fetching data:", error);
          throw new Error("Error fetching data");
        } else {
          console.log("data", data);
          fetchedData = data;
          console.log("fetchedData", fetchedData);
          loading = false;
        }
      } catch (error) {
        console.error("Error during onMount:", error);
      }
    });
   /**
    *  const { data, error } = await supabase
  .from('countries')
  .select()
   */
</script>

<div class="container mx-auto p-8 space-y-8 max-w-3xl">

    {#if !loading && fetchedData}
        <StandardText paragraphs={fetchedData[0]} />
    {:else}
        <Moon />
    {/if}

</div>

