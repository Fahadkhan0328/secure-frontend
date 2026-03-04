<script lang="ts">
  import { useClerkContext } from 'svelte-clerk';
  import Chart from 'chart.js/auto'; 

  const ctx = useClerkContext();
  
  // --- CONFIGURATION: Replace this with your generated Railway URL ---
// --- CONFIGURATION: Your live Railway URL ---
const API_BASE_URL = "https://secure-backend-production-bbab.up.railway.app";1
  let token = $state("Click the button to reveal your secure JWT...");
  let isLoadingToken = $state(false);
  
  let backendResponse = $state("Waiting to fetch data from Python...");
  let isFetchingData = $state(false);
  let selectedTrace = $state(0);
  let traceMetadata = $state<Record<string, any> | null>(null);

  let chartCanvas = $state<HTMLCanvasElement | null>(null);
  let chartInstance: Chart | null = null;
  let hasData = $state(false);

  async function getJwt() {
    isLoadingToken = true;
    try {
      const rawToken = await ctx.session?.getToken(); 
      if (rawToken) token = rawToken;
      else token = "No active session found! Try logging out and back in.";
    } catch (error) {
      console.error(error);
      token = "Error fetching token!";
    }
    isLoadingToken = false;
  }

  async function fetchSecureData() {
    isFetchingData = true;
    traceMetadata = null; 
    try {
      const rawToken = await ctx.session?.getToken(); 
      if (!rawToken) {
        backendResponse = "Error: No token available. Are you logged in?";
        isFetchingData = false;
        return;
      }

      // Updated to use the live Railway URL
      const res = await fetch(`${API_BASE_URL}/api/leakage-data?trace_index=${selectedTrace}`, {
        method: "GET",
        headers: {
          "Authorization": `Bearer ${rawToken}`,
          "Content-Type": "application/json"
        }
      });

      if (res.ok) {
        const data = await res.json();
        const previewData = { ...data, data_preview: "[... array hidden for UI preview ...]" };
        backendResponse = JSON.stringify(previewData, null, 2); 
        
        if (data.metadata) {
            traceMetadata = data.metadata;
        }
        
        if (data.data_preview) {
          hasData = true;
          setTimeout(() => drawChart(data.data_preview), 0);
        }
      } else {
        backendResponse = `Access Denied! Server blocked the request (Status ${res.status}).`;
      }

    } catch (error) {
      console.error(error);
      backendResponse = `Connection Error! Could not reach the cloud server at ${API_BASE_URL}`;
    }
    isFetchingData = false;
  }

  function drawChart(dataPoints: number[]) {
    if (!chartCanvas) return;
    if (chartInstance) chartInstance.destroy();

    const labels = dataPoints.map((_, i) => `Pt ${i + 1}`);

    chartInstance = new Chart(chartCanvas, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          label: `Trace #${selectedTrace} Amplitude`,
          data: dataPoints,
          borderColor: '#8b5cf6', 
          backgroundColor: 'rgba(139, 92, 246, 0.1)',
          borderWidth: 1.5, 
          pointRadius: 0,   
          tension: 0,       
          fill: false       
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { labels: { color: '#e5e7eb', font: { size: 14 } } }
        },
        scales: {
          x: { ticks: { color: '#9ca3af' }, grid: { color: '#374151' } },
          y: { ticks: { color: '#9ca3af' }, grid: { color: '#374151' } }
        }
      }
    });
  }
</script>

