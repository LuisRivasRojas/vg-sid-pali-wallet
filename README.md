# SID - Pali Wallet Demo

**Actividad Individual — Sistemas Distribuidos (SID)**

Demo práctica con Svelte que se conecta a Pali Wallet para iniciar sesión, leer el address y el saldo de la wallet.

---

## Cómo correr el proyecto

```bash
npm install
npm run dev
```

Abrir en el navegador: `http://localhost:5173`

> Requiere tener **Pali Wallet** instalada como extensión en Chrome.  
> Descarga: https://pali.syscoin.org

---

## Tecnologías usadas

- **Svelte 5** — framework JavaScript reactivo
- **Vite** — bundler y servidor de desarrollo
- **Pali Wallet** — wallet Web3 compatible con EIP-1193 (`window.ethereum`)
- **EIP-1193 Provider API** — estándar para comunicarse con wallets desde el navegador

---

## ¿Cómo funciona la conexión?

1. La app detecta `window.ethereum` inyectado por Pali Wallet en el navegador
2. Usa `eth_requestAccounts` para solicitar permiso y obtener el address
3. Usa `eth_getBalance` para obtener el saldo en wei (hexadecimal) y lo convierte a ETH
4. Escucha eventos `accountsChanged` y `chainChanged` para mantener la UI actualizada

---

## Funcionalidades

- ✅ Iniciar sesión con Pali Wallet
- ✅ Leer y mostrar el address de la wallet
- ✅ Leer y mostrar el saldo en ETH
- ✅ Mostrar la red conectada
- ✅ Copiar address al portapapeles
- ✅ Actualizar saldo manualmente
- ✅ Desconectar wallet

---

## Parte 1: Investigación — Casos de uso de Smart Contracts en dApps

### Caso 1: Uniswap (DeFi — Exchange Descentralizado)

**¿Qué es?**  
Uniswap es un protocolo de intercambio descentralizado (DEX) que opera sobre Ethereum y otras redes EVM compatibles.

**¿Cómo usa Smart Contracts?**  
En lugar de un libro de órdenes tradicional, Uniswap usa un modelo llamado **Automated Market Maker (AMM)**. Los smart contracts gestionan "pools de liquidez" donde los usuarios depositan pares de tokens. El contrato calcula automáticamente el precio de intercambio usando la fórmula `x * y = k`.

**¿Por qué es relevante?**
- No existe una empresa central que custodie los fondos
- Los intercambios se ejecutan directamente entre el usuario y el contrato
- Cualquier persona puede proveer liquidez y ganar comisiones
- Opera 24/7 sin intermediarios
- Tiene más de $3 billones en volumen de trading diario en sus mejores días

**Contrato principal:** `UniswapV3Factory`, `SwapRouter` — desplegados en Ethereum Mainnet  
**URL:** https://uniswap.org

---

### Caso 2: Aave (DeFi — Préstamos y Depósitos Descentralizados)

**¿Qué es?**  
Aave es un protocolo de lending descentralizado que permite a los usuarios depositar criptomonedas para ganar intereses, o tomar préstamos usando sus activos como colateral.

**¿Cómo usa Smart Contracts?**  
Los smart contracts de Aave actúan como el "banco" del sistema. Cuando depositas ETH, el contrato emite tokens `aETH` que representan tu depósito más los intereses acumulados. Para tomar un préstamo, el contrato verifica automáticamente que tu colateral sea suficiente y ejecuta el préstamo sin necesidad de aprobación manual.

También implementa **Flash Loans**: préstamos sin colateral que se toman y devuelven en la misma transacción, usados en arbitraje y liquidaciones.

**¿Por qué es relevante?**
- Las tasas de interés se ajustan algorítmicamente según la oferta y demanda
- No existe un comité que apruebe o rechace préstamos
- Tiene más de $10 billones en valor bloqueado (TVL)
- Usado activamente en Perú y Latinoamérica como alternativa bancaria

**Contrato principal:** `LendingPool` — desplegado en Ethereum, Polygon, Avalanche  
**URL:** https://aave.com

---

## Oportunidad de dApp para el Perú

**Propuesta: Sistema de remesas descentralizadas Perú ↔ Exterior**

Millones de peruanos en el exterior envían dinero a sus familias. Las remesas tradicionales cobran entre 5-10% de comisión. Una dApp basada en smart contracts en Polygon (red de bajo costo) podría:
- Reducir las comisiones a menos del 1%
- Ejecutar transferencias en minutos en lugar de días
- Usar stablecoins (USDC) para evitar la volatilidad
- Dar acceso a personas no bancarizadas mediante solo un smartphone y una wallet
