<script lang="ts">
    import { onMount } from 'svelte';

    interface Buddy {
        id: number;
        name: string;
        age: string;
    }

    let buddies: Buddy[] = [];
    let isLoading = true;
    let error: string | null = null;
    let newName: string = '';
    let newAge: string = '';
    let editingBuddy: Buddy | null = null;
    let editName: string = '';
    let editAge: string = '';

    const API_BASE_URL = "https://flasksvelte.pythonanywhere.com";

async function fetchBuddies(sortBy: string = '', order: string = '') {
    isLoading = true;
    error = null;
    try {
        let url = `${API_BASE_URL}/readJSON`;
        if (sortBy && order) {
            url += `?sort_by=${sortBy}&order=${order}`;
        }
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP status ${response.status}`);
        const data: any[] = await response.json();
        buddies = data.map(b => ({ ...b, id: Number(b.id) }));
    } catch (e: any) {
        error = `Error fetching data: ${e.message}`;
    } finally {
        isLoading = false;
    }
}    

    async function addBuddy() {
        if (!newName || !newAge) return;
        try {
            const response = await fetch(`${API_BASE_URL}/writeJSON`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ name: newName, age: newAge }),
            });
            if (!response.ok) throw new Error(`HTTP status ${response.status}`);
            newName = '';
            newAge = '';
            await fetchBuddies();
        } catch (e: any) {
            error = `Error adding buddy: ${e.message}`;
        }
    }

    async function deleteBuddy(id: number) {
        if (!confirm(`Are you sure you want to delete buddy ID ${id}?`)) return;
        try {
            const response = await fetch(`${API_BASE_URL}/deleteJSON/${id}`, { method: 'DELETE' });
            if (!response.ok) throw new Error(`HTTP status ${response.status}`);
            await fetchBuddies();
        } catch (e: any) {
            error = `Error deleting buddy: ${e.message}`;
        }
    }

    function startEdit(buddy: Buddy) {
        editingBuddy = buddy;
        editName = buddy.name;
        editAge = buddy.age;
    }

    async function saveEdit() {
        if (!editingBuddy) return;
        try {
            const response = await fetch(`${API_BASE_URL}/editJSON/${editingBuddy.id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ name: editName, age: editAge }),
            });
            if (!response.ok) throw new Error(`HTTP status ${response.status}`);
            editingBuddy = null;
            await fetchBuddies();
        } catch (e: any) {
            error = `Error updating buddy: ${e.message}`;
        }
    }

    function cancelEdit() {
        editingBuddy = null;
    }

    onMount(fetchBuddies);

    const getAgeColor = (age: string) => {
        const numAge = parseInt(age);
        if (numAge < 20) return 'color-young';
        if (numAge < 40) return 'color-middle';
        return 'color-old';
    }
</script>

<div class="sort-buttons">
    <button on:click={() => fetchBuddies('name', 'asc')}>Name ↑</button>
    <button on:click={() => fetchBuddies('name', 'desc')}>Name ↓</button>
    <button on:click={() => fetchBuddies('age', 'asc')}>Age ↑</button>
    <button on:click={() => fetchBuddies('age', 'desc')}>Age ↓</button>
</div>

