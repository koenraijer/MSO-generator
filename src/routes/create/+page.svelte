<script>
    import { writable } from "svelte/store";
    import { localStorageStore } from '@skeletonlabs/skeleton';
    import { toastStore } from '@skeletonlabs/skeleton';
    import { modalStore } from '@skeletonlabs/skeleton';
    import SelectComponent from "$lib/components/SelectComponent.svelte";
    import TextComponent from "$lib/components/TextComponent.svelte";
    import BooleanComponent from "$lib/components/BooleanComponent.svelte";
    import Moon from "$lib/components/Moon.svelte";
    import { supabase } from "$lib/supabaseClient.js";
    import { tick } from "svelte";
    import { goto } from "$app/navigation";
    import { focusTrap } from '@skeletonlabs/skeleton';
    import { popup } from '@skeletonlabs/skeleton';
    import {onMount} from 'svelte';
    let shiftKeyPressed = false;
    let isFocused = true;

    let optionInput;
    let paragraphNameInput
    let typeInput
    
    let contentInput
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

        currentParagraphIndex = $paragraphs.length;
        currentParagraphName = newParagraph.name;

        paragraphs.update((para) => [...para, newParagraph]);
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

        if (elementType === "select" && $selectOptions.length === 0) {
                const t = {
                    message: 'Please add at least one option for the select element.',
                    background: 'variant-filled-warning',
                };
                toastStore.trigger(t);
                return; // Do not proceed with adding the element
        }

        if (elementType !== "text") {
            element.current = 0; // Add the 'current' property with a default value of 0
        }

        paragraphs.update((para) => {
            para[currentParagraphIndex].elements.push(element);
            return [...para];
        });

        elementContent = "";
        selectOptions.set([]);
        typeInput.focus() // THis doesn't work right now
    }
  
    function addOption() {
      if (tempSelectOption) {
        selectOptions.update((opts) => [
          ...opts,
          { value: tempSelectOption, label: tempSelectOption },
        ]);
        tempSelectOption = "";
      } else {
        if (!shiftKeyPressed) {
            const t = {
                message: 'Please enter an option value.',
                background: 'variant-filled-warning',
            };
            toastStore.trigger(t);
        }

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
            if (currentParagraphIndex >= 0 && currentParagraphIndex < para.length) {
                para[currentParagraphIndex].name = currentParagraphName;
                return [...para];
            }
            return para;
        });
    }

    function clearMemory() {
        const confirm = {
            type: 'confirm',
            // Data
            title: 'Do you want to clear the memory?',
            body: 'Press confirm if you want to clear the memory.',
            // TRUE if confirm pressed, FALSE if cancel pressed
            response: (r) => {
                if (r) {
                    localStorage.clear();
                    paragraphs.set([])
                    addParagraph();
                    const t = {
                        message: 'Memory cleared.',
                        background: 'variant-filled-success',
                    };
                    toastStore.trigger(t);
                }
            }
        };
        modalStore.trigger(confirm);
    }

    function uuidv4() {
        return ([1e7]+-1e3+-4e3+-8e3+-1e11).replace(/[018]/g, c =>
            (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
        );
        }

    let loading = false;

    async function createStandardText() {
        loading = true;
        let uuid = uuidv4();
        try {
            // Insert the JSON object into the Standardtexts table
            const { data, error } = await supabase
                .from('Standardtexts')
                .insert([{ uuid: uuid, json: JSON.stringify($paragraphs) }]);

            if (error) {
                throw error;
            }

            // Show a success toast
            const t = {
                message: 'Standard text created successfully.',
                background: 'variant-filled-success',
            };
            toastStore.trigger(t);

            // Navigate to another page or perform any other desired action
            goto(`/${uuid}`); // Redirect to the newly created standard text page
            
        } catch (error) {
            console.error('Error creating standard text:', error);

            // Show an error toast
            const t = {
                message: 'Error creating standard text.',
                background: 'variant-filled-warning',
            };
            toastStore.trigger(t);
        } finally {
            loading = false;
        }
    }
    
    let menuPopupSettings = {
        // Set the event as: click | hover | hover-click | focus | focus-click
        event: 'hover-click',
        // Provide a matching 'data-popup' value.
        target: 'menuPopup'
    };

    let rightColumnHeight;
    let ulHeight;
    $: leftColumnStyle = `max-height: ${rightColumnHeight}px;`;

    // Information popups
    let informationContents = [
        {
            title: 'What is this?',
            content: 'This is a simple tool to create standard texts/templates for those that do not have the luxury of an EHR with this functionality built-in. You can create a standard text by adding paragraphs and elements to the paragraphs.',
        },
        {
            title: 'Add, rename and delete paragraphs.',
            content: `Select the paragraphs you want to add elements to by pressing on the gray circle to the left of its name. \n\n Rename paragraphs by clicking on the paragraph's name. Delete them by clicking on the trashcan to its right. A paragraph provides structure to your standard text, but it will not be part of the final text.`,
        },
        {
            title: 'What are elements?',
            content: 'An element is a part of a paragraph. They come in three types: text, select elements and yes/no elements.',
        },
        {
            title: 'What is a text element?',
            content: `A text element is simply that: normal text. Use it when you want to add text to a paragraph that doesn't need any special formatting.`,
        },
        {
            title: 'What is a select element?',
            content: 'A select element allows you to choose between multiple options. They are the reason standard texts are so useful.',
        },
        {
            title: 'What is a yes/no element?',
            content: 'A yes/no element allows you to define a piece of text that is only added to the final text if you want it to.',
        },
        {
            title: 'Preview your standard text.',
            content: 'This shows what your standard text would look like if you would generate it right now.',
        }
    ]
    let info1PopupSettings = {
        // Set the event as: click | hover | hover-click | focus | focus-click
        event: 'click',
        // Provide a matching 'data-popup' value.
        target: 'info1Popup'
    };
    let info2PopupSettings = {
        // Set the event as: click | hover | hover-click | focus | focus-click
        event: 'click',
        // Provide a matching 'data-popup' value.
        target: 'info2Popup'
    };
    let info3PopupSettings = {
        // Set the event as: click | hover | hover-click | focus | focus-click
        event: 'click',
        // Provide a matching 'data-popup' value.
        target: 'info3Popup'
    };
    let info4PopupSettings = {
        // Set the event as: click | hover | hover-click | focus | focus-click
        event: 'click',
        // Provide a matching 'data-popup' value.
        target: 'info4Popup'
    };

    const showWelcomeToast = () => {
        const t = {
        message: 'Welcome! This is a simple tool to create standard texts for healthcare professionals dealing with archaic EHR software that does not have this functionality built-in. luxury of an EHR with this functionality built-in. Click on the information icons to learn more about the different features.',
        autohide: false
        };
        toastStore.trigger(t);
    };

    onMount(() => {
        if (localStorage.getItem('firstVisit') === null) {
        showWelcomeToast();
        localStorage.setItem('firstVisit', 'false');
        }
    });

  </script>
  
<div class="container mx-auto p-8 space-y-8 max-w-3xl flex-col flex-nowrap">

        <button use:popup={menuPopupSettings} class="absolute top-0 right-6 my-6 bg-surface-200 p-2 rounded-full aspect-square text-xl">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256"><path d="M144,128a16,16,0,1,1-16-16A16,16,0,0,1,144,128ZM60,112a16,16,0,1,0,16,16A16,16,0,0,0,60,112Zm136,0a16,16,0,1,0,16,16A16,16,0,0,0,196,112Z"></path></svg>
        </button>
        <div data-popup="menuPopup" class="-translate-x-4 -translate-y-6 w-56 card flex-col !justify-items-end !flex gap-y-4 p-2">
            <button class="btn variant-filled-surface rounded-md" on:click={clearMemory}>Clear memory</button>
            <a href="/mso" class="btn variant-filled-surface rounded-md" on:click={clearMemory}>Mentale status onderzoek</a>
        </div>

    <div>
        <h1 class="text-2xl mb-8 md:mb-12">
            The standard text generator
            <svg use:popup={info3PopupSettings} xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5 inline fill-surface-400 -translate-y-[0.125rem]" viewBox="0 0 256 256"><path d="M108,84a16,16,0,1,1,16,16A16,16,0,0,1,108,84Zm128,44A108,108,0,1,1,128,20,108.12,108.12,0,0,1,236,128Zm-24,0a84,84,0,1,0-84,84A84.09,84.09,0,0,0,212,128Zm-72,36.68V132a20,20,0,0,0-20-20,12,12,0,0,0-4,23.32V168a20,20,0,0,0,20,20,12,12,0,0,0,4-23.32Z"></path></svg>
        </h1>
        <div data-popup="info3Popup" class="card variant-filled-secondary w-72">
            <header class="card-header">
                <h4>{informationContents[0].title}</h4>
            </header>
            <section class="p-4">{informationContents[0].content}</section>
            <div class="arrow variant-filled-secondary" />
        </div>
        <div class="grid grid-cols-1 md:grid-cols-5 gap-x-4 md:gap-x-8 lg:gap-x-12"> 
            <div class="flex flex-col col-span-2 md:w-fit w-full h-auto pb-8 md:pb-0 !z-100" style={leftColumnStyle} bind:clientHeight="{ulHeight}">
                <span class="block mb-1">
                    Paragraphs
                    <svg use:popup={info1PopupSettings} xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5 inline fill-surface-400 -translate-y-[0.125rem]" viewBox="0 0 256 256"><path d="M108,84a16,16,0,1,1,16,16A16,16,0,0,1,108,84Zm128,44A108,108,0,1,1,128,20,108.12,108.12,0,0,1,236,128Zm-24,0a84,84,0,1,0-84,84A84.09,84.09,0,0,0,212,128Zm-72,36.68V132a20,20,0,0,0-20-20,12,12,0,0,0-4,23.32V168a20,20,0,0,0,20,20,12,12,0,0,0,4-23.32Z"></path></svg>
                </span>
                <div data-popup="info1Popup" class="card variant-filled-secondary w-72">
                    <header class="card-header">
                        <h4>{informationContents[1].title}</h4>
                    </header>
                    <section class="p-4">{informationContents[1].content}</section>
                    <div class="arrow variant-filled-secondary" />
                </div>
                <ul
                class="h-fit btn-group-vertical bg-surface-100 overflow-x-none rounded-md overflow {ulHeight >= rightColumnHeight ? 'overflow-y-scroll' : 'overflow-y-none'}"
                >
                {#each $paragraphs as paragraph, index}
                <li
                    class="inline-block relative group"
                >
                    <div class="flex flex-row flex-nowrap">
                    <button on:click={() => switchParagraph(index)}>
                        <span class="border rounded-full border-surface-400 p-2 {currentParagraphIndex === index ? '!bg-surface-400' : ''}"></span>
                    </button>
                    <input
                        type="text"
                        bind:value={paragraph.name}
                        class="input !border-none rounded-md w-full bg-transparent"
                        placeholder="Enter paragraph title..."
                        on:input={() => updateParagraphName(index, paragraph.name)}
                    />
                    <span
                        on:click={() => deleteParagraph(index)}
                        class="cursor-pointer p-1 inline !border-none self-center pr-2 hover:text-surface-400"
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
                            d="M216,48H40a12,12,0,0,0,0,24h4V208a20,20,0,0,0,20,20H192a20,20,0,0,0,20-20V72h4a12,12,0,0,0,0-24ZM188,204H68V72H188ZM76,20A12,12,0,0,1,88,8h80a12,12,0,0,1,0,24H88A12,12,0,0,1,76,20Z"
                        ></path>
                        </svg>
                    </span>
                    </div>
                </li>
                {/each}
                </ul>
                <button
                type="button"
                class="w-full mt-8 md:mt-10 p-2 btn variant-filled-surface bg-surface-400 text-white rounded-md"
                on:click={addParagraph}
                >
                    <svg
                        xmlns="http://www.w3.org/2000/svg"
                        width="32"
                        height="32"
                        fill="currentColor"
                        class="w-5 h-5 mr-2"
                        viewBox="0 0 256 256"
                    >
                        <path
                        d="M128,20A108,108,0,1,0,236,128,108.12,108.12,0,0,0,128,20Zm0,192a84,84,0,1,1,84-84A84.09,84.09,0,0,1,128,212Zm52-84a12,12,0,0,1-12,12H140v28a12,12,0,0,1-24,0V140H88a12,12,0,0,1,0-24h28V88a12,12,0,0,1,24,0v28h28A12,12,0,0,1,180,128Z"
                        ></path>
                    </svg>
                    Add paragraph
                </button>
            </div>
            <div class="col-span-3 h-fit" bind:clientHeight={rightColumnHeight}>
                <div class="md:invisible visible">
                    <hr class="md:my-0 my-8">
                </div>

                <div data-popup="info2Popup" class="card variant-filled-secondary w-96">
                    <header class="card-header">
                        <h4>{informationContents[2].title}</h4>
                    </header>
                    <section class="p-4">{informationContents[2].content}</section>
                    <section class="p-4">
                        <h5 class="font-semibold mb-1">{informationContents[3].title}</h5>
                        <p>{informationContents[3].content}</p>
                    </section>
                    <section class="p-4">
                        <h5 class="font-semibold mb-1">{informationContents[4].title}</h5>
                        <p>{informationContents[4].content}</p>
                    </section>
                    <section class="p-4">
                        <h5 class="font-semibold mb-1">{informationContents[5].title}</h5>
                        <p>{informationContents[5].content}</p>
                    </section>
                    <div class="arrow variant-filled-secondary" />
                </div>
                <form use:focusTrap={isFocused} class="h-full">
                    <div class="mb-6 md:mb-8 w-fit">
                        <div class="inline -translate-y-2">
                            <label for="type" class="inline">Element Type
                            </label>
                            <svg use:popup={info2PopupSettings} xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5 inline fill-surface-400 -translate-y-[0.125rem]" viewBox="0 0 256 256"><path d="M108,84a16,16,0,1,1,16,16A16,16,0,0,1,108,84Zm128,44A108,108,0,1,1,128,20,108.12,108.12,0,0,1,236,128Zm-24,0a84,84,0,1,0-84,84A84.09,84.09,0,0,0,212,128Zm-72,36.68V132a20,20,0,0,0-20-20,12,12,0,0,0-4,23.32V168a20,20,0,0,0,20,20,12,12,0,0,0,4-23.32Z"></path></svg>
                        </div>
                        <select
                            bind:value={elementType}
                            id="type"
                            class="input !border-none rounded-md"
                            bind:this={typeInput}
                            on:change={async () => {
                                await tick();
                                if (elementType !== 'select') {
                                    contentInput.focus();
                                } else {
                                    optionInput.focus();
                                }
                            }}
                        >
                            <option value="text">Text</option>
                            <option value="select">Select</option>
                            <option value="boolean">Boolean</option>
                        </select>
                    </div>
    
                    {#if elementType !== "select"}
                        <div class="mb-6 md:mb-6">
                            <label for="content" class="block mb-1">Content</label>
                            <input
                                type="text"
                                bind:value={elementContent}
                                id="content"
                                class="input !border-none rounded-md"
                                placeholder="Enter content"
                                on:keydown={(e) => {
                                    if (e.key === 'Enter') { addElement(); typeInput.focus(); }
                                }}
                                bind:this={contentInput}
                            />
                        </div>
                    {:else}
                        <div class="mb-4">
                            <label for="option" class="block mb-2">Select Options</label>
                            <div class="flex">
                                <input
                                    type="text"
                                    bind:value={tempSelectOption}
                                    bind:this={optionInput}
                                    id="option"
                                    class="input !border-none !rounded-l-md rounded-r-none"
                                    placeholder="Enter option"
                                    on:keyup={(e) => {
                                        if (e.key === 'Shift') { shiftKeyPressed = false; }
                                    }}
                                    on:keydown={(e) => {
                                        if (e.key === 'ArrowUp') { typeInput.focus(); }
                                        if (e.key === 'Enter' && !shiftKeyPressed) { addOption(); optionInput.focus(); }
                                        if (e.key === 'Shift') { shiftKeyPressed = true; }
                                        if (e.key === 'Enter' && shiftKeyPressed) { addOption(); addElement(); typeInput.focus(); }
                                    }}
                                />
                                <button type="button" class="p-2 text-white btn variant-filled-secondary rounded-l-none px-3" on:click={() => { addOption(); optionInput.focus(); }}>
                                    <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256"><path d="M128,20A108,108,0,1,0,236,128,108.12,108.12,0,0,0,128,20Zm0,192a84,84,0,1,1,84-84A84.09,84.09,0,0,1,128,212Zm52-84a 12,12,0,0,1-12,12H140v28a12,12,0,0,1-24,0V140H88a12,12,0,0,1,0-24h28V88a12,12,0,0,1,24,0v28h28A12,12,0,0,1,180,128Z"></path></svg>
                                </button>
                            </div>
                            <ul class="mt-2">
                            {#each $selectOptions as option, index}
                                <li class="flex items-center mb-1">
                                <span class="mr-2">{option.label}</span>
                                    <button
                                    type="button"
                                    class="btn variant-filled-warning text-sm ml-2"
                                    on:click={() => removeOption(index)}
                                    >
                                        <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256"><path d="M216,36H68.53a20.09,20.09,0,0,0-17.15,9.71L5.71,121.83a12,12,0,0,0,0,12.34l45.67,76.12A20.09,20.09,0,0,0,68.53,220H216a20,20,0,0,0,20-20V56A20,20,0,0,0,216,36Zm-4,160H70.8L30,128,70.8,60H212ZM103.51,143.51,119,128l-15.52-15.51a12,12,0,0,1,17-17L136,111l15.51-15.52a12,12,0,0,1,17,17L153,128l15.52,15.51a12,12,0,0,1-17,17L136,145l-15.51,15.52a12,12,0,0,1-17-17Z"></path></svg>
                                    </button>
                                </li>
                            {/each}
                            </ul>
                        </div>
                    {/if}
    
                    <button type="button" class="btn variant-filled-surface bg-surface-400 text-white rounded-md mt-4 w-full" on:click|preventDefault={addElement} on:keydown={(e) => { if (e.key === 'Enter') addElement();typeInput.focus()}}>
                        <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5 mr-2" viewBox="0 0 256 256"><path d="M128,20A108,108,0,1,0,236,128,108.12,108.12,0,0,0,128,20Zm0,192a84,84,0,1,1,84-84A84.09,84.09,0,0,1,128,212Zm52-84a12,12,0,0,1-12,12H140v28a12,12,0,0,1-24,0V140H88a12,12,0,0,1,0-24h28V88a12,12,0,0,1,24,0v28h28A12,12,0,0,1,180,128Z"></path></svg>
                        Add element
                    </button>   
                </form>
            </div>
        </div>
    
        <hr class="my-4 md:my-6">

        <label class="mb-2">
            Preview
            <svg use:popup={info4PopupSettings} xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5 inline fill-surface-400 -translate-y-[0.125rem]" viewBox="0 0 256 256"><path d="M108,84a16,16,0,1,1,16,16A16,16,0,0,1,108,84Zm128,44A108,108,0,1,1,128,20,108.12,108.12,0,0,1,236,128Zm-24,0a84,84,0,1,0-84,84A84.09,84.09,0,0,0,212,128Zm-72,36.68V132a20,20,0,0,0-20-20,12,12,0,0,0-4,23.32V168a20,20,0,0,0,20,20,12,12,0,0,0,4-23.32Z"></path></svg>
        </label>
        <div data-popup="info4Popup" class="card variant-filled-secondary w-72">
            <header class="card-header">
                <h4>{informationContents[6].title}</h4>
            </header>
            <section class="p-4">{informationContents[6].content}</section>
            <div class="arrow variant-filled-secondary" />
        </div>
        <div class="bg-surface-100 p-4 rounded-md">
            {#each $paragraphs as paragraph, paragraphIndex}
            {#if paragraph.name}
                <h3 class:mb-4={paragraphIndex === 0} class:mt-0={paragraphIndex === 0} class:my-4={paragraphIndex !== 0}>{paragraph.name}</h3>
            {:else}
                <h3 class="my-4 text-surface-600" class:mb-4={paragraphIndex === 0} class:mt-0={paragraphIndex === 0} class:my-4={paragraphIndex !== 0}>Nameless paragraph</h3>
            {/if}
            <p>
            <!--If no paragraph contains elements-->
            {#if paragraph.elements.length === 0}
                <span class="text-surface-400">Start by adding some elements to this paragraph...</span>
            {/if}
            {#each paragraph.elements as element, elementIndex}
                <span class="group inline relative">
                    {#if element.type === 'text'}
                        <TextComponent content={element.content} />
                    {:else if element.type === 'boolean'}
                        <BooleanComponent content={element.content} disabled/>
                    {:else if element.type === 'select'}
                            <SelectComponent
                            bind:bindValue={element.content[element.current].value}
                            options={element.content}
                            disabled
                            />
                    {/if}
                    <button
                        type="button"
                        class="btn variant-filled-surface text-sm ml-2 group-hover:visible invisible p-1 absolute -top-[100%] right-0"
                        on:click={() => removeElement(paragraphIndex, elementIndex)}
                    >
                    <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256"><path d="M216,36H68.53a20.09,20.09,0,0,0-17.15,9.71L5.71,121.83a12,12,0,0,0,0,12.34l45.67,76.12A20.09,20.09,0,0,0,68.53,220H216a20,20,0,0,0,20-20V56A20,20,0,0,0,216,36Zm-4,160H70.8L30,128,70.8,60H212ZM103.51,143.51,119,128l-15.52-15.51a12,12,0,0,1,17-17L136,111l15.51-15.52a12,12,0,0,1,17,17L153,128l15.52,15.51a12,12,0,0,1-17,17L136,145l-15.51,15.52a12,12,0,0,1-17-17Z"></path></svg>
                    </button>
                </span>
            {/each}
            </p>
        {/each}
        </div>

    </div>

    <button class="btn rounded-md w-full flex items-center text-white variant-filled-primary" on:click={createStandardText}>
        <div class="flex flex-row flex-nowrap w-fit">
            {#if loading}
            <Moon class="absolute inset-0 mx-auto my-auto" size=20 borderWidth=2 />
            {:else}
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="w-5 h-5 inset-0 mx-auto my-auto" viewBox="0 0 256 256"><path d="M216.49,79.51l-56-56A12,12,0,0,0,152,20H56A20,20,0,0,0,36,40V216a20,20,0,0,0,20,20H200a20,20,0,0,0,20-20V88A12,12,0,0,0,216.49,79.51ZM160,57l23,23H160ZM60,212V44h76V92a12,12,0,0,0,12,12h48V212Zm104-60a12,12,0,0,1-12,12H140v12a12,12,0,0,1-24,0V164H104a12,12,0,0,1,0-24h12V128a12,12,0,0,1,24,0v12h12A12,12,0,0,1,164,152Z"></path></svg>
            {/if}
            <span class="ml-1">Create StandardText</span>
        </div>
    </button>
</div>    