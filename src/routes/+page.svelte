<script>
	import { onMount } from 'svelte';
	import { clipboard } from '@skeletonlabs/skeleton';
	import SelectComponent from '$lib/components/SelectComponent.svelte';
	import { fade } from 'svelte/transition';
	let showThumbsUp = false;

	function handleCopyButtonClick() {
		showThumbsUp = true;
		setTimeout(() => {
		showThumbsUp = false;
		}, 1500);
	}

	let combinedText = '';
	
	function updateText() {
		const geen1Text = geen1 ? " geen" : "";
		const geen2Text = geen2 ? " geen" : "";
		combinedText = [
		`De patiënt ${selfcare}. Er is ${ageDifference} verschil tussen de biologische- en kalenderleeftijd. Er is sprake van ${weight}. Patiënt ${contact} contact. De klachtenpresentatie is ${presentation} met de klachteninhoud.`,
		`Het bewustzijn is ${consciousness}. De aandacht is ${attention}. De concentratie is ${concentration}, en de patiënt is ${orientation}. ${memory} is ${intelligence}. Patiënt is ${insight} intelligent. Ziekte-besef ${hallucinations}. Er zijn ${formalThought} hallucinaties. Het formele denken is ${formalThinking}. Inhoudelijk is er sprake van ${contentThinking}.`,
		`De stemming is ${mood}, en het affect is ${affect}. De patiënt kampt met ${patientIssues}. Er is${geen2Text} sprake van ${suicidality}.`,
		`De mimiek is ${mimic}. De spraak is ${speech}. Er is${geen1Text} sprake van ${behavior}.`
		].join('\n\n');
	}
	
	onMount(() => {
		updateText();
	});
	
	$: updateText();
	
    const paragraphsJSON = {
  "1": {
    select: {
      type: "selfcare",
      content: [
        { value: "heeft goede zelfzorg", label: "heeft goede zelfzorg" },
        { value: "vertoont tekenen van zelfverwaarlozing", label: "vertoont tekenen van zelfverwaarlozing" },
        { value: "vertoont tekenen van overdreven zelfzorg", label: "vertoont tekenen van overdreven zelfzorg" }
      ]
    },
    select: {
      type: "ageDifference",
      content: [
        { value: "geen", label: "geen" },
        { value: "enigszins", label: "enigszins" },
        { value: "duidelijk", label: "duidelijk" }
      ]
    },
    select: {
      type: "weight",
      content: [
        { value: "een gezond gewicht", label: "een gezond gewicht" },
        { value: "opvallend overgewicht", label: "opvallend overgewicht" },
        { value: "opvallend ondergewicht", label: "opvallend ondergewicht" }
      ]
    },
    select: {
      type: "contact",
      content: [
        { value: "maakt normaal", label: "maakt normaal" },
        { value: "is overdreven in", label: "is overdreven in" },
        { value: "is afwerend in", label: "is afwerend in" }
      ]
    },
    select: {
      name: "presentation",
      content: [
        { value: "consistent", label: "consistent" },
        { value: "inconsistent", label: "inconsistent" }
      ]
    },
    text: {
      type: "text",
      content: "text"
    }
  }
};
	let insight
	let formalThought
	let consciousness;
	let attention;
	let concentration;
	let orientation;
	let memory;
	let intelligence;
	let hallucinations;
	let formalThinking;
	let contentThinking;
	let mood;
	let affect;
	let patientIssues;
	let suicidality;
	let mimic;
	let speech;
	let behavior;

    let geen1 = false;
	let geen2 = false;
	let plural1 = "zijn"

	$: if(memory == "Zowel het korte- als langetermijngeheugen") {
		plural1 = "zijn";
	} else {
		plural1 = "is";
	}
</script>

