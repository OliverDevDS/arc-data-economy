# Arc Data Economy

> An open, autonomous AI agent marketplace running on Arc Testnet — where agents create, execute, audit, and settle data processing jobs onchain in USDC, with gasless micropayments via Circle Nanopayments.

**Live Dashboard:** https://oliverdevds.github.io/arc-data-economy/  
**Operator:** `0x19868832e52e6bf64aed577de060782a63210897`  
**Network:** Arc Testnet · Chain ID 5042002  
**Standards:** ERC-8004 (Agent Identity) · ERC-8183 (Agentic Commerce)  
**Hackathon:** [Agentic Economy on Arc — LabLab.ai](https://lablab.ai/ai-hackathons/nano-payments-arc)  
**Track:** 🤖 Agent-to-Agent Payment Loop

---

## What Is This?

Arc Data Economy is a fully autonomous ecosystem where AI agents operate a continuous data processing marketplace — without any human intervention between cycles. Jobs are created, executed by real AI (Groq LLaMA 3.3), audited, and settled onchain in USDC every ~90 seconds.

Every approved job triggers two payment layers:

1. **Circle Nanopayment (x402)** — executor pays JobFactory gaslessly via EIP-3009 authorization, settled offchain in batches by Circle Gateway. Cost: `$0.0001 × tokens_used`, well below the $0.01 threshold.
2. **Onchain settlement** — job lifecycle recorded on Arc Testnet via ERC-8183 AgenticCommerce contract (createJob → setBudget → approve → fund → submit → complete → reputation).

This proves that agent-to-agent commerce at sub-cent granularity is now economically viable — something impossible with traditional gas-per-transaction models.

---

## Why This Would Fail Without Nanopayments

With traditional onchain gas:

- A $0.0001 payment would cost more in gas than its own value (gas overhead: 1,000%–5,000% on most chains).
- High-frequency agent loops (1 payment per job, every 90s) would be economically destroyed by gas costs.
- Even on low-cost chains, batching and per-token pricing are impractical without offchain aggregation.

With Circle Nanopayments:
- Each payment signs an EIP-3009 authorization **offchain** — zero gas per payment.
- Circle Gateway aggregates hundreds of payments into a single onchain batch.
- The per-transaction gas cost drops to effectively zero for the agent.
- True pay-per-token billing at `$0.0001/token` becomes viable at scale.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Arc Data Economy v2                         │
│                                                                 │
│  JobFactory-v1 (Producer)                                       │
│       │                                                         │
│       │  1. Generates job input via Groq                        │
│       │  2. Starts x402 server (Express + Circle Gateway)       │
│       ▼                                                         │
│  Groq LLaMA 3.3-70b (Executor — real AI processing)            │
│       │                                                         │
│       │  translate / summarize / extract /                      │
│       │  clean-csv / classify / validate-schema                 │
│       ▼                                                         │
│  Auditor-v1 (Validator — job-type-specific checks)             │
│       │                                                         │
│       │  ✅ APPROVED → two payment flows:                       │
│       │                                                         │
│       ├──▶ Circle Nanopayment (x402 / EIP-3009)                │
│       │    └─ GatewayClient.pay() → gasless offchain            │
│       │       Circle Gateway settles in batch onchain           │
│       │                                                         │
│       └──▶ AgenticCommerce (ERC-8183 onchain settlement)        │
│            createJob → setBudget → approve → fund               │
│            → submit → complete → giveFeedback                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Circle Technologies Used

| Technology | How Used |
|---|---|
| **Arc Testnet** | All transactions settle on Arc (Chain ID 5042002, USDC as gas) |
| **USDC** | Native token for all payments and gas fees |
| **Circle Developer Wallets** | 5 agent wallets managed via Circle API |
| **Circle Nanopayments (x402)** | Per-token gasless billing between agents via EIP-3009 and Circle Gateway |
| **Circle Gateway** | GatewayWallet deposit + batched offchain settlement |

---

## Registered Agents (ERC-8004)

| Agent | ID | Role | Address |
|---|---|---|---|
| JobFactory-v1 | #1720 | Producer | `0xbb7c447ce2a48c592d72c0845e4d747946c39a97` |
| DataWrangler-v1 | #1721 | Executor | `0xad28062b28bc7d17a956cdfcaad6d85912d654aa` |
| DataWrangler-v2 | #1722 | Executor | `0x1dddea7735459ac23b8e22939b27cf4109f482b9` |
| Translator-v1 | #1723 | Executor | `0x4805452dbddf0cda1b250c3b126011de247176e1` |
| Auditor-v1 | #1724 | Auditor | `0xbdaa89f73e5eee812b493745b9b57c474abf3952` |

---

## Smart Contracts (Arc Testnet)

| Contract | Address | Standard |
|---|---|---|
| AgenticCommerce | `0x0747EEf0706327138c69792bF28Cd525089e4583` | ERC-8183 |
| AgentBidBoard | `0xFb72B52eaF2b1A2e0cf96F8eDA1386288fC74ad9` | Custom |
| IdentityRegistry | `0x8004A818BFB912233c491871b3d84c89A494BD9e` | ERC-8004 |
| ReputationRegistry | `0x8004B663056A597Dffe9eCcC1965A193B7388713` | Custom |
| GatewayWallet | `0x0077777d7EBA4688BDeF3E311b846F25870A19B9` | Circle Gateway |
| USDC | `0x3600000000000000000000000000000000000000` | ERC-20 |

---

## Economic Model

Each job cycle produces two types of payments:

**Nanopayment (gasless, per-token):**
```
payment = max(tokens_used × $0.0001, $0.001)
example: 1,000 tokens → $0.10 (well below $0.01/token threshold)
```

**Onchain settlement ($0.10 USDC per job):**
- Funded via escrow in AgenticCommerce
- Released to executor on audit approval
- Reputation score recorded onchain

**Transaction frequency:** 1 complete cycle every ~90 seconds, generating 7+ onchain transactions and 1 gasless nanopayment per cycle.

---

## Job Types

The ecosystem processes 6 real data tasks via Groq LLaMA 3.3-70b:

| Job Type | What It Does | Auditor Check |
|---|---|---|
| `translate` | Translates documents to target language | Script detection, length ratio |
| `summarize` | Compresses documents to key points | Length reduction, no repetition |
| `extract` | Extracts structured data from documents | JSON validity, required fields |
| `clean-csv` | Deduplicates and normalizes CSV data | Row count, format consistency |
| `classify` | Assigns category labels to content | Valid category, confidence |
| `validate-schema` | Validates JSON against schemas | EIP-3009/payment payload schemas |

---

## Nanopayments Integration (x402)

The v2 loop integrates Circle Gateway x402 for every approved job:

```typescript
// 1. JobFactory runs x402-protected Express server
const gateway = createGatewayMiddleware({
  sellerAddress: JOBFACTORY_ADDRESS,
  networks: ["eip155:5042002"], // Arc Testnet
});
app.post("/job/submit", gateway.require("$0.01"), handler);

// 2. Executor pays gaslessly after each approved job
const client = new GatewayClient({ chain: "arcTestnet", privateKey });
const { data, status } = await client.pay("http://localhost:4402/job/submit");
// → Signs EIP-3009 offchain, Circle settles in batch → zero gas
```

The executor's GatewayWallet balance is funded once onchain; all subsequent payments are gasless EIP-3009 authorizations aggregated by Circle Gateway.

---

## Live Stats

- **1,500+ completed job cycles** onchain (v1 loop)
- **Job ID 4290+** on AgenticCommerce contract (v2 loop)
- **5 active agents** under ERC-8004
- **6 job types** with real AI processing
- **$0.0001/token** nanopayment pricing
- **100/100** audit scores maintained
- **~90 second** average cycle time
- **Arc Block Explorer:** https://testnet.arcscan.app

---

## Repositories

| Repo | Description |
|---|---|
| [arc-data-economy](https://github.com/OliverDevDS/arc-data-economy) | Live dashboard (GitHub Pages) |
| [arc-agent-marketplace](https://github.com/OliverDevDS/arc-agent-marketplace) | Smart contracts |

---

## Circle Skills Contributions

PRs submitted to [circlefin/skills](https://github.com/circlefin/skills/pulls):

- PR #9, #10, #12, #14, #15, #17

---

## Running Locally

```bash
# Clone
git clone https://github.com/OliverDevDS/arc-data-economy
cd arc-data-economy

# Install
npm install

# Configure .env
GROQ_API_KEY=...
CIRCLE_API_KEY=...
CIRCLE_ENTITY_SECRET=...
DEPLOY_KEY=0x...   # For Circle Gateway x402 payments

# Run v2 loop (real AI + Nanopayments + onchain settlement)
npx tsx --env-file=.env arc-data-economy-v2.ts
```

**Requirements:** Node.js 18+, Arc Testnet USDC (from [Circle Faucet](https://faucet.circle.com)), Groq API key.

---

## Circle Product Feedback

**Products used:** Arc Testnet, USDC, Circle Developer Wallets, Circle Nanopayments (x402), Circle Gateway.

**What worked well:**
- `@circle-fin/x402-batching` SDK was well-designed — `GatewayClient.deposit()` and `.pay()` worked seamlessly on Arc Testnet after discovering the correct endpoint (`/v1/x402/` not `/gateway/v1/x402/`).
- Arc Testnet is listed in `/v1/x402/supported` (`eip155:5042002`) with correct GatewayWallet and USDC addresses.
- Developer Wallets API is rock-solid — 1,500+ transactions without a single failure.
- The `createGatewayMiddleware` Express integration is clean and production-ready.

**Issues encountered:**
- The `createGatewayMiddleware` initially returned 503 because `networks` config required the correct CAIP-2 format (`eip155:5042002`) — not the raw chain ID. Documentation could be clearer.
- The `GATEWAY_API_TESTNET` base URL in the SDK (`https://gateway-api-testnet.circle.com`) differs from the documented path prefix in some docs. Took time to discover `/v1/x402/` is the correct prefix.
- `DEPLOY_KEY` (raw private key) is required for `GatewayClient` alongside `CIRCLE_API_KEY` for Developer Wallets — having two separate key types for the same project adds friction. A unified auth flow would improve DX.

**Recommendations:**
- Add a step-by-step Arc Testnet x402 quickstart to the docs (most examples use Base Sepolia).
- Document the CAIP-2 network string format requirement explicitly in `createGatewayMiddleware` config.
- Consider a `GatewayClient` constructor that accepts a Circle entity secret instead of raw private key for projects already using Developer Wallets.

---

## Author

**OliverDevDS**  
GitHub: https://github.com/OliverDevDS  
Operator: `0x19868832e52e6bf64aed577de060782a63210897`

---

## License

MIT
