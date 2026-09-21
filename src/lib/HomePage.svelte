<script>
  let { account = null, isConnected = false, onConnect } = $props();
</script>

<div class="home">

  <!-- Hero -->
  <section class="hero">
    <div class="hero-label">Proyecto de Innovación Blockchain · SID 2024</div>
    <h1 class="hero-title">Pali Wallet <span class="hero-accent">DApp</span></h1>
    <p class="hero-desc">
      Aplicación descentralizada para interactuar con redes blockchain EVM mediante Pali Wallet.
      Envía transacciones, consulta saldos y gestiona redes desde una sola interfaz.
    </p>
    {#if isConnected}
      <div class="hero-status">
        <span class="status-dot"></span>
        Wallet conectada — <code>{account?.slice(0,6)}...{account?.slice(-4)}</code>
      </div>
    {:else}
      <button class="btn-primary" onclick={onConnect}>Conectar Pali Wallet</button>
    {/if}
  </section>

  <!-- Descripción -->
  <section class="card-section">
    <div class="section-header">
      <div class="section-tag">01</div>
      <h2>Descripción</h2>
    </div>
    <div class="prose">
      <p>
        Esta DApp es un proyecto académico individual desarrollado para el curso de
        <strong>Sistemas Distribuidos (SID)</strong> del Instituto Valle Grande.
        Permite interactuar directamente con redes blockchain compatibles con EVM a través de
        <strong>Pali Wallet</strong>, el proveedor oficial de Syscoin, implementando el estándar
        <strong>EIP-1193</strong> para comunicación con el proveedor inyectado en el navegador.
      </p>
      <p>
        La aplicación opera completamente en el lado del cliente, sin backend ni servidor intermediario.
        Toda la lógica de firma, consulta y gestión de redes se ejecuta localmente en el navegador
        a través de <code>window.ethereum</code>.
      </p>
    </div>
  </section>

  <!-- Objetivos -->
  <section class="card-section">
    <div class="section-header">
      <div class="section-tag">02</div>
      <h2>Objetivos</h2>
    </div>
    <div class="objectives-grid">
      <div class="obj-card">
        <div class="obj-number">01</div>
        <h4>General</h4>
        <p>Desarrollar una DApp funcional que demuestre la integración con redes blockchain EVM aplicando conceptos de sistemas distribuidos.</p>
      </div>
      <div class="obj-card">
        <div class="obj-number">02</div>
        <h4>Interoperabilidad</h4>
        <p>Soportar múltiples redes blockchain (UTXO y EVM) con cambio dinámico de red y adición automática de redes no configuradas.</p>
      </div>
      <div class="obj-card">
        <div class="obj-number">03</div>
        <h4>Transacciones</h4>
        <p>Ejecutar y monitorear transferencias de cuenta a cuenta capturando el hash para verificación en exploradores de bloques.</p>
      </div>
      <div class="obj-card">
        <div class="obj-number">04</div>
        <h4>Consulta de Saldos</h4>
        <p>Consultar el saldo de cualquier dirección en múltiples redes simultáneamente, con y sin sesión de wallet iniciada.</p>
      </div>
    </div>
  </section>

  <!-- Beneficios -->
  <section class="card-section">
    <div class="section-header">
      <div class="section-tag">03</div>
      <h2>Beneficios</h2>
    </div>
    <div class="benefits-list">
      {#each [
        { title: 'No custodial', desc: 'El usuario mantiene control total de sus claves privadas. La aplicación nunca almacena ni accede a información sensible.' },
        { title: 'Sin backend', desc: 'Toda la lógica se ejecuta en el navegador usando window.ethereum. No requiere servidor ni base de datos.' },
        { title: 'Multi-red', desc: 'Soporte para Syscoin, Ethereum, Polygon, BNB Chain, Avalanche y más. Las redes nuevas se añaden automáticamente.' },
        { title: 'Transparencia', desc: 'Cada transacción genera un hash único e inmutable verificable en exploradores de bloques públicos.' },
        { title: 'Consulta masiva', desc: 'Verifica el saldo de cualquier dirección en 11 redes EVM simultáneamente mediante llamadas RPC directas.' },
        { title: 'Descentralizado', desc: 'Demuestra en la práctica los principios de sistemas distribuidos: nodos, consenso e inmutabilidad.' },
      ] as b}
        <div class="benefit-row">
          <span class="benefit-title">{b.title}</span>
          <span class="benefit-sep">—</span>
          <span class="benefit-desc">{b.desc}</span>
        </div>
      {/each}
    </div>
  </section>

  <!-- Características -->
  <section class="card-section">
    <div class="section-header">
      <div class="section-tag">04</div>
      <h2>Características</h2>
    </div>
    <div class="features-grid">
      {#each [
        { label: 'Conexión con Pali Wallet', desc: 'Integración EIP-1193, detección automática y reconexión al recargar la página.' },
        { label: 'Envío de Transacciones', desc: 'Transferencias EVM cuenta a cuenta con validación de dirección y captura del hash.' },
        { label: 'Cambio de Red', desc: 'Soporte UTXO y EVM. Redes no configuradas se añaden automáticamente con wallet_addEthereumChain.' },
        { label: 'Consulta Simple', desc: 'Verifica el saldo de cualquier dirección en una red específica, con o sin sesión.' },
        { label: 'Consulta Multi-red', desc: 'Consulta simultánea en 11 redes EVM mediante Promise.allSettled y RPC públicos.' },
        { label: 'Interfaz Responsiva', desc: 'Diseño adaptable a escritorio y móvil con navegación lateral y por pestañas.' },
      ] as f, i}
        <div class="feature-item">
          <span class="feature-index">{String(i + 1).padStart(2, '0')}</span>
          <div>
            <div class="feature-label">{f.label}</div>
            <div class="feature-desc">{f.desc}</div>
          </div>
        </div>
      {/each}
    </div>
  </section>

  <!-- Desarrollador -->
  <section class="card-section">
    <div class="section-header">
      <div class="section-tag">05</div>
      <h2>Desarrollador</h2>
    </div>
    <div class="dev-card">
      <div class="dev-avatar">
        <span>LR</span>
      </div>
      <div class="dev-body">
        <div class="dev-name">Luis Rivas</div>
        <div class="dev-role">Desarrollador · Estudiante SID</div>
        <div class="dev-inst">Instituto Valle Grande · 2024</div>
        <div class="dev-tags">
          <span class="tag">Svelte 5</span>
          <span class="tag">EVM / Web3</span>
          <span class="tag">Pali Wallet</span>
          <span class="tag">EIP-1193</span>
          <span class="tag">JavaScript</span>
        </div>
        <p class="dev-bio">
          Proyecto individual para el curso de Sistemas Distribuidos.
          Enfocado en demostrar la integración de aplicaciones web con redes blockchain
          mediante el estándar EIP-1193 y Pali Wallet como proveedor.
        </p>
      </div>
    </div>
  </section>

