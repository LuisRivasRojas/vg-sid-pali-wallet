<script>
  let { account = null, isConnected = false, onConnect } = $props();

  let toAddress  = $state('');
  let amount     = $state('');
  let txHash     = $state(null);
  let txStatus   = $state(null);   // 'pending' | 'success' | 'error'
  let txError    = $state(null);
  let isSending  = $state(false);
  let txHistory  = $state([]);
  let currentChainId = $state(null);

  // Mapa de exploradores por chainId
  const EXPLORERS = {
    1: 'https://etherscan.io/tx/',
    5: 'https://goerli.etherscan.io/tx/',
    11155111: 'https://sepolia.etherscan.io/tx/',
    137: 'https://polygonscan.com/tx/',
    80001: 'https://mumbai.polygonscan.com/tx/',
    80002: 'https://amoy.polygonscan.com/tx/',
    56: 'https://bscscan.com/tx/',
    97: 'https://testnet.bscscan.com/tx/',
    57: 'https://explorer.syscoin.org/tx/',
    5700: 'https://tanenbaum.io/tx/',
    570: 'https://explorer.rollux.com/tx/',
    57000: 'https://rollux.tanenbaum.io/tx/',
    43114: 'https://snowtrace.io/tx/',
    250: 'https://ftmscan.com/tx/',
    42161: 'https://arbiscan.io/tx/',
    10: 'https://optimistic.etherscan.io/tx/',
  };

  async function loadChainId() {
    try {
      const cid = await window.ethereum.request({ method: 'eth_chainId' });
      currentChainId = parseInt(cid, 16);
    } catch (_) {}
  }

  function explorerUrl(hash) {
    const base = EXPLORERS[currentChainId];
    return base ? base + hash : null;
  }

  function isValidAddress(addr) {
    return /^0x[0-9a-fA-F]{40}$/.test(addr);
  }

  function ethToWeiHex(val) {
    const wei = BigInt(Math.round(parseFloat(val) * 1e18));
    return '0x' + wei.toString(16);
  }

  async function sendTransaction() {
    txError = null; txHash = null; txStatus = null;

    if (!toAddress.trim()) { txError = 'Ingresa la dirección destino.'; return; }
    if (!isValidAddress(toAddress.trim())) { txError = 'Dirección inválida — debe comenzar con 0x y tener 42 caracteres.'; return; }
    if (toAddress.trim().toLowerCase() === account?.toLowerCase()) { txError = 'La dirección destino no puede ser la misma que la de origen.'; return; }
    if (!amount || parseFloat(amount) <= 0) { txError = 'Ingresa un monto mayor a 0.'; return; }

    isSending = true;
    txStatus = 'pending';

    try {
      await loadChainId();
      const hash = await window.ethereum.request({
        method: 'eth_sendTransaction',
        params: [{ from: account, to: toAddress.trim(), value: ethToWeiHex(amount) }],
      });

      txHash   = hash;
      txStatus = 'success';

      txHistory = [{
        hash,
        to:        toAddress.trim(),
        amount,
        chainId:   currentChainId,
        time:      new Date().toLocaleTimeString('es-PE'),
        explorerUrl: explorerUrl(hash),
      }, ...txHistory];

      toAddress = ''; amount = '';

    } catch (e) {
      txStatus = 'error';
      if (e.code === 4001)   txError = 'Transacción rechazada por el usuario.';
      else if (e.code === -32603) txError = 'Error interno: saldo insuficiente o gas inválido.';
      else txError = `Error: ${e.message}`;
    } finally {
      isSending = false;
    }
  }

  async function copyText(text) {
    await navigator.clipboard.writeText(text);
  }

  function shortAddr(addr) { return `${addr.slice(0,8)}...${addr.slice(-6)}`; }
  function shortHash(h)    { return `${h.slice(0,12)}...${h.slice(-8)}`; }
</script>

