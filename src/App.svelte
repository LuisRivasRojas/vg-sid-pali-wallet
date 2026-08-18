<script>
  import { onMount } from 'svelte';

  // Estado de la aplicación
  let account = $state(null);
  let balance = $state(null);
  let isConnecting = $state(false);
  let error = $state(null);
  let isConnected = $state(false);
  let networkName = $state(null);

  // Verificar si Pali Wallet está instalada
  function isPaliWalletAvailable() {
    return typeof window !== 'undefined' && typeof window.ethereum !== 'undefined';
  }

  // Conectar con Pali Wallet
  async function connectWallet() {
    error = null;

    if (!isPaliWalletAvailable()) {
      error = 'Pali Wallet no está instalada. Por favor instálala desde pali.syscoin.org';
      return;
    }

    isConnecting = true;

    try {
      // Solicitar acceso a las cuentas
      const accounts = await window.ethereum.request({
        method: 'eth_requestAccounts'
      });

      if (accounts.length === 0) {
        error = 'No se encontraron cuentas. Asegúrate de tener una cuenta en Pali Wallet.';
        return;
      }

      account = accounts[0];
      isConnected = true;

      // Leer el saldo usando eth_getBalance
      await loadBalance(account);

      // Leer la red actual
      await loadNetwork();

      // Escuchar cambios de cuenta
      window.ethereum.on('accountsChanged', handleAccountsChanged);
      window.ethereum.on('chainChanged', handleChainChanged);

    } catch (err) {
      if (err.code === 4001) {
        error = 'Conexión rechazada. El usuario denegó el acceso a la wallet.';
      } else {
        error = `Error al conectar: ${err.message}`;
      }
    } finally {
      isConnecting = false;
    }
  }

  // Leer saldo
  async function loadBalance(address) {
    try {
      // eth_getBalance devuelve el saldo en wei (hexadecimal)
      const balanceWei = await window.ethereum.request({
        method: 'eth_getBalance',
        params: [address, 'latest']
      });

      // Convertir de wei (hex) a ETH
      const balanceBigInt = BigInt(balanceWei);
      const balanceEth = Number(balanceBigInt) / 1e18;
      balance = balanceEth.toFixed(6);
    } catch (err) {
      balance = 'Error al leer saldo';
    }
  }

  // Leer red
  async function loadNetwork() {
    try {
      const chainId = await window.ethereum.request({ method: 'eth_chainId' });
      const chainIdNum = parseInt(chainId, 16);
      const networks = {
        1: 'Ethereum Mainnet',
        5: 'Goerli Testnet',
        11155111: 'Sepolia Testnet',
        137: 'Polygon Mainnet',
        80001: 'Mumbai Testnet',
        57: 'Syscoin Mainnet',
        5700: 'Syscoin Testnet (Tanenbaum)',
      };
      networkName = networks[chainIdNum] || `Chain ID: ${chainIdNum}`;
    } catch (err) {
      networkName = 'Red desconocida';
    }
  }

  // Manejar cambio de cuenta
  function handleAccountsChanged(accounts) {
    if (accounts.length === 0) {
      disconnectWallet();
    } else {
      account = accounts[0];
      loadBalance(account);
    }
  }

  // Manejar cambio de red
  function handleChainChanged() {
    window.location.reload();
  }

  // Desconectar
  function disconnectWallet() {
    account = null;
    balance = null;
    networkName = null;
    isConnected = false;
    error = null;

    if (window.ethereum) {
      window.ethereum.removeListener('accountsChanged', handleAccountsChanged);
      window.ethereum.removeListener('chainChanged', handleChainChanged);
    }
  }

  // Formatear address para mostrar (0x1234...5678)
  function formatAddress(addr) {
    if (!addr) return '';
    return `${addr.slice(0, 6)}...${addr.slice(-4)}`;
  }

  // Copiar address al portapapeles
  async function copyAddress() {
    if (account) {
      await navigator.clipboard.writeText(account);
      alert('Address copiada al portapapeles');
    }
  }

  onMount(() => {
    // Verificar si ya hay una conexión activa al cargar
    if (isPaliWalletAvailable()) {
      window.ethereum.request({ method: 'eth_accounts' }).then(accounts => {
        if (accounts.length > 0) {
          account = accounts[0];
          isConnected = true;
          loadBalance(account);
          loadNetwork();
          window.ethereum.on('accountsChanged', handleAccountsChanged);
          window.ethereum.on('chainChanged', handleChainChanged);
        }
      });
    }
  });
