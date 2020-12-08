<script>
  import { onMount } from "svelte";
  // import { LayerCake, Svg } from "layercake";
  import random from "../../utils/random";
  import aVsAn from "../../utils/aVsAn";
  import Icon from "../helpers/Icon.svelte";
  import prompts from "../../data/prompts.json";
  import madLibs from "../../data/madLibs.json";
  import examples from "../../data/examples.json";

  let index, lib, libPrompts, title;

  let currentInput = 0,
    currentPromptIndex = 0,
    exampleIndex = 0;

  async function getLib() {
    let formattedPrompts = [];

    prompts.forEach(d => {
      formattedPrompts.push(`<::${d.type}/>`);
    });

    index = random(0, madLibs.length);
    exampleIndex = random(0, 3);
    lib = formatLib(madLibs[index].text);
    libPrompts = getPrompts(lib);

    title = madLibs[index].title;

    function formatLib(text) {
      const wrds = text.split(/([ ,."]+)/),
        formattedWrds = [];

      wrds.forEach((d, i) => {
        const text = d,
          index = i,
          prompt = formattedPrompts.indexOf(d) > -1,
          promptType = prompt ? prompts.find(s => s.type === d.replace("<::", "").replace("/>", "")).type : null,
          promptDesc = prompt ? prompts.find(s => s.type === d.replace("<::", "").replace("/>", "")).description : null,
          value = null;

        formattedWrds.push({ text, index, prompt, promptType, promptDesc, value });
      })

      return formattedWrds;
    }

    function getPrompts(wrds) {
      const thesePrompts = [];

      wrds.forEach((d, i) => {
        if (d.prompt) {
          const type = d.promptType,
            desc = d.promptDesc,
            index = d.index,
            lead = aVsAn(d.promptDesc);

          thesePrompts.push({ type, desc, lead, index });
        }
      });

      thesePrompts.forEach((d, i) => {
        d.i = i;
        d.lead = i > 0 && thesePrompts[i - 1].desc === d.desc ? "another" : d.lead;
      })

      currentPromptIndex = thesePrompts[currentInput].index;

      return thesePrompts;
    }
  }

  let promise = getLib();

  const handleEnter = () => {
    if (currentInput < libPrompts.length) {
      currentInput += 1;
      currentPromptIndex = libPrompts[currentInput] ? libPrompts[currentInput].index : currentPromptIndex;
    }

    lib = findVowels(lib);
    exampleIndex = random(0, 3);
  }

  function findVowels(wrds) {
    wrds.forEach((d, i) => {
      d.text = d.text === "a/an" ? aVsAn(wrds[i + 1].value || "sam") : d.text;
      if (d.text === "a/an") console.log(d.text)
    })

    return wrds;
  }

</script>

<section id="demo">
  <div>
    <h1>Mad Libs</h1>
  </div>

  {#await promise}

    <p>Loading...</p>

  {:then}

    {#each libPrompts as prompt, i (prompt.i)}
      <div class:isvisible="{i === currentInput}" class="input-container" id="prompt-input-{prompt.i}">
        {#if i === 0}
          <p>First, we need {prompt.lead} {prompt.desc}</p>
        {:else if i === libPrompts.length - 1}
          <p>And lastly, {prompt.lead} {prompt.desc}</p>
        {:else}
          <p>Okay, now {prompt.lead} {prompt.desc}</p>
        {/if}
        <form on:submit|preventDefault={handleEnter}>
          <input bind:value={lib[prompt.index].value} placeholder={examples[prompt.type][exampleIndex]} type="text" class={prompt.type} />
        </form>
      </div>
    {/each}

    <div class:isvisible="{currentInput === libPrompts.length}" class="title">
      <h2>{title}</h2>
    </div>

    <div>
      {#each lib as word, i (word.index)}
        {#if !word.prompt}
          <div class:isvisible="{currentInput === libPrompts.length}" class="word" id="word-{word.index}">
            <span class="inner">
              {word.text}
            </span>
          </div>
        {:else}
          <div class:isactive="{word.index === currentPromptIndex && currentInput < libPrompts.length}" class="prompt {word.promptType}" id="prompt-{word.index}">
            <span class="inner">
              {word.value || "  "}
            </span>
          </div>
        {/if}
      {/each}
    </div>
  {/await}

  <div>
    <p>Using the Pudding's <a href="https://github.com/the-pudding/svelte-starter">starter template</a> for Svelte <Icon name="feather" /> and Microsoft's <a href="https://www.microsoft.com/en-us/download/details.aspx?id=55593">Dataset for Fill-in-the-Blank Humor</a>.</p>
  </div>
</section>

<style>
  section {
    margin: 0 auto;
    padding: 1rem;
    width: 100%;
    max-width: 640px;
  }

  div {
    margin-bottom: 2rem;
  }

  h1 {
    font-weight: 700;
  }

  input {
    text-align: center;
    border: none;
    border-bottom: 2px solid yellow;
    border-radius: 0;
    background-color: #efefef;
    color: #121212 !important;
  }

  input:focus {
    box-shadow: none;
    background-color: #dedede;
  }

  span {
  }

  .title {
    display: none;
  }

  .title.isvisible {
    display: block;
  }

  .input-container {
    display: none;
  }

  .input-container.isvisible {
    display: block;
  }

  .word {
    display: inline-flex;
    align-items: center;
    margin: 0 4px 8px 0;
    height: 32px;
    background-color: rgba(239, 239, 239, 0.5);
    transition: 250ms;
  }

  .word .inner {
    opacity: 0;
  }

  .word.isvisible {
    background-color: #fff;
  }

  .word.isvisible .inner {
    opacity: 1;
  }

  .prompt {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    margin: 0 4px 8px 0;
    min-width: 64px;
    border-bottom: 2px solid red;
    transition: 250ms;
    font-weight: bold;
  }

  .prompt.isactive {
    transform: scale(1.1) translate(0, -8px);
    box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.05);
    height: 32px;
    background-color: white;
  }

  input.animal, .prompt.animal {
    color: yellow;
    border-color: yellow;
  }

  input.animal_plural, .prompt.animal_plural {
    color: yellow;
    border-color: yellow;
  }

  input.body, .prompt.body {
    color: orange;
    border-color: orange;
  }

  input.body_plural, .prompt.body_plural {
    color: orange;
    border-color: orange;
  }

  input.food, .prompt.food {
    color: cyan;
    border-color: cyan;
  }

  input.food_plural, .prompt.food_plural {
    color: cyan;
    border-color: cyan;
  }

  input.jj, .prompt.jj {
    color: pink;
    border-color: pink;
  }

  input.liquid, .prompt.liquid {
    color: magenta;
    border-color: magenta;
  }

  input.nn, .prompt.nn {
    color: red;
    border-color: red;
  }

  input.nns, .prompt.nns {
    color: red;
    border-color: red;
  }

  input.rb, .prompt.rb {
    color: purple;
    border-color: purple;
  }

  input.vb, .prompt.vb {
    color: green;
    border-color: green;
  }

  input.vbd, .prompt.vbd {
    color: green;
    border-color: green;
  }

  input.vbg, .prompt.vbg {
    color: green;
    border-color: green;
  }

  input.vbn, .prompt.vbn {
    color: green;
    border-color: green;
  }

  input.vbz, .prompt.vbz {
    color: green;
    border-color: green;
  }

  .inner {
    white-space: nowrap;
  }
</style>
