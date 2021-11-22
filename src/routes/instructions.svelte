<script>
  import Info from '$lib/data/info.json'
  import { goto } from '$app/navigation'

  const {
    instructions
  } = Info;
</script>

<h2 style={'width: 100%; text-align: center'}>Instructions</h2>
<p class='question'>
  {instructions}
</p>
<button on:click={() => goto('/quiz')}>Got it!</button>
<button class='disagree' on:click={() => goto('/')}>Wait, nevermind!</button>
