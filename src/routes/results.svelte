<script>
  import Info from '$lib/data/info.json';
  import Values from '$lib/data/values.json';
  import Questions from '$lib/data/questions.json';
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';

  const {
    values
  } = Values;

  const {
    query,
  } = page;

  let results = {}
  if(query) {
    for (const [key, value] of query) {
      results[key] = value;
    }
  }
  console.log(results)
  console.log('query', query)

  const {
    name,
  } = Info;
</script>

<svelte:head>
  {name} Results
</svelte:head>

<h1>Results</h1>

{#each values as {slug, name, leftValue, rightValue}}
  <h2>{name[0].toUpperCase() + name.slice(1)} Axis: <span class='weight-300'>{'result'}</span></h2>
  <div class='axis'>
    <img class='icon' src="icons/{leftValue.name}.svg" alt={leftValue.name} />
    <div class='bar' style="border-right-style: solid; background-color: {leftValue.color}"><div class='text_wrapper'>{results[slug]}%</div></div>
    <div class='bar' style="border-left-style: solid; text-align: right; background-color: {rightValue.color}"><div class='text_wrapper'>{'50'}%</div></div>
    <img class='icon' src="icons/{rightValue.name}.svg" alt={rightValue.name} />
  </div>
{/each}
<h2>Closest Match:</h2>
<p>
  Ideological matching is a work in progress, and is much less accurate than the values and axes.
</p>
<p>
  You can send these results by copying and pasting the URL at the top of the page or using the
  image below. Think your matched ideology was wrong? Want to help us calibrate the test? Send the
  results along with your political ideology to us at eightvalues@gmail.com, or send us any
  comments, questions, or criticism.
</p>
<hr/>
<button>Back</button>


<style>
.axis {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bar {
  width: 50%;
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
