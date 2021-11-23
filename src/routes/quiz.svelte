<script>
  import Info from '$lib/data/info.json';
  import Values from '$lib/data/values.json';
  import Questions from '$lib/data/questions.json';
  import { resultsStore } from '$lib/resultsStore'
  import { goto } from '$app/navigation';

  const {
    name
  } = Info;

  const {
    values
  } = Values;

  const {
    questions
  } = Questions;

  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  const randomizedQuestions = shuffle(questions);

  let questionNumber = 0;
  $: questionText = randomizedQuestions[questionNumber].question;
  $: firstQuestion = questionNumber <= 0;

  let maxes = {};
  let results = {};
  for (const { slug } of values) {
    maxes[slug] = 0;
    results[slug] = Array(questions.length);
  }

  for (const question of randomizedQuestions) {
    for (const [slug, value] of Object.entries(question.effect)) {
      maxes[slug] += Math.abs(value);
    }
  }

  function nextQuestion(multiplier) {

    const questionEffect = randomizedQuestions[questionNumber].effect;

    for (const [slug, value] of Object.entries(questionEffect)) {
      results[slug][questionNumber] = value * multiplier;
    }
    console.log(results);

    questionNumber += 1;
    if (questionNumber >= questions.length) {
      calcResults();
    }
  }

  function calcResults() {
    let results = {};
    for (const [slug, subresult] in Object.entries(results)) {
      let total = subresult.reduce((acc, value) => acc += (value ? value : 0), 0);
      results[slug] = (100 * (maxes[slug] + total) / (2 * maxes[slug])).toFixed(1);
    }
    resultsStore.set(results);
    const query = '?' + Object.entries(results).map(([slug, value]) => slug + '=' + value).join('&');
    goto('/results' + query);
  }

  function previousQuestion() {
    if (!firstQuestion) {
      questionNumber -= 1;
    }
  }

</script>

<svelte:head>
  <title>{name} Quiz</title>
</svelte:head>

<h2 style='text-align: center'>Question {questionNumber + 1} of {questions.length}</h2>
<p class='question'>
  {questionText}
</p>
<button class='stronglyAgree' on:click={() => nextQuestion(1.0)}>Strongly Agree</button>
<button class='agree' on:click={() => nextQuestion(0.5)}>Agree</button>
<button class='neutral' on:click={() => nextQuestion(0)}>Neutral</button>
<button class='disagree' on:click={() => nextQuestion(-0.5)}>Disagree</button>
<button class='stronglyDisagree' on:click={() => nextQuestion(-1)}>Strongly Disagree</button>
<button class='neutral' on:click={() => nextQuestion(0)}>Don't Know/Understand</button>
<button class='small_button' class:small_button_disabled={firstQuestion} on:click={previousQuestion}>Back</button>

<style>
  .small_button {
    background-color: #333;
    font-family: 'Montserrat', sans-serif;
    border: none;
    border-radius: 8pt;
    padding: 8pt;
    width: 10%;
    min-width: 100pt;
    text-align: center;
    text-decoration: none;
    display: block;
    font-size: 18pt;
    margin: -2px auto;
    cursor: pointer;
  }

  .small_button:hover, .small_button:focus {
    background: #222;
  }

  .small_button_disabled {
    background-color: #ddd;
    color: #888;
    border: 2px solid #888;
    cursor: not-allowed;
  }

  .small_button_disabled:hover, .small_button_disabled:focus {
    background-color: #ddd;
  }
</style>
