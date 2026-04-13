---
name: agentapi
description: "One API suite for agent AI tools with pay-per-use on-chain payments. Generate images and video by paying with USDC on X Layer via x402 protocol."
version: "1.0.0"
author: "ClawEnjoyer"
homepage: https://github.com/EnjoyingClaw/agentapi
tags:
 - x402
 - payment
 - ai
 - image-generation
 - video-generation
 - xlayer
 - onchainos
 - hackathon
---

# AgentAPI

One API suite for agent AI tools with pay-per-use on-chain payments. Generate images and video by paying with USDC on X Layer via the x402 protocol.

**Hosted API:** `https://x402-image-paywall-production.up.railway.app`  
**Price:** 0.05 USDC / image · 0.20 USDC / video  
**Networks:** X Layer (gas-free) · Ethereum · Base · Arbitrum  
**Uniswap compatible:** Use `pay-with-any-token` skill on Base/Ethereum/Arbitrum  

---

## Pre-flight Checks

Before using this skill, ensure:

1. The `onchainos` CLI is installed and configured (`npx skills add okx/onchainos-skills`)
2. You have a funded wallet with USDC on X Layer (or another supported chain)
3. Your wallet is logged in (`onchainos wallet status`)

---

## Quick Start

### Step 1: Send a generate request (will return 402)

```bash
curl -X POST https://x402-image-paywall-production.up.railway.app/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "a futuristic city at sunset"}'
# → HTTP 402, PAYMENT-REQUIRED header contains payment details
```

### Step 2: Decode the payment requirement

```js
const payHeader = response.headers.get('payment-required');
const decoded = JSON.parse(Buffer.from(payHeader, 'base64').toString());
// decoded.accepts contains payment options for X Layer
```

### Step 3: Pay using onchainos

Make sure you are logged in (`onchainos wallet status`), then:

```bash
onchainos payment x402-pay \
  --accepts '[{"scheme":"exact","network":"eip155:196","amount":"50000","payTo":"0xe5a24a32eafa471845f658f95118dcdfcc9ecc2a","asset":"0x74b7f16337b8972027f6196a17a631ac6de26d22","maxTimeoutSeconds":300}]'
# → returns { signature, authorization }
```

### Step 4: Replay with payment proof

```bash
# The payment response gives you authorization + signature
# Construct the PAYMENT-SIGNATURE header with both accepted + payload
curl -X POST https://x402-image-paywall-production.up.railway.app/generate \
  -H "Content-Type: application/json" \
  -H "PAYMENT-SIGNATURE: <base64-encoded-payment-payload>" \
  -d '{"prompt": "a futuristic city at sunset"}'
# → { "success": true, "image_url": "https://...", "tx_hash": "0x..." }
```

---

## Commands

### Generate Image

```bash
curl -X POST https://x402-image-paywall-production.up.railway.app/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "your image description"}'
```

**When to use**: When user wants to generate an AI image.

**Output**: Returns HTTP 402 initially. After payment, returns `{ success, image_url, prompt, paid_by, network, amount_usdc, tx_hash }`

### Generate Video

```bash
curl -X POST https://x402-image-paywall-production.up.railway.app/generate-video \
  -H "Content-Type: application/json" \
  -d '{"prompt": "your video description"}'
```

**When to use**: When user wants to generate an AI video (2s, 1280x720, 24fps via Google Veo 3).

**Output**: Same as image, but with video_url.

### Health Check

```bash
curl https://x402-image-paywall-production.up.railway.app/health
```

**When to use**: To verify the API is running.

---

## Payment Details

The 402 response includes accepts for all supported chains:

| Chain | Network | USDC Address | Gas |
|-------|---------|-------------|-----|
| X Layer | eip155:196 | `0x74b7f16337b8972027f6196a17a631ac6de26d22` | Free |
| Ethereum | eip155:1 | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | ~$0.50 |
| Base | eip155:8453 | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` | ~$0.001 |
| Arbitrum | eip155:42161 | `0xaf88d065e77c8cC2239327C5EDb3A432268e5831` | ~$0.01 |

**Amount:** 50000 (= 0.05 USDC) for images · 200000 (= 0.20 USDC) for video  
**Scheme:** `exact` (EIP-3009 transferWithAuthorization)  

---

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| HTTP 402 Payment Required | Payment not included or invalid | Include valid PAYMENT-SIGNATURE header |
| "Invalid payment" | Signature expired or invalid | Re-sign with fresh authorization |
| "Insufficient payment amount" | Not enough USDC sent | Ensure wallet has sufficient balance |
| "Payment authorization expired" | nonce validBefore passed | Get fresh signature with new nonce |
| ECONNREFUSED | API endpoint down | Check health endpoint or try again later |

---

## Security Notices

- This skill performs on-chain transactions on X Layer and other EVM chains
- Requires wallet signing via onchainos TEE (keys never leave secure enclave)
- Payments are non-refundable — ensure the amount is correct before signing
- Always verify the tx_hash after payment for confirmation

---

## Self-Hosting

```bash
git clone https://github.com/EnjoyingClaw/agentapi
cd agentapi
cp .env.example .env   # fill in your keys
npm install
npm start
```

Required env vars:
- `EDEN_AI_KEY` — Eden AI API key (for image/video generation)
- `WALLET_ADDRESS` — Your EVM wallet to receive payments
- `PRICE_USDC` — Image price in minimal units (default: 50000 = 0.05 USDC)
- `PRICE_VIDEO_USDC` — Video price (default: 200000 = 0.20 USDC)
- `SERVER_PRIVATE_KEY` — Server wallet key for on-chain redemption
- `PORT` — Server port (default: 3000)

---

## Skill Routing

- For token swaps → use `okx-dex-swap` skill
- For wallet balances → use `okx-agentic-wallet` or `okx-wallet-portfolio` skill
- For security scanning → use `okx-security` skill
