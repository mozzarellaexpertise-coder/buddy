<script lang="ts">
  import { onMount } from 'svelte';

  interface Buddy {
    id: number;
    name: string;
    age: number;
  }

  let buddies: Buddy[] = [];
  let isLoading = true;
  let error: string | null = null;

  const API_URL = "https://flasksvelte.pythonanywhere.com";

  // New buddy form state
  let newName = '';
  let newAge: number | '' = '';

  // Editing state
  let editingId: number | null = null;
  let editName = '';
  let editAge: number | '' = '';

  // Fetch all buddies
  async function fetchBuddies() {
    isLoading = true;
    try {
      const res = await fetch(`${API_URL}/readJSON`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      buddies = await res.json();
    } catch (e: any) {
      error = e.message || "Failed to load buddies";
    } finally {
      isLoading = false;
    }
  }

  onMount(fetchBuddies);

  // Helper: color badge
  const getAgeColor = (age: number) => {
    if (age < 20) return 'color-young';
    if (age < 40) return 'color-middle';
    return 'color-old';
  }

  // CREATE
  async function addBuddy() {
    if (!newName || !newAge) return;
    try {
      const res = await fetch(`${API_URL}/writeJSON`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: newName, age: newAge })
      });
      const data = await res.json();
      if (res.ok) {
        buddies = [...buddies, data.received];
        newName = '';
        newAge = '';
      } else {
        alert(data.error);
      }
    } catch (e: any) {
      alert(e.message);
    }
  }

  // DELETE
  async function deleteBuddy(id: number) {
    if (!confirm('Delete this buddy?')) return;
    try {
      const res = await fetch(`${API_URL}/deleteJSON/${id}`, { method: 'DELETE' });
      if (res.ok) {
        buddies = buddies.filter(b => b.id !== id);
      } else {
        const data = await res.json();
        alert(data.error);
      }
    } catch (e: any) {
      alert(e.message);
    }
  }

  // EDIT
  function startEdit(buddy: Buddy) {
    editingId = buddy.id;
    editName = buddy.name;
    editAge = buddy.age;
  }

  async function saveEdit(id: number) {
    if (!editName || !editAge) return;
    try {
      const res = await fetch(`${API_URL}/editJSON/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: editName, age: editAge })
      });
      const data = await res.json();
      if (res.ok) {
        buddies = buddies.map(b => b.id === id ? { id, name: editName, age: editAge } : b);
        editingId = null;
      } else {
        alert(data.error);
      }
    } catch (e: any) {
      alert(e.message);
    }
  }

  function cancelEdit() {
    editingId = null;
  }

</script>

<div class="data-container">
  <div class="header-band"></div>
  <div class="card-content">
    <h2 class="title">👥 Buddy Records</h2>

    {#if isLoading}
      <div class="loading-state">
        <div class="spinner"></div>
        <p>Loading buddies... hang tight!</p>
      </div>
    {:else if error}
      <div class="error-state">
        <span class="icon">⚠️</span>
        <h3>Connection Error</h3>
        <p>{error}</p>
      </div>
    {:else}
      <!-- ADD NEW BUDDY -->
      <div class="add-buddy">
        <input type="text" placeholder="Name" bind:value={newName} />
        <input type="number" placeholder="Age" bind:value={newAge} />
        <button on:click={addBuddy}>Add Buddy</button>
      </div>

      <!-- LIST BUDDIES -->
      {#if buddies.length > 0}
        <div class="buddy-grid">
          <div class="grid-header">Name</div>
          <div class="grid-header age-header">Age</div>
          <div class="grid-header">Actions</div>

          {#each buddies as buddy (buddy.id)}
            {#if editingId === buddy.id}
              <div class="grid-cell">
                <input type="text" bind:value={editName} />
              </div>
              <div class="grid-cell">
                <input type="number" bind:value={editAge} />
              </div>
              <div class="grid-cell">
                <button on:click={() => saveEdit(buddy.id)}>💾 Save</button>
                <button on:click={cancelEdit}>❌ Cancel</button>
              </div>
            {:else}
              <div class="grid-cell name-cell">{buddy.name}</div>
              <div class="grid-cell age-cell">
                <span class="age-badge {getAgeColor(buddy.age)}">{buddy.age}</span>
              </div>
              <div class="grid-cell">
                <button on:click={() => startEdit(buddy)}>✏️ Edit</button>
                <button on:click={() => deleteBuddy(buddy.id)}>🗑️ Delete</button>
              </div>
            {/if}
          {/each}
        </div>
      {:else}
        <div class="empty-state">
          <span class="icon">😶</span>
          <p>No buddy records found!</p>
        </div>
      {/if}
    {/if}
  </div>
</div>

<style>
  /* Keep your polished styles: same as before */
  /* Add small tweaks for Add/Edit buttons */
  .add-buddy {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
  }
  .add-buddy input {
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #ccc;
    flex: 1;
  }
  .add-buddy button {
    padding: 8px 16px;
    border: none;
    border-radius: 6px;
    background: #2ecc71;
    color: white;
    cursor: pointer;
    font-weight: 600;
  }
  .add-buddy button:hover {
    background: #27ae60;
  }
  .buddy-grid {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr;
    border: 1px solid #e5e5e5;
    border-radius: 8px;
    overflow: hidden;
  }
  .grid-header {
    padding: 12px 18px;
    background-color: #f7f9fc;
    font-weight: 600;
    border-bottom: 1px solid #e5e5e5;
    text-transform: uppercase;
  }
  .grid-cell {
    padding: 12px 18px;
    border-bottom: 1px solid #f0f0f0;
  }
  button {
    margin-right: 5px;
    padding: 4px 8px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
  }
  button:hover {
    opacity: 0.85;
  }
</style>