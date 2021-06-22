<script lang="ts">
  import { currentSelector, decimals, symbol, targetSelector, url } from "./config";
  import { tweened } from "svelte/motion";
  import { derived } from "svelte/store";

  let cacheBypassIncrement = 0;
  const parser = new DOMParser();

  function getValue(doc: Document, selector: string): number | null {
    const element = doc.querySelector(selector);
    if(element === null) return null;

    const text = element.textContent;
    if(text === null) return null;

    return parseFloat(text);
  }

  const current = tweened(0, { duration: 1000 });
  const target = tweened(0, { duration: 500 });
  const percentage = derived([current, target], (([currentVal, targetVal]) => {
    if(targetVal === 0) return 0;
    else return 100 * currentVal / targetVal;
  }));

  function updateVariables() {
    cacheBypassIncrement++;
    fetch(`https://api.allorigins.win/raw?url=${encodeURIComponent(url)}?${cacheBypassIncrement}=1`, {
      cache: "no-store"
    }).then(res => res.text())
      .then(html => parser.parseFromString(html, 'text/html'))
      .then(doc => ({
        currentVal: getValue(doc, currentSelector),
        targetVal: getValue(doc, targetSelector)
      }))
      .then(({currentVal, targetVal}) => {
        if(currentVal !== null) current.set(currentVal, {});
        if(targetVal !== null) target.set(targetVal, {});
      });
  }

  setInterval(updateVariables, 10 * 1000);
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
    margin: 4px;
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