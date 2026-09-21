<script>
  let { account = null, isConnected = false, onConnect } = $props();

  let queryAddress  = $state('');
  let selectedChain = $state('');
  let singleResult  = $state(null);
  let singleLoading = $state(false);
  let singleError   = $state(null);

  let massAddress = $state('');
  let massResults = $state([]);
  let massLoading = $state(false);
  let massError   = $state(null);

  const NETWORKS = [
    { chainIdHex: '0x1',      chainId: 1,       name: 'Ethereum',          symbol: 'ETH',   rpc: 'https://rpc.ankr.com/eth' },
    { chainIdHex: '0xaa36a7', chainId: 11155111, name: 'Sepolia',           symbol: 'ETH',   rpc: 'https://rpc.sepolia.org' },
    { chainIdHex: '0x89',     chainId: 137,      name: 'Polygon',           symbol: 'MATIC', rpc: 'https://polygon-rpc.com' },
    { chainIdHex: '0x38',     chainId: 56,       name: 'BNB Smart Chain',   symbol: 'BNB',   rpc: 'https://bsc-dataseed.binance.org' },
    { chainIdHex: '0x39',     chainId: 57,       name: 'Syscoin',           symbol: 'SYS',   rpc: 'https://rpc.syscoin.org' },
    { chainIdHex: '0x1644',   chainId: 5700,     name: 'Syscoin Tanenbaum', symbol: 'tSYS',  rpc: 'https://rpc.tanenbaum.io' },
    { chainIdHex: '0x23a',    chainId: 570,      name: 'Rollux',            symbol: 'SYS',   rpc: 'https://rpc.rollux.com' },
    { chainIdHex: '0xa86a',   chainId: 43114,    name: 'Avalanche',         symbol: 'AVAX',  rpc: 'https://api.avax.network/ext/bc/C/rpc' },
    { chainIdHex: '0xfa',     chainId: 250,      name: 'Fantom',            symbol: 'FTM',   rpc: 'https://rpc.ftm.tools' },
    { chainIdHex: '0xa4b1',   chainId: 42161,    name: 'Arbitrum One',      symbol: 'ETH',   rpc: 'https://arb1.arbitrum.io/rpc' },
    { chainIdHex: '0xa',      chainId: 10,       name: 'Optimism',          symbol: 'ETH',   rpc: 'https://mainnet.optimism.io' },
  ];

  function isValidAddr(addr) {
    return /^0x[0-9a-fA-F]{40}$/.test(addr);
  }

  function weiToToken(hex) {
    return (Number(BigInt(hex)) / 1e18).toFixed(6);
  }

  async function rpcBalance(rpcUrl, address) {
    const res = await fetch(rpcUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ jsonrpc: '2.0', method: 'eth_getBalance', params: [address, 'latest'], id: 1 }),
    });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    if (data.error) throw new Error(data.error.message);
    return data.result;
  }

  // ── Consulta simple ────────────────────────────────────────────
  async function querySingle() {
    singleError = null; singleResult = null;
    const addr = (isConnected && !queryAddress.trim()) ? account : queryAddress.trim();

    if (!addr) { singleError = 'Ingresa una dirección o conecta tu wallet.'; return; }
    if (!isValidAddr(addr)) { singleError = 'Dirección inválida (0x + 40 caracteres hexadecimales).'; return; }
    if (!selectedChain)    { singleError = 'Selecciona una red.'; return; }

    const network = NETWORKS.find(n => n.chainIdHex === selectedChain);
    if (!network) return;
    singleLoading = true;

    try {
      let hex;
      if (isConnected) {
        try {
          const cur = await window.ethereum.request({ method: 'eth_chainId' });
          hex = cur === selectedChain
            ? await window.ethereum.request({ method: 'eth_getBalance', params: [addr, 'latest'] })
            : await rpcBalance(network.rpc, addr);
        } catch (_) { hex = await rpcBalance(network.rpc, addr); }
      } else {
        hex = await rpcBalance(network.rpc, addr);
      }

      singleResult = { address: addr, balance: weiToToken(hex), symbol: network.symbol, network: network.name };
    } catch (e) {
      singleError = `Error al consultar: ${e.message}`;
    } finally {
      singleLoading = false;
    }
  }

  // ── Consulta masiva ────────────────────────────────────────────
  async function queryMass() {
    massError = null;
    const addr = massAddress.trim();
    if (!addr) { massError = 'Ingresa una dirección.'; return; }
    if (!isValidAddr(addr)) { massError = 'Dirección inválida.'; return; }

    massLoading = true;
    massResults = NETWORKS.map(n => ({ ...n, balance: null, status: 'loading' }));

    await Promise.allSettled(NETWORKS.map(async (n, i) => {
      try {
        const hex = await rpcBalance(n.rpc, addr);
        massResults = massResults.map((r, j) => j === i ? { ...r, balance: weiToToken(hex), status: 'ok' } : r);
      } catch (_) {
        massResults = massResults.map((r, j) => j === i ? { ...r, status: 'error' } : r);
      }
    }));

    massLoading = false;
  }

  function useMyAddress()     { if (account) queryAddress = account; }
  function useMyAddressMass() { if (account) massAddress  = account; }

  function shortAddr(addr) { return addr ? `${addr.slice(0,6)}...${addr.slice(-4)}` : ''; }

  async function copyText(t) { await navigator.clipboard.writeText(t); }

  // Estadísticas reactivas usando $derived de Svelte 5
  let massStats = $derived({
    ok:  massResults.filter(r => r.status === 'ok').length,
    err: massResults.filter(r => r.status === 'error').length,
  });
