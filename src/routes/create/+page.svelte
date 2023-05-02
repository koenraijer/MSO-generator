<script>
    import { writable } from "svelte/store";
  
    let elementType = "text";
    let elementContent = "";
    let selectOptions = writable([]);
    let tempSelectOption = "";
  
    let paragraphs = writable([]);
    let currentParagraphIndex = -1;
  
    let jsonElements = writable([]);
  
    function addParagraph() {
      const newParagraph = {
        name: "",
        elements: [],
      };
      paragraphs.update((para) => [...para, newParagraph]);
      currentParagraphIndex = $paragraphs.length - 1;
    }
  
    function addElement() {
      const element = {
        type: elementType,
        content: elementType === "select" ? $selectOptions : elementContent,
      };
  
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
  
    function removeElement(index) {
      paragraphs.update((para) => {
        para[currentParagraphIndex].elements.splice(index, 1);
        return [...para];
      });
    }
  
    function switchParagraph(index) {
      currentParagraphIndex = index;
    }
  </script>
  
  <div class="container mx-auto p-4">
    <h1 class="text-2xl mb-4">Add JSON elements</h1>
  
    <div class="mb-4">
      <button type="button" class="p-2 bg-green-500 text-white mb-6" on:click={addParagraph}>Add paragraph</button>
      <ul class="mb-4">
        {#each $paragraphs as paragraph, index}
          <li class="inline-block mr-2">
            <button
              type="button"
              class="p-2 bg-blue-500 text-white"
              on:click={() => switchParagraph(index)}
            >{paragraph.name || `Paragraph ${index + 1}`}</button>
          </li>
        {/each}
      </ul>
    </div>
  
    {#if currentParagraphIndex !== -1}
      <div class="mb-4">
        <label for="paragraphName" class="block mb-2">Paragraph Name</label>
        <input
          type="text"
          bind:value={$paragraphs[currentParagraphIndex].name}
          id="paragraphName"
          class="w-full p-2 border border-gray-300"
          placeholder="Enter paragraph name"
        />
      </div>
    {/if}
  
    <!-- Add the rest of the existing code here -->
    <!-- ... -->
  
    <!-- Replace the <pre> tag at the end with the following code -->

  
    <div class="mb-4">
      <label for="type" class="block mb-2">Element Type</label>
      <select bind:value={elementType} id="type" class="w-full p-2 border border-gray-300">
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
          class="w-full p-2 border border-gray-300"
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
            class="flex-grow p-2 border border-gray-300 mr-2"
            placeholder="Enter option"
          />
          <button type="button" class="p-2 bg-blue-500 text-white" on:click={addOption}>Add option</button>
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
  
    <button type="button" class="p-2 bg-green-500 text-white mb-6" on:click={addElement}>  element</button>

    <div class="border-t border-gray-300 pt-4">
      <h2 class="text-xl mb-4">JSON Elements</h2>
      <div class="h-64 overflow-auto mb-4">
        <ul>
          {#each $jsonElements as element, index}
            <li class="p-2 mb-2 border border-gray-300">
              <pre class="whitespace-pre-wrap break-all">{JSON.stringify(element, null, 2)}</pre>
              <button type="button" class="mt-1 p-1 bg-red-500 text-white" on:click={() => removeElement(index)}>Remove</button>
            </li>
          {/each}
        </ul>
      </div>
    </div>
    <pre>{JSON.stringify($paragraphs, null, 2)}</pre>
  </div>