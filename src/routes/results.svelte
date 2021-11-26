<script>
  import Info from '$lib/data/info.json';
  import Values from '$lib/data/values.json';
  import Ideologies from '$lib/data/idealogies.json';
  import { resultsStore } from '$lib/resultsStore';
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';
  import { onMount } from 'svelte';

  const {
    values
  } = Values;

  let results = {};
  for (const value of values) {
    results[value.slug] = '50.0';
  }
  for (const [key, value] of $page.query) {
    results[key] = value;
  }
  if (Object.keys($resultsStore).length !== 0) {
    results = $resultsStore;
  }


  let complimentResults = {};
  for (const [slug, value] of Object.entries(results)) {
    complimentResults[slug] = (100 - parseFloat(value)).toFixed(1);
  }

  const {
    name,
    version,
    contact,
    url,
  } = Info;

  function getLabel(labels, value) {
    if (value > 100) {
      return '';
    } else if (value > 90) {
      return labels[0];
    } else if (value > 75) {
      return labels[1];
    } else if (value > 60) {
      return labels[2];
    } else if (value >= 40) {
      return labels[3];
    } else if (value >= 25) {
      return labels[4];
    } else if (value >= 10) {
      return labels[5];
    } else if (value >= 0) {
      return labels[6];
    } else {
      return '';
    }
  }

  const {
    ideologies
  } = Ideologies;

  async function calculateIdeology() {
    let ideology = '';
    let lowestDistance = Infinity;
    for (const { name, stats } of ideologies) {
      let dist = 0;

      for (const { slug, weight } of values) {
        dist += Math.pow(Math.abs(stats[slug] - results[slug]), weight);
      }

      if (dist < lowestDistance) {
        ideology = name;
        lowestDistance = dist;
      }
    }
    return ideology;
  }

  const barSectionHeight = 130;

  function getImageWidth() {
    return 800;
  }

  function getImageHeight() {
    return 160 + (barSectionHeight * values.length);
  }

  let imageSrc = ''
  async function generateImage() {
    const c = document.createElement('canvas');
    c.width = getImageWidth();
    c.height = getImageHeight();

    const ctx = c.getContext("2d");

    ctx.fillStyle = "#EEEEEE";
    ctx.fillRect(0, 0, 800, getImageHeight());



    //Title
    ctx.fillStyle = "#222222";
    let titleSize = 81;
    do {
      titleSize -= 1;
      ctx.font = "700 " + titleSize + "px Montserrat";
    } while(ctx.measureText(name).width > 460)
    const height = ctx.measureText(name).actualBoundingBoxAscent;
    const expected = 59;
    const offset = expected - height
    ctx.textAlign = "left";
    ctx.fillText(name, 20, 90 - offset);
    //Ideology
    ctx.font = "50px Montserrat";
    ctx.fillText(await calculateIdeology(), 20, 140 - offset);
    //url + version
    ctx.textAlign = "right"
    let urlSize = 31;
    do {
      urlSize -= 1;
      ctx.font = "700 " + urlSize + "px Montserrat";
    } while(ctx.measureText(url).width > 280)
    ctx.fillText(url, 780, 60);
    ctx.font = "30px Montserrat";
    ctx.fillText(version, 780, 90);

    for(const [index, result] of Object.entries(values)) {
      const {
        name,
        slug,
        labels,
        leftValue,
        rightValue,
      } = result;

      //Axis name
      const capitalizedName = name[0].toUpperCase() + name.slice(1);
      ctx.font = "300 30px Montserrat";
      ctx.textAlign = "center";
      ctx.fillText(capitalizedName + " Axis: " + getLabel(labels, parseFloat(results[slug])), 400, 172 + (index * barSectionHeight));

      //left image
      let leftImg = new Image();
      leftImg.src = "icons/" + leftValue.name + ".svg";
      ctx.drawImage(leftImg, 20, 170 + (index * barSectionHeight), 100, 100);
      //right image
      let rightImg = new Image();
      rightImg.src = "icons/" + rightValue.name + ".svg";
      ctx.drawImage(rightImg, 680, 170 + (index * barSectionHeight), 100, 100);

      //background of bar
      ctx.fillStyle = "#222222";
      ctx.fillRect(120, 180 + (index * barSectionHeight), 560, 80);

      //left bar
      ctx.fillStyle = leftValue.color;
      const leftSize = 5.6 * results[slug] - 2;
      ctx.fillRect(120, 184 + (index * barSectionHeight), leftSize, 72);

      //right bar
      ctx.fillStyle = rightValue.color;
      const rightSize = 5.6 * complimentResults[slug] - 2;
      const rightPos = 680 - rightSize;
      ctx.fillRect(rightPos, 184 + (index * barSectionHeight), rightSize, 72);

      //percents
      //left percent
      if(results[slug] > 30) {
        ctx.fillStyle = "#222222";
        ctx.textAlign = "left";
        ctx.font = "50px Montserrat";
        ctx.fillText(results[slug] + "%", 130, 237.5 + (index * barSectionHeight));
      }

      if(complimentResults[slug] > 30) {
        ctx.fillStyle = "#222222";
        ctx.textAlign = "right";
        ctx.font = "50px Montserrat";
        ctx.fillText(complimentResults[slug] + "%", 670, 237.5 + (index * barSectionHeight));
      }

    }

    //Uncomment for debug region prints
    /*
    ctx.fillStyle = "#FF0000";
    ctx.fillRect(500, 0, 1, 100);
    ctx.fillRect(780, 0, 1, 100);
    ctx.fillStyle = "#0000FF";
    ctx.fillRect(20, 0, 1, 100);
    ctx.fillRect(480, 0, 1, 100);
    ctx.fillStyle = "#00FF00";
    ctx.fillRect(0, 150, 700, 1);
    ctx.fillRect(0, 280, 700, 1);
     */

    return c.toDataURL();
  }

  onMount(async () => {
    imageSrc = await generateImage();
  })
</script>

<svelte:head>
  <title>{name} Results</title>
</svelte:head>

<h1>Results</h1>

{#each values as { slug, name, labels, leftValue, rightValue }}
  <h2>{name[0].toUpperCase() + name.slice(1)} Axis: <span
    style='font-weight: 300'>{getLabel(labels, parseFloat(results[slug]))}</span></h2>
  <div class='axis'>
    <img class='icon' src='icons/{leftValue.name}.svg' alt={leftValue.name} />
    <div class='bar' style='border-right-style: solid; background-color: {leftValue.color}; width: {results[slug]}%'>
      {#if (parseFloat(results[slug]) >= 6)}
        <div class='text_wrapper'>{results[slug]}%</div>
      {/if}
    </div>
    <div
      class='bar'
      style='border-left-style: solid; text-align: right; background-color: {rightValue.color}; width: {complimentResults[slug]}%'
    >
      {#if (parseFloat(complimentResults[slug]) >= 6)}
        <div class='text_wrapper'>{complimentResults[slug]}%</div>
      {/if}
    </div>
    <img class='icon' src='icons/{rightValue.name}.svg' alt={rightValue.name} />
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
  results along with your political ideology to us at {contact.email}, or send us any comments,
  questions, or criticism.
</p>
<hr />
<div class='image_out' style="width: {getImageWidth()}px; height: {getImageHeight()}px">
  <img src={imageSrc} style='width: 100%; height: 100%'/>
</div>
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

  .image_out {
    margin: auto;
    border-color: #444444;
    border-style: solid;
    border-width: 2px;
    border-radius: 8pt;
    overflow: hidden;
  }

  img {
    height: 128pt;
  }
</style>
