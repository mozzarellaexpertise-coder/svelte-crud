<script>
  import { onMount } from "svelte";
  import { supabase } from "$lib/supabase.js";

  let items = [];
  let name = "";
  let price = "";

  let editId = null;
  let editName = "";
  let editPrice = "";

  // -------------------------------
  // Fetch all items
  // -------------------------------
  async function loadItems() {
    const { data, error } = await supabase.from("items").select("*").order("id");
    if (error) console.error(error);
    else items = data;
  }

  // -------------------------------
  // Create new item
  // -------------------------------
  async function addItem() {
    if (!name || !price) return alert("Fill all fields");
    const { data, error } = await supabase.from("items").insert([{ name, price }]);
    if (error) console.error(error);
    else {
      items = [...items, ...data];
      name = "";
      price = "";
    }
  }

  // -------------------------------
  // Delete item
  // -------------------------------
  async function deleteItem(id) {
    const { error } = await supabase.from("items").delete().eq("id", id);
    if (error) console.error(error);
    else items = items.filter((i) => i.id !== id);
  }

  // -------------------------------
  // Start editing
  // -------------------------------
  function startEdit(item) {
    editId = item.id;
    editName = item.name;
    editPrice = item.price;
  }

  // -------------------------------
  // Update item
  // -------------------------------
  async function updateItem() {
    const { data, error } = await supabase
      .from("items")
      .update({ name: editName, price: editPrice })
      .eq("id", editId);
    if (error) console.error(error);
    else {
      items = items.map((i) => (i.id === editId ? data[0] : i));
      editId = null;
      editName = "";
      editPrice = "";
    }
  }

  onMount(() => {
    loadItems();
  });
</script>

<style>
  body { font-family: system-ui, sans-serif; padding: 2rem; }
  input { margin: 0.2rem; padding: 0.4rem; }
  button { margin: 0.2rem; padding: 0.4rem 0.8rem; }
  li { margin: 0.5rem 0; }
</style>

<h1>🦏 Inventory CRUD</h1>

<!-- Create New Item -->
<div>
  <input type="text" placeholder="Name" bind:value={name} />
  <input type="number" placeholder="Price" bind:value={price} />
  <button on:click={addItem}>Add Item</button>
</div>

<hr />

<!-- Items List -->
<ul>
  {#each items as item}
    <li>
      {#if editId === item.id}
        <input type="text" bind:value={editName} />
        <input type="number" bind:value={editPrice} />
        <button on:click={updateItem}>Save</button>
        <button on:click={() => (editId = null)}>Cancel</button>
      {:else}
        <strong>{item.name}</strong> - ${item.price}
        <button on:click={() => startEdit(item)}>Edit</button>
        <button on:click={() => deleteItem(item.id)}>Delete</button>
      {/if}
    </li>
  {/each}
</ul>