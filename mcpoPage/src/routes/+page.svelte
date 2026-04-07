<script>
    // Svelte 5 Runes for state management
    let query = $state("");
    let results = $state(null);
    let isLoading = $state(false);
    let error = $state(null);

    async function handleSearch() {
    isLoading = true;
    try {
        const response = await fetch('http://localhost:9001/rag_search_ddgs', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                query: query,        // Your search term from the input
                num_results: 10,     // Total sites to check
                top_k: 5,            // Best results to return
                include_urls: true   // Keep the links for UI citations
            })
        });

        if (!response.ok) throw new Error("Check Docker logs - Request failed");

        const data = await response.json();
        // Depending on the tool, the results are usually in data.results or data.content
        results = data.results || data.content; 
    } catch (err) {
        error = err.message;
    } finally {
        isLoading = false;
    }
}
</script>

<main class="container">
    <h1>AI Research Assistant</h1>
    
    <div class="search-box">
        <input 
            bind:value={query} 
            placeholder="Ask a technical or research question..." 
            onkeydown={(e) => e.key === 'Enter' && handleSearch()}
        />
        <button onclick={handleSearch} disabled={isLoading}>
            {isLoading ? "Searching..." : "Analyze"}
        </button>
    </div>

    {#if error}
        <p class="error">{error}</p>
    {/if}

    <div class="dashboard">
    {#if results}
        <section class="results-list">
            <h2>Top Sources & Evidence</h2>
            {#each results as item}
                <div class="card">
                    <p>{item.content || item.text || "No content found"}</p>
                    
                    {#if item.url}
                        <a href={item.url} target="_blank" class="source-link">
                            Source: {new URL(item.url).hostname}
                        </a>
                    {/if}
                </div>
            {/each}
        </section>
    {/if}
</div>
</main>

<style>
    .container { max-width: 800px; margin: 2rem auto; padding: 1rem; font-family: sans-serif; }
    .search-box { display: flex; gap: 10px; margin-bottom: 2rem; }
    input { flex: 1; padding: 0.8rem; border: 1px solid #ccc; border-radius: 8px; }
    button { padding: 0.8rem 1.5rem; background: #007bff; color: white; border: none; border-radius: 8px; cursor: pointer; }
    button:disabled { background: #ccc; }
    .card { padding: 1rem; border: 1px solid #eee; border-radius: 8px; margin-bottom: 1rem; background: #f9f9f9; }
    .error { color: red; font-weight: bold; }
    .card {
    background: #fdfdfd;
    border: 1px solid #e0e0e0;
    padding: 1.5rem;
    margin-bottom: 1rem;
    border-radius: 12px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.02);
}

.source-link {
    display: inline-block;
    margin-top: 10px;
    font-size: 0.8rem;
    color: #007bff;
    text-decoration: none;
    font-weight: 500;
}

.source-link:hover {
    text-decoration: underline;
}
</style>