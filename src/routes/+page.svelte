<script lang="ts">
  let position = $state(0);

  function handle(e: KeyboardEvent) {
    console.log(e.which);

    let direction = 0;
    let multiplier = 0;

    if (['ArrowRight', 'ArrowDown'].includes(e.key)) {
      direction = 1;
    } else if (['ArrowLeft', 'ArrowUp'].includes(e.key)) {
      direction = -1;
    }
  }
</script>

<svelte:window onkeydown={handle} />

<main class="flex flex-col items-center justify-center h-screen">
  <div class="flex flex-row gap-4">
    {#each { length: 3 } as _}
      <div class="size-12 border border-black"></div>
    {/each}
  </div>
  <button onclick={() => position++}>increase</button>
  <button onclick={() => position--}>decrease</button>
  <div class="flex flex-wrap size-96 relative">
    {#each { length: 16 } as _, i}
      <div class:active={position == i} class="size-24 border border-black"></div>
    {/each}
  </div>
</main>

<style>
  .active {
    border: 5px red solid;
  }
</style>