<div class="container">
    <h1>💾 Buddy CRUD</h1>

    <div class="add-form">
        <input type="text" placeholder="Name" bind:value={newName} />
        <input type="number" placeholder="Age" bind:value={newAge} />
        <button on:click={addBuddy} disabled={!newName || !newAge}>Add</button>
    </div>

    {#if error}
        <div class="error">{error}</div>
    {/if}

    {#if isLoading}
        <div class="loading">Loading buddies…</div>
    {:else if buddies.length === 0}
        <div class="empty">No buddies found.</div>
    {:else}
        <div class="buddy-list">
            {#each buddies as buddy, index (buddy.id)}
                <div class="buddy-card">
                    <div class="buddy-info">
                        <div class="serial">{index + 1}.</div>
                        {#if editingBuddy && editingBuddy.id === buddy.id}
                            <input type="text" bind:value={editName} class="edit-input" />
                            <input type="number" bind:value={editAge} class="edit-input" />
                        {:else}
                            <div class="name">{buddy.name}</div>
                            <div class="age-badge {getAgeColor(buddy.age)}">{buddy.age}</div>
                        {/if}
                    </div>
                    <div class="buddy-actions">
                        {#if editingBuddy && editingBuddy.id === buddy.id}
                            <button class="save" on:click={saveEdit}>Save</button>
                            <button class="cancel" on:click={cancelEdit}>Cancel</button>
                        {:else}
                            <button class="edit" on:click={() => startEdit(buddy)}>Edit</button>
                            <button class="delete" on:click={() => deleteBuddy(buddy.id)}>Delete</button>
                        {/if}
                    </div>
                </div>
            {/each}
        </div>
    {/if}
</div>

<style>
    :global(body) {
        margin: 0;
        font-family: 'Inter', sans-serif;
        background: linear-gradient(135deg, #3498db, #9b59b6);
        min-height: 100vh;
        display: flex;
        justify-content: center;
        padding: 20px;
    }

    .container {
        background: #fff;
        border-radius: 12px;
        padding: 20px;
        width: 100%;
        max-width: 500px;
        box-shadow: 0 10px 25px rgba(0,0,0,0.15);
    }

    h1 {
        text-align: center;
        color: #2c3e50;
        margin-bottom: 20px;
    }

    .add-form {
        display: flex;
        flex-direction: column;
        gap: 10px;
        margin-bottom: 20px;
    }

    .add-form input {
        padding: 10px;
        border-radius: 8px;
        border: 1px solid #ccc;
        font-size: 1rem;
    }

    .add-form button {
        padding: 10px;
        border-radius: 8px;
        border: none;
        background-color: #2ecc71;
        color: white;
        font-weight: bold;
        cursor: pointer;
        transition: 0.2s;
    }

    .add-form button:hover:enabled {
        background-color: #27ae60;
    }

    .loading, .empty, .error {
        text-align: center;
        margin: 20px 0;
        font-weight: 600;
    }

    .error {
        color: #e74c3c;
    }

    .buddy-list {
        display: flex;
        flex-direction: column;
        gap: 15px;
    }

    .buddy-card {
        background: #fefefe;
        border-radius: 10px;
        padding: 15px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        box-shadow: 0 5px 15px rgba(0,0,0,0.05);
        transition: transform 0.2s;
    }

    .buddy-card:hover {
        transform: scale(1.02);
    }

    .buddy-info {
        display: flex;
        align-items: center;
        gap: 10px;
        flex-wrap: wrap;
    }

    .serial {
        font-weight: bold;
        color: #34495e;
    }

    .name {
        font-weight: 600;
        font-size: 1rem;
    }

    .age-badge {
        padding: 4px 10px;
        border-radius: 12px;
        font-size: 0.85rem;
        font-weight: 700;
        text-align: center;
        min-width: 40px;
    }

    .color-young { background-color: #e6f7ff; color: #1890ff; }
    .color-middle { background-color: #f6ffed; color: #52c41a; }
    .color-old { background-color: #fff1f0; color: #f5222d; }

    .buddy-actions {
        display: flex;
        gap: 8px;
        flex-wrap: wrap;
    }

    .buddy-actions button {
        padding: 6px 12px;
        border-radius: 8px;
        border: none;
        font-size: 0.85rem;
        font-weight: bold;
        cursor: pointer;
        transition: 0.2s;
    }

    .edit { background-color: #3498db; color: white; }
    .edit:hover { background-color: #2980b9; }
    .delete { background-color: #e74c3c; color: white; }
    .delete:hover { background-color: #c0392b; }
    .save { background-color: #2ecc71; color: white; }
    .save:hover { background-color: #27ae60; }
    .cancel { background-color: #95a5a6; color: white; }
    .cancel:hover { background-color: #7f8c8d; }

    .edit-input {
        padding: 6px;
        border-radius: 8px;
        border: 1px solid #ccc;
        font-size: 0.9rem;
    }

    @media (max-width: 500px) {
        .buddy-card {
            flex-direction: column;
            align-items: flex-start;
            gap: 10px;
        }

        .buddy-actions {
            width: 100%;
            justify-content: space-around;
        }
    }
</style>