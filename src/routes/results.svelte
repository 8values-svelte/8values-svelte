<script>
  import Info from '$lib/data/info.json';
  import Values from '$lib/data/values.json';
  import Ideologies from '$lib/data/idealogies.json'
  import { resultsStore } from '$lib/resultsStore'
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';

  const {
    values
  } = Values;

  const {
    query,
  } = page;

  let results = {}
  let complimentResults = {}
  for(const value of values) {
    results[value.slug] = "50.0"
  }
  if(Object.keys($resultsStore).length !== 0) {
    console.log("Found store")
    results = $resultsStore;
  }
  if(query) {
    console.log("Found query")
    for (const [key, value] of query) {
      results[key] = value;
    }
  }

  for(const [slug, value] of Object.entries(results)) {
    complimentResults[slug] = (100 - parseFloat(value)).toFixed(1);
  }

  const {
    name,
    contact,
  } = Info;

  function getLabel(labels, value) {
    if (value > 100) {
      return ""
    } else if (value > 90) {
      return labels[0]
    } else if (value > 75) {
      return labels[1]
    } else if (value > 60) {
      return labels[2]
    } else if (value >= 40) {
      return labels[3]
    } else if (value >= 25) {
      return labels[4]
    } else if (value >= 10) {
      return labels[5]
    } else if (value >= 0) {
      return labels[6]
    } else {
      return ""
    }
  }

  const {
    ideologies
  } = Ideologies;

  async function calculateIdeology() {
    let ideology = "";
    let lowestDistance = Infinity;
    for(const {name, stats} of ideologies) {
      let dist = 0;

      for(const {slug, weight} of values) {
        dist += Math.pow(Math.abs(stats[slug] - results[slug]), weight)
      }

      if (dist < lowestDistance) {
        ideology = name;
        lowestDistance = dist;
      }
    }
    return ideology;
  }

  async function generateImage() {
    const c = document.createElement("canvas")
  }
</script>

<svelte:head>
  <title>{name} Results</title>
</svelte:head>

<h1>Results</h1>

{#each values as {slug, name, labels, leftValue, rightValue}}
  <h2>{name[0].toUpperCase() + name.slice(1)} Axis: <span style='font-weight: 300'>{getLabel(labels, parseFloat(results[slug]))}</span></h2>
  <div class='axis'>
    <img class='icon' src="icons/{leftValue.name}.svg" alt={leftValue.name} />
    <div class='bar' style="border-right-style: solid; background-color: {leftValue.color}; width: {results[slug]}%">
      {#if (parseFloat(results[slug]) >= 6)}
        <div class='text_wrapper'>{results[slug]}%</div>
      {/if}
    </div>
    <div
      class='bar'
      style="border-left-style: solid; text-align: right; background-color: {rightValue.color}; width: {complimentResults[slug]}%"
    >
      {#if (parseFloat(complimentResults[slug]) >= 6)}
        <div class='text_wrapper'>{complimentResults[slug]}%</div>
      {/if}
    </div>
    <img class='icon' src="icons/{rightValue.name}.svg" alt={rightValue.name} />
  </div>
{/each}
<h2>Closest Match:
  <span style='font-weight: 300'>
    {#await calculateIdeology()}
      Calculating...
    {:then result}
      {result}
    {:catch error}
      Unknown
    {/await}
  </span>
</h2>
<p>
  Ideological matching is a work in progress, and is much less accurate than the values and axes.
</p>
<p>
  You can send these results by copying and pasting the URL at the top of the page or using the
  image below. Think your matched ideology was wrong? Want to help us calibrate the test? Send the
  results along with your political ideology to us at
  <a href={'mailto:' + contact.email}>{contact.email}</a>,
  or send us any comments, questions, or criticism.
</p>
<hr/>
<button on:click={() => goto('/')}>Back</button>


<style>
.axis {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bar {
  height: 36pt;
  line-height: 36pt;
  padding: 8pt;
  margin-top: 4pt;
  margin-bottom: 4pt;
  border-width: 4px;
  border-right-width: 1px;
  border-top-style: solid;
  border-bottom-style: solid;
  border-color: #222222;
  background-color: #eeeeee;
  display: block;
  width: 20%;
}

.text_wrapper {
  font-family: 'Montserrat', sans-serif;
  font-size: 36pt;
  line-height: 36pt;
  color: #222222;
  display: inline-block;
}

img {
  height: 128pt;
}
</style>
