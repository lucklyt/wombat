<script>
  // FieldMask editor: internal UI uses an array of strings but syncs state as a single comma-separated string.
  import { onMount } from "svelte";
  import InputLabel from "../controls/InputLabel.svelte";
  import CrossButton from "../controls/CrossButton.svelte";

  export let field; // { name, kind }
  export let state; // backing state object
  export let key;   // map key (optional)
  export let idx;   // repeated index (optional)

  let valKey; // key into state
  let labelColor;
  let removeable;

  // Internal representation: array of { v: string }
  let paths = [];

  onMount(() => {
    initFromState();
  });

  function initFromState() {
    valKey = key !== undefined ? key : idx >= 0 ? idx : field.name;
    const raw = state[valKey];
    if (typeof raw === 'string' && raw.trim() !== '') {
      paths = raw.split(',').map(s => ({ v: s.trim() })).filter(p => p.v.length > 0);
    } else if (Array.isArray(raw)) { // defensive, if backend someday sends array
      paths = raw.map(s => ({ v: String(s).trim() })).filter(p => p.v.length > 0);
    } else {
      paths = [];
    }
  }

  $: {
    valKey = key !== undefined ? key : idx >= 0 ? idx : field.name;
    labelColor = key !== undefined ? "var(--accent-color3)" : idx >= 0 ? "var(--accent-color2)" : undefined;
    removeable = idx >= 0;
  }

  // Sync state whenever paths change
  $: state[valKey] = paths.map(p => p.v).filter(v => v.length > 0).join(',');

  function addPath() {
    paths = [...paths, { v: '' }];
  }

  function removePath(i) {
    paths.splice(i, 1);
    paths = paths; // trigger update
  }
</script>

<style>
  .fm-wrapper {
    min-width: 400px;
    margin-bottom: var(--padding);
  }
  .label-row {
    display: flex;
    align-items: center;
  }
  .paths {
    padding-left: var(--padding);
    position: relative;
  }
  .path-item {
    display: flex;
    align-items: center;
    margin-bottom: calc(var(--padding) * 0.5);
  }
  .path-item input {
    flex: 1;
    background-color: var(--bg-input-color);
    border: var(--border);
    outline: none;
    color: var(--text-color);
    padding: var(--padding);
    font-size: var(--font-size);
  }
  .path-item input::placeholder { color: var(--text-color3); }
  .add-btn { margin-left: auto; }
  .empty-hint { color: var(--text-color3); font-size: 12px; padding-left: var(--padding); }
</style>

<div class="fm-wrapper">
  <div class="label-row">
    <InputLabel on:remove {removeable} label={field.name} color={labelColor} hint={field.kind} block />
    <CrossButton class="add-btn" color={isWin ? "#a3be8c" : "var(--green-color)"} add on:click={addPath} />
  </div>
  <div class="paths">
    {#if paths.length === 0}
      <div class="empty-hint">camelStyle</div>
    {/if}
    {#each paths as p, i}
      <div class="path-item">
        <CrossButton color="var(--red-color)" on:click={() => removePath(i)} />
        <input placeholder="field.path" bind:value={p.v} />
      </div>
    {/each}
  </div>
</div>
