<script>
  let { account = null, isConnected = false, onConnect } = $props();

  let isSwitching   = $state(false);
  let switchError   = $state(null);
  let switchSuccess = $state(null);
  let currentChainId = $state(null);

  // ── Redes EVM ─────────────────────────────────────────────────
  const EVM_NETWORKS = [
    {
      chainId: '0x1', chainIdNum: 1,
      name: 'Ethereum', fullName: 'Ethereum Mainnet',
      symbol: 'ETH', type: 'Mainnet',
      rpcUrls: ['https://mainnet.infura.io/v3/9aa3d95b3bc440fa88ea12eaa4456161'],
      blockExplorerUrls: ['https://etherscan.io'],
      nativeCurrency: { name: 'Ether', symbol: 'ETH', decimals: 18 },
    },
    {
      chainId: '0xaa36a7', chainIdNum: 11155111,
      name: 'Sepolia', fullName: 'Sepolia Testnet',
      symbol: 'ETH', type: 'Testnet',
      rpcUrls: ['https://rpc.sepolia.org'],
      blockExplorerUrls: ['https://sepolia.etherscan.io'],
      nativeCurrency: { name: 'Sepolia ETH', symbol: 'ETH', decimals: 18 },
    },
    {
      chainId: '0x89', chainIdNum: 137,
      name: 'Polygon', fullName: 'Polygon Mainnet',
      symbol: 'MATIC', type: 'Mainnet',
      rpcUrls: ['https://polygon-rpc.com'],
      blockExplorerUrls: ['https://polygonscan.com'],
      nativeCurrency: { name: 'MATIC', symbol: 'MATIC', decimals: 18 },
    },
    {
      chainId: '0x13882', chainIdNum: 80002,
      name: 'Amoy', fullName: 'Polygon Amoy Testnet',
      symbol: 'MATIC', type: 'Testnet',
      rpcUrls: ['https://rpc-amoy.polygon.technology'],
      blockExplorerUrls: ['https://amoy.polygonscan.com'],
      nativeCurrency: { name: 'MATIC', symbol: 'MATIC', decimals: 18 },
    },
    {
      chainId: '0x38', chainIdNum: 56,
      name: 'BNB Chain', fullName: 'BNB Smart Chain',
      symbol: 'BNB', type: 'Mainnet',
      rpcUrls: ['https://bsc-dataseed.binance.org'],
      blockExplorerUrls: ['https://bscscan.com'],
      nativeCurrency: { name: 'BNB', symbol: 'BNB', decimals: 18 },
    },
    {
      chainId: '0x61', chainIdNum: 97,
      name: 'BNB Testnet', fullName: 'BNB Smart Chain Testnet',
      symbol: 'tBNB', type: 'Testnet',
      rpcUrls: ['https://data-seed-prebsc-1-s1.binance.org:8545'],
      blockExplorerUrls: ['https://testnet.bscscan.com'],
      nativeCurrency: { name: 'tBNB', symbol: 'tBNB', decimals: 18 },
    },
    {
      chainId: '0x39', chainIdNum: 57,
      name: 'Syscoin', fullName: 'Syscoin Mainnet',
      symbol: 'SYS', type: 'Mainnet',
      rpcUrls: ['https://rpc.syscoin.org'],
      blockExplorerUrls: ['https://explorer.syscoin.org'],
      nativeCurrency: { name: 'Syscoin', symbol: 'SYS', decimals: 18 },
    },
    {
      chainId: '0x1644', chainIdNum: 5700,
      name: 'Tanenbaum', fullName: 'Syscoin Tanenbaum Testnet',
      symbol: 'tSYS', type: 'Testnet',
      rpcUrls: ['https://rpc.tanenbaum.io'],
      blockExplorerUrls: ['https://tanenbaum.io'],
      nativeCurrency: { name: 'tSYS', symbol: 'tSYS', decimals: 18 },
    },
    {
      chainId: '0x23a', chainIdNum: 570,
      name: 'Rollux', fullName: 'Rollux Mainnet (L2)',
      symbol: 'SYS', type: 'L2 Mainnet',
      rpcUrls: ['https://rpc.rollux.com'],
      blockExplorerUrls: ['https://explorer.rollux.com'],
      nativeCurrency: { name: 'Syscoin', symbol: 'SYS', decimals: 18 },
    },
    {
      chainId: '0xdea8', chainIdNum: 57000,
      name: 'Rollux Testnet', fullName: 'Rollux Testnet (L2)',
      symbol: 'tSYS', type: 'L2 Testnet',
      rpcUrls: ['https://rpc-tanenbaum.rollux.com'],
      blockExplorerUrls: ['https://rollux.tanenbaum.io'],
      nativeCurrency: { name: 'tSYS', symbol: 'tSYS', decimals: 18 },
    },
    {
      chainId: '0xa86a', chainIdNum: 43114,
      name: 'Avalanche', fullName: 'Avalanche C-Chain',
      symbol: 'AVAX', type: 'Mainnet',
      rpcUrls: ['https://api.avax.network/ext/bc/C/rpc'],
      blockExplorerUrls: ['https://snowtrace.io'],
      nativeCurrency: { name: 'AVAX', symbol: 'AVAX', decimals: 18 },
    },
    {
      chainId: '0xa', chainIdNum: 10,
      name: 'Optimism', fullName: 'Optimism Mainnet',
      symbol: 'ETH', type: 'L2 Mainnet',
      rpcUrls: ['https://mainnet.optimism.io'],
      blockExplorerUrls: ['https://optimistic.etherscan.io'],
      nativeCurrency: { name: 'Ether', symbol: 'ETH', decimals: 18 },
    },
  ];

  const UTXO_NETWORKS = [
    { name: 'Syscoin UTXO', symbol: 'SYS', desc: 'Red nativa UTXO de Syscoin. Gestionada desde el modo UTXO de Pali Wallet.' },
    { name: 'Bitcoin',      symbol: 'BTC', desc: 'Red Bitcoin. Soportada en el modo UTXO de Pali Wallet.' },
    { name: 'Litecoin',     symbol: 'LTC', desc: 'Red Litecoin. Soportada en el modo UTXO de Pali Wallet.' },
    { name: 'Dogecoin',     symbol: 'DOGE', desc: 'Red Dogecoin. Soportada en el modo UTXO de Pali Wallet.' },
  ];

  const TYPE_STYLE = {
    'Mainnet':    { color: 'var(--success)',  bg: 'var(--success-dim)' },
    'Testnet':    { color: 'var(--text-muted)', bg: 'var(--bg-overlay)' },
    'L2 Mainnet': { color: 'var(--accent)',   bg: 'var(--accent-dim)' },
    'L2 Testnet': { color: 'var(--text-muted)', bg: 'var(--bg-overlay)' },
  };

  $effect(() => { if (isConnected) loadChain(); });

  async function loadChain() {
    try {
      const cid = await window.ethereum.request({ method: 'eth_chainId' });
      currentChainId = parseInt(cid, 16);
    } catch (_) {}
  }

  async function switchTo(network) {
    switchError = null; switchSuccess = null;
    isSwitching = true;
    try {
      await window.ethereum.request({
        method: 'wallet_switchEthereumChain',
        params: [{ chainId: network.chainId }],
      });
      currentChainId = network.chainIdNum;
      switchSuccess = `Red cambiada a ${network.fullName}`;
    } catch (e) {
      if (e.code === 4902 || e.code === -32603) {
        try {
          await window.ethereum.request({
            method: 'wallet_addEthereumChain',
            params: [{
              chainId:           network.chainId,
              chainName:         network.fullName,
              nativeCurrency:    network.nativeCurrency,
              rpcUrls:           network.rpcUrls,
              blockExplorerUrls: network.blockExplorerUrls,
            }],
          });
          currentChainId = network.chainIdNum;
          switchSuccess = `Red ${network.fullName} añadida y activada`;
        } catch (ae) {
          switchError = ae.code === 4001
            ? 'El usuario rechazó añadir la red.'
            : `Error al añadir la red: ${ae.message}`;
        }
      } else if (e.code === 4001) {
        switchError = 'El usuario rechazó el cambio de red.';
      } else {
        switchError = `Error: ${e.message}`;
      }
    } finally {
      isSwitching = false;
    }
  }

  function currentNetworkName() {
    const n = EVM_NETWORKS.find(n => n.chainIdNum === currentChainId);
    return n ? n.fullName : currentChainId ? `Chain ID: ${currentChainId}` : 'Desconocida';
  }

  function isActive(n) { return n.chainIdNum === currentChainId; }
