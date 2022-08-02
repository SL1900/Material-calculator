<script>
  import { v4 as uuidv4 } from "uuid";

  import Item from "./Item.svelte";

  let modalShown = false;

  class MaterialItem {
    constructor(name, count, children = []) {
      this.name = name;
      this.count = count;
      this.children = children;
      this.uuid = uuidv4();
    }
  }

  $: list = localStorage.getItem("LastList")
    ? JSON.parse(localStorage.getItem("LastList"))
    : [
        new MaterialItem("Potter", 1, [
          new MaterialItem("Ceramic", 5, [
            new MaterialItem("Clay", 2, [
              new MaterialItem("Sand", 2),
              new MaterialItem("Water", 1),
            ]),
          ]),
          new MaterialItem("Water", 1),
        ]),
      ];

  let total = [];

  $: list && updateTotal() && saveData();

  function saveData() {
    console.log("Saving");
    localStorage.setItem("LastList", JSON.stringify(list));
  }
  function updateTotal() {
    total = get_counts(list);
    return true;
  }

  function get_counts(container, multiplier = 1) {
    let totals = {};
    for (let item of container) {
      if (!item) continue;

      if (!totals[item.name] && !item.children.length) {
        totals[item.name] = item.count * multiplier;
      } else if (!item.children.length) {
        totals[item.name] += item.count * multiplier;
      }

      let child_totals = get_counts(item.children, item.count * multiplier);

      if (Object.entries(child_totals).length == 0) continue;

      for (let child_total of Object.entries(child_totals).map((e) => {
        return { name: e[0], count: e[1], children: e[2] || [] };
      })) {
        if (!totals[child_total.name])
          totals[child_total.name] = child_total.count;
        else totals[child_total.name] += child_total.count;
      }
    }

    return totals;
  }

  function AddChild({ detail: data }) {
    let route = getItemRoute(data.uuid, list);
    let pointer = list[route[0]];

    for (let i = 1; i < route.length; i++) {
      pointer = pointer.children[route[i]];
    }

    pointer.children.push(new MaterialItem("New Item", 1, []));

    list = list;
  }
  function RemoveChild({ detail: data }) {
    let route = getItemRoute(data.uuid, list);
    if (route.found) route = [route.index];

    let pointer = list[route[0]];

    for (let i = 1; i < route.length - 1; i++) {
      pointer = pointer.children[route[i]];
    }

    pointer.children.splice([route[route.length - 1]], 1);
    list = list;
  }

  function getItemRoute(uuid, container) {
    for (let [index, item] of container.entries()) {
      if (item.uuid == uuid)
        return {
          found: true,
          index,
        };

      if (!item.children.length) continue;

      let res = getItemRoute(uuid, item.children);
      if (res) {
        if (res.found) return [index, res.index];

        return [index, ...res];
      }
    }

    return false;
  }
  function ChangeCount({ detail: data }) {
    let route = getItemRoute(data.uuid, list);
    if (route.found) route = [route.index];

    let pointer = list[route[0]];

    for (let i = 1; i < route.length; i++) {
      pointer = pointer.children[route[i]];
    }

    pointer.count = data.count;

    list = list;
  }

  function formRouteString() {
    let result_string = "";

    let route = getItemRoute(modal_data.uuid, list);
    if (route.found) route = [route.index];

    let pointer = list[route[0]];

    for (let i = 1; i < route.length; i++) {
      pointer = pointer.children[route[i]];

      result_string += `${pointer.name} -> `;
    }

    return result_string.replace(/\s->\s$/gi, "");
  }
  function exportJSON() {
    navigator.clipboard.writeText(JSON.stringify(list));
  }
  function nameChange({ detail: data }) {
    let route = getItemRoute(data.uuid, list);
    if (route.found) route = [route.index];

    let pointer = list[route[0]];

    for (let i = 1; i < route.length; i++) {
      pointer = pointer.children[route[i]];
    }

    pointer.name = data.name;

    list = list;
  }
</script>

<main>
  <div class="app-title">MATERIAL CALCULATOR</div>
  <div class="mainBody">
    <div class="left">
      {#each list as item}
        <Item
          {...item}
          on:CountChange={ChangeCount}
          on:RemoveChild={RemoveChild}
          on:AddChild={AddChild}
          on:NameChange={nameChange}
          sum={false}
          editable={true}
        />
      {/each}
    </div>
    <div class="right">
      {#each list as item}
        <Item
          {...item}
          on:CountChange={ChangeCount}
          on:RemoveChild={RemoveChild}
          on:AddChild={AddChild}
          on:NameChange={nameChange}
          sum={true}
          editable={false}
        />
      {/each}
    </div>
  </div>
  <div class="total-label">TOTAL:</div>
  (bullshit)
  <div class="total">
    {#each Object.entries(total) as item}
      <div class="total_item">
        <div class="total_name">
          {item[0]}
        </div>
        <div>
          x {item[1]}
        </div>
      </div>
    {/each}
  </div>
  <button on:click={exportJSON}>Export JSON</button>
</main>

<style>
  main {
    display: flex;
    position: absolute;
    inset: 0;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .app-title {
    font-weight: 900;
    font-size: 3.5rem;
    text-align: center;
  }
  .mainBody {
    display: flex;
    flex: 1;
    overflow: auto;
  }
  .mainBody > * {
    margin: 10px;
  }
  .total-label {
    font-weight: 700;
    font-size: 2rem;
  }
  .total {
    display: flex;
    justify-content: space-between;
  }
  .total_item {
    display: flex;
    padding: 2px;
    border: 2px solid gray;
    margin: 5px;
    flex-wrap: wrap;
  }
  .total_name {
    margin-right: 5px;
    font-weight: 600;
  }
</style>
