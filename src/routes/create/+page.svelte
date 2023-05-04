<script>
    import { writable } from "svelte/store";
    import { localStorageStore } from '@skeletonlabs/skeleton';
    import SelectComponent from "$lib/components/SelectComponent.svelte";
    import TextComponent from "$lib/components/TextComponent.svelte";
    import BooleanComponent from "$lib/components/BooleanComponent.svelte";

    let elementType = "text";
    let elementContent = "";
    let selectOptions = writable([]);
    let tempSelectOption = "";
  
    let paragraphs = localStorageStore('paragraphs', [
    {
        name: "Paragraph 1",
        elements: [],
    },
    ]);

    let currentParagraphIndex = 0;
    let currentParagraphName = "";
  
    function addParagraph() {
      const newParagraph = {
        name: `Paragraph ${$paragraphs.length + 1}`,
        elements: [],
      };
      paragraphs.update((para) => [...para, newParagraph]);
      currentParagraphIndex = $paragraphs.length - 1;
    }
    
    function deleteParagraph(index) {
        if (index >= 0 && index < $paragraphs.length) {
            paragraphs.update((para) => para.filter((_, i) => i !== index));
            if (currentParagraphIndex === index) {
                currentParagraphIndex = -1;
                currentParagraphName = "";
            } else if (currentParagraphIndex > index) {
                currentParagraphIndex--;
            }
        }
    }

    function addElement() {
        const element = {
            type: elementType,
            content: elementType === "select" ? $selectOptions : elementContent,
        };

        if (elementType === "select") {
            element.current = 0; // Add the 'current' property with a default value of 0
        }

        paragraphs.update((para) => {
            para[currentParagraphIndex].elements.push(element);
            return [...para];
        });

        elementContent = "";
        selectOptions.set([]);
        }
  
    function addOption() {
      if (tempSelectOption) {
        selectOptions.update((opts) => [
          ...opts,
          { value: tempSelectOption, label: tempSelectOption },
        ]);
        tempSelectOption = "";
      }
    }
  
    function removeOption(index) {
      selectOptions.update((opts) => opts.filter((_, i) => i !== index));
    }
  
    function removeElement(paragraphIndex, elementIndex) {
        paragraphs.update((para) => {
        para[paragraphIndex].elements.splice(elementIndex, 1);
        return [...para];
        });
    }
  
    function switchParagraph(index) {
        currentParagraphIndex = index;
        currentParagraphName = $paragraphs[currentParagraphIndex].name;
    }

    $: if (currentParagraphIndex !== -1) {
    paragraphs.update((para) => {
        para[currentParagraphIndex].name = currentParagraphName;
        return [...para];
    });
}

  </script>
  
  <div class="container mx-auto p-8 space-y-8 max-w-3xl">
    <h1 class="text-2xl mb-4">Add JSON elements</h1>
  
    <div class="mb-4">
      <ul class="mb-4 btn-group">
        {#each $paragraphs as paragraph, index}
        <li class="inline-block variant-soft-surface relative group">
            <div class="flex flex-row flex-nowrap">
            <button
                type="button"
                class="p-2 !pr-1 !mr-0"
                on:click={() => switchParagraph(index)}
            >{paragraph.name || `Paragraph ${index + 1}`}</button>
            <span on:click={() => deleteParagraph(index)} class="cursor-pointer p-1 inline !border-none self-center pr-2">
                <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-4 h-4" viewBox="0 0 256 256"><path d="M216,48H40a12,12,0,0,0,0,24h4V208a20,20,0,0,0,20,20H192a20,20,0,0,0,20-20V72h4a12,12,0,0,0,0-24ZM188,204H68V72H188ZM76,20A12,12,0,0,1,88,8h80a12,12,0,0,1,0,24H88A12,12,0,0,1,76,20Z"></path></svg>
            </span>
            </div>
        </li>
        {/each}
        <li class="text-white hover:text-black">
            <button type="button" class="p-2 btn variant-filled-secondary rounded-r-full !rounded-l-none" on:click={addParagraph}>
                <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5 mr-2" viewBox="0 0 256 256"><path d="M128,20A108,108,0,1,0,236,128,108.12,108.12,0,0,0,128,20Zm0,192a84,84,0,1,1,84-84A84.09,84.09,0,0,1,128,212Zm52-84a12,12,0,0,1-12,12H140v28a12,12,0,0,1-24,0V140H88a12,12,0,0,1,0-24h28V88a12,12,0,0,1,24,0v28h28A12,12,0,0,1,180,128Z"></path></svg>
                Add paragraph
            </button>
        </li>
      </ul>
    </div>
  
    {#if currentParagraphIndex !== -1}
      <div class="mb-4">
        <label for="paragraphName" class="block mb-2">Paragraph Name</label>
        <input
            type="text"
            bind:value={currentParagraphName}
            id="paragraphName"
            class="input !border-none rounded-md"
            placeholder="Enter paragraph name"
        />
      </div>
    {/if}
  
    <div class="mb-4">
      <label for="type" class="block mb-2">Element Type</label>
      <select bind:value={elementType} id="type" class="input !border-none rounded-md"
      >
        <option value="text">Text</option>
        <option value="select">Select</option>
        <option value="boolean">Boolean</option>
      </select>
    </div>
  
    {#if elementType !== "select"}
      <div class="mb-4">
        <label for="content" class="block mb-2">Content</label>
        <input
          type="text"
          bind:value={elementContent}
          id="content"
          class="input !border-none rounded-md"
          placeholder="Enter content"
        />
      </div>
    {:else}
      <div class="mb-4">
        <label for="option" class="block mb-2">Select Options</label>
        <div class="flex">
          <input
            type="text"
            bind:value={tempSelectOption}
            id="option"
            class="input !border-none !rounded-l-md rounded-r-none"
            placeholder="Enter option"
          />
          <button type="button" class="p-2  text-white btn variant-filled-secondary rounded-l-none px-3" on:click={addOption}>
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256"><path d="M128,20A108,108,0,1,0,236,128,108.12,108.12,0,0,0,128,20Zm0,192a84,84,0,1,1,84-84A84.09,84.09,0,0,1,128,212Zm52-84a12,12,0,0,1-12,12H140v28a12,12,0,0,1-24,0V140H88a12,12,0,0,1,0-24h28V88a12,12,0,0,1,24,0v28h28A12,12,0,0,1,180,128Z"></path></svg>

            </button>
        </div>
        <ul class="mt-2">
          {#each $selectOptions as option, index}
            <li class="flex items-center mb-1">
              <span class="mr-2">{option.label}</span>
              <button type="button" class="p-1 bg-red-500 text-white" on:click={() => removeOption(index)}>Remove</button>
            </li>
          {/each}
        </ul>
      </div>
    {/if}
  
    <button type="button" class="btn variant-filled-tertiary text-white" on:click={addElement}>
        <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5 mr-2" viewBox="0 0 256 256"><path d="M128,20A108,108,0,1,0,236,128,108.12,108.12,0,0,0,128,20Zm0,192a84,84,0,1,1,84-84A84.09,84.09,0,0,1,128,212Zm52-84a12,12,0,0,1-12,12H140v28a12,12,0,0,1-24,0V140H88a12,12,0,0,1,0-24h28V88a12,12,0,0,1,24,0v28h28A12,12,0,0,1,180,128Z"></path></svg>
        Add element
    </button>

    <div class="pt-4">
        {#each $paragraphs as paragraph, paragraphIndex}
        <h2>{paragraph.name}</h2>
        {#each paragraph.elements as element, elementIndex}
          {#if element.type === 'text'}
            <TextComponent content={element.content} />
          {:else if element.type === 'boolean'}
            <BooleanComponent content={element.content} />
          {:else if element.type === 'select'}
                <SelectComponent
                bind:bindValue={element.content[element.current].value}
                options={element.content}
                bind:current={element.current}
                />
          {/if}
          <button
            type="button"
            class="mt-1 p-1 bg-red-500 text-white"
            on:click={() => removeElement(paragraphIndex, elementIndex)}
          >
            Remove
          </button>
        {/each}
      {/each}
      
    <pre>{JSON.stringify($paragraphs, null, 2)}</pre>
  </div>

  </div>    