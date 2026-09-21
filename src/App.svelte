<script>
  import { onMount } from 'svelte';
  import HomePage        from './lib/HomePage.svelte';
  import TransactionPage from './lib/TransactionPage.svelte';
  import NetworkPage     from './lib/NetworkPage.svelte';
  import BalancePage     from './lib/BalancePage.svelte';

  // ── Estado global de la wallet ────────────────────────────────
  let account      = $state(null);
  let networkName  = $state(null);
  let chainId      = $state(null);
  let isConnected  = $state(false);
  let isConnecting = $state(false);
  let error        = $state(null);

  // ── Navegación ────────────────────────────────────────────────
  let currentPage = $state('home');

  const NAV_ITEMS = [
    { id: 'home',        label: 'Inicio',         short: 'Inicio' },
    { id: 'transaction', label: 'Transacciones',   short: 'Enviar' },
    { id: 'network',     label: 'Redes',           short: 'Redes' },
    { id: 'balance',     label: 'Consultar Saldo', short: 'Saldo' },
  ];

  // ── Wallet ────────────────────────────────────────────────────
  function isPaliAvailable() {
    return typeof window !== 'undefined' && typeof window.ethereum !== 'undefined';
  }

  async function connectWallet() {
    error = null;
    if (!isPaliAvailable()) {
      error = 'Pali Wallet no está instalada. Visita pali.syscoin.org';
      return;
    }
    isConnecting = true;
    try {
      const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
      if (!accounts.length) { error = 'No se encontraron cuentas.'; return; }
      account     = accounts[0];
      isConnected = true;
      await loadNetwork();
      window.ethereum.on('accountsChanged', onAccountsChanged);
      window.ethereum.on('chainChanged',    onChainChanged);
    } catch (e) {
      error = e.code === 4001 ? 'Conexión rechazada por el usuario.' : `Error: ${e.message}`;
    } finally {
      isConnecting = false;
    }
  }

  async function loadNetwork() {
    try {
      const cid = await window.ethereum.request({ method: 'eth_chainId' });
      chainId = parseInt(cid, 16);
      const NAMES = {
        1:'Ethereum', 11155111:'Sepolia', 137:'Polygon', 80001:'Mumbai',
        80002:'Amoy', 56:'BNB Chain', 97:'BNB Testnet', 57:'Syscoin',
        5700:'Tanenbaum', 570:'Rollux', 57000:'Rollux Testnet',
        43114:'Avalanche', 250:'Fantom', 42161:'Arbitrum', 10:'Optimism',
      };
      networkName = NAMES[chainId] || `Chain ${chainId}`;
    } catch (_) { networkName = 'Desconocida'; }
  }

  function onAccountsChanged(accs) {
    if (!accs.length) disconnectWallet(); else { account = accs[0]; }
  }
  function onChainChanged() { loadNetwork(); }

  function disconnectWallet() {
    account = null; networkName = null; chainId = null;
    isConnected = false; error = null;
    if (window.ethereum) {
      window.ethereum.removeListener('accountsChanged', onAccountsChanged);
      window.ethereum.removeListener('chainChanged',    onChainChanged);
    }
  }

  function shortAddr(addr) {
    return addr ? `${addr.slice(0,6)}...${addr.slice(-4)}` : '';
  }

  async function copyAddress() {
    if (account) await navigator.clipboard.writeText(account);
  }

  onMount(() => {
    if (isPaliAvailable()) {
      window.ethereum.request({ method: 'eth_accounts' })
        .then(accs => {
          if (accs.length) {
            account = accs[0]; isConnected = true;
            loadNetwork();
            window.ethereum.on('accountsChanged', onAccountsChanged);
            window.ethereum.on('chainChanged',    onChainChanged);
          }
        }).catch(() => {});
    }
  });
</script>