</div>

<style>
  .home {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  /* ── Hero ────────────────────────────────────────────────────── */
  .hero {
    padding: 3rem 2.5rem;
    background: var(--bg-surface);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-xl);
    position: relative;
    overflow: hidden;
  }

  .hero::before {
    content: '';
    position: absolute;
    inset: 0;
    background: radial-gradient(ellipse 60% 50% at 70% 50%, rgba(79,110,247,0.07), transparent);
    pointer-events: none;
  }

  .hero-label {
    font-size: 0.72rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: var(--text-muted);
    margin-bottom: 1rem;
  }

  .hero-title {
    font-size: clamp(2rem, 5vw, 3rem);
    font-weight: 800;
    letter-spacing: -0.03em;
    line-height: 1.1;
    color: var(--text-primary);
    margin-bottom: 1rem;
  }

  .hero-accent {
    color: var(--accent);
  }

  .hero-desc {
    font-size: 1rem;
    color: var(--text-secondary);
    max-width: 520px;
    line-height: 1.7;
    margin-bottom: 2rem;
  }

  .hero-status {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.85rem;
    color: var(--text-secondary);
    background: var(--success-dim);
    border: 1px solid rgba(34,197,94,0.2);
    padding: 0.5rem 1rem;
    border-radius: var(--radius-md);
  }

  .status-dot {
    width: 7px;
    height: 7px;
    border-radius: 50%;
    background: var(--success);
    flex-shrink: 0;
  }

  .hero-status code {
    font-family: var(--font-mono);
    color: var(--text-accent);
    font-size: 0.78rem;
  }

  /* ── Secciones ──────────────────────────────────────────────── */
  .card-section {
    background: var(--bg-surface);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-xl);
    padding: 2rem;
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .section-header {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding-bottom: 1rem;
    border-bottom: 1px solid var(--border-subtle);
  }

  .section-tag {
    font-size: 0.65rem;
    font-weight: 800;
    letter-spacing: 0.1em;
    color: var(--accent);
    background: var(--accent-dim);
    border: 1px solid var(--accent-glow);
    padding: 0.25rem 0.6rem;
    border-radius: var(--radius-sm);
    font-family: var(--font-mono);
  }

  .section-header h2 {
    font-size: 1.1rem;
    font-weight: 700;
    color: var(--text-primary);
    letter-spacing: -0.01em;
  }

  /* ── Descripción ────────────────────────────────────────────── */
  .prose {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }

  .prose p {
    font-size: 0.9rem;
    color: var(--text-secondary);
    line-height: 1.75;
  }

  .prose strong { color: var(--text-primary); font-weight: 600; }

  .prose code {
    font-family: var(--font-mono);
    font-size: 0.82rem;
    color: var(--text-accent);
    background: var(--accent-dim);
    padding: 0.1rem 0.4rem;
    border-radius: 4px;
  }

  /* ── Objetivos ──────────────────────────────────────────────── */
  .objectives-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1rem;
  }

  .obj-card {
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-lg);
    padding: 1.25rem;
    transition: border-color 0.2s;
  }

  .obj-card:hover { border-color: var(--border-default); }

  .obj-number {
    font-size: 1.75rem;
    font-weight: 900;
    font-family: var(--font-mono);
    color: var(--accent-glow);
    line-height: 1;
    margin-bottom: 0.75rem;
  }

  .obj-card h4 {
    font-size: 0.85rem;
    font-weight: 700;
    color: var(--text-primary);
    margin-bottom: 0.4rem;
  }

  .obj-card p {
    font-size: 0.8rem;
    color: var(--text-muted);
    line-height: 1.6;
  }

  /* ── Beneficios ─────────────────────────────────────────────── */
  .benefits-list {
    display: flex;
    flex-direction: column;
    gap: 0;
  }

  .benefit-row {
    display: flex;
    align-items: baseline;
    gap: 0.75rem;
    padding: 0.75rem 0;
    border-bottom: 1px solid var(--border-subtle);
    font-size: 0.875rem;
  }

  .benefit-row:last-child { border-bottom: none; }

  .benefit-title {
    font-weight: 700;
    color: var(--text-primary);
    white-space: nowrap;
    min-width: 140px;
  }

  .benefit-sep {
    color: var(--text-muted);
    flex-shrink: 0;
  }

  .benefit-desc {
    color: var(--text-secondary);
    line-height: 1.5;
  }

  /* ── Características ────────────────────────────────────────── */
  .features-grid {
    display: flex;
    flex-direction: column;
    gap: 0;
  }

  .feature-item {
    display: flex;
    align-items: flex-start;
    gap: 1rem;
    padding: 1rem 0;
    border-bottom: 1px solid var(--border-subtle);
  }

  .feature-item:last-child { border-bottom: none; }

  .feature-index {
    font-size: 0.7rem;
    font-weight: 800;
    font-family: var(--font-mono);
    color: var(--accent);
    padding-top: 0.1rem;
    min-width: 24px;
  }

  .feature-label {
    font-size: 0.875rem;
    font-weight: 600;
    color: var(--text-primary);
    margin-bottom: 0.2rem;
  }

  .feature-desc {
    font-size: 0.8rem;
    color: var(--text-muted);
    line-height: 1.5;
  }

  /* ── Desarrollador ──────────────────────────────────────────── */
  .dev-card {
    display: flex;
    gap: 2rem;
    align-items: flex-start;
  }

  .dev-avatar {
    width: 72px;
    height: 72px;
    border-radius: 50%;
    background: var(--accent);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.25rem;
    font-weight: 800;
    color: #fff;
    flex-shrink: 0;
    letter-spacing: -0.02em;
    border: 2px solid var(--accent-glow);
  }

  .dev-body {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
    flex: 1;
  }

  .dev-name {
    font-size: 1.25rem;
    font-weight: 700;
    color: var(--text-primary);
    letter-spacing: -0.01em;
  }

  .dev-role {
    font-size: 0.85rem;
    color: var(--text-accent);
  }

  .dev-inst {
    font-size: 0.78rem;
    color: var(--text-muted);
    margin-bottom: 0.4rem;
  }

  .dev-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 0.35rem;
    margin-bottom: 0.5rem;
  }

  .tag {
    font-size: 0.7rem;
    font-weight: 600;
    color: var(--text-secondary);
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    padding: 0.2rem 0.6rem;
    border-radius: 20px;
  }

  .dev-bio {
    font-size: 0.83rem;
    color: var(--text-muted);
    line-height: 1.6;
  }

  /* Botón primario */
  .btn-primary {
    background: var(--accent);
    color: #fff;
    border: none;
    padding: 0.7rem 1.75rem;
    border-radius: var(--radius-md);
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.15s, box-shadow 0.15s;
    font-family: var(--font-sans);
  }

  .btn-primary:hover {
    background: var(--accent-hover);
    box-shadow: 0 0 0 3px var(--accent-dim);
  }

  @media (max-width: 600px) {
    .hero { padding: 2rem 1.25rem; }
    .card-section { padding: 1.25rem; }
    .dev-card { flex-direction: column; align-items: center; text-align: center; }
    .dev-tags { justify-content: center; }
    .benefit-title { min-width: unset; }
    .benefit-row { flex-direction: column; gap: 0.2rem; }
    .benefit-sep { display: none; }
  }
</style>
