<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  export let enabled = false;
  export let list = [];

  let save_name = "";

  function selectItem(index) {
    dispatch("SidebarSelect", { index });
  }

  function deleteItem(index) {
    dispatch("SidebarDelete", { index });
  }

  let save_cooldown = false;
  function saveCurrentItem() {
    if (save_name && !save_cooldown) {
      save_name = save_name.replace(/\s+$/gi, "");
      dispatch("SidebarSave", { save_name });

      save_cooldown = true;
      setTimeout(() => {
        save_cooldown = false;
      }, 500);
    }
  }
</script>

<main class={enabled ? "shown" : ""}>
  <!-- + -->
  <div class="body">
    <div class="title">Saved recipes</div>
    <div class="list">
      {#each list as item, index}
        <div
          class="item"
          {item}
          on:click={() => {
            selectItem(index);
          }}
        >
          <div class="name">{item.name}</div>
          <div
            class="delete-btn"
            on:click={() => {
              deleteItem(index);
            }}
          >
            X
          </div>
        </div>
      {/each}
    </div>
    <div class="bottom-row">
      <div>Save name:</div>
      <input
        type="text"
        class="save-name-input"
        bind:value={save_name}
        placeholder="Enter your item name here"
      />
      <button class="save-btn" on:click={saveCurrentItem}>Save</button>
    </div>
  </div>
  <div class="arrow" on:click={() => (enabled = !enabled)}>
    {enabled ? "<<" : ">>"}
  </div>
</main>

<style>
  main {
    --arrow-width: 20px;
    position: absolute;
    inset: 0;
    width: 50%;
    display: flex;
    flex-direction: row;
    transform: translate(calc(-100% + var(--arrow-width) - 26px));
    background-color: rgb(225, 225, 225);
    transition: all 0.25s;
  }
  .shown {
    transform: translate(-30px);
  }
  .title {
    font-weight: 600;
    font-size: 1.25rem;
    display: flex;
    justify-content: center;
    user-select: none;
  }
  .body {
    flex: 1;
    display: flex;
    flex-direction: column;
  }
  .arrow {
    width: var(--arrow-width);
    border: 2px solid gray;
    margin: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    user-select: none;
  }
  .item {
    border: 2px solid gray;
    border-radius: 5px;
    padding: 5px;
    margin: 2px;
    user-select: none;
    display: flex;
  }
  .name {
    flex: 1;
  }
  .list {
    display: flex;
    flex-direction: column;
    overflow: auto;
    flex: 1;
  }
  .bottom-row {
    display: flex;
    justify-content: center;
  }
  .bottom-row div {
    display: flex;
    align-items: center;
    margin-bottom: 0.5rem;
  }
  .save-btn {
    margin-left: 0.5rem;
  }
</style>
