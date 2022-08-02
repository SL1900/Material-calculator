<script>
  import Item from "./Item.svelte";
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  export let name = "Undefined";
  export let count = sum;
  export let children = [];
  export let sum = false;
  export let miltiplier = 1;
  export let uuid;
  export let editable = false;
  let name_temp = name;

  $: count_res = sum ? count * miltiplier : count;
  function addChild() {
    dispatch("AddChild", { uuid });
  }
  function removeChild() {
    dispatch("RemoveChild", { uuid });
  }
  function countChange() {
    dispatch("CountChange", { uuid, count });
  }
  function nameChange() {
    dispatch("NameChange", { uuid, name: name_temp });
  }
</script>

<main id={uuid}>
  <div class="text">
    {#if editable}
      <input
        type="text"
        class="name"
        bind:value={name_temp}
        on:input={nameChange}
      />
    {:else}
      <div class="name">{name}</div>
    {/if}
    <div>
      x
      {#if sum}
        {count_res}
      {:else}
        <input
          class="item-count-input"
          type="number"
          min="1"
          bind:value={count}
          on:input={countChange}
          on:change={countChange}
        />
      {/if}
    </div>
    {#if !sum}
      <div class="btn" on:click={addChild}>+</div>
      <div class="btn" on:click={removeChild}>-</div>
    {/if}
  </div>
  {#if children.length}
    <div class="children">
      {#each children as item}
        <Item
          {...item}
          {sum}
          {editable}
          on:RemoveChild
          on:AddChild
          on:CountChange
          on:NameChange
          miltiplier={count_res}
        />
      {/each}
    </div>
  {/if}
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    border: 2px solid gray;
    margin: 2px;
    padding: 5px;
  }
  .text {
    display: flex;
    align-items: center;
  }
  .name {
    font-weight: 600;
    margin-right: 5px;
  }
  .children {
    padding-left: 10px;
  }
  .btn {
    margin: 2px;
    border: 2px solid lightgray;
    height: 50%;
    user-select: none;
    cursor: pointer;
    margin: 2px;
    padding: 2px 4px;
    min-width: 10px;
    text-align: center;
  }
  .item-count-input {
    margin: 0;
    width: 80px;
    height: 30px;
  }
</style>
