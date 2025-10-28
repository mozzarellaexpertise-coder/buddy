<script lang="ts">
    import { onMount } from 'svelte';

    // 1. Define the TypeScript interface for type safety
    interface Buddy {
        name: string;
        age: number; // Assuming the server returns age as a number
    }

    // 2. Reactive state variables
    let buddies: Buddy[] = [];
    let isLoading = true;
    let error: string | null = null;

    // NOTE: Replace this with your actual server endpoint URL
    const API_URL = "https://flasksvelte.pythonanywhere.com/readJSON";

    // 3. Data Fetching Logic using onMount
    onMount(async () => {
        try {
            const response = await fetch(API_URL);

            // Check for HTTP errors (e.g., 404, 500)
            if (!response.ok) {
                throw new Error(`Failed to fetch data: HTTP status ${response.status}`);
            }

            // Parse the JSON response
            const data: Buddy[] = await response.json();
            
            // Update the state reactively
            buddies = data; 
        } catch (e: any) {
            // Catch network or JSON parsing errors
            error = e.message || "An unknown error occurred while fetching data.";
        } finally {
            // Ensure loading state is always updated
            isLoading = false;
        }
    });

    // Helper function to dynamically color the age display
    const getAgeColor = (age: number) => {
        if (age < 20) return 'color-young';
        if (age < 40) return 'color-middle';
        return 'color-old';
    }
</script>

<div class="data-container">
    <div class="header-band"></div>
    <div class="card-content">
        <h2 class="title">
            <span role="img" aria-label="People">👥</span> Buddy Records
        </h2>

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
                <p class="error-tip">Please check if the server at <strong>{API_URL}</strong> is accessible.</p>
            </div>

        {:else if buddies.length > 0}
            <div class="buddy-grid">
                <div class="grid-header">Name</div>
                <div class="grid-header age-header">Age</div>

                {#each buddies as buddy (buddy.name)}
                    <div class="grid-cell name-cell">{buddy.name}</div>
                    <div class="grid-cell age-cell">
                        <span class="age-badge {getAgeColor(buddy.age)}">{buddy.age}</span>
                    </div>
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
    /* Global Styling for the page */
    :global(body) {
        font-family: 'Inter', 'Segoe UI', Roboto, sans-serif;
        background: #f0f4f8; /* Very light, cool gray background */
        margin: 0;
        padding: 0;
    }

    /* Container Card Style */
    .data-container {
        max-width: 700px; /* Slightly wider */
        margin: 60px auto;
        background: #fff;
        border-radius: 16px; /* Smoother corners */
        overflow: hidden;
        /* Modern, deep shadow for a "floating" look */
        box-shadow: 0 10px 30px rgba(44, 62, 80, 0.1), 0 0 0 1px rgba(0, 0, 0, 0.05);
        transition: transform 0.3s ease;
    }

    .data-container:hover {
        transform: translateY(-3px); /* Subtle lift on hover */
    }

    /* Decorative Header Band */
    .header-band {
        height: 8px;
        /* Attractive gradient effect */
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

    /* --- Data Grid Styles (Replacing the Table) --- */
    .buddy-grid {
        display: grid;
        grid-template-columns: 3fr 1fr; /* Name is wider than Age */
        border: 1px solid #e5e5e5;
        border-radius: 8px;
        overflow: hidden;
    }

    .grid-header {
        padding: 12px 18px;
        background-color: #f7f9fc;
        color: #34495e;
        font-weight: 600;
        border-bottom: 1px solid #e5e5e5;
        border-right: 1px solid #e5e5e5;
        text-transform: uppercase;
        font-size: 0.9rem;
    }
    
    .grid-header.age-header {
        border-right: none;
    }

    .grid-cell {
        padding: 14px 18px;
        border-bottom: 1px solid #f0f0f0;
        background-color: #ffffff;
        transition: background-color 0.2s ease;
    }

    .buddy-grid > div:nth-child(4n + 3), 
    .buddy-grid > div:nth-child(4n + 4) {
        background-color: #fcfcfc; /* Subtle striping effect */
    }

    .grid-cell:hover {
        background-color: #e8f0fe; /* Light blue hover */
    }

    .grid-cell.name-cell {
        font-weight: 500;
        color: #333;
        border-right: 1px solid #f0f0f0;
    }

    /* --- Age Badge/Pill for better presentation --- */
    .age-badge {
        display: inline-block;
        padding: 4px 10px;
        border-radius: 12px;
        font-size: 0.85rem;
        font-weight: 700;
        text-align: center;
        min-width: 40px;
    }

    .color-young {
        background-color: #e6f7ff;
        color: #1890ff; /* Blue */
    }
    .color-middle {
        background-color: #f6ffed;
        color: #52c41a; /* Green */
    }
    .color-old {
        background-color: #fff1f0;
        color: #f5222d; /* Red/Maroon */
    }

    /* --- Loading and Error States --- */
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

    /* Spinner Animation */
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