<div class="container mx-auto p-8 space-y-8 max-w-3xl">
	<h2>MSO Generator</h2>
	<br>
	<h3>Cognitieve functies</h3>
  <p class="flex flex-wrap gap-y-4 items-center">
    <span>De patiënt&nbsp;</span>
	<SelectComponent bind:bindValue={selfcare} options={selfcareOptions} />
    <select class="select mr-0" bind:value={selfcare}>
      <option value="heeft goede zelfzorg">heeft goede zelfzorg</option>
      <option value="vertoont tekenen van zelfverwaarlozing"
        >vertoont tekenen van zelfverwaarlozing</option
      >
      <option value="vertoont tekenen van overdreven zelfzorg"
        >vertoont tekenen van overdreven zelfzorg</option
      >
    </select>
    <span>. Er is &nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={ageDifference}>
      <option value="geen">geen</option>
      <option value="enigszins">enigszins</option>
      <option value="duidelijk">duidelijk</option>
    </select>
    <span
      >verschil tussen de biologische- en kalenderleeftijd. Er is sprake van</span
    >
    <select class="select w-fit inline-flex" bind:value={weight}>
      <option value="een gezond gewicht">een gezond gewicht</option>
      <option value="opvallend overgewicht">opvallend overgewicht</option>
      <option value="opvallend ondergewicht">opvallend ondergewicht</option>
    </select>
    <span>. Patiënt &nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={contact}>
      <option value="maakt normaal">maakt normaal</option>
      <option value="is overdreven in">is overdreven in</option>
      <option value="is afwerend in">is afwerend in</option>
    </select>
    <span>contact. De klachtenpresentatie is &nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={presentation}>
      <option value="consistent">consistent</option>
      <option value="inconsistent">inconsistent</option>
    </select>
    <span>&nbsp;met de klachteninhoud.</span>
  </p>
  <hr>
  <h3>Affectieve functies</h3>
  <p class="flex flex-wrap gap-y-4 items-center">
    <span>Het bewustzijn is &nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={consciousness}>
      <option value="helder">helder</option>
      <option value="vernauwd">vernauwd</option>
      <option value="somnolent">somnolent</option>
      <option value="soporeus">soporeus</option>
      <option value="subcomateus">subcomateus</option>
      <option value="stuporeus">stuporeus</option>
    </select>
    <span>. De aandacht is &nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={attention}>
      <option value="goed te trekken en vast te houden"
        >goed te trekken en vast te houden</option
      >
      <option value="niet te trekken en niet vast te houden"
        >niet te trekken en niet vast te houden</option
      >
      <option value="te trekken maar niet vast te houden"
        >te trekken maar niet vast te houden</option
      >
    </select>
    <span>. De concentratie is &nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={concentration}>
      <option value="goed">goed</option>
      <option value="matig">matig</option>
      <option value="slecht">slecht</option>
    </select>
    <span>, en de patiënt is&nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={orientation}>
      <option value="georiënteerd in triplo">georiënteerd in triplo</option>
      <option value="slecht georiënteerd in tijd"
        >slecht georiënteerd in tijd</option
      >
      <option value="slecht georiënteerd in plaats"
        >slecht georiënteerd in plaats</option
      >
      <option value="slecht georiënteerd in persoon"
        >slecht georiënteerd in persoon</option
      >
    </select>
    <span>.</span>
    <select class="select w-fit inline-flex" bind:value={memory}>
      <option value="Zowel het korte- als langetermijngeheugen">Zowel het korte- als langetermijngeheugen</option>
      <option value="Het kortetermijngeheugen">Het kortetermijngeheugen</option>
      <option value="Het langetermijngeheugen">Het langetermijngeheugen</option>
    </select>
    <span>&nbsp;{plural1}&nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={intelligence}>
      <option value="ongestoord">ongestoord</option>
      <option value="gestoord">gestoord</option>
    </select>
    <span>. Patiënt is&nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={insight}>
      <option value="benedengemiddeld">benedengemiddeld</option>
      <option value="gemiddeld">gemiddeld</option>
      <option value="bovengemiddeld">bovengemiddeld</option>
    </select>
    <span>&nbsp;intelligent. Ziekte-besef </span>
    <select class="select w-fit inline-flex" bind:value={hallucinations}>
      <option value="en ziekte-inzicht zijn aanwezig."
        >en ziekte-inzicht zijn aanwezig</option
      >
      <option value="is afwezig">is afwezig</option>
    </select>
    <span>. Er zijn&nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={formalThought}>
      <option value="geen">geen</option>
      <option value="akoestische">akoestische</option>
      <option value="visuele">visuele</option>
      <option value="gustatoire">gustatoire</option>
      <option value="olfactorische">olfactorische</option>
      <option value="tactiele">tactiele</option>
    </select>
    <span>hallucinaties. Het formele denken is &nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={formalThinking}>
      <option value="geremd">geremd</option>
      <option value="gestuwd">gestuwd</option>
      <option value="wijdlopig">wijdlopig</option>
    </select>
    <span>. Inhoudelijk is er sprake van </span>
    <select class="select w-fit inline-flex" bind:value={contentThinking}>
      <option value="betrekkingsideeën">betrekkingsideeën</option>
      <option value="waanwaarnemingen">waanwaarnemingen</option>
      <option value="wanen">wanen</option>
      <option value="gedachte-armoede">gedachte-armoede</option>
      <option value="preoccupatie">preoccupatie</option>
      <option value="obsessie">obsessie</option>
      <option value="rumineren">rumineren</option>
    </select>
    <span>. </span>
  </p>
  <hr>
  <h3>Conatieve functies</h3>
  <p class="flex flex-wrap gap-y-4 items-center">
    <span>De stemming is&nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={mood}>
      <option value="normofoor">normofoor</option>
      <option value="somber">somber</option>
      <option value="anhedonisch">anhedonisch</option>
      <option value="apathisch">apathisch</option>
      <option value="angstig">angstig</option>
      <option value="dysfoor">dysfoor</option>
      <option value="eufoor">eufoor</option>
    </select>
    <span>, en het affect is&nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={affect}>
      <option value="vlak">vlak</option>
      <option value="normaal">normaal</option>
      <option value="labiel">labiel</option>
      <option value="incongruent">incongruent</option>
      <option value="aangezet">aangezet</option>
    </select>
    <span>. De patiënt kampt met&nbsp;</span>
    <select class="select w-fit inline-flex" bind:value={patientIssues}>
      <option value="doorslaapproblemen">doorslaapproblemen</option>
      <option value="moeheid">moeheid</option>
      <option value="dagschommeling">dagschommeling</option>
      <option value="eetlustverlies">eetlustverlies</option>
      <option value="gewichtsverlies">gewichtsverlies</option>
      <option value="libidoverlies">libidoverlies</option>
    </select>
    <span>. Er is </span>
    <!--Insert "geen" button-->
	{#if !geen2}
	<button on:click={() => {geen2 = !geen2; updateText()}} class="btn variant-filled mx-2 bg-surface-400 py-1 rounded-md px-2">
		<span>
			<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256"><path d="M228,128a12,12,0,0,1-12,12H140v76a12,12,0,0,1-24,0V140H40a12,12,0,0,1,0-24h76V40a12,12,0,0,1,24,0v76h76A12,12,0,0,1,228,128Z"></path></svg>
		</span>
		<span>geen</span>
	</button>
	{:else}
	<span>&nbsp;geen</span>
	<button on:click={() => {geen2 = !geen2; updateText()}} class="btn variant-filled p-1 mx-1 !bg-surface-600">
		<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-4 h-4" viewBox="0 0 256 256"><path d="M228,128a12,12,0,0,1-12,12H40a12,12,0,0,1,0-24H216A12,12,0,0,1,228,128Z"></path></svg>
	</button>
	{/if}
    <span>&nbsp;sprake van</span>
    <select class="select w-fit inline-flex" bind:value={suicidality}>
      <option value="suïcidaliteit">suïcidaliteit</option>
      <option value="passieve suicidale ideatie"
        >passieve suicidale ideatie</option
      >
      <option value="actieve suicidale ideatie"
        >actieve suicidale ideatie</option
      >
    </select> <span>. </span>
  </p>
  <p class="flex flex-wrap gap-y-4 items-center">
    <span>De mimiek is </span>
    <select class="select w-fit inline-flex" bind:value={mimic}>
      <option value="normaal">normaal</option>
      <option value="vertraagd">vertraagd</option>
      <option value="afwezig">afwezig</option>
      <option value="overdreven">overdreven</option>
    </select> <span>. De spraak is </span>
    <select class="select" bind:value={speech}>
      <option value="normaal">normaal</option>
      <option value="monotoon">monotoon</option>
      <option value="levendig">levendig</option>
    </select> <span>. Er is</span>
	<!--Insert "geen" button-->
	{#if !geen1}
	<button on:click={() => {geen1 = !geen1; updateText()}} class="btn variant-filled mx-2 bg-surface-400 py-1 rounded-md px-2">
		<span>
			<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256"><path d="M228,128a12,12,0,0,1-12,12H140v76a12,12,0,0,1-24,0V140H40a12,12,0,0,1,0-24h76V40a12,12,0,0,1,24,0v76h76A12,12,0,0,1,228,128Z"></path></svg>
		</span>
		<span>geen</span>
	</button>
	{:else}
	<span>&nbsp;geen</span>
	<button on:click={() => {geen1 = !geen1; updateText()}} class="btn variant-filled p-1 mx-1 !bg-surface-600">
		<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-4 h-4" viewBox="0 0 256 256"><path d="M228,128a12,12,0,0,1-12,12H40a12,12,0,0,1,0-24H216A12,12,0,0,1,228,128Z"></path></svg>
	</button>
	{/if}
		<span>sprake van </span>
    <select class="select" bind:value={behavior}>
      <option value="avolitie">avolitie</option>
      <option value="impulsief gedrag">impulsief gedrag</option>
      <option value="drangmatig gedrag">drangmatig gedrag</option>
      <option value="dwangmatig gedrag">dwangmatig gedrag</option>
    </select> <span>.</span>
  </p>

<div>
	<h2 class="">Resultaat</h2>
</div>
  	<!-- Source -->
	<textarea class="textarea" rows="15" value={combinedText} data-clipboard="exampleInput"></textarea>

	<!-- Trigger -->
	<button class="btn variant-filled" on:click={handleCopyButtonClick} use:clipboard={combinedText}>Copy</button>
	{#if showThumbsUp}
	<span class="text-xl bg-surface-200 rounded-full p-1" out:fade={{ duration: 250 }}>&#x1F44D;</span>
	{/if}
</div>