</script>

<main>
  <div class="container">
    <!-- Header -->
    <div class="header">
      <div class="logo-area">
        <div class="wallet-icon">🔐</div>
        <div>
          <h1>Pali Wallet Demo</h1>
          <p class="subtitle">SID - Sistemas Distribuidos</p>
        </div>
      </div>

      {#if isConnected}
        <div class="status-badge connected">
          <span class="dot"></span> Conectado
        </div>
      {:else}
        <div class="status-badge disconnected">
          <span class="dot"></span> Desconectado
        </div>
      {/if}
    </div>

    <!-- Error -->
    {#if error}
      <div class="alert error">
        <span>⚠️</span>
        <p>{error}</p>
      </div>
    {/if}

    <!-- No conectado -->
    {#if !isConnected}
      <div class="connect-card">
        <div class="connect-illustration">🦊</div>
        <h2>Conecta tu Pali Wallet</h2>
        <p>Haz clic en el botón para iniciar sesión con tu wallet y ver tu address y saldo.</p>

        {#if !isPaliWalletAvailable()}
          <div class="alert warning">
            <span>⚠️</span>
            <p>Pali Wallet no detectada. Instálala desde <a href="https://pali.syscoin.org" target="_blank">pali.syscoin.org</a></p>
          </div>
        {/if}

        <button
          class="btn-connect"
          onclick={connectWallet}
          disabled={isConnecting}
        >
          {#if isConnecting}
            <span class="spinner"></span> Conectando...
          {:else}
            🔗 Conectar Pali Wallet
          {/if}
        </button>
      </div>

    <!-- Conectado -->
    {:else}
      <div class="wallet-cards">

        <!-- Card: Address -->
        <div class="card">
          <div class="card-header">
            <div class="card-icon">👤</div>
            <h3>Address de la Wallet</h3>
          </div>
          <div class="address-display">
            <code class="address-full">{account}</code>
            <code class="address-short">{formatAddress(account)}</code>
          </div>
          <button class="btn-copy" onclick={copyAddress}>
            📋 Copiar Address
          </button>
        </div>

        <!-- Card: Saldo -->
        <div class="card">
          <div class="card-header">
            <div class="card-icon">💰</div>
            <h3>Saldo</h3>
          </div>
          {#if balance !== null}
            <div class="balance-display">
              <span class="balance-amount">{balance}</span>
              <span class="balance-unit">ETH</span>
            </div>
          {:else}
            <p class="loading">Cargando saldo...</p>
          {/if}
          <button class="btn-refresh" onclick={() => loadBalance(account)}>
            🔄 Actualizar saldo
          </button>
        </div>

        <!-- Card: Red -->
        <div class="card">
          <div class="card-header">
            <div class="card-icon">🌐</div>
            <h3>Red Conectada</h3>
          </div>
          <div class="network-display">
            <span class="network-name">{networkName ?? 'Cargando...'}</span>
          </div>
        </div>

      </div>

      <!-- Botón desconectar -->
      <button class="btn-disconnect" onclick={disconnectWallet}>
        🔌 Desconectar
      </button>
    {/if}

    <!-- Footer informativo -->
    <div class="info-section">
      <h3>¿Cómo funciona?</h3>
      <div class="steps">
        <div class="step">
          <div class="step-num">1</div>
          <p>La app detecta <code>window.ethereum</code> inyectado por Pali Wallet en el navegador</p>
        </div>
        <div class="step">
          <div class="step-num">2</div>
          <p>Usa <code>eth_requestAccounts</code> para solicitar acceso y obtener el address</p>
        </div>
        <div class="step">
          <div class="step-num">3</div>
          <p>Usa <code>eth_getBalance</code> para leer el saldo en wei y lo convierte a ETH</p>
        </div>
      </div>
    </div>

  </div>
</main>

<style>
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  :global(body) {
    background: #0f0f1a;
    color: #e2e8f0;
    font-family: 'Segoe UI', system-ui, sans-serif;
    min-height: 100vh;
  }

  main {
    min-height: 100vh;
    padding: 2rem 1rem;
    background: linear-gradient(135deg, #0f0f1a 0%, #1a1a2e 50%, #16213e 100%);
  }

  .container {
    max-width: 800px;
    margin: 0 auto;
  }

  /* Header */
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 2rem;
    padding: 1.5rem;
    background: rgba(255,255,255,0.05);
    border-radius: 16px;
    border: 1px solid rgba(255,255,255,0.1);
  }

  .logo-area {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .wallet-icon {
    font-size: 2.5rem;
  }

  h1 {
    font-size: 1.5rem;
    font-weight: 700;
    background: linear-gradient(135deg, #6366f1, #8b5cf6);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .subtitle {
    font-size: 0.8rem;
    color: #64748b;
    margin-top: 0.2rem;
  }

  .status-badge {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.4rem 1rem;
    border-radius: 20px;
    font-size: 0.85rem;
    font-weight: 600;
  }

  .status-badge.connected {
    background: rgba(34, 197, 94, 0.15);
    color: #22c55e;
    border: 1px solid rgba(34, 197, 94, 0.3);
  }

  .status-badge.disconnected {
    background: rgba(100, 116, 139, 0.15);
    color: #94a3b8;
    border: 1px solid rgba(100, 116, 139, 0.3);
  }

  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: currentColor;
  }

  /* Alerts */
  .alert {
    display: flex;
    align-items: flex-start;
    gap: 0.75rem;
    padding: 1rem 1.25rem;
    border-radius: 12px;
    margin-bottom: 1.5rem;
    font-size: 0.9rem;
  }

  .alert.error {
    background: rgba(239, 68, 68, 0.1);
    border: 1px solid rgba(239, 68, 68, 0.3);
    color: #fca5a5;
  }

  .alert.warning {
    background: rgba(234, 179, 8, 0.1);
    border: 1px solid rgba(234, 179, 8, 0.3);
    color: #fde047;
    margin-top: 1rem;
  }

  .alert a {
    color: inherit;
    font-weight: 600;
  }

  /* Connect card */
  .connect-card {
    text-align: center;
    padding: 3rem 2rem;
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(255,255,255,0.08);
    border-radius: 20px;
    margin-bottom: 2rem;
  }

  .connect-illustration {
    font-size: 4rem;
    margin-bottom: 1rem;
  }

  .connect-card h2 {
    font-size: 1.5rem;
    margin-bottom: 0.75rem;
    color: #e2e8f0;
  }

  .connect-card p {
    color: #94a3b8;
    margin-bottom: 2rem;
    line-height: 1.6;
  }

  /* Botones */
  .btn-connect {
    background: linear-gradient(135deg, #6366f1, #8b5cf6);
    color: white;
    border: none;
    padding: 0.9rem 2.5rem;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    transition: all 0.2s ease;
  }

  .btn-connect:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(99, 102, 241, 0.4);
  }

  .btn-connect:disabled {
    opacity: 0.7;
    cursor: not-allowed;
  }

  .spinner {
    width: 16px;
    height: 16px;
    border: 2px solid rgba(255,255,255,0.3);
    border-top-color: white;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
    display: inline-block;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  /* Wallet cards */
  .wallet-cards {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1rem;
    margin-bottom: 1.5rem;
  }

  .card {
    background: rgba(255,255,255,0.04);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 16px;
    padding: 1.5rem;
    transition: border-color 0.2s;
  }

  .card:hover {
    border-color: rgba(99, 102, 241, 0.4);
  }

  .card-header {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    margin-bottom: 1rem;
  }

  .card-icon {
    font-size: 1.5rem;
  }

  .card-header h3 {
    font-size: 1rem;
    color: #94a3b8;
    font-weight: 500;
  }

  /* Address */
  .address-display {
    background: rgba(0,0,0,0.3);
    border-radius: 10px;
    padding: 0.75rem 1rem;
    margin-bottom: 1rem;
    word-break: break-all;
  }

  .address-full {
    font-size: 0.8rem;
    color: #a5b4fc;
    display: block;
  }

  .address-short {
    display: none;
    font-size: 1rem;
    color: #a5b4fc;
  }

  @media (max-width: 480px) {
    .address-full { display: none; }
    .address-short { display: block; }
  }

  .btn-copy {
    background: rgba(99, 102, 241, 0.15);
    color: #a5b4fc;
    border: 1px solid rgba(99, 102, 241, 0.3);
    padding: 0.5rem 1rem;
    border-radius: 8px;
    cursor: pointer;
    font-size: 0.85rem;
    transition: all 0.2s;
  }

  .btn-copy:hover {
    background: rgba(99, 102, 241, 0.3);
  }

  /* Balance */
  .balance-display {
    display: flex;
    align-items: baseline;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  .balance-amount {
    font-size: 2rem;
    font-weight: 700;
    color: #22c55e;
  }

  .balance-unit {
    font-size: 1rem;
    color: #64748b;
  }

  .loading {
    color: #64748b;
    font-style: italic;
    margin-bottom: 1rem;
  }

  .btn-refresh {
    background: rgba(34, 197, 94, 0.1);
    color: #86efac;
    border: 1px solid rgba(34, 197, 94, 0.25);
    padding: 0.5rem 1rem;
    border-radius: 8px;
    cursor: pointer;
    font-size: 0.85rem;
    transition: all 0.2s;
  }

  .btn-refresh:hover {
    background: rgba(34, 197, 94, 0.2);
  }

  /* Network */
  .network-display {
    background: rgba(0,0,0,0.3);
    border-radius: 10px;
    padding: 0.75rem 1rem;
  }

  .network-name {
    color: #fbbf24;
    font-weight: 600;
    font-size: 1rem;
  }

  /* Disconnect */
  .btn-disconnect {
    width: 100%;
    background: rgba(239, 68, 68, 0.1);
    color: #fca5a5;
    border: 1px solid rgba(239, 68, 68, 0.25);
    padding: 0.75rem;
    border-radius: 12px;
    cursor: pointer;
    font-size: 0.9rem;
    font-weight: 500;
    transition: all 0.2s;
    margin-bottom: 2rem;
  }

  .btn-disconnect:hover {
    background: rgba(239, 68, 68, 0.2);
  }

  /* Info section */
  .info-section {
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(255,255,255,0.07);
    border-radius: 16px;
    padding: 1.5rem;
  }

  .info-section h3 {
    font-size: 0.9rem;
    color: #64748b;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 1.25rem;
  }

  .steps {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .step {
    display: flex;
    align-items: flex-start;
    gap: 1rem;
  }

  .step-num {
    background: linear-gradient(135deg, #6366f1, #8b5cf6);
    color: white;
    width: 28px;
    height: 28px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.8rem;
    font-weight: 700;
    flex-shrink: 0;
  }

  .step p {
    color: #94a3b8;
    font-size: 0.875rem;
    line-height: 1.6;
    padding-top: 0.2rem;
  }

  .step code {
    background: rgba(99, 102, 241, 0.15);
    color: #a5b4fc;
    padding: 0.1rem 0.4rem;
    border-radius: 4px;
    font-size: 0.8rem;
  }
</style>
