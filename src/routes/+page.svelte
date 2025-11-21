<script>
 import { onMount } from "svelte";
 // 🛑 NO LONGER NEEDED: import { supabase } from "$lib/supabase.js";
 
 // Define the API URL for your server
 const API_URL = "http://localhost:5000/items";

 let items = [];
 let name = "";
 let price = "";

 let editId = null;
 let editName = "";
 let editPrice = "";

 // -------------------------------
 // 1. Fetch all items (READ)
 // -------------------------------
 async function loadItems() {
  try {
   const response = await fetch(API_URL);
   if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
   
   // The API returns the array of items directly
   items = await response.json();
  } catch (error) {
   console.error("Could not fetch items:", error);
  }
 }

 // -------------------------------
 // 2. Create new item (CREATE)
 // -------------------------------
 async function addItem() {
  if (!name || !price) return alert("Fill all fields, ya hear?");
  
  try {
   const response = await fetch(API_URL, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name, price: Number(price) }), // Convert price to number if needed
   });

   if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);

   // The API returns the array of newly created items
   const newItems = await response.json();
   
   // Update local state: add the new item(s) to the list
   items = [...items, ...newItems];
   name = "";
   price = "";
  } catch (error) {
   console.error("Could not add item:", error);
  }
 }

 // -------------------------------
 // 3. Delete item (DELETE)
 // -------------------------------
 async function deleteItem(id) {
  try {
   const response = await fetch(`${API_URL}/${id}`, {
    method: "DELETE",
   });

   if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);

   // Update local state: remove the item from the list
   items = items.filter((i) => i.id !== id);
  } catch (error) {
   console.error("Could not delete item:", error);
  }
 }

 // -------------------------------
 // 4. Start editing (Helper function)
 // -------------------------------
 function startEdit(item) {
  editId = item.id;
  editName = item.name;
  editPrice = item.price;
 }

 // -------------------------------
 // 5. Update item (UPDATE)
 // -------------------------------
 async function updateItem() {
  try {
   const response = await fetch(`${API_URL}/${editId}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name: editName, price: Number(editPrice) }),
   });

   if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);

   // The API returns the array of updated items (usually just one)
   const updatedRows = await response.json();
   const updatedItem = updatedRows[0];
   
   // Update local state: replace the old item with the updated one
   items = items.map((i) => (i.id === editId ? updatedItem : i));
   
   // Reset the edit state
   editId = null;
   editName = "";
   editPrice = "";
  } catch (error) {
   console.error("Could not update item:", error);
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

<div>
 <input type="text" placeholder="Name" bind:value={name} />
 <input type="number" placeholder="Price" bind:value={price} />
 <button on:click={addItem}>Add Item</button>
</div>

<hr />

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