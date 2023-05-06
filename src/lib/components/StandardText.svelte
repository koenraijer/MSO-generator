<script>
    export let paragraphs;
  
    import SelectComponent from "$lib/components/SelectComponent.svelte";
    import TextComponent from "$lib/components/TextComponent.svelte";
    import BooleanComponent from "$lib/components/BooleanComponent.svelte";
    import { onMount, afterUpdate } from "svelte";
    import {fade} from "svelte/transition";
    import { clipboard } from '@skeletonlabs/skeleton';

    let localParagraphs = JSON.parse(paragraphs.json);
        
    console.log(localParagraphs)

    let jsonString = "";
  
    function handleSelectChange(paragraphIndex, elementIndex, value, type) {
        if (type === "boolean") {
            localParagraphs[paragraphIndex].elements[elementIndex].current = value;
        } else {
            localParagraphs[paragraphIndex].elements[elementIndex].current = value;
        }
    }

    const getUpdatedJSONString = () => {
      jsonString = ""; // Clear jsonString before reconstructing it
      for (const paragraph of localParagraphs) {
        for (const element of paragraph.elements) {
          if (element.type === "text") {
            jsonString += element.content;
          } else if (element.type === "select") {
            jsonString += element.content[element.current].value;
          } else if (element.type === "boolean") {
            if (element.current === 1) {
              jsonString += element.content;
            }
          }
        }
      }
    };
  
    onMount(() => {
      getUpdatedJSONString();
    });
  
    afterUpdate(() => {
      getUpdatedJSONString();
    });
  
    $: {
      localParagraphs;
      getUpdatedJSONString();
    }

    let showThumbsUp = false;

    function handleCopyButtonClick() {
        showThumbsUp = true;
        setTimeout(() => {
        showThumbsUp = false;
        }, 1500);
    }
  </script>
  
  <div class="pt-4">
    {#each localParagraphs as paragraph, paragraphIndex}
      <h2 class="mb-4">{paragraph.name}</h2>
      <p class="!leading-10">
        {#each paragraph.elements as element, elementIndex}
          <span class="group inline relative">
            {#if element.type === 'text'}
              <TextComponent content={element.content} />
            {:else if element.type === 'boolean'}
              &nbsp;
              <BooleanComponent
                content={element.content}
                on:update={(event) => handleSelectChange(paragraphIndex, elementIndex, event.detail, "boolean")}
              />
              &nbsp;
            {:else if element.type === 'select'}
              &nbsp;<SelectComponent
                bindValue={element.content[localParagraphs[paragraphIndex].elements[elementIndex].current].value}
                options={element.content}
                current={localParagraphs[paragraphIndex].elements[elementIndex].current}
                on:change={(event) => handleSelectChange(paragraphIndex, elementIndex, event.detail)}
              />
              {/if}
            </span>
          {/each}
        </p>
      {/each}
      </div>
      
        	<!-- Source -->
	<textarea class="textarea" rows="15" value={jsonString} data-clipboard="exampleInput"></textarea>

	<!-- Trigger -->
	<button class="btn variant-filled" on:click={handleCopyButtonClick} use:clipboard={jsonString}>Copy</button>
	{#if showThumbsUp}
	<span class="text-xl bg-surface-200 rounded-full p-1" out:fade={{ duration: 250 }}>&#x1F44D;</span>
	{/if}
