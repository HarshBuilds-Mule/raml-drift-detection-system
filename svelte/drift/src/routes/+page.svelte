<script>
  // Imports for Svelte's optimized animation system
  import { fade, scale } from 'svelte/transition';
  import { flip } from 'svelte/animate';

  // --- Reactive Form State (Svelte 5 Runes) ---
  let groupId = $state("87ae0ee6-029e-46db-b459-5580fd38c9de");
  let assetId = $state("driftraml");
  let version = $state("1.0.1");

  // --- Reactive UI State (Svelte 5 Runes) ---
  let loading = $state(false);
  let error = $state(null);
  let results = $state([]); // Initialized as an empty array

  // --- Derived State (Performance boost: only recalculates when 'results' changes) ---
  let hasResults = $derived(results.length > 0);

  // --- API Interaction Logic ---
  async function checkDrift(event) {
    event.preventDefault(); // Standard Svelte 5 event handling

    // UI state reset
    loading = true;
    error = null;
    results = []; // Clear old results to trigger entrance animation for new ones

    try {
      const response = await fetch("https://drift-detection-system-i97py.5sc6y6-3.usa-e2.cloudhub.io/drift", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ groupId, assetId, version })
      });

      if (!response.ok) {
        throw new Error(`Error: ${response.status} - ${response.statusText}`);
      }

      const data = await response.json();

      // Handle both successful array responses and Mule string messages (like "no new version")
      if (typeof data === 'string') {
        error = data;
      } else if (Array.isArray(data)) {
        results = data;
      } else {
        error = "Unexpected response format from API.";
      }

    } catch (err) {
      error = err.message + " (Check if the Mule app is running and CORS is configured on port 8081).";
    } finally {
      loading = false;
    }
  }
</script>