<div class="page">

  <!-- Encabezado de página -->
  <div class="page-header">
    <h2 class="page-title">Transacciones</h2>
    <p class="page-sub">Transferencia de cuenta a cuenta en la red activa</p>
  </div>

  {#if !isConnected}
    <div class="empty-state">
      <div class="empty-icon">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" width="32" height="32">
          <rect x="3" y="11" width="18" height="11" rx="2"/>
          <path d="M7 11V7a5 5 0 0110 0v4"/>
        </svg>
      </div>
      <h3>Wallet no conectada</h3>
      <p>Necesitas conectar tu Pali Wallet para enviar transacciones.</p>
      <button class="btn-primary" onclick={onConnect}>Conectar Pali Wallet</button>
    </div>

  {:else}

    <!-- Origen -->
    <div class="origin-bar">
      <span class="origin-label">Desde</span>
      <code class="origin-addr">{account}</code>
    </div>

    <!-- Formulario -->
    <div class="form-card">
      <div class="form-field">
        <label for="to-address" class="field-label">Dirección destino</label>
        <input
          id="to-address"
          type="text"
          bind:value={toAddress}
          placeholder="0x742d35Cc6634C0532925a3b8D4C9b4A..."
          class="field-input"
          class:field-invalid={toAddress && !isValidAddress(toAddress)}
          disabled={isSending}
        />
        {#if toAddress && !isValidAddress(toAddress)}
          <span class="field-error">Dirección inválida</span>
        {/if}
      </div>

      <div class="form-field">
        <label for="amount" class="field-label">Monto (moneda nativa de la red)</label>
        <div class="amount-wrap">
          <input
            id="amount"
            type="number"
            bind:value={amount}
            placeholder="0.001"
            min="0"
            step="0.0001"
            class="field-input"
            disabled={isSending}
          />
          <span class="amount-unit">ETH / SYS</span>
        </div>
      </div>

      {#if txError}
        <div class="alert-error">{txError}</div>
      {/if}

      <button
        class="btn-primary btn-full"
        onclick={sendTransaction}
        disabled={isSending || !toAddress || !amount}
      >
        {#if isSending}
          <span class="spinner"></span>
          Esperando confirmación de la wallet...
        {:else}
          Enviar transacción
        {/if}
      </button>
    </div>

    <!-- Resultado -->
    {#if txStatus === 'success' && txHash}
      <div class="result-card">
        <div class="result-header">
          <div class="result-badge success">Transacción enviada</div>
          <p class="result-hint">La transacción fue aceptada. Puede tardar unos segundos en confirmarse.</p>
        </div>

        <div class="hash-block">
          <div class="hash-row-label">Hash de transacción</div>
          <div class="hash-row">
            <code class="hash-value hash-full">{txHash}</code>
            <code class="hash-value hash-short">{shortHash(txHash)}</code>
            <button class="btn-copy" onclick={() => copyText(txHash)} title="Copiar hash">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="14" height="14">
                <rect x="9" y="9" width="13" height="13" rx="2"/>
                <path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1"/>
              </svg>
            </button>
          </div>
        </div>

        {#if explorerUrl(txHash)}
          <a href={explorerUrl(txHash)} target="_blank" rel="noopener noreferrer" class="btn-explorer">
            Ver en explorador de bloques
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="13" height="13">
              <path d="M18 13v6a2 2 0 01-2 2H5a2 2 0 01-2-2V8a2 2 0 012-2h6"/>
              <polyline points="15 3 21 3 21 9"/>
              <line x1="10" y1="14" x2="21" y2="3"/>
            </svg>
          </a>
        {:else}
          <p class="no-explorer-note">
            Busca el hash manualmente en el explorador de tu red.
          </p>
        {/if}
      </div>
    {/if}

    <!-- Historial -->
    {#if txHistory.length > 0}
      <div class="history-section">
        <div class="history-title">Historial de la sesión</div>
        <div class="history-list">
          {#each txHistory as tx}
            <div class="history-row">
              <div class="history-left">
                <div class="history-meta">
                  <span class="hlabel">Para</span>
                  <code>{shortAddr(tx.to)}</code>
                </div>
                <div class="history-meta">
                  <span class="hlabel">Hash</span>
                  <code class="hmuted">{shortHash(tx.hash)}</code>
                </div>
              </div>
              <div class="history-right">
                <span class="history-amount">{tx.amount}</span>
                <span class="history-time">{tx.time}</span>
                <div class="history-actions">
                  <button class="btn-icon-sm" onclick={() => copyText(tx.hash)} title="Copiar hash">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="12" height="12">
                      <rect x="9" y="9" width="13" height="13" rx="2"/>
                      <path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1"/>
                    </svg>
                  </button>
                  {#if tx.explorerUrl}
                    <a href={tx.explorerUrl} target="_blank" rel="noopener noreferrer" class="btn-icon-sm" title="Ver en explorador">
                      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="12" height="12">
                        <path d="M18 13v6a2 2 0 01-2 2H5a2 2 0 01-2-2V8a2 2 0 012-2h6"/>
                        <polyline points="15 3 21 3 21 9"/>
                        <line x1="10" y1="14" x2="21" y2="3"/>
                      </svg>
                    </a>
                  {/if}
                </div>
              </div>
            </div>
          {/each}
        </div>
      </div>
    {/if}

    <!-- Nota técnica -->
    <div class="tech-note">
      <div class="tech-note-title">Notas técnicas</div>
      <ul>
        <li>La transacción se firma localmente en Pali Wallet. Las claves privadas nunca salen del dispositivo.</li>
        <li>El <strong>hash</strong> es el identificador único e inmutable de la transacción en la blockchain.</li>
        <li>El gas fee es estimado y gestionado automáticamente por Pali Wallet según el estado de la red.</li>
        <li>Usa el hash para rastrear el estado (pendiente / confirmado) en el explorador de bloques.</li>
      </ul>
    </div>

  {/if}

</div>

<style>
  .page { display: flex; flex-direction: column; gap: 1.25rem; }

  /* Encabezado */
  .page-header { margin-bottom: 0.25rem; }
  .page-title  { font-size: 1.5rem; font-weight: 700; color: var(--text-primary); letter-spacing: -0.02em; }
  .page-sub    { font-size: 0.85rem; color: var(--text-muted); margin-top: 0.25rem; }

  /* Empty state */
  .empty-state {
    text-align: center;
    padding: 4rem 2rem;
    background: var(--bg-surface);
    border: 1px dashed var(--border-default);
    border-radius: var(--radius-xl);
  }

  .empty-icon {
    width: 64px;
    height: 64px;
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-lg);
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 1.25rem;
    color: var(--text-muted);
  }

  .empty-state h3 { font-size: 1.1rem; color: var(--text-primary); margin-bottom: 0.4rem; }
  .empty-state p  { font-size: 0.875rem; color: var(--text-muted); margin-bottom: 1.5rem; }

  /* Origen */
  .origin-bar {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.625rem 1rem;
    background: var(--success-dim);
    border: 1px solid rgba(34,197,94,0.18);
    border-radius: var(--radius-md);
    flex-wrap: wrap;
  }

  .origin-label {
    font-size: 0.68rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--success);
  }

  .origin-addr {
    font-family: var(--font-mono);
    font-size: 0.78rem;
    color: var(--text-accent);
    word-break: break-all;
  }

  /* Formulario */
  .form-card {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-xl);
    padding: 1.75rem;
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
  }

  .form-field { display: flex; flex-direction: column; gap: 0.45rem; }

  .field-label {
    font-size: 0.8rem;
    font-weight: 600;
    color: var(--text-secondary);
    letter-spacing: 0.01em;
  }

  .field-input {
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-md);
    padding: 0.7rem 1rem;
    color: var(--text-primary);
    font-size: 0.875rem;
    transition: border-color 0.15s, box-shadow 0.15s;
    width: 100%;
  }

  .field-input:focus {
    outline: none;
    border-color: var(--accent);
    box-shadow: 0 0 0 3px var(--accent-dim);
  }

  .field-input::placeholder { color: var(--text-muted); }
  .field-input:disabled { opacity: 0.5; cursor: not-allowed; }
  .field-input.field-invalid { border-color: var(--danger); }

  .field-error {
    font-size: 0.75rem;
    color: #fca5a5;
  }

  .amount-wrap { position: relative; }
  .amount-wrap .field-input { padding-right: 80px; }

  .amount-unit {
    position: absolute;
    right: 1rem;
    top: 50%;
    transform: translateY(-50%);
    font-size: 0.72rem;
    font-weight: 700;
    color: var(--text-muted);
    pointer-events: none;
    letter-spacing: 0.04em;
  }

  .alert-error {
    padding: 0.75rem 1rem;
    background: var(--danger-dim);
    border: 1px solid rgba(239,68,68,0.22);
    border-radius: var(--radius-md);
    font-size: 0.82rem;
    color: #fca5a5;
    line-height: 1.5;
  }

  /* Botones */
  .btn-primary {
    background: var(--accent);
    color: #fff;
    border: none;
    padding: 0.7rem 1.75rem;
    border-radius: var(--radius-md);
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    transition: background 0.15s, box-shadow 0.15s;
    font-family: var(--font-sans);
  }

  .btn-primary:hover:not(:disabled) {
    background: var(--accent-hover);
    box-shadow: 0 0 0 3px var(--accent-dim);
  }

  .btn-primary:disabled { opacity: 0.5; cursor: not-allowed; }
  .btn-full { width: 100%; justify-content: center; }

  /* Resultado */
  .result-card {
    background: var(--bg-surface);
    border: 1px solid rgba(34,197,94,0.2);
    border-radius: var(--radius-xl);
    padding: 1.75rem;
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
  }

  .result-header { display: flex; flex-direction: column; gap: 0.4rem; }

  .result-badge {
    display: inline-block;
    font-size: 0.7rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    padding: 0.25rem 0.7rem;
    border-radius: 20px;
    width: fit-content;
  }

  .result-badge.success {
    background: var(--success-dim);
    color: var(--success);
    border: 1px solid rgba(34,197,94,0.2);
  }

  .result-hint { font-size: 0.8rem; color: var(--text-muted); }

  /* Hash */
  .hash-block {
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-md);
    padding: 1rem;
  }

  .hash-row-label {
    font-size: 0.65rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: var(--text-muted);
    margin-bottom: 0.5rem;
  }

  .hash-row {
    display: flex;
    align-items: center;
    gap: 0.625rem;
    flex-wrap: wrap;
  }

  .hash-value {
    font-family: var(--font-mono);
    font-size: 0.78rem;
    color: var(--text-accent);
    flex: 1;
    word-break: break-all;
  }

  .hash-full { display: block; }
  .hash-short { display: none; }

  @media (max-width: 540px) {
    .hash-full  { display: none; }
    .hash-short { display: block; }
  }

  .btn-copy {
    background: var(--bg-overlay);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-sm);
    padding: 0.35rem 0.5rem;
    cursor: pointer;
    color: var(--text-secondary);
    display: flex;
    align-items: center;
    transition: border-color 0.15s, color 0.15s;
    flex-shrink: 0;
  }

  .btn-copy:hover { border-color: var(--accent-glow); color: var(--text-accent); }

  .btn-explorer {
    display: inline-flex;
    align-items: center;
    gap: 0.4rem;
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    color: var(--text-secondary);
    padding: 0.55rem 1rem;
    border-radius: var(--radius-md);
    font-size: 0.82rem;
    font-weight: 500;
    text-decoration: none;
    transition: border-color 0.15s, color 0.15s;
    width: fit-content;
  }

  .btn-explorer:hover { border-color: var(--accent-glow); color: var(--text-primary); text-decoration: none; }

  .no-explorer-note {
    font-size: 0.8rem;
    color: var(--text-muted);
  }

  /* Historial */
  .history-section {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-xl);
    padding: 1.5rem;
  }

  .history-title {
    font-size: 0.75rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-muted);
    margin-bottom: 1rem;
  }

  .history-list { display: flex; flex-direction: column; }

  .history-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    padding: 0.75rem 0;
    border-bottom: 1px solid var(--border-subtle);
    flex-wrap: wrap;
  }

  .history-row:last-child { border-bottom: none; }

  .history-left  { display: flex; flex-direction: column; gap: 0.25rem; }
  .history-right { display: flex; align-items: center; gap: 0.75rem; }

  .history-meta  { display: flex; align-items: center; gap: 0.4rem; }

  .hlabel {
    font-size: 0.65rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-muted);
  }

  .history-meta code {
    font-family: var(--font-mono);
    font-size: 0.78rem;
    color: var(--text-accent);
  }

  .hmuted { color: var(--text-muted) !important; }

  .history-amount {
    font-size: 0.875rem;
    font-weight: 700;
    color: var(--success);
  }

  .history-time {
    font-size: 0.72rem;
    color: var(--text-muted);
  }

  .history-actions { display: flex; gap: 0.3rem; }

  .btn-icon-sm {
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-sm);
    padding: 0.3rem 0.4rem;
    cursor: pointer;
    color: var(--text-muted);
    display: flex;
    align-items: center;
    text-decoration: none;
    transition: border-color 0.15s, color 0.15s;
  }

  .btn-icon-sm:hover { border-color: var(--border-default); color: var(--text-secondary); }

  /* Nota técnica */
  .tech-note {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-left: 3px solid var(--accent);
    border-radius: var(--radius-md);
    padding: 1.25rem;
  }

  .tech-note-title {
    font-size: 0.72rem;
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
    gap: 0.45rem;
  }

  .tech-note li {
    font-size: 0.8rem;
    color: var(--text-muted);
    padding-left: 1rem;
    position: relative;
    line-height: 1.55;
  }

  .tech-note li::before {
    content: '—';
    position: absolute;
    left: 0;
    color: var(--accent);
    font-weight: 700;
  }

  .tech-note strong { color: var(--text-secondary); }

  /* Spinner */
  .spinner {
    width: 14px; height: 14px;
    border: 2px solid rgba(255,255,255,0.25);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    display: inline-block;
    flex-shrink: 0;
  }

  @keyframes spin { to { transform: rotate(360deg); } }
</style>
