<script lang="ts">
  import { onDestroy } from 'svelte';
  import { SvelteMap } from 'svelte/reactivity';

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
  let operator = $state<keyof typeof f>('_');
  let number_slots = $state<Array<(typeof numbers)[number]>>([]);
  let map = $state(new SvelteMap<number, (typeof numbers)[number] | keyof typeof f>());

  let interval: any;

  function handle(e: KeyboardEvent) {
    if (e.code == 'Space') {
      let item = map.get(position);
      if (typeof item === 'string' && operator == '_') {
        operator = item;
        map.delete(position);
      } else if (typeof item === 'number' && number_slots.length < 2) {
        number_slots.push(item);
        map.delete(position);
      }

      let val_calculated = false;
      let operator_dropped = false;

      if (number_slots.length == 2 && operator != '_') {
        for (let i = 0; i < map.size; i++) {
          let val = map.get(i);
          if (!val_calculated && val === undefined) {
            // @ts-expect-error
            map.set(i, f[operator](...number_slots));
            number_slots = [];
            val_calculated = true;
            continue;
          }

          if (!operator_dropped && val === undefined) {
            map.set(i, operator);
            operator = '_';
            operator_dropped = true;
            continue;
          }

          if (val_calculated && operator_dropped) return;
        }
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

  // svelte-ignore state_referenced_locally
  for (let i = 0; i < cell_count * cell_count; i++) {
    if (i % 5 == 0 || i % 4 == 0) continue;

    if (i % 3 == 0) {
      rand = operators[Math.floor(Math.random() * operators.length)];
    } else {
      rand = [1, 2, 3][Math.floor(Math.random() * 3)];
    }

    map.set(i, rand);
  }

  function restart_interval() {
    clearInterval(interval);
    time = 0;

    interval = setInterval(() => {
      time += 1;
      if (time >= 13) {
        console.log('game over');
      }
    }, 1000);
  }

  restart_interval();

  onDestroy(() => {
    clearInterval(interval);
  });
</script>

<svelte:window onkeydown={handle} />

<main class="flex flex-col items-center justify-center h-screen gap-8">
  <button onclick={restart_interval}>restart</button>
  <div class="flex flex-row gap-4">
    <span>{number_slots[0]}</span>
    <span>{operator == '_' ? '' : operator}</span>
    <span>{number_slots[1]}</span>
  </div>
  <div class="flex flex-wrap size-96 relative">
    <div class="absolute -top-12 -right-12">{time}</div>
    {#each { length: 16 } as _, i}
      <div
        class:active={position == i}
        class="size-24 border border-black flex items-center justify-center"
      >
        {map.get(i)}
      </div>
    {/each}
  </div>
</main>

<style>
  .active {
    border: 5px red solid;
  }

  span {
    @apply flex items-center justify-center size-12 border border-black;
  }
</style>
