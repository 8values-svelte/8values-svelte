<script context='module' lang='ts'>
  export const prerender = true;
</script>

<script lang='ts'>
  import Info from '$lib/data/info.json';
  import Values from '$lib/data/values.json';
  import Questions from '$lib/data/questions.json';
  import converter from 'number-to-words';
  import { goto } from '$app/navigation';

  const {
    name,
    description,
    typeOfQuiz,
    repo,
    contact
  } = Info;

  const {
    values
  } = Values;

  const {
    questions
  } = Questions;

  const names = values
    .map(value => {
      const name = value.name;
      return name[0].toUpperCase() + name.slice(1);
    });

  const namesList = names.slice(0, -1).join(', ') + ' and ' + names.slice(-1);

  const numAxes = converter.toWords(values.length);
  const numValues = converter.toWords(values.length * 2);

</script>

<svelte:head>
  <title>{name}</title>
</svelte:head>

<div class='center icons_box'>
  {#each values as { name, leftValue, rightValue }}
    <div class='icons_column'>
      <div class='icons_header'>
        {name.toUpperCase()}
      </div>
      <a href='#anchor'>
        <img class='icon' src={'icons/' + leftValue.name + '.svg'} alt={leftValue.name} />
      </a>
      <a href='#anchor'>
        <img class='icon' src={'icons/' + rightValue.name + '.svg'} alt={rightValue.name} />
      </a>
    </div>
  {/each}
</div>
<br>
<button on:click={() => goto('/instructions')}>Click here to start!</button>
<br>
<hr>
<h2>What is {name}?</h2>
<p>
  {description}
  <br>
  <br>
  You will be presented with a statement, and then you will answer your opinion on the statement,
  from <b>strongly Agree</b> to <b>Strongly Disagree</b>, with each answer slightly affecting your
  scores. At the end of the quiz, your answers will be compared to the maximum possible for each value,
  thus giving you a percentage. Answer honestly
  <br>
  Answer honestly!
  <br>
  <br>
  This specific version is a Svelte based rewrite of the original 8values quiz. It is made to be
  extendable and easily modifiable, while cleaning up the codebase and smoothing out the user
  experience. Check out this guide on the github for how to make your own version!
  <br>
  <br>
  There are <b><u>{questions.length}</u></b> questions in the test.
</p>
<h2><a id='anchor'>What are the {numValues} values?</a></h2>
<p>
  There are {numAxes} independent axes - {namesList} - and each has two opposing values assigned to them.
  <br>
  <br>
  They are:
</p>
<div class='explanation_bg'>
  {#each values as { name, leftValue, rightValue }}
    <div class='spacer'>
      <div class='explanation_blurb_left'>
        <p class='value' style={'color:' + leftValue.color}>
          <b>{leftValue.name.toUpperCase()}</b>
        </p>
        <p class='blurb_text'>
          {leftValue.description}
        </p>
      </div>
      <div class='explanation_axis'>
        <p class='axis_name'>
          {name.toUpperCase()}
        </p>
        <img class='arrow' src='double_arrow.svg' alt='double ended arrow' />
      </div>
      <div class='explanation_blurb_right'>
        <p class='value' style={'color:' + rightValue.color}>
          <b>{rightValue.name.toUpperCase()}</b>
        </p>
        <p class='blurb_text'>
          {rightValue.description}
        </p>
      </div>
    </div>
  {/each}
</div>

<h2>What's the "Closest Match" mean at the bottom of the results?</h2>
<p>
  In addition to matching you to the eight values, the quiz also attempts to match you to a
  {typeOfQuiz} ideology. This is a work in progress and is much less accurate than the values and axes,
  so don't take it too seriously. If you disagree with your assigned ideology, send us an email with
  your scores, matched ideology, and preferred ideology, and we'll look into adjusting the system.
  Thanks!
</p>
<h2>I don't like my scores!</h2>
<p>
  ¯\_(ツ)_/¯
  <br />
  If you have any suggestions or constructive criticism, feel free to contact us!
<p>
<ul>
  <li>
    Email: {contact.email}
  </li>
  {#if (contact.github)}
    <li>
      Github: <a href="https://github.com/{contact.github}">@{contact.github}</a>
    </li>
  {/if}
</ul>

<p>
  or open an issue on the Github Repository here: <a href={repo}>Github Page</a>
</p>


<style>
  .icons_box {
    background-color: #eeeeee;
    border-radius: 8px;
    margin: auto;
    padding: 6pt;
    width: 50%;
    min-width: 488pt;
    display: flex;
    font-size: 24pt;
    gap: 7pt;
    text-align: center;
  }

  .icons_column {
    display: flex;
    flex-grow: 1;
    flex-direction: column;
    gap: 7pt;
  }

  .icons_header {
    color: #333333;
    font-size: 19pt;
    font-family: 'Montserrat', serif;
    display: inline-block;
    padding-bottom: 8.65pt;
    text-align: center;
  }

  .icon {
    transition: transform 0.3s;
    width: 100%;
  }

  .icon:hover {
    transform: scale(1.05);
  }

  .explanation_bg {
    background-color: #eeeeee;
    border-radius: 25px;
    margin-top: 15px;
  }

  .spacer {
    display: flex;
  }

  .spacer > div {
    margin-top: 10px;
    display: inline-block;
    *display: inline;
    text-align: center;
  }

  .explanation_blurb_left {
    vertical-align: top;
    width: 37%;
    margin-left: 1%;
    margin-right: 1%;
  }

  .axis_name {
    color: #333333;
    font-size: 19pt;
    font-family: 'Montserrat';
    margin-top: 50px;
  }

  .arrow {
    width: 60%;
    height: auto;
  }

  .explanation_axis {
    width: 20.9%;
    vertical-align: top;
  }

  .explanation_blurb_right {
    vertical-align: top;
    width: 37%;
    margin-left: 1%;
    margin-right: 1%;
  }


</style>
