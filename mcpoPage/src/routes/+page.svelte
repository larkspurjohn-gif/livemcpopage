<script>
    // Svelte 5 Runes for state management
    let query = $state("");
    let results = $state(null);
    let isLoading = $state(false);
    let error = $state(null);

    async function handleSearch() {
        if (!query) return;
        isLoading = true;
        error = null;
        
        try {
            const response = await fetch('http://localhost:9001/rag_search_ddgs', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    query: query,
                    num_results: 10,
                    top_k: 5,
                    include_urls: true
                })
            });

            if (!response.ok) throw new Error("MCP Server Error: Check if Docker is running on port 9001");

            const data = await response.json();
            // Maps the data to ensure we handle different possible key names from the MCP tool
            results = data.results || data.content || []; 
        } catch (err) {
            error = err.message;
        } finally {
            isLoading = false;
        }
    }

    // Helper to format the relevance score as a percentage
    const getScore = (item) => {
        const s = item.score ?? item.relevance ?? item.rank ?? 0;
        return Math.round(s * 100);
    };
</script>

<main class="container">
    <header>
        <h1>AI Research Assistant</h1>
    </header>
    
    <div class="search-box">
        <input 
            bind:value={query} 
            placeholder="Type question or statement here" 
            onkeydown={(e) => e.key === 'Enter' && handleSearch()}
        />
        <button onclick={handleSearch} disabled={isLoading}>
            {isLoading ? "Searching..." : "Analyze"}
        </button>
    </div>

    {#if error}
        <div class="error-banner">
            {error}
        </div>
    {/if}

    <div class="dashboard">
        {#if isLoading}
            <div class="loading-state">
                <div class="spinner"></div>
                <p>Scouring web sources and ranking relevance...</p>
            </div>
        {/if}

        {#if results && !isLoading}
            <section class="results-list">
                <h2>Top Sources & Evidence</h2>
                {#each results as item}
                    <div class="card">
                        <div class="card-meta">
                            <span class="badge" class:high={getScore(item) > 70}>
    {#if getScore(item) > 0}
        {getScore(item)}% Match
    {:else}
        Result Found
    {/if}
</span>
                            {#if item.url}
                                <span class="hostname">{new URL(item.url).hostname}</span>
                            {/if}
                        </div>
                        
                        <p class="content">{item.content || item.text || "No snippet available."}</p>
                        
                        {#if item.url}
                            <a href={item.url} target="_blank" class="source-link">
                                View Source →
                            </a>
                        {/if}
                    </div>
                {/each}
            </section>
        {/if}
    </div>
</main>

<style>
    :global(body) { background-color: #f8fafc; color: #1e293b; margin: 0; }
    .container { max-width: 900px; margin: 3rem auto; padding: 0 1.5rem; font-family: 'Inter', system-ui, sans-serif; }
    
    header { margin-bottom: 2rem; text-align: center; }
    h1 { margin: 0; font-size: 2.25rem; font-weight: 800; color: #0f172a; }

    .search-box { display: flex; gap: 12px; background: white; padding: 8px; border-radius: 16px; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1); border: 1px solid #e2e8f0; }
    input { flex: 1; padding: 0.75rem 1rem; border: none; font-size: 1rem; outline: none; }
    button { padding: 0.75rem 2rem; background: #2563eb; color: white; border: none; border-radius: 12px; font-weight: 600; cursor: pointer; transition: all 0.2s; }
    button:hover { background: #1d4ed8; }
    button:disabled { background: #94a3b8; cursor: not-allowed; }

    .dashboard { margin-top: 3rem; }
    
    .card { background: white; padding: 1.5rem; margin-bottom: 1.25rem; border-radius: 16px; border: 1px solid #e2e8f0; transition: transform 0.2s; }
    .card:hover { transform: translateY(-2px); border-color: #cbd5e1; }
    
    .card-meta { display: flex; align-items: center; gap: 12px; margin-bottom: 1rem; }
    .badge { padding: 4px 10px; border-radius: 6px; font-size: 0.75rem; font-weight: 700; background: #f1f5f9; color: #475569; }
    .badge.high { background: #dcfce7; color: #166534; }
    .hostname { font-size: 0.85rem; color: #94a3b8; font-weight: 500; }

    .content { line-height: 1.6; color: #334155; margin: 0; font-size: 1rem; }
    .source-link { display: inline-block; margin-top: 1.25rem; color: #2563eb; text-decoration: none; font-size: 0.875rem; font-weight: 600; }
    
    .error-banner { background: #fef2f2; color: #991b1b; padding: 1rem; border-radius: 12px; border: 1px solid #fee2e2; margin-bottom: 2rem; }
    
    .loading-state { text-align: center; padding: 3rem; color: #64748b; }
    .spinner { width: 30px; height: 30px; border: 3px solid #e2e8f0; border-top-color: #2563eb; border-radius: 50%; animation: spin 1s linear infinite; margin: 0 auto 1rem; }
    @keyframes spin { to { transform: rotate(360deg); } }
</style>