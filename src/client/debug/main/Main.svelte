<script>
  export let client;

  import JSONTree from 'svelte-json-tree-auto/src/Root.svelte';
  import Move from './Move.svelte';
  import Controls from './Controls.svelte';
  import PlayerInfo from './PlayerInfo.svelte';
  import { AssignShortcuts } from '../utils/shortcuts';

  const shortcuts = AssignShortcuts(client.moves, 'mlia');

  function SanitizeCtx(ctx) {
    let r = {};
    for (const key in ctx) {
      if (!key.startsWith('_')) {
        r[key] = ctx[key];
      }
    }
    return r;
  }

  let { playerID, moves, events } = client;
  let ctx = {};
  let G = {};
  client.subscribe((state) => {
    if (state) ({ G, ctx } = state);
    ({ playerID, moves, events } = client);
  });
</script>

<style>
  .tree {
    --json-tree-font-family: monospace;
    --json-tree-font-size: 14px;
    --json-tree-null-color: #757575;
  }

  label {
    font-weight: bold;
    font-size: 1.1em;
    display: inline;
  }

  h3 {
    text-transform: uppercase;
  }

  li {
    list-style: none;
    margin: none;
    margin-bottom: 5px;
  }
</style>

<section>
  <h3>Controls</h3>
  <Controls {client} />
</section>

<section>
  <h3>Players</h3>
  <PlayerInfo
    on:change={(e) => client.updatePlayerID(e.detail.playerID)}
    ctx={ctx}
    playerID={playerID}
  />
</section>

<section>
  <h3>Moves</h3>
  {#each Object.entries(moves) as [name, fn]}
    <li>
      <Move shortcut={shortcuts[name]} {fn} {name} />
    </li>
  {/each}
</section>

<section>
  <h3>Events</h3>

  <div class="events">
  {#if ctx.activePlayers && events.endStage}
    <li>
      <Move name="endStage" shortcut={7} fn={events.endStage} />
    </li>
  {/if}
  {#if events.endTurn}
    <li>
      <Move name="endTurn" shortcut={8} fn={events.endTurn} />
    </li>
  {/if}
  {#if ctx.phase && events.endPhase}
    <li>
      <Move name="endPhase" shortcut={9} fn={events.endPhase} />
    </li>
  {/if}
  </div>
</section>

<section class="tree">
  <label>G</label>
  <JSONTree value={G} />
</section>

<section class="tree">
  <label>ctx</label>
  <JSONTree value={SanitizeCtx(ctx)} />
</section>