<main>
  <div class="app-container">
    <header class="app-header">
      <h1><span class="mule-blue">API</span> Drift Detector</h1>
      <p>Analyze and identify critical schema drift in your Anypoint Exchange RAML specs.</p>
    </header>

    <!-- FORM SECTION: Polished layout with fast feedback states -->
    <section class="card form-section">
      <form onsubmit={checkDrift}>
        <div class="form-grid">
          <div class="form-group">
            <label for="groupId">Organization / Group ID</label>
            <input type="text" id="groupId" bind:value={groupId} placeholder="Enter Group ID" required />
          </div>

          <div class="form-group">
            <label for="assetId">API Asset ID</label>
            <input type="text" id="assetId" bind:value={assetId} placeholder="Enter Asset ID" required />
          </div>

          <div class="form-group version-group">
            <label for="version">Target Version</label>
            <input type="text" id="version" bind:value={version} placeholder="1.0.1" required />
          </div>
        </div>

        <div class="form-actions">
          <button type="submit" class="primary-btn" disabled={loading} class:is-loading={loading}>
            {#if loading}
              <span class="spinner"></span> Analyzing Spec...
            {:else}
              Analyze Spec for Drift
            {/if}
          </button>
        </div>
      </form>
    </section>

    <!-- RESULTS/INFO SECTION: Conditional display of results, loading, and error states -->
    <section class="results-section">
      {#if loading}
        <div class="info-card loading-state">
          <h2>Analyzing API Drift</h2>
          <p>Please wait while we extract and compare the RAML specs from Exchange...</p>
        </div>
      {:else if error}
        <div class="info-card error-state" transition:fade>
          <h2>Analysis Alert</h2>
          <p>{error}</p>
        </div>
      {:else if hasResults}
        <div class="results-header" transition:fade>
          <h2>Analysis Complete: <span class="drift-count">{results.length} Potential Drift(s) Detected</span></h2>
        </div>
        
        <div class="results-grid">
          <!-- Efficient Svelte Keyed Each Block for smooth list reordering and transitions -->
          {#each results as result, index (result.description)}
            <div 
              class="result-card {result.changeType.toLowerCase()}" 
              transition:scale|global="{{delay: index * 100, duration: 400, opacity: 0.5, start: 0.8}}"
              animate:flip={{duration: 400}}
            >
              <div class="result-header">
                <span class="badge {result.changeType.toLowerCase()}">
                  {result.changeType}
                </span>
                <span class="level-badge">{result.level} level</span>
              </div>
              <p class="description">{result.description}</p>
            </div>
          {/each}
        </div>
      {:else}
        <!-- Initial landing state or "No Drift" success state -->
        <div class="info-card intro-state" transition:fade>
          <h2>Specification Comparison</h2>
          <p>This tool proactively identifies modifications made to your API specification that could potentially impact existing consumer applications. Breaking changes will be highlighted in red, while new optional feature additions are shown in green.</p>
          <p>Fill in the form above and click 'Analyze Spec' to begin.</p>
        </div>
      {/if}
    </section>
  </div>
</main>

<style>
  /* --- Production-ready Base Styles and CSS Variables --- */
  :global(html) {
    box-sizing: border-box;
    font-size: 16px;
  }

  :global(*, *:before, *:after) {
    box-sizing: inherit;
  }

  :global(body) {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;
    background-color: #f7fafc;
    color: #2d3748;
    margin: 0;
    line-height: 1.6;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  /* --- Svelte component styles (component-scoped for speed) --- */
  .app-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem 1rem;
    display: flex;
    flex-direction: column;
    gap: 2rem;
  }

  .app-header {
    text-align: center;
    margin-bottom: 1rem;
  }

  .app-header h1 {
    font-size: 2.5rem;
    margin: 0 0 0.5rem;
    color: #1a202c;
    font-weight: 800;
  }

  .mule-blue { color: #00a2df; } /* MuleSoft Blue accent */

  .app-header p {
    font-size: 1.1rem;
    color: #718096;
    margin: 0;
    max-width: 700px;
    margin: 0 auto;
  }

  /* --- Common Card Styles --- */
  .card, .info-card, .result-card {
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05), 0 1px 3px rgba(0, 0, 0, 0.03);
    padding: 2rem;
  }

  /* --- Form Section Styling (Responsive and Fast) --- */
  .form-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
  }

  /* Ensure version group is smaller if space allows */
  @media (min-width: 768px) {
    .version-group { grid-column: span 1; }
  }

  .form-group label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 600;
    font-size: 0.9rem;
    color: #4a5568;
  }

  .form-group input {
    width: 100%;
    padding: 0.75rem 1rem;
    border: 2px solid #e2e8f0;
    border-radius: 8px;
    font-size: 1rem;
    transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  }

  .form-group input:focus {
    outline: none;
    border-color: #00a2df;
    box-shadow: 0 0 0 3px rgba(0, 162, 223, 0.15);
  }

  .form-actions {
    text-align: right;
  }

  /* --- Fast/Polished Buttons & States --- */
  .primary-btn {
    background-color: #00a2df;
    color: white;
    border: none;
    padding: 0.8rem 2rem;
    font-size: 1rem;
    font-weight: 700;
    border-radius: 8px;
    cursor: pointer;
    transition: background-color 0.15s ease-in-out, transform 0.1s ease-in-out;
    display: inline-flex;
    align-items: center;
    gap: 0.75rem;
  }

  .primary-btn:hover:not(:disabled) {
    background-color: #0081b3;
    transform: translateY(-1px);
  }

  .primary-btn:active:not(:disabled) {
    transform: translateY(1px);
  }

  .primary-btn:disabled {
    background-color: #cbd5e0;
    cursor: not-allowed;
    opacity: 0.8;
  }

  /* Polished Loading Spinner */
  .primary-btn.is-loading {
    padding-left: 1.5rem;
  }

  .spinner {
    width: 18px;
    height: 18px;
    border: 2px solid white;
    border-top-color: transparent;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  /* --- Results Section Styling (High Impact UX) --- */
  .results-section {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .results-header h2 {
    font-size: 1.25rem;
    font-weight: 700;
    color: #4a5568;
    margin: 0;
  }

  .results-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .result-card {
    padding: 1.5rem;
    border-left: 6px solid #e2e8f0; /* Default border */
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }

  /* Instant Visual Cue on Hover for results */
  .result-card:hover {
    transform: translateX(4px);
    box-shadow: 0 6px 10px rgba(0, 0, 0, 0.08), 0 2px 4px rgba(0, 0, 0, 0.05);
  }

  /* Breaking vs Feature semantic coloring */
  .result-card.breaking {
    border-left-color: #f56565; /* Bold Red for breaking */
  }

  .result-card.feature {
    border-left-color: #48bb78; /* Polished Green for feature */
  }

  .result-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 0.75rem;
  }

  .badge {
    padding: 0.35rem 1rem;
    border-radius: 9999px;
    font-size: 0.7rem;
    font-weight: 800;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  .badge.breaking {
    background-color: #fff5f5;
    color: #c53030;
  }

  .badge.feature {
    background-color: #f0fff4;
    color: #276749;
  }

  .level-badge {
    font-size: 0.8rem;
    color: #a0aec0;
    font-family: Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  }

  .description {
    margin: 0;
    font-size: 1rem;
    line-height: 1.6;
    color: #2d3748;
  }

  /* --- UI Info States (Loading, Error, Intro) --- */
  .info-card h2 {
    font-size: 1.5rem;
    margin-top: 0;
    margin-bottom: 1rem;
  }

  .info-card p {
    margin: 0 0 1rem;
    color: #4a5568;
    max-width: 800px;
  }

  .info-card p:last-child {
    margin-bottom: 0;
  }

  .error-state {
    background-color: #fff5f5;
    border: 2px solid #feb2b2;
    color: #c53030;
  }

  .error-state p { color: #c53030; }

  .loading-state h2, .intro-state h2 { color: #2d3748; }

  /* Responsive styling adjust for small mobile screens */
  @media (max-width: 600px) {
    .app-header h1 { font-size: 2rem; }
    .primary-btn { width: 100%; justify-content: center; }
  }
</style>