<script lang="ts">
  import { onDestroy } from 'svelte';
  import { SvelteMap } from 'svelte/reactivity';

  // rules
  // total number cannot exceed 13
  //
  // TODO: cycle operators
  // sum all numbers when cycle is completed

  const numbers = [1, 2, 3] as const;

  const f = {
    '+': (x: number, y: number) => x + y,
    '-': (x: number, y: number) => x - y,
    x: (x: number, y: number) => x * y,
    _: () => undefined
  };
  const operators = ['+', '-', 'x'] as const;

  let time = $state(0);
  let cell_count = $state(4);
  let position = $state(0);
  let operator = $state<keyof typeof f>('+');
  let operator_index = $state(0);
  let number_slots = $state<Array<(typeof numbers)[number]>>([]);
  let map = $state(new SvelteMap<number, (typeof numbers)[number] | keyof typeof f>());

  let interval: any;

  function handle(e: KeyboardEvent) {
    if (e.code == 'Space') {
      let item = map.get(position);
      if (typeof item === 'number' && number_slots.length < 2) {
        number_slots.push(item);
        map.delete(position);
      }

      if (number_slots.length == 2) {
        // @ts-expect-error
        const res = f[operator](...number_slots);
        if (res != 0) {
          // @ts-expect-error
          map.set(position, res);
        } else {
          map.delete(position);
        }

        operator_index++;
        operator = operators[operator_index % 3];
        number_slots = [];
      }

      return;
    }

    let effect = 0;

    switch (e.code) {
      case 'ArrowRight':
        if ((position + 1) % cell_count != 0) {
          effect = 1;
        }
        break;
      case 'ArrowLeft':
        if (position % cell_count != 0) {
          effect = -1;
        }
        break;
      case 'ArrowUp':
        if (position - cell_count >= 0) {
          position -= cell_count;
        }
        break;
      case 'ArrowDown':
        if (position + cell_count < cell_count * cell_count) {
          position += cell_count;
        }
        break;
    }

    position += effect;
  }

  let rand: any;

  function restart_game() {
    map = new SvelteMap();
    restart_interval();
    // svelte-ignore state_referenced_locally
    for (let i = 0; i < cell_count * cell_count; i++) {
      if (i % 3 == 0) {
        continue;
      } else {
        rand = numbers[Math.floor(Math.random() * 3)];
      }

      map.set(i, rand);
    }
  }

  function restart_interval() {
    clearInterval(interval);
    time = 0;

    interval = setInterval(() => {
      time += 1;
      if (time >= 13) {
        restart_game();
      }
    }, 1000);
  }

  restart_game();

  onDestroy(() => {
    clearInterval(interval);
  });
</script>

<svelte:window onkeydown={handle} />

<main
  class="flex flex-col items-center justify-center h-screen gap-8 bg-purple-950 text-yellow-200"
>
  <button onclick={restart_interval}>restart</button>
  <div class="flex flex-row gap-4">
    <div class="cell">{number_slots[0]}</div>
    <div class="cell">{operator == '_' ? '' : operator}</div>
    <div class="cell">{number_slots[1]}</div>
  </div>
  <div class="flex flex-wrap size-96 relative">
    <div class="absolute -top-12 -right-12 text-3xl">{time + 1}</div>
    {#each { length: 16 } as _, i}
      <div class:active={position == i} class="cell">
        {map.get(i)}
      </div>
    {/each}
  </div>
</main>

<style>
  .cell {
    @apply shadow-inner bg-purple-900 font-bold text-5xl size-24  flex items-center justify-center;
  }

  .active {
    @apply bg-purple-700;
  }
</style>