</script>

<div class="page">

  <div class="page-header">
    <h2 class="page-title">Redes</h2>
    <p class="page-sub">Cambia entre redes EVM y UTXO. Las redes no configuradas se añaden automáticamente.</p>
  </div>

  {#if !isConnected}
    <div class="empty-state">
      <div class="empty-icon">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" width="32" height="32">
          <circle cx="12" cy="12" r="10"/>
          <line x1="2" y1="12" x2="22" y2="12"/>
          <path d="M12 2a15.3 15.3 0 010 20M12 2a15.3 15.3 0 000 20"/>
        </svg>
      </div>
      <h3>Wallet no conectada</h3>
      <p>Conecta tu Pali Wallet para cambiar de red.</p>
      <button class="btn-primary" onclick={onConnect}>Conectar Pali Wallet</button>
    </div>

  {:else}

    <!-- Red activa -->
    <div class="active-net-bar">
      <div class="active-net-left">
        <div class="active-dot"></div>
        <span class="active-net-label">Red activa</span>
        <span class="active-net-name">{currentNetworkName()}</span>
        {#if currentChainId}<span class="active-chain-id">Chain ID: {currentChainId}</span>{/if}
      </div>
      <button class="btn-ghost-sm" onclick={loadChain} title="Actualizar">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="14" height="14">
          <polyline points="23 4 23 10 17 10"/>
          <path d="M20.49 15a9 9 0 11-2.12-9.36L23 10"/>
        </svg>
      </button>
    </div>

    <!-- Alertas -->
    {#if switchError}
      <div class="alert alert-error">{switchError}</div>
    {/if}
    {#if switchSuccess}
      <div class="alert alert-success">{switchSuccess}</div>
    {/if}

    <!-- Redes EVM -->
    <div class="net-section">
      <div class="net-section-header">
        <span class="section-badge">EVM Networks</span>
        <span class="section-meta">
          <code>wallet_switchEthereumChain</code> · <code>wallet_addEthereumChain</code>
        </span>
      </div>

      <div class="net-table">
        <div class="net-table-head">
          <span>Red</span>
          <span>Símbolo</span>
          <span>Tipo</span>
          <span>Chain ID</span>
          <span></span>
        </div>
        {#each EVM_NETWORKS as net}
          {@const active = isActive(net)}
          {@const ts = TYPE_STYLE[net.type] ?? TYPE_STYLE['Testnet']}
          <div class="net-row" class:net-row-active={active}>
            <span class="net-name" class:net-name-active={active}>{net.name}</span>
            <span class="net-symbol">{net.symbol}</span>
            <span class="net-type" style="color:{ts.color}; background:{ts.bg}">{net.type}</span>
            <span class="net-chainid">{net.chainIdNum}</span>
            <div class="net-action">
              {#if active}
                <span class="active-mark">Activa</span>
              {:else}
                <button
                  class="btn-switch"
                  onclick={() => switchTo(net)}
                  disabled={isSwitching}
                >
                  {#if isSwitching}
                    <span class="spinner"></span>
                  {:else}
                    Cambiar
                  {/if}
                </button>
              {/if}
            </div>
          </div>
        {/each}
      </div>
    </div>

    <!-- Redes UTXO -->
    <div class="net-section">
      <div class="net-section-header">
        <span class="section-badge utxo">UTXO Networks</span>
        <span class="section-meta">Gestionadas en el modo UTXO de Pali Wallet</span>
      </div>

      <div class="utxo-notice">
        Las redes UTXO (Bitcoin, Syscoin UTXO) utilizan el modelo Unspent Transaction Output y
        se gestionan en el <strong>modo UTXO</strong> de Pali Wallet, separado del modo EVM.
        No es posible cambiar a modo UTXO mediante <code>window.ethereum</code>.
      </div>

      <div class="utxo-table">
        {#each UTXO_NETWORKS as net}
          <div class="utxo-row">
            <div class="utxo-main">
              <span class="utxo-name">{net.name}</span>
              <span class="utxo-sym">{net.symbol}</span>
            </div>
            <span class="utxo-desc">{net.desc}</span>
          </div>
        {/each}
      </div>
    </div>

    <!-- Flujo técnico -->
    <div class="tech-note">
      <div class="tech-title">Flujo de cambio de red</div>
      <ol>
        <li>Se llama a <code>wallet_switchEthereumChain</code> con el <code>chainId</code> en hexadecimal.</li>
        <li>Si la wallet lanza el error <strong>4902</strong>, la red no está configurada.</li>
        <li>Se llama a <code>wallet_addEthereumChain</code> con RPC, explorador y moneda nativa.</li>
        <li>La wallet solicita confirmación al usuario y activa la red automáticamente.</li>
      </ol>
    </div>

  {/if}

</div>

<style>
  .page { display: flex; flex-direction: column; gap: 1.25rem; }

  .page-header { margin-bottom: 0.25rem; }
  .page-title { font-size: 1.5rem; font-weight: 700; color: var(--text-primary); letter-spacing: -0.02em; }
  .page-sub   { font-size: 0.85rem; color: var(--text-muted); margin-top: 0.25rem; }

  /* Empty state */
  .empty-state {
    text-align: center;
    padding: 4rem 2rem;
    background: var(--bg-surface);
    border: 1px dashed var(--border-default);
    border-radius: var(--radius-xl);
  }

  .empty-icon {
    width: 64px; height: 64px;
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-lg);
    display: flex; align-items: center; justify-content: center;
    margin: 0 auto 1.25rem;
    color: var(--text-muted);
  }

  .empty-state h3 { font-size: 1.1rem; color: var(--text-primary); margin-bottom: 0.4rem; }
  .empty-state p  { font-size: 0.875rem; color: var(--text-muted); margin-bottom: 1.5rem; }

  /* Red activa */
  .active-net-bar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    padding: 0.75rem 1rem;
    background: var(--success-dim);
    border: 1px solid rgba(34,197,94,0.18);
    border-radius: var(--radius-md);
  }

  .active-net-left {
    display: flex;
    align-items: center;
    gap: 0.625rem;
    flex-wrap: wrap;
  }

  .active-dot {
    width: 7px; height: 7px;
    border-radius: 50%;
    background: var(--success);
    flex-shrink: 0;
    animation: pulse 2.5s infinite;
  }

  @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.4} }

  .active-net-label {
    font-size: 0.68rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--success);
  }

  .active-net-name {
    font-size: 0.875rem;
    font-weight: 600;
    color: var(--text-primary);
  }

  .active-chain-id {
    font-size: 0.72rem;
    color: var(--text-muted);
    font-family: var(--font-mono);
  }

  .btn-ghost-sm {
    background: transparent;
    border: none;
    color: var(--text-muted);
    cursor: pointer;
    padding: 0.25rem;
    border-radius: var(--radius-sm);
    display: flex; align-items: center;
    transition: color 0.15s;
  }
  .btn-ghost-sm:hover { color: var(--text-primary); }

  /* Alertas */
  .alert {
    padding: 0.75rem 1rem;
    border-radius: var(--radius-md);
    font-size: 0.82rem;
    font-weight: 500;
  }
  .alert-error   { background: var(--danger-dim);  border: 1px solid rgba(239,68,68,0.2);  color: #fca5a5; }
  .alert-success { background: var(--success-dim); border: 1px solid rgba(34,197,94,0.2); color: var(--success); }

  /* Sección de red */
  .net-section {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-xl);
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .net-section-header {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    flex-wrap: wrap;
  }

  .section-badge {
    font-size: 0.7rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    padding: 0.25rem 0.7rem;
    border-radius: 20px;
    background: var(--accent-dim);
    color: var(--text-accent);
    border: 1px solid var(--accent-glow);
  }

  .section-badge.utxo {
    background: var(--warning-dim);
    color: var(--warning);
    border-color: rgba(245,158,11,0.25);
  }

  .section-meta {
    font-size: 0.75rem;
    color: var(--text-muted);
  }

  .section-meta code {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--text-accent);
    background: var(--accent-dim);
    padding: 0.1rem 0.3rem;
    border-radius: 3px;
  }

  /* Tabla EVM */
  .net-table { display: flex; flex-direction: column; }

  .net-table-head {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr 1fr auto;
    gap: 0.5rem;
    padding: 0.4rem 0.75rem;
    font-size: 0.65rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-muted);
    border-bottom: 1px solid var(--border-subtle);
    margin-bottom: 0.25rem;
  }

  .net-row {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr 1fr auto;
    gap: 0.5rem;
    align-items: center;
    padding: 0.625rem 0.75rem;
    border-radius: var(--radius-md);
    transition: background 0.15s;
  }

  .net-row:hover { background: rgba(255,255,255,0.02); }
  .net-row-active { background: rgba(34,197,94,0.04) !important; }

  .net-name {
    font-size: 0.85rem;
    font-weight: 600;
    color: var(--text-secondary);
  }

  .net-name-active { color: var(--text-primary); }

  .net-symbol {
    font-size: 0.78rem;
    font-family: var(--font-mono);
    color: var(--text-muted);
  }

  .net-type {
    display: inline-block;
    font-size: 0.62rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    padding: 0.15rem 0.5rem;
    border-radius: 20px;
    width: fit-content;
  }

  .net-chainid {
    font-size: 0.72rem;
    font-family: var(--font-mono);
    color: var(--text-muted);
  }

  .net-action { display: flex; justify-content: flex-end; min-width: 72px; }

  .active-mark {
    font-size: 0.7rem;
    font-weight: 700;
    color: var(--success);
    letter-spacing: 0.05em;
  }

  .btn-switch {
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    color: var(--text-secondary);
    padding: 0.3rem 0.75rem;
    border-radius: var(--radius-sm);
    font-size: 0.75rem;
    font-weight: 600;
    cursor: pointer;
    transition: border-color 0.15s, color 0.15s;
    display: flex; align-items: center; gap: 0.35rem;
    font-family: var(--font-sans);
  }

  .btn-switch:hover:not(:disabled) { border-color: var(--accent-glow); color: var(--text-primary); }
  .btn-switch:disabled { opacity: 0.45; cursor: not-allowed; }

  /* UTXO */
  .utxo-notice {
    font-size: 0.82rem;
    color: var(--text-muted);
    line-height: 1.6;
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-left: 3px solid var(--warning);
    border-radius: var(--radius-md);
    padding: 0.875rem 1rem;
  }

  .utxo-notice strong { color: var(--warning); }
  .utxo-notice code {
    font-family: var(--font-mono);
    font-size: 0.75rem;
    color: var(--text-accent);
    background: var(--accent-dim);
    padding: 0.1rem 0.3rem;
    border-radius: 3px;
  }

  .utxo-table { display: flex; flex-direction: column; }

  .utxo-row {
    display: flex;
    align-items: baseline;
    gap: 1rem;
    padding: 0.625rem 0.75rem;
    border-bottom: 1px solid var(--border-subtle);
    flex-wrap: wrap;
  }

  .utxo-row:last-child { border-bottom: none; }

  .utxo-main { display: flex; align-items: center; gap: 0.625rem; min-width: 140px; }
  .utxo-name { font-size: 0.85rem; font-weight: 600; color: var(--text-secondary); }
  .utxo-sym  { font-size: 0.72rem; font-family: var(--font-mono); color: var(--text-muted); }
  .utxo-desc { font-size: 0.78rem; color: var(--text-muted); line-height: 1.5; }

  /* Nota técnica */
  .tech-note {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-left: 3px solid var(--accent);
    border-radius: var(--radius-md);
    padding: 1.25rem;
  }

  .tech-title {
    font-size: 0.72rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-muted);
    margin-bottom: 0.75rem;
  }

  .tech-note ol {
    list-style: none;
    counter-reset: step;
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
  }

  .tech-note li {
    counter-increment: step;
    font-size: 0.8rem;
    color: var(--text-muted);
    padding-left: 1.5rem;
    position: relative;
    line-height: 1.55;
  }

  .tech-note li::before {
    content: counter(step);
    position: absolute;
    left: 0;
    font-size: 0.65rem;
    font-weight: 800;
    font-family: var(--font-mono);
    color: var(--accent);
    top: 0.05rem;
  }

  .tech-note code {
    font-family: var(--font-mono);
    font-size: 0.75rem;
    color: var(--text-accent);
    background: var(--accent-dim);
    padding: 0.1rem 0.3rem;
    border-radius: 3px;
  }

  .tech-note strong { color: var(--warning); }

  /* Botones */
  .btn-primary {
    background: var(--accent); color: #fff; border: none;
    padding: 0.7rem 1.75rem; border-radius: var(--radius-md);
    font-size: 0.9rem; font-weight: 600; cursor: pointer;
    transition: background 0.15s, box-shadow 0.15s;
    font-family: var(--font-sans);
  }
  .btn-primary:hover { background: var(--accent-hover); box-shadow: 0 0 0 3px var(--accent-dim); }

  /* Spinner */
  .spinner {
    width: 12px; height: 12px;
    border: 2px solid rgba(255,255,255,0.2);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    display: inline-block;
  }
  @keyframes spin { to { transform: rotate(360deg); } }

  /* Responsive tabla */
  @media (max-width: 640px) {
    .net-table-head { display: none; }
    .net-row {
      grid-template-columns: 1fr 1fr;
      grid-template-rows: auto auto;
    }
    .net-chainid { display: none; }
  }
</style>
