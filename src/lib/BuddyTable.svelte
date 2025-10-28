<script lang="ts">
    import { onMount } from 'svelte';

    // 1. Define the TypeScript interface (matches your Flask response)
    interface Buddy {
        id: number; // IMPORTANT: Required for Delete/Update operations
        name: string;
        age: string; // Keep as string for now, as Flask stores it as TEXT
    }

    // 2. Reactive state variables
    let buddies: Buddy[] = [];
    let isLoading = true;
    let error: string | null = null;
    let newName: string = ''; // For adding new buddy
    let newAge: string = '';  // For adding new buddy
    let editingBuddy: Buddy | null = null; // Buddy currently being edited
    let editName: string = '';
    let editAge: string = '';

    // NOTE: Replace this with your actual server URL (e.g., your PythonAnywhere URL)
    const API_BASE_URL = "https://flasksvelte.pythonanywhere.com"; // Use your local host for testing

    // --- Core Data Fetching (Read) ---
    async function fetchBuddies() {
        isLoading = true;
        error = null;
        try {
            const response = await fetch(`${API_BASE_URL}/readJSON`);
            if (!response.ok) {
                throw new Error(`HTTP status ${response.status}`);
            }
            // Ensure ID is parsed as number for safety, age remains string
            const data: any[] = await response.json();
            buddies = data.map(b => ({ ...b, id: Number(b.id) }));
        } catch (e: any) {
            error = `Error fetching data: ${e.message}`;
        } finally {
            isLoading = false;
        }
    }

    // --- Create New Buddy ---
    async function addBuddy() {
        if (!newName || !newAge) return;

        try {
            const response = await fetch(`${API_BASE_URL}/writeJSON`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ name: newName, age: newAge }),
            });

            if (!response.ok) {
                 throw new Error(`HTTP status ${response.status}`);
            }
            
            // Clear inputs and refresh the list
            newName = '';
            newAge = '';
            await fetchBuddies();
        } catch (e: any) {
            error = `Error adding buddy: ${e.message}`;
        }
    }

    // --- Delete Buddy ---
    async function deleteBuddy(id: number) {
        if (!confirm(`Are you sure you want to delete buddy ID ${id}?`)) return;

        try {
            const response = await fetch(`${API_BASE_URL}/deleteJSON/${id}`, {
                method: 'DELETE',
            });

            if (!response.ok) {
                 throw new Error(`HTTP status ${response.status}`);
            }
            
            // Refresh the list
            await fetchBuddies();
        } catch (e: any) {
            error = `Error deleting buddy: ${e.message}`;
        }
    }

    // --- Update Buddy: Start Editing ---
    function startEdit(buddy: Buddy) {
        editingBuddy = buddy;
        editName = buddy.name;
        editAge = buddy.age;
    }

    // --- Update Buddy: Save Changes ---
    async function saveEdit() {
        if (!editingBuddy) return;

        try {
            const response = await fetch(`${API_BASE_URL}/editJSON/${editingBuddy.id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ name: editName, age: editAge }),
            });

            if (!response.ok) {
                 throw new Error(`HTTP status ${response.status}`);
            }

            // Exit edit mode and refresh the list
            editingBuddy = null;
            await fetchBuddies();
        } catch (e: any) {
            error = `Error updating buddy: ${e.message}`;
        }
    }

    function cancelEdit() {
        editingBuddy = null;
    }

    // Run the initial data fetch when the component mounts
    onMount(fetchBuddies);

    // Simple helper for dynamic age color (reused from previous answer)
    const getAgeColor = (age: string) => {
        const numAge = parseInt(age);
        if (numAge < 20) return 'color-young';
        if (numAge < 40) return 'color-middle';
        return 'color-old';
    }
</script>

<div class="data-container">
    <div class="header-band"></div>
    <div class="card-content">
        <h2 class="title">
            <span role="img" aria-label="Database">💾</span> Buddy CRUD Interface
        </h2>

        {#if error}
            <div class="error-state">
                <span class="icon">⚠️</span>
                <h3>Request Failed</h3>
                <p>{error}</p>
                <p class="error-tip">Please check your console and ensure the Flask server is running at <strong>{API_BASE_URL}</strong>.</p>
            </div>
        {/if}

        <div class="add-form-container">
            <input type="text" placeholder="Name" bind:value={newName} />
            <input type="number" placeholder="Age" bind:value={newAge} />
            <button on:click={addBuddy} disabled={!newName || !newAge}>Add New Buddy</button>
        </div>

        {#if isLoading}
            <div class="loading-state">
                <div class="spinner"></div>
                <p>Loading buddies... hang tight!</p>
            </div>

        {:else if buddies.length > 0}
            <div class="buddy-grid">
                <div class="grid-header id-header">ID</div>
                <div class="grid-header name-header">Name</div>
                <div class="grid-header age-header">Age</div>
                <div class="grid-header action-header">Actions</div>

                {#each buddies as buddy (buddy.id)}
                    
                    {#if editingBuddy && editingBuddy.id === buddy.id}
                        <div class="grid-cell">{buddy.id}</div>
                        <div class="grid-cell"><input type="text" bind:value={editName} /></div>
                        <div class="grid-cell"><input type="number" bind:value={editAge} /></div>
                        <div class="grid-cell action-cell edit-actions">
                            <button class="save-btn" on:click={saveEdit}>Save</button>
                            <button class="cancel-btn" on:click={cancelEdit}>Cancel</button>
                        </div>
                    {:else}
                        <div class="grid-cell id-cell">{buddy.id}</div>
                        <div class="grid-cell name-cell">{buddy.name}</div>
                        <div class="grid-cell age-cell">
                            <span class="age-badge {getAgeColor(buddy.age)}">{buddy.age}</span>
                        </div>
                        <div class="grid-cell action-cell">
                            <button class="edit-btn" on:click={() => startEdit(buddy)}>Edit</button>
                            <button class="delete-btn" on:click={() => deleteBuddy(buddy.id)}>Delete</button>
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
    </div>
</div>

<style>
    /* Add form styling */
    .add-form-container {
        display: flex;
        gap: 10px;
        margin-bottom: 30px;
        padding: 15px;
        border: 1px solid #e0e0e0;
        border-radius: 8px;
        background-color: #f9f9f9;
    }
    .add-form-container input {
        flex-grow: 1;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 4px;
    }
    .add-form-container button {
        background-color: #2ecc71;
        color: white;
        border: none;
        padding: 10px 15px;
        border-radius: 4px;
        cursor: pointer;
        transition: background-color 0.2s;
    }
    .add-form-container button:hover:enabled {
        background-color: #27ae60;
    }
    .add-form-container button:disabled {
        background-color: #bdc3c7;
        cursor: not-allowed;
    }

    /* Grid layout updated for actions and ID */
    .buddy-grid {
        display: grid;
        grid-template-columns: 0.5fr 2fr 0.8fr 1.5fr; /* ID | Name | Age | Actions */
        border: 1px solid #e5e5e5;
        border-radius: 8px;
        overflow: hidden;
    }

    .grid-header {
        text-transform: uppercase;
        font-size: 0.9rem;
        padding: 12px 10px;
        text-align: center;
        /* Inherited styles for background, color, borders */
        background-color: #f7f9fc;
        color: #34495e;
        font-weight: 600;
        border-bottom: 1px solid #e5e5e5;
        border-right: 1px solid #e5e5e5;
    }
    .grid-header.action-header {
        border-right: none;
    }
    .grid-cell {
        padding: 10px 10px;
        border-bottom: 1px solid #f0f0f0;
        background-color: #ffffff;
        text-align: center;
    }
    .grid-cell:nth-child(4n + 4) {
        border-right: none;
    }
    .grid-cell.name-cell {
        text-align: left;
    }
    
    /* Action Buttons */
    .action-cell {
        display: flex;
        justify-content: center;
        gap: 5px;
    }
    .edit-actions input {
        width: 90%;
        padding: 5px;
    }
    .edit-btn, .delete-btn, .save-btn, .cancel-btn {
        border: none;
        padding: 6px 10px;
        border-radius: 4px;
        cursor: pointer;
        font-size: 0.85rem;
    }

    .edit-btn {
        background-color: #3498db;
        color: white;
    }
    .edit-btn:hover {
        background-color: #2980b9;
    }
    .delete-btn {
        background-color: #e74c3c;
        color: white;
    }
    .delete-btn:hover {
        background-color: #c0392b;
    }
    .save-btn {
        background-color: #2ecc71;
        color: white;
    }
    .save-btn:hover {
        background-color: #27ae60;
    }
    .cancel-btn {
        background-color: #95a5a6;
        color: white;
    }
    .cancel-btn:hover {
        background-color: #7f8c8d;
    }
    
    /* Inherited styles from previous answer (container, loading, error, etc.) */
    :global(body) {
        font-family: 'Inter', 'Segoe UI', Roboto, sans-serif;
        background: #f0f4f8;
        margin: 0;
        padding: 0;
    }
    .data-container {
        max-width: 800px; /* Slightly wider for the action column */
        margin: 60px auto;
        background: #fff;
        border-radius: 16px;
        overflow: hidden;
        box-shadow: 0 10px 30px rgba(44, 62, 80, 0.1), 0 0 0 1px rgba(0, 0, 0, 0.05);
        transition: transform 0.3s ease;
    }
    .data-container:hover {
        transform: translateY(-3px);
    }
    .header-band {
        height: 8px;
        background: linear-gradient(90deg, #3498db 0%, #2ecc71 100%); 
    }
    .card-content {
        padding: 30px;
    }
    .title {
        text-align: center;
        color: #2c3e50;
        font-size: 1.8rem;
        font-weight: 700;
        margin-bottom: 30px;
        position: relative;
    }
    .title::after {
        content: '';
        display: block;
        width: 50px;
        height: 3px;
        background: #2ecc71;
        margin: 10px auto 0;
        border-radius: 2px;
    }
    .buddy-grid > div:nth-child(8n + 5), 
    .buddy-grid > div:nth-child(8n + 6),
    .buddy-grid > div:nth-child(8n + 7),
    .buddy-grid > div:nth-child(8n + 8) {
        background-color: #fcfcfc; /* Subtle striping effect every 4 cells (one row) */
    }
    .grid-cell:hover {
        background-color: #e8f0fe;
    }
    .age-badge {
        display: inline-block;
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
    .loading-state, .error-state, .empty-state {
        text-align: center;
        padding: 40px 20px;
        border-radius: 8px;
        margin: 20px 0;
        background-color: #f7f9fc;
    }
    .loading-state p {
        color: #3498db;
        font-style: italic;
    }
    .spinner {
        border: 4px solid rgba(52, 152, 219, 0.2);
        border-top: 4px solid #3498db;
        border-radius: 50%;
        width: 30px;
        height: 30px;
        animation: spin 1s linear infinite;
        margin: 0 auto 15px;
    }
    @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
    }
    .error-state {
        background: #fff0f0;
        border: 1px solid #ffccc7;
    }
    .error-state .icon {
        font-size: 2.5rem;
        display: block;
        margin-bottom: 10px;
    }
    .error-state h3 {
        color: #e74c3c;
        margin: 0 0 5px 0;
    }
    .error-tip {
        font-size: 0.85rem;
        color: #7f8c8d;
        margin-top: 10px;
    }
    .empty-state .icon {
        font-size: 2.5rem;
        color: #bdc3c7;
        display: block;
        margin-bottom: 10px;
    }
</style>