<div class="max-w-5xl mx-auto py-20 px-4 mt-10 text-left">
  <div class="bg-white/80 backdrop-blur-md shadow-2xl rounded-3xl p-10 border border-gray-100">
    <div class="flex justify-between items-center mb-6">
        <h1 class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-600 to-purple-600">
            Secure Dashboard
        </h1>
        <div class="flex items-center gap-2 px-3 py-1 bg-green-100 text-green-700 rounded-full text-xs font-bold uppercase tracking-widest">
            <span class="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
            Cloud Live
        </div>
    </div>
    
    {#if ctx.user}
      <p class="text-xl text-gray-600 mb-10 text-left">
        Welcome back, <span class="font-bold text-indigo-600">{ctx.user.firstName || 'User'}</span>! 
      </p>
    {/if}

    <div class="space-y-6">
      
      <div class="bg-gray-50/50 p-8 rounded-2xl border border-gray-100 shadow-inner">
        <h2 class="text-2xl font-bold mb-4 text-gray-800 flex items-center gap-2">
          🎟️ 1. Authentication Status
        </h2>
        <button 
          onclick={getJwt} 
          disabled={isLoadingToken}
          class="bg-gray-800 hover:bg-gray-900 text-white px-6 py-2 rounded-lg shadow font-semibold transition-all mb-6 disabled:opacity-50">
          {isLoadingToken ? 'Refreshing...' : 'Reveal JWT'}
        </button>
        <div class="relative bg-gray-900 rounded-xl p-4 shadow-xl border border-gray-800 h-24 overflow-y-auto">
          <p class="text-green-400 font-mono text-xs break-all leading-relaxed">{token}</p>
        </div>
      </div>

      <div class="bg-indigo-50/50 p-8 rounded-2xl border border-indigo-100 shadow-inner">
        <h2 class="text-2xl font-bold mb-2 text-indigo-900 flex items-center gap-2">
          🐍 2. Cloud Data Analysis
        </h2>
        
        <div class="mt-6 mb-2">
          <!-- svelte-ignore a11y_label_has_associated_control -->
          <label class="block text-sm font-bold text-indigo-900 mb-2">
            🔍 Target Trace Index (0 - 59999):
          </label>
          <input 
            type="number" 
            bind:value={selectedTrace} 
            min="0" 
            max="59999"
            class="w-48 px-4 py-3 border-2 border-indigo-200 rounded-xl focus:ring-4 focus:ring-indigo-500/20 focus:border-indigo-500 outline-none transition-all font-mono text-lg"
          />
        </div>

        <button 
          onclick={fetchSecureData} 
          disabled={isFetchingData}
          class="bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-700 hover:to-purple-700 text-white px-8 py-3 rounded-xl shadow-lg hover:shadow-xl font-bold transition-all transform hover:-translate-y-0.5 flex items-center gap-2 mb-6 mt-4 disabled:opacity-50">
          {isFetchingData ? 'Fetching from Railway...' : 'Fetch Secure Data'}
        </button>

        {#if traceMetadata}
        <div class="mb-6 bg-gray-900 rounded-xl p-6 shadow-xl border border-gray-800 relative">
            <div class="absolute top-0 left-0 w-full h-1 bg-gradient-to-r from-green-400 to-emerald-500 rounded-t-xl"></div>
            <h3 class="text-lg font-bold text-green-400 mb-4 flex items-center gap-2">
                🔐 Cryptographic Metadata (Trace #{selectedTrace})
            </h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                {#each Object.entries(traceMetadata) as [key, value]}
                    <div class="bg-gray-800 p-3 rounded-lg border border-gray-700">
                        <div class="text-gray-400 font-mono text-xs uppercase tracking-wider mb-1">{key}</div>
                        <div class="text-white font-mono text-sm break-all">
                            {Array.isArray(value) ? `[ ${value.join(', ')} ]` : value}
                        </div>
                    </div>
                {/each}
            </div>
        </div>
        {/if}

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
          <div class="relative bg-gray-900 rounded-xl p-6 shadow-xl border border-gray-800 h-80 overflow-y-auto">
            <div class="absolute top-0 left-0 w-full h-1 bg-gradient-to-r from-blue-400 to-indigo-500 rounded-t-xl"></div>
            <pre class="text-blue-300 font-mono text-xs whitespace-pre-wrap">{backendResponse}</pre>
          </div>

          <div class="relative bg-gray-900 rounded-xl p-6 shadow-xl border border-gray-800 h-80 flex flex-col justify-center items-center">
            <div class="absolute top-0 left-0 w-full h-1 bg-gradient-to-r from-pink-500 to-purple-500 rounded-t-xl"></div>
            {#if hasData}
              <div class="w-full h-full relative">
                 <canvas bind:this={chartCanvas}></canvas>
              </div>
            {:else}
              <div class="text-gray-500 animate-pulse flex flex-col items-center">
                <span class="text-4xl mb-3">📊</span>
                <p>Waiting for cloud data...</p>
              </div>
            {/if}
          </div>
        </div>
      </div>
    </div>
  </div>
</div>