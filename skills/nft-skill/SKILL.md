---
name: nft-skill
description: Base区块链上的自主AI艺术家代理，用于生成、进化、铸造、上架和推广NFT艺术作品。 当用户需要创作AI艺术、铸造ERC-721 NFT、市场上架、监控链上销售、触发艺术进化或在X/Twitter上发布空投时触发。
metadata:
  version: 1.1.0
  author: AI Artist
  license: MIT
  openclaw:
    emoji: 🎨
    homepage: https://github.com/Numba1ne/nft-skill
    requires:
      bins:
      - node
      - npm
    env:
      BASE_RPC_URL: ${BASE_RPC_URL}
      BASE_PRIVATE_KEY: ${BASE_PRIVATE_KEY}
      NFT_CONTRACT_ADDRESS: ${NFT_CONTRACT_ADDRESS}
      MARKETPLACE_ADDRESS: ${MARKETPLACE_ADDRESS}
      PINATA_API_KEY: ${PINATA_API_KEY}
      PINATA_SECRET: ${PINATA_SECRET}
      LLM_PROVIDER: ${LLM_PROVIDER}
      OPENROUTER_API_KEY: ${OPENROUTER_API_KEY}
      GROQ_API_KEY: ${GROQ_API_KEY}
      OLLAMA_BASE_URL: ${OLLAMA_BASE_URL}
      IMAGE_PROVIDER: ${IMAGE_PROVIDER}
      STABILITY_API_KEY: ${STABILITY_API_KEY}
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      X_CONSUMER_KEY: ${X_CONSUMER_KEY}
      X_CONSUMER_SECRET: ${X_CONSUMER_SECRET}
      X_ACCESS_TOKEN: ${X_ACCESS_TOKEN}
      X_ACCESS_SECRET: ${X_ACCESS_SECRET}
    install:
    - id: npm-install
      kind: shell
      command: cd {baseDir} && npm install && npm run build
      bins:
      - node
      label: Install NFT Skill dependencies
---

# NFT Skill for OpenClaw

Allows an OpenClaw agent 迁移到 autonomously 生成 art, mint NFTs, 列表 on marketplace, 监控 sales, evolve based on milestones, and post social updates.

## When 迁移到 Use This Skill

- User asks 迁移到 生成 AI art or procedural digital art
- User wants 迁移到 mint an NFT on Base
- User wants 迁移到 列表 an NFT for sale on the marketplace
- User wants 迁移到 监控 on-chain NFT sales
- User wants 迁移到 evolve art style after a sales milestone
- User wants 迁移到 tweet or announce a new NFT drop on X (Twitter)
- User mentions "NFT", "mint", "Base blockchain", "AI art", "digital art", or "marketplace listing"

## 设置 (First 运行)

Before first use, ensure the project is built:

```bash
cd {baseDir} && npm install && npm run build
```

The user 必须 populate a `.env` file with their keys:

```bash
cp {baseDir}/.env.example {baseDir}/.env
```

Required variables: `BASE_RPC_URL`, `BAS`BAS`_PRIVATE_KE`FT_CONTRAC``NFT_CONTRACT_ADDRESS`
`MARKETPLACE_ADDRESS`, `PINATA_API`PINATA_API`KEY`ECRET`, `PIN`ECRET`_CODE_3__DER``LLM_PROVIDER`

迁移到 部署 contracts (one-time 设置):

```bash
cd {baseDir} && npm run deploy:testnet   # Base Sepolia testnet
cd {baseDir} && npm run deploy:mainnet   # Base mainnet
```

Contract addresses are automatically written 迁移到 `.env` after deployment.

## Tools

All tools 输出 JSON. The agent 应该 look for the final line matching `{"status":"success",...}` or `{"status":"erro`{"status":"erro`",...}`

---

### 1. 生成 — 生成 Art

生成 new art and 上传 迁移到 IPFS.

```bash
cd {baseDir} && npm run cli -- generate --generation <number> --theme "<description>"
```

**参数:**

| Flag | Type | Required | 描述 |
|------|------|----------|-------------|
| `-g, --generation` | number | yes | Generation number (determines evolution state) |
| `-t, --theme` | string | yes | Art theme 描述 sent 迁移到 LLM |

**输出:**
```json
{"status": "success", "result": {"imagePath": "...", "metadata": {...}, "metadataUri": "Qm..."}}
```

**示例:**
```bash
cd {baseDir} && npm run cli -- generate --generation 1 --theme "neon cyberpunk city"
```

---

### 2. mint — Mint NFT

Mint a new ERC721 token on Base with an IPFS metadata URI.

```bash
cd {baseDir} && npm run cli -- mint --metadata-uri <uri>
```

**参数:**

| Flag | Type | Required | 描述 |
|------|------|----------|-------------|
| `-m, --metadata-uri` | string | yes | IPFS metadata URI (e.g. `Qm...` or`Qm...``Qm...`CODE_3__

**输出:**
```json
{"status": "success", "result": {"tokenId": "1", "txHash": "0x...", "blockNumber": 12345, "gasUsed": "80000"}}
```

**示例:**
```bash
cd {baseDir} && npm run cli -- mint --metadata-uri QmXyz123abc
```

---

### 3. 列表 — 列表 NFT on Marketplace

列表 a minted NFT for sale on the marketplace.

```bash
cd {baseDir} && npm run cli -- list --token-id <id> --price <eth>
```

**参数:**

| Flag | Type | Required | 描述 |
|------|------|----------|-------------|
| `-i, --token-id` | string | yes | Token ID 迁移到 列表 |
| `-p, --price` | string | yes | Listing price in ETH (e.g. `"0`"0`05"`

**输出:**
```json
{"status": "success", "result": {"success": true, "price": "0.05", "txHash": "0x..."}}
```

**示例:**
```bash
cd {baseDir} && npm run cli -- list --token-id 1 --price 0.05
```

---

### 4. 监控 — 监控 Sales

监视 for sales events in real-time. Streams JSON 迁移到 stdout until interrupted (Ctrl+C).

```bash
cd {baseDir} && npm run cli -- monitor [--from-block <number>]
```

**参数:**

| Flag | Type | Required | 描述 |
|------|------|----------|-------------|
| `-f, --from-block` | number | no | Replay missed sales from this block before live monitoring |

**输出 (per sale):**
```json
{"status": "sale", "result": {"buyer": "0x...", "tokenId": "1", "price": "0.05", "txHash": "0x...", "blockNumber": 12345}}
```

**示例:**
```bash
cd {baseDir} && npm run cli -- monitor --from-block 12000000
```

---

### 5. evolve — Evolve Agent

Trigger the evolution logic when sales milestones are met.

```bash
cd {baseDir} && npm run cli -- evolve --proceeds <eth> --generation <number> --trigger "<reason>"
```

**参数:**

| Flag | Type | Required | 描述 |
|------|------|----------|-------------|
| `-p, --proceeds` | string | yes | Total ETH proceeds earned so far |
| `-g, --generation` | number | yes | Current generation number |
| `--trigger` | string | yes | Human-readable reason for evolution |

**输出:**
```json
{"status": "success", "result": {"previousGeneration": 1, "newGeneration": 2, "improvements": [...], "newAbilities": [...]}}
```

**示例:**
```bash
cd {baseDir} && npm run cli -- evolve --proceeds "0.5" --generation 1 --trigger "Sold 3 NFTs"
```

---

### 6. tweet — Post 迁移到 X

Post an 更新 迁移到 X (Twitter).

```bash
cd {baseDir} && npm run cli -- tweet --content "<text>"
```

**参数:**

| Flag | Type | Required | 描述 |
|------|------|----------|-------------|
| `-c, --content` | string | yes | Tweet text (auto-truncated 迁移到 280 chars) |

**输出:**
```json
{"status": "success", "result": "tweet_id_string"}
```

**示例:**
```bash
cd {baseDir} && npm run cli -- tweet --content "New AI art drop incoming! #AIArt #Base"
```

---

## Typical Workflow

A full autonomous cycle the agent 应该 follow:

1. **生成** art with a theme → 接收 metadata URI
2. **Mint** the NFT with that URI → 接收 token ID
3. **列表** the NFT on the marketplace at a price
4. **Tweet** about the new listing
5. **监控** sales for purchase events
6. **Evolve** when a sales milestone is reached
7. Repeat from step 1 with the new generation number

## 错误 Handling

- If a 命令 返回 `{"status":"error",...}`, 读取 the `message` fiel`message`ort `message`e user.
- Common issues: missing `.env` variables, insufficient wallet balance, network RPC errors.
- For wallet balance issues, suggest the user funds their Base wallet.
- For missing env vars, remind the user 迁移到 populate `{baseDir}/.env`.

## Environment Variables

| Variable | Required | 描述 |
|----------|----------|-------------|
| `BASE_RPC_URL` | yes | Base network RPC endpoint |
| `BASE_PRIVATE_KEY` | yes* | Wallet private key (or use `PRIVATE`PRIVATE`KEY_FILE`
| `PRIVATE_KEY_FILE` | no | Path 迁移到 file containing the private key (safer alternative 迁移到 env var) |
| `NFT_CONTRACT_ADDRESS` | yes | Deployed NFTArt contract address |
| `MARKETPLACE_ADDRESS` | yes | Deployed NFTMarketplace contract address |
| `PINATA_API_KEY` | yes | Pinata IPFS API key |
| `PINATA_SECRET` | yes | Pinata IPFS secret |
| `LLM_PROVIDER` | yes | `ope`ope`route__COD__CODE_`ollama``o`ollama`
| `LLM_MODEL` | no | Model ID override |
| `OPENROUTER_API_KEY` | if LLM_PROVIDER=openrouter | OpenRouter API key |
| `GROQ_API_KEY` | if LLM_PROVIDER=groq | Groq API key |
| `OLLAMA_BASE_URL` | if LLM_PROVIDER=ollama | Ollama base URL |
| `IMAGE_PROVIDER` | no | `stabi`stabi`ity`le__CO`dalle`_CODE_4__dural`fault) |
| `IMAGE_MODEL` | no | Image model override |
| `STABILITY_API_KEY` | if IMAGE_PROVIDER=stability | Stability AI key |
| `OPENAI_API_KEY` | if IMAGE_PROVIDER=dalle | OpenAI key for DALL-E |
| `X_CONSUMER_KEY` | for tweet | X API consumer key |
| `X_CONSUMER_SECRET` | for tweet | X API consumer secret |
| `X_ACCESS_TOKEN` | for tweet | X access token |
| `X_ACCESS_SECRET` | for tweet | X access token secret |
| `BASESCAN_API_KEY` | no | For contract verification on Basescan |