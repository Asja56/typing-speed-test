<script lang="ts">
  let innerWidth = $state(0);
  let isMobile = $derived(innerWidth < 640);

  const difficulties = ["Easy", "Medium", "Hard"];
  const modes = ["Timed (60s)", "Passage"];

  let { selectedDifficulty = $bindable(), selectedMode = $bindable() } =
    $props();

  function selectDifficulty(value: string) {
    selectedDifficulty = value;
  }

  function selectMode(value: string) {
    selectedMode = value;
  }
</script>

<svelte:window bind:innerWidth />

{#if isMobile}
  <div class="flex gap-4">
    <div class="flex flex-col gap-2 grow">
      <select
        id="difficulty-select"
        bind:value={selectedDifficulty}
        class="bg-neutral-800 text-xs text-white border border-neutral-500 rounded px-4 py-1 focus:outline-none focus:border-blue-400"
      >
        {#each difficulties as difficulty}
          <option value={difficulty}>{difficulty}</option>
        {/each}
      </select>
    </div>

    <div class="flex flex-col gap-2 grow">
      <select
        id="mode-select"
        bind:value={selectedMode}
        class="bg-neutral-800 text-xs text-white border border-neutral-500 rounded px-4 py-1 focus:outline-none focus:border-blue-400"
      >
        {#each modes as mode}
          <option value={mode}>{mode}</option>
        {/each}
      </select>
    </div>
  </div>
{:else}
  <div class="flex justify-between gap-2">
    <div class="separator flex gap-2 items-center">
      <p class="pl-1">Difficulty:</p>
      {#each difficulties as difficulty}
        <button
          onclick={(e) => selectDifficulty(difficulty)}
          class="chip"
          class:chip-selected={selectedDifficulty === difficulty}
        >
          {difficulty}
        </button>
      {/each}
    </div>

    <div class="flex gap-2 items-center">
      <p class="pl-1">Mode:</p>
      {#each modes as mode}
        <button
          onclick={(e) => selectMode(mode)}
          class="chip"
          class:chip-selected={selectedMode === mode}
        >
          {mode}
        </button>
      {/each}
    </div>
  </div>
{/if}