<div class="shell">

  <!-- ── Sidebar ─────────────────────────────────────────────── -->
  <aside class="sidebar">

    <div class="brand">
      <div class="brand-mark">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M12 2L2 7l10 5 10-5-10-5z"/>
          <path d="M2 17l10 5 10-5"/>
          <path d="M2 12l10 5 10-5"/>
        </svg>
      </div>
      <div class="brand-text">
        <span class="brand-name">Pali DApp</span>
        <span class="brand-sub">SID · Valle Grande</span>
      </div>
    </div>

    <nav class="nav">
      {#each NAV_ITEMS as item}
        <button
          class="nav-item"
          class:active={currentPage === item.id}
          onclick={() => currentPage = item.id}
        >
          <span class="nav-indicator"></span>
          {item.label}
        </button>
      {/each}
    </nav>

    <div class="sidebar-footer">
      {#if isConnected}
        <div class="wallet-panel">
          <div class="wallet-panel-header">
            <div class="status-indicator connected"></div>
            <span class="wallet-panel-label">Conectado</span>
            <button class="btn-icon-ghost" onclick={disconnectWallet} title="Desconectar">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="14" height="14">
                <line x1="18" y1="6" x2="6" y2="18"/>
                <line x1="6" y1="6" x2="18" y2="18"/>
              </svg>
            </button>
          </div>
          <button class="wallet-addr-btn" onclick={copyAddress}>
            <code>{shortAddr(account)}</code>
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="12" height="12">
              <rect x="9" y="9" width="13" height="13" rx="2"/>
              <path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1"/>
            </svg>
          </button>
          {#if networkName}
            <div class="wallet-network">
              <span class="net-dot"></span>
              {networkName}
            </div>
          {/if}
        </div>
      {:else}
        <button class="btn-connect" onclick={connectWallet} disabled={isConnecting}>
          {#if isConnecting}
            <span class="spinner"></span> Conectando...
          {:else}
            Conectar Wallet
          {/if}
        </button>
        {#if !isPaliAvailable()}
          <a href="https://pali.syscoin.org" target="_blank" rel="noopener noreferrer" class="install-hint">
            Instalar Pali Wallet →
          </a>
        {/if}
      {/if}

      {#if error}
        <div class="sidebar-error">{error}</div>
      {/if}
    </div>

  </aside>

  <!-- ── Barra mobile ─────────────────────────────────────────── -->
  <header class="topbar">
    <div class="topbar-brand">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18">
        <path d="M12 2L2 7l10 5 10-5-10-5z"/>
        <path d="M2 17l10 5 10-5"/>
        <path d="M2 12l10 5 10-5"/>
      </svg>
      Pali DApp
    </div>
    <div class="topbar-wallet">
      {#if isConnected}
        <div class="topbar-connected">
          <div class="status-indicator connected"></div>
          <code>{shortAddr(account)}</code>
        </div>
      {:else}
        <button class="btn-connect-sm" onclick={connectWallet} disabled={isConnecting}>
          Conectar
        </button>
      {/if}
    </div>
  </header>

  <!-- ── Tabs mobile ──────────────────────────────────────────── -->
  <nav class="bottom-tabs">
    {#each NAV_ITEMS as item}
      <button
        class="bottom-tab"
        class:active={currentPage === item.id}
        onclick={() => currentPage = item.id}
      >
        {item.short}
      </button>
    {/each}
  </nav>

  <!-- ── Contenido ────────────────────────────────────────────── -->
  <main class="main">
    <div class="page-wrap">
      {#if currentPage === 'home'}
        <HomePage {account} {isConnected} onConnect={connectWallet} />
      {:else if currentPage === 'transaction'}
        <TransactionPage {account} {isConnected} onConnect={connectWallet} />
      {:else if currentPage === 'network'}
        <NetworkPage {account} {isConnected} onConnect={connectWallet} />
      {:else if currentPage === 'balance'}
        <BalancePage {account} {isConnected} onConnect={connectWallet} />
      {/if}
    </div>
  </main>

</div>

<style>
  /* ── Shell layout ───────────────────────────────────────────── */
  .shell {
    display: grid;
    grid-template-columns: 220px 1fr;
    grid-template-areas: 'sidebar main';
    min-height: 100vh;
    background: var(--bg-base);
  }

  /* ── Sidebar ────────────────────────────────────────────────── */
  .sidebar {
    grid-area: sidebar;
    display: flex;
    flex-direction: column;
    gap: 2rem;
    padding: 1.5rem 1rem;
    background: var(--bg-surface);
    border-right: 1px solid var(--border-subtle);
    position: sticky;
    top: 0;
    height: 100vh;
    overflow-y: auto;
  }

  /* Brand */
  .brand {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0 0.25rem;
  }

  .brand-mark {
    width: 36px;
    height: 36px;
    background: var(--accent-dim);
    border: 1px solid var(--accent-glow);
    border-radius: var(--radius-md);
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--accent);
    flex-shrink: 0;
  }

  .brand-mark svg { width: 16px; height: 16px; }

  .brand-name {
    display: block;
    font-size: 0.9rem;
    font-weight: 700;
    color: var(--text-primary);
    letter-spacing: -0.01em;
  }

  .brand-sub {
    display: block;
    font-size: 0.65rem;
    color: var(--text-muted);
    letter-spacing: 0.04em;
    text-transform: uppercase;
  }

  /* Nav */
  .nav {
    display: flex;
    flex-direction: column;
    gap: 2px;
    flex: 1;
  }

  .nav-item {
    position: relative;
    display: flex;
    align-items: center;
    gap: 0.625rem;
    width: 100%;
    padding: 0.6rem 0.75rem;
    background: transparent;
    border: none;
    border-radius: var(--radius-md);
    color: var(--text-muted);
    font-size: 0.85rem;
    font-weight: 500;
    cursor: pointer;
    text-align: left;
    transition: background 0.15s, color 0.15s;
    font-family: var(--font-sans);
  }

  .nav-item:hover {
    background: rgba(255,255,255,0.04);
    color: var(--text-secondary);
  }

  .nav-item.active {
    background: var(--accent-dim);
    color: var(--text-accent);
    font-weight: 600;
  }

  .nav-indicator {
    width: 3px;
    height: 14px;
    border-radius: 2px;
    background: transparent;
    flex-shrink: 0;
    transition: background 0.15s;
  }

  .nav-item.active .nav-indicator {
    background: var(--accent);
  }

  /* Footer de sidebar */
  .sidebar-footer {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  /* Panel de wallet conectada */
  .wallet-panel {
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-lg);
    padding: 0.875rem;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .wallet-panel-header {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .wallet-panel-label {
    font-size: 0.72rem;
    font-weight: 700;
    color: var(--success);
    text-transform: uppercase;
    letter-spacing: 0.06em;
    flex: 1;
  }

  .btn-icon-ghost {
    background: transparent;
    border: none;
    color: var(--text-muted);
    cursor: pointer;
    padding: 0.2rem;
    border-radius: var(--radius-sm);
    display: flex;
    align-items: center;
    justify-content: center;
    transition: color 0.15s, background 0.15s;
  }

  .btn-icon-ghost:hover {
    color: var(--danger);
    background: var(--danger-dim);
  }

  .wallet-addr-btn {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.5rem;
    background: rgba(0,0,0,0.2);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-sm);
    padding: 0.4rem 0.6rem;
    cursor: pointer;
    color: var(--text-accent);
    font-size: 0.8rem;
    transition: border-color 0.15s;
  }

  .wallet-addr-btn:hover { border-color: var(--accent-glow); }
  .wallet-addr-btn code { font-family: var(--font-mono); font-size: 0.78rem; }

  .wallet-network {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    font-size: 0.72rem;
    color: var(--warning);
    font-weight: 500;
  }

  .net-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: var(--warning);
    flex-shrink: 0;
  }

  /* Indicador de estado */
  .status-indicator {
    width: 7px;
    height: 7px;
    border-radius: 50%;
    flex-shrink: 0;
  }

  .status-indicator.connected {
    background: var(--success);
    box-shadow: 0 0 0 2px rgba(34,197,94,0.2);
    animation: pulse-green 2.5s infinite;
  }

  @keyframes pulse-green {
    0%, 100% { opacity: 1; }
    50%       { opacity: 0.45; }
  }

  /* Botón conectar sidebar */
  .btn-connect {
    width: 100%;
    background: var(--accent);
    color: #fff;
    border: none;
    padding: 0.65rem 1rem;
    border-radius: var(--radius-md);
    font-size: 0.85rem;
    font-weight: 600;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.4rem;
    transition: background 0.15s, box-shadow 0.15s;
    font-family: var(--font-sans);
  }

  .btn-connect:hover:not(:disabled) {
    background: var(--accent-hover);
    box-shadow: 0 0 0 3px var(--accent-dim);
  }

  .btn-connect:disabled { opacity: 0.55; cursor: not-allowed; }

  .install-hint {
    font-size: 0.72rem;
    color: var(--accent);
    text-align: center;
    padding: 0.2rem;
    transition: color 0.15s;
    text-decoration: none;
  }

  .install-hint:hover { color: var(--text-primary); }

  .sidebar-error {
    font-size: 0.73rem;
    color: #fca5a5;
    background: var(--danger-dim);
    border: 1px solid rgba(239,68,68,0.2);
    border-radius: var(--radius-sm);
    padding: 0.5rem 0.625rem;
    line-height: 1.45;
  }

  /* ── Main ───────────────────────────────────────────────────── */
  .main {
    grid-area: main;
    overflow-y: auto;
    min-height: 100vh;
  }

  .page-wrap {
    max-width: 880px;
    margin: 0 auto;
    padding: 2.5rem 2rem;
  }

  /* ── Mobile topbar ──────────────────────────────────────────── */
  .topbar {
    display: none;
    position: fixed;
    top: 0; left: 0; right: 0;
    z-index: 200;
    background: rgba(8,12,20,0.92);
    backdrop-filter: blur(12px);
    border-bottom: 1px solid var(--border-subtle);
    padding: 0.75rem 1.25rem;
    align-items: center;
    justify-content: space-between;
  }

  .topbar-brand {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.9rem;
    font-weight: 700;
    color: var(--text-primary);
  }

  .topbar-brand svg { color: var(--accent); }

  .topbar-connected {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    font-size: 0.78rem;
    color: var(--text-secondary);
  }

  .topbar-connected code {
    font-family: var(--font-mono);
    font-size: 0.75rem;
    color: var(--text-accent);
  }

  .btn-connect-sm {
    background: var(--accent);
    color: #fff;
    border: none;
    padding: 0.4rem 0.875rem;
    border-radius: var(--radius-sm);
    font-size: 0.8rem;
    font-weight: 600;
    cursor: pointer;
    font-family: var(--font-sans);
    transition: background 0.15s;
  }

  .btn-connect-sm:hover:not(:disabled) { background: var(--accent-hover); }
  .btn-connect-sm:disabled { opacity: 0.55; }

  /* ── Bottom tabs mobile ─────────────────────────────────────── */
  .bottom-tabs {
    display: none;
    position: fixed;
    bottom: 0; left: 0; right: 0;
    z-index: 200;
    background: rgba(8,12,20,0.96);
    backdrop-filter: blur(12px);
    border-top: 1px solid var(--border-subtle);
    padding: 0.4rem 0;
  }

  .bottom-tab {
    flex: 1;
    background: transparent;
    border: none;
    color: var(--text-muted);
    font-size: 0.72rem;
    font-weight: 500;
    padding: 0.5rem 0.25rem;
    cursor: pointer;
    transition: color 0.15s;
    font-family: var(--font-sans);
  }

  .bottom-tab.active { color: var(--accent); font-weight: 700; }

  /* ── Responsive ─────────────────────────────────────────────── */
  @media (max-width: 768px) {
    .shell {
      grid-template-columns: 1fr;
      grid-template-areas: 'main';
      padding-top: 52px;
      padding-bottom: 64px;
    }

    .sidebar    { display: none; }
    .topbar     { display: flex; }
    .bottom-tabs { display: flex; }
    .page-wrap  { padding: 1.25rem 1rem; }
  }

  /* ── Spinner ────────────────────────────────────────────────── */
  .spinner {
    width: 13px;
    height: 13px;
    border: 2px solid rgba(255,255,255,0.25);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    display: inline-block;
  }

  @keyframes spin { to { transform: rotate(360deg); } }
</style>
