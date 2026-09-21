# AS241S6 — Pali Wallet DApp

**Proyecto de Innovación Blockchain · Sistemas Distribuidos (SID)**  
**Instituto Valle Grande · 2024**  
**Desarrollador:** Luis Rivera

---

## Descripción

Aplicación descentralizada (DApp) desarrollada con Svelte 5 que permite interactuar con redes blockchain EVM a través de Pali Wallet. Implementa el estándar EIP-1193 para comunicación con el proveedor `window.ethereum`, sin requerir backend ni servidor intermediario.

---

## Funcionalidades implementadas

### Actividad 1 — Página de Inicio
Página de presentación del proyecto con las siguientes secciones:
- Descripción general del sistema
- Objetivos del proyecto
- Beneficios de la solución
- Características técnicas implementadas
- Card del desarrollador

### Actividad 2 — Transacciones de cuenta a cuenta
- Formulario de envío con validación de dirección y monto
- Firma mediante `eth_sendTransaction` a través de Pali Wallet
- Captura y visualización del hash de transacción
- Enlace directo al explorador de bloques según la red activa (Etherscan, Polygonscan, Syscoin Explorer, etc.)
- Historial de transacciones de la sesión activa

### Actividad 3 — Cambio de red por proveedor
- Soporte para 11 redes EVM: Ethereum, Sepolia, Polygon, BNB Chain, Syscoin, Tanenbaum, Rollux, Avalanche, Fantom, Arbitrum y Optimism
- Cambio de red mediante `wallet_switchEthereumChain`
- Adición automática de redes no configuradas mediante `wallet_addEthereumChain` (error 4902)
- Sección informativa de redes UTXO: Syscoin UTXO, Bitcoin, Litecoin y Dogecoin

### Actividad 4 — Consulta de saldo por address
- **Consulta simple:** saldo de cualquier dirección en una red específica, con o sin sesión iniciada
- **Consulta masiva:** saldo de una dirección en 11 redes EVM simultáneamente mediante `Promise.allSettled` y llamadas RPC directas

---

## Tecnologías

| Tecnología | Versión | Uso |
|---|---|---|
| Svelte | 5 | Framework frontend reactivo |
| Vite | 8 | Bundler y servidor de desarrollo |
| Pali Wallet | — | Proveedor EIP-1193 (`window.ethereum`) |
| EIP-1193 | — | Estándar de comunicación con wallets |

---

## Requisitos previos

- [Node.js](https://nodejs.org) v18 o superior
- [Pali Wallet](https://pali.syscoin.org) instalada como extensión en Chrome o Brave

---

## Instalación y ejecución

```bash
# Clonar el repositorio
git clone https://github.com/NOMBRE-ORG/AS241S6_06_PaliWalletDApp.git
cd AS241S6_06_PaliWalletDApp

# Instalar dependencias
npm install

# Iniciar servidor de desarrollo
npm run dev
```

Abrir en el navegador: `http://localhost:5173`

```bash
# Generar build de producción
npm run build

# Previsualizar el build
npm run preview
```

---

## Estructura del proyecto

```
src/
├── App.svelte              # Shell principal, navegación y estado global de wallet
├── app.css                 # Sistema de diseño (tokens CSS, reset global)
├── main.js                 # Punto de entrada
└── lib/
    ├── HomePage.svelte     # Actividad 1 — Página de inicio
    ├── TransactionPage.svelte  # Actividad 2 — Transacciones
    ├── NetworkPage.svelte  # Actividad 3 — Cambio de red
    └── BalancePage.svelte  # Actividad 4 — Consulta de saldo
```

---

## Flujo técnico

```
Usuario
  └── Pali Wallet (extensión de navegador)
        └── window.ethereum (EIP-1193 Provider)
              ├── eth_requestAccounts   → Conectar wallet
              ├── eth_getBalance        → Consultar saldo
              ├── eth_sendTransaction   → Enviar transacción
              ├── eth_chainId           → Obtener red activa
              ├── wallet_switchEthereumChain → Cambiar red
              └── wallet_addEthereumChain    → Añadir red nueva
```

---

## Redes soportadas

| Red | Chain ID | Tipo | Símbolo |
|---|---|---|---|
| Ethereum | 1 | EVM Mainnet | ETH |
| Sepolia | 11155111 | EVM Testnet | ETH |
| Polygon | 137 | EVM Mainnet | MATIC |
| BNB Smart Chain | 56 | EVM Mainnet | BNB |
| Syscoin | 57 | EVM Mainnet | SYS |
| Syscoin Tanenbaum | 5700 | EVM Testnet | tSYS |
| Rollux | 570 | L2 Mainnet | SYS |
| Avalanche | 43114 | EVM Mainnet | AVAX |
| Fantom | 250 | EVM Mainnet | FTM |
| Arbitrum One | 42161 | L2 Mainnet | ETH |
| Optimism | 10 | L2 Mainnet | ETH |

---

## Rama de trabajo

```
main      → rama principal / producción
develop   → rama de desarrollo activa
```

---

## Investigación — Casos de uso de Smart Contracts

### Caso 1: Uniswap — Exchange Descentralizado (DEX)

Uniswap es un protocolo de intercambio descentralizado que opera sobre Ethereum y redes EVM compatibles. En lugar de un libro de órdenes tradicional, utiliza el modelo **Automated Market Maker (AMM)**: los smart contracts gestionan pools de liquidez donde el precio se calcula automáticamente mediante la fórmula `x * y = k`. No existe una entidad central que custodie fondos; los intercambios se ejecutan directamente entre el usuario y el contrato, operando 24/7 sin intermediarios.

Contratos principales: `UniswapV3Factory`, `SwapRouter` — [uniswap.org](https://uniswap.org)

### Caso 2: Aave — Protocolo de Préstamos Descentralizados

Aave permite a los usuarios depositar criptomonedas para ganar intereses o tomar préstamos usando sus activos como colateral. Los smart contracts actúan como el núcleo del sistema: al depositar ETH, el contrato emite tokens `aETH` que acumulan intereses de forma automática. Las tasas se ajustan algorítmicamente según oferta y demanda, sin comités de aprobación. Implementa además **Flash Loans**: préstamos sin colateral que se emiten y devuelven en una sola transacción, utilizados en arbitraje y liquidaciones.

Contrato principal: `LendingPool` — [aave.com](https://aave.com)

### Propuesta para el Perú: Sistema de Remesas Descentralizadas

Las remesas tradicionales desde el exterior hacia Perú cobran entre 5% y 10% de comisión. Una DApp sobre Polygon con stablecoins (USDC) podría reducir esa comisión a menos del 1%, ejecutar transferencias en minutos y dar acceso a personas no bancarizadas mediante únicamente un smartphone y una wallet. Los smart contracts garantizarían la ejecución sin intermediarios y con total transparencia.

---

*Proyecto académico — Sistemas Distribuidos · Instituto Valle Grande · 2024*