</script>

<div class="page">

  <div class="page-header">
    <h2 class="page-title">Consultar Saldo</h2>
    <p class="page-sub">Verifica el saldo de cualquier dirección en una red específica o en múltiples redes a la vez</p>
  </div>

  <!-- ─── Sección 1: Consulta por red ─────────────────────────── -->
  <div class="query-card">
    <div class="query-card-header">
      <div class="query-num">01</div>
      <div>
        <h3>Consulta por red</h3>
        <p>Saldo de una dirección en una red específica</p>
      </div>
      <div class="session-badge" class:connected={isConnected}>
        {isConnected ? 'Con sesión' : 'Sin sesión'}
      </div>
    </div>

    <div class="form-row">
      <div class="form-field flex1">
        <label class="field-label" for="q-addr">Dirección</label>
        <div class="input-group">
          <input
            id="q-addr"
            type="text"
            bind:value={queryAddress}
            placeholder={isConnected
              ? `Vacío para usar wallet (${shortAddr(account)})`
              : '0x742d35Cc6634C0532925...'}
            class="field-input"
            class:field-invalid={queryAddress && !isValidAddr(queryAddress)}
          />
          {#if isConnected}
            <button class="btn-inline" onclick={useMyAddress}>Mi wallet</button>
          {/if}
        </div>
        {#if queryAddress && !isValidAddr(queryAddress)}
          <span class="field-error">Dirección inválida</span>
        {/if}
        {#if !queryAddress && isConnected}
          <span class="field-hint">Se consultará: <code>{account}</code></span>
        {/if}
      </div>

      <div class="form-field w200">
        <label class="field-label" for="q-chain">Red</label>
        <select id="q-chain" bind:value={selectedChain} class="field-select">
          <option value="">Selecciona una red</option>
          {#each NETWORKS as n}
            <option value={n.chainIdHex}>{n.name} ({n.symbol})</option>
          {/each}
        </select>
      </div>
    </div>

    {#if singleError}
      <div class="alert alert-error">{singleError}</div>
    {/if}

    <button class="btn-primary" onclick={querySingle} disabled={singleLoading}>
      {#if singleLoading}
        <span class="spinner"></span> Consultando...
      {:else}
        Consultar saldo
      {/if}
    </button>

    <!-- Resultado simple -->
    {#if singleResult}
      <div class="result-block">
        <div class="result-row">
          <span class="rlabel">Red</span>
          <span class="rvalue">{singleResult.network}</span>
        </div>
        <div class="result-row">
          <span class="rlabel">Dirección</span>
          <div class="rvalue addr-row">
            <code>{singleResult.address}</code>
            <button class="btn-copy-xs" onclick={() => copyText(singleResult.address)} title="Copiar dirección" aria-label="Copiar dirección">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="12" height="12">
                <rect x="9" y="9" width="13" height="13" rx="2"/>
                <path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1"/>
              </svg>
            </button>
          </div>
        </div>
        <div class="result-row">
          <span class="rlabel">Saldo</span>
          <div class="balance-display">
            <span class="balance-num">{singleResult.balance}</span>
            <span class="balance-sym">{singleResult.symbol}</span>
          </div>
        </div>
      </div>
    {/if}
  </div>

  <!-- ─── Sección 2: Consulta masiva ──────────────────────────── -->
  <div class="query-card">
    <div class="query-card-header">
      <div class="query-num">02</div>
      <div>
        <h3>Consulta masiva</h3>
        <p>Saldo de una dirección en {NETWORKS.length} redes simultáneamente</p>
      </div>
      <span class="net-count-badge">{NETWORKS.length} redes</span>
    </div>

    <div class="form-row">
      <div class="form-field flex1">
        <label class="field-label" for="m-addr">Dirección</label>
        <div class="input-group">
          <input
            id="m-addr"
            type="text"
            bind:value={massAddress}
            placeholder="0x742d35Cc6634C0532925a3b8D4C9b4A..."
            class="field-input"
            class:field-invalid={massAddress && !isValidAddr(massAddress)}
          />
          {#if isConnected}
            <button class="btn-inline" onclick={useMyAddressMass}>Mi wallet</button>
          {/if}
        </div>
        {#if massAddress && !isValidAddr(massAddress)}
          <span class="field-error">Dirección inválida</span>
        {/if}
      </div>
    </div>

    {#if massError}
      <div class="alert alert-error">{massError}</div>
    {/if}

    <button class="btn-primary" onclick={queryMass} disabled={massLoading}>
      {#if massLoading}
        <span class="spinner"></span> Consultando {NETWORKS.length} redes...
      {:else}
        Consulta masiva — {NETWORKS.length} redes
      {/if}
    </button>

    <!-- Resultados masivos -->
    {#if massResults.length > 0}
      <div class="mass-header-bar">
        <span class="mass-addr-label">
          Resultados para
          <code>{shortAddr(massAddress)}</code>
        </span>
        <div class="mass-stats">
          <span class="stat-ok">{massStats.ok} OK</span>
          {#if massStats.err > 0}
            <span class="stat-err">{massStats.err} Error</span>
          {/if}
        </div>
      </div>

      <div class="mass-table">
        <div class="mass-table-head">
          <span>Red</span>
          <span>Saldo</span>
          <span>Símbolo</span>
          <span>Estado</span>
        </div>
        {#each massResults as r}
          <div class="mass-row" class:mass-row-ok={r.status==='ok'} class:mass-row-err={r.status==='error'}>
            <span class="mass-net">{r.name}</span>
            <span class="mass-bal">
              {#if r.status === 'loading'}
                <span class="spinner-xs"></span>
              {:else if r.status === 'error'}
                <span class="no-resp">—</span>
              {:else}
                {r.balance}
              {/if}
            </span>
            <span class="mass-sym">{r.symbol}</span>
            <span class="mass-status" class:ok={r.status==='ok'} class:err={r.status==='error'} class:loading={r.status==='loading'}>
              {r.status === 'ok' ? 'OK' : r.status === 'error' ? 'Error' : '...'}
            </span>
          </div>
        {/each}
      </div>
    {/if}
  </div>

  <!-- Notas -->
  <div class="tech-note">
    <div class="tech-title">Notas técnicas</div>
    <ul>
      <li>La consulta masiva usa RPC públicos directos — funciona sin wallet conectada.</li>
      <li>Cuando hay sesión activa y la red coincide, se usa <code>window.ethereum</code>; de lo contrario se hace llamada RPC directa.</li>
      <li>Los saldos UTXO (Bitcoin, Syscoin UTXO) requieren APIs específicas y no están incluidos en la consulta masiva EVM.</li>
      <li>Los RPCs públicos pueden tener limitación de solicitudes — errores ocasionales son normales.</li>
    </ul>
  </div>

</div>

<style>
  .page { display: flex; flex-direction: column; gap: 1.25rem; }

  .page-header { margin-bottom: 0.25rem; }
  .page-title  { font-size: 1.5rem; font-weight: 700; color: var(--text-primary); letter-spacing: -0.02em; }
  .page-sub    { font-size: 0.85rem; color: var(--text-muted); margin-top: 0.25rem; }

  /* Card de consulta */
  .query-card {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-xl);
    padding: 1.75rem;
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
  }

  .query-card-header {
    display: flex;
    align-items: center;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .query-num {
    width: 32px; height: 32px;
    background: var(--accent);
    color: #fff;
    border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    font-size: 0.78rem;
    font-weight: 800;
    font-family: var(--font-mono);
    flex-shrink: 0;
  }

  .query-card-header h3 {
    font-size: 1rem;
    font-weight: 700;
    color: var(--text-primary);
    margin-bottom: 0.1rem;
  }

  .query-card-header p {
    font-size: 0.78rem;
    color: var(--text-muted);
  }

  .session-badge {
    margin-left: auto;
    font-size: 0.68rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    padding: 0.2rem 0.6rem;
    border-radius: 20px;
    background: var(--bg-elevated);
    color: var(--text-muted);
    border: 1px solid var(--border-default);
  }

  .session-badge.connected {
    background: var(--success-dim);
    color: var(--success);
    border-color: rgba(34,197,94,0.2);
  }

  .net-count-badge {
    margin-left: auto;
    font-size: 0.68rem;
    font-weight: 700;
    letter-spacing: 0.05em;
    padding: 0.2rem 0.6rem;
    border-radius: 20px;
    background: var(--accent-dim);
    color: var(--text-accent);
    border: 1px solid var(--accent-glow);
  }

  /* Formulario */
  .form-row {
    display: flex;
    gap: 0.875rem;
    align-items: flex-start;
    flex-wrap: wrap;
  }

  .form-field { display: flex; flex-direction: column; gap: 0.4rem; }
  .flex1 { flex: 1; min-width: 200px; }
  .w200  { width: 200px; flex-shrink: 0; }

  .field-label {
    font-size: 0.78rem;
    font-weight: 600;
    color: var(--text-secondary);
  }

  .input-group {
    display: flex;
    gap: 0.4rem;
  }

  .field-input {
    flex: 1;
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-md);
    padding: 0.65rem 0.875rem;
    color: var(--text-primary);
    font-size: 0.85rem;
    transition: border-color 0.15s, box-shadow 0.15s;
    min-width: 0;
  }

  .field-input:focus {
    outline: none;
    border-color: var(--accent);
    box-shadow: 0 0 0 3px var(--accent-dim);
  }

  .field-input::placeholder { color: var(--text-muted); font-size: 0.78rem; }
  .field-input.field-invalid { border-color: var(--danger); }

  .field-select {
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-md);
    padding: 0.65rem 0.875rem;
    color: var(--text-primary);
    font-size: 0.85rem;
    cursor: pointer;
    width: 100%;
    transition: border-color 0.15s;
  }

  .field-select:focus {
    outline: none;
    border-color: var(--accent);
    box-shadow: 0 0 0 3px var(--accent-dim);
  }

  .field-select option { background: var(--bg-elevated); }

  .btn-inline {
    background: var(--bg-overlay);
    border: 1px solid var(--border-default);
    color: var(--text-secondary);
    padding: 0.5rem 0.75rem;
    border-radius: var(--radius-md);
    cursor: pointer;
    font-size: 0.75rem;
    font-weight: 600;
    white-space: nowrap;
    transition: border-color 0.15s, color 0.15s;
    font-family: var(--font-sans);
    flex-shrink: 0;
  }

  .btn-inline:hover { border-color: var(--accent-glow); color: var(--text-primary); }

  .field-error { font-size: 0.73rem; color: #fca5a5; }

  .field-hint {
    font-size: 0.73rem;
    color: var(--text-muted);
  }

  .field-hint code {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--text-accent);
    background: var(--accent-dim);
    padding: 0.1rem 0.3rem;
    border-radius: 3px;
  }

  .alert {
    padding: 0.7rem 0.875rem;
    border-radius: var(--radius-md);
    font-size: 0.82rem;
  }

  .alert-error { background: var(--danger-dim); border: 1px solid rgba(239,68,68,0.2); color: #fca5a5; }

  /* Botón */
  .btn-primary {
    background: var(--accent); color: #fff; border: none;
    padding: 0.7rem 1.5rem; border-radius: var(--radius-md);
    font-size: 0.875rem; font-weight: 600; cursor: pointer;
    display: inline-flex; align-items: center; gap: 0.5rem;
    transition: background 0.15s, box-shadow 0.15s;
    font-family: var(--font-sans);
    align-self: flex-start;
  }

  .btn-primary:hover:not(:disabled) { background: var(--accent-hover); box-shadow: 0 0 0 3px var(--accent-dim); }
  .btn-primary:disabled { opacity: 0.5; cursor: not-allowed; }

  /* Resultado simple */
  .result-block {
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-lg);
    overflow: hidden;
  }

  .result-row {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 0.75rem 1rem;
    border-bottom: 1px solid var(--border-subtle);
    flex-wrap: wrap;
  }

  .result-row:last-child { border-bottom: none; }

  .rlabel {
    font-size: 0.68rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-muted);
    min-width: 72px;
  }

  .rvalue {
    font-size: 0.875rem;
    color: var(--text-primary);
  }

  .addr-row {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  .addr-row code {
    font-family: var(--font-mono);
    font-size: 0.78rem;
    color: var(--text-accent);
    word-break: break-all;
  }

  .btn-copy-xs {
    background: transparent;
    border: 1px solid var(--border-default);
    border-radius: var(--radius-sm);
    padding: 0.2rem 0.35rem;
    cursor: pointer;
    color: var(--text-muted);
    display: flex; align-items: center;
    flex-shrink: 0;
    transition: border-color 0.15s, color 0.15s;
  }

  .btn-copy-xs:hover { border-color: var(--accent-glow); color: var(--text-accent); }

  .balance-display {
    display: flex;
    align-items: baseline;
    gap: 0.4rem;
  }

  .balance-num {
    font-size: 1.75rem;
    font-weight: 800;
    color: var(--success);
    font-family: var(--font-mono);
    letter-spacing: -0.02em;
    line-height: 1;
  }

  .balance-sym {
    font-size: 0.875rem;
    font-weight: 600;
    color: var(--text-muted);
  }

  /* Resultados masivos */
  .mass-header-bar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .mass-addr-label {
    font-size: 0.8rem;
    color: var(--text-muted);
  }

  .mass-addr-label code {
    font-family: var(--font-mono);
    color: var(--text-accent);
    background: var(--accent-dim);
    padding: 0.1rem 0.35rem;
    border-radius: 3px;
    font-size: 0.75rem;
  }

  .mass-stats { display: flex; gap: 0.625rem; font-size: 0.72rem; font-weight: 700; }
  .stat-ok  { color: var(--success); }
  .stat-err { color: var(--danger); }

  /* Tabla masiva */
  .mass-table {
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-lg);
    overflow: hidden;
  }

  .mass-table-head {
    display: grid;
    grid-template-columns: 2fr 2fr 1fr 1fr;
    gap: 0.5rem;
    padding: 0.4rem 0.875rem;
    font-size: 0.62rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-muted);
    border-bottom: 1px solid var(--border-subtle);
    background: rgba(0,0,0,0.15);
  }

  .mass-row {
    display: grid;
    grid-template-columns: 2fr 2fr 1fr 1fr;
    gap: 0.5rem;
    align-items: center;
    padding: 0.6rem 0.875rem;
    border-bottom: 1px solid var(--border-subtle);
    transition: background 0.15s;
  }

  .mass-row:last-child { border-bottom: none; }
  .mass-row:hover { background: rgba(255,255,255,0.02); }
  .mass-row-ok  { }
  .mass-row-err { opacity: 0.55; }

  .mass-net { font-size: 0.82rem; font-weight: 500; color: var(--text-secondary); }
  .mass-bal { font-size: 0.85rem; font-weight: 700; color: var(--success); font-family: var(--font-mono); }
  .mass-sym { font-size: 0.72rem; font-family: var(--font-mono); color: var(--text-muted); }
  .no-resp  { color: var(--text-muted); font-weight: 400; }

  .mass-status {
    font-size: 0.62rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.06em;
  }

  .mass-status.ok      { color: var(--success); }
  .mass-status.err     { color: var(--danger); }
  .mass-status.loading { color: var(--text-muted); }

  /* Nota técnica */
  .tech-note {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-left: 3px solid var(--accent);
    border-radius: var(--radius-md);
    padding: 1.25rem;
  }

  .tech-title {
    font-size: 0.7rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-muted);
    margin-bottom: 0.75rem;
  }

  .tech-note ul {
    list-style: none;
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
  }

  .tech-note li {
    font-size: 0.8rem;
    color: var(--text-muted);
    padding-left: 1rem;
    position: relative;
    line-height: 1.55;
  }

  .tech-note li::before { content: '—'; position: absolute; left: 0; color: var(--accent); font-weight: 700; }

  .tech-note code {
    font-family: var(--font-mono);
    font-size: 0.75rem;
    color: var(--text-accent);
    background: var(--accent-dim);
    padding: 0.1rem 0.3rem;
    border-radius: 3px;
  }

  /* Spinners */
  .spinner {
    width: 14px; height: 14px;
    border: 2px solid rgba(255,255,255,0.25);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    display: inline-block;
    flex-shrink: 0;
  }

  .spinner-xs {
    width: 11px; height: 11px;
    border: 2px solid rgba(165,180,252,0.2);
    border-top-color: var(--text-accent);
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    display: inline-block;
  }

  @keyframes spin { to { transform: rotate(360deg); } }

  @media (max-width: 540px) {
    .w200 { width: 100%; }
    .mass-table-head { display: none; }
    .mass-row { grid-template-columns: 2fr 2fr 1fr; }
    .mass-status { display: none; }
  }
</style>
