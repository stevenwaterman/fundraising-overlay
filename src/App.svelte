<script lang="ts">
  import { currentSelector, decimals, symbol, targetSelector, url } from "./config";
  import { tweened } from "svelte/motion";
  import { derived } from "svelte/store";

  const parser = new DOMParser();

  let html: string | null = null;

  let doc: Document | null;
  $: doc = html === null ? null : parser.parseFromString(html, 'text/html');


  let currentNode: Element | null;
  $: currentNode = doc === null ? null : doc.querySelector(currentSelector);

  let currentAmountString: string | null;
  $: currentAmountString = currentNode === null ? null : currentNode.textContent;

  let currentAmountNumber: number | null;
  $: currentAmountNumber = currentAmountString === null ? null : parseFloat(currentAmountString);


  let targetNode: Element | null;
  $: targetNode = doc === null ? null : doc.querySelector(targetSelector);

  let targetAmountString: string | null;
  $: targetAmountString = targetNode === null ? null : targetNode.textContent;

  let targetAmountNumber: number | null;
  $: targetAmountNumber = targetAmountString === null ? null : parseFloat(targetAmountString);


  const current = tweened(0, { duration: 1000 });
  const target = tweened(0, { duration: 500 });
  const percentage = derived([current, target], (([currentVal, targetVal]) => {
    if(targetVal === 0) return 0;
    else return 100 * currentVal / targetVal;
  }));

  $: if(currentAmountNumber !== null) current.set(currentAmountNumber, {});
  $: if(targetAmountNumber !== null) target.set(targetAmountNumber, {});


  function updateVariables() {
    fetch(`https://api.allorigins.win/get?url=${encodeURIComponent(url)}`)
      .then(res => res.json())
      .then(body => body.contents)
      .then(text => html = text);
  }

  setTimeout(updateVariables, 10 * 1000);
  updateVariables();
</script>

<style>
  :global(body) {
    padding: 0;
    overflow: hidden;
  }

  .barBackground {
    width: 100vw;
    height: 50vh;
    border-radius: 25vh;
    background-color: #fdffff;
  }

  .barForeground {
    height: 50vh;
    border-radius: 25vh;
    background-color: #34506a;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .percentage {
    font-size: 20vh;
    color: #fdffff;
  }

  .textBackground {
    height: 50vh;
    width: 100vw;
    display: flex;
    justify-content: center;
    align-items: flex-start;
  }

  .totals {
    font-size: 8vw;
    color: #fdffff;
  }
</style>



<div class="barBackground">
  <div class="barForeground" style={`width: ${$percentage}vw`}>
    <span class="percentage">
      {$percentage.toFixed(0)}%
    </span>
  </div>
</div>

<div class="textBackground">
  <span class="totals">
    {symbol}{$current.toFixed(decimals)} / {symbol}{$target.toFixed(decimals)} 
  </span>
</div>