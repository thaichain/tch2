# ThaiChain Node v2

Full node configuration for **ThaiChain v2** — an EVM-compatible blockchain powered by [Tempo](https://github.com/tempoxyz/tempo).

## Network Details

| Parameter       | Value       |
|-----------------|-------------|
| Chain ID        | `7`         |
| Native Token    | TCH (TCH TOKEN) |
| Decimals        | 18          |
| Consensus       | Proof of Authority (PoA) |
| Epoch Length    | 302,400     |
| Base Fee        | 20 Gwei     |
| EVM Forks       | All enabled (Homestead → Prague) |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- At least **50 GB** of free disk space
- Ports `3000`, `3001`, `3002`, `3003`, `8545`, `8546` must be available

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/thaichain/node-v2.git
cd node-v2
```

### 2. Generate an enode key

Before starting the node, generate a P2P secret key:

```bash
# The node expects the key at config/enode.key
# You can generate one using any Ethereum key generation tool
# or let the node generate it on first run if it doesn't exist.
```

### 3. Download snapshot (recommended)

Instead of syncing from genesis, you can download a snapshot to get started quickly:

```bash
# Create data directory
mkdir -p data/full-node

# Download the latest snapshot (~22 seconds)
docker run --rm \
  -v ./data:/data \
  -v ./config:/config \
  ghcr.io/tempoxyz/tempo:latest \
  download \
  --manifest-url https://snapshot.thaichain.org/thaichain/manifest.json \
  --datadir /data/full-node \
  --chain /config/genesis.json \
  --skip-consensus=false \
  --force \
  --non-interactive
```

This downloads the execution layer state and consensus data from the latest snapshot, significantly reducing sync time.

### 4. Start the node

```bash
docker compose up -d
```

### 5. Verify the node is running

```bash
# Check logs
docker compose logs -f

# Check RPC endpoint
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

## Ports

| Port   | Purpose           |
|--------|-------------------|
| `3000` | Consensus P2P     |
| `3001` | Execution P2P     |
| `3002` | Metrics           |
| `3003` | Auth RPC          |
| `8545` | HTTP RPC          |
| `8546` | WebSocket RPC     |

## Directory Structure

```
node-v2/
├── config/
│   ├── genesis.json    # Chain genesis configuration
│   └── enode.key       # P2P secret key (generated)
├── data/               # Blockchain data (created at runtime)
├── docker-compose.yml  # Docker Compose configuration
└── README.md
```

## Configuration

### Genesis

The `config/genesis.json` contains the chain specification including:

- Pre-deployed system contracts (Multicall3, CREATE2 Deployer, Validator Registry)
- Native token configuration (TCH TOKEN)
- Validator set and PoA parameters

### Docker Compose

The node connects to the following trusted peers by default:

- `116.202.106.98:3001`
- `135.181.193.101:3001`

And follows the network via WebSocket at `ws://119.59.123.107:8546`.

## RPC Usage

### HTTP

```bash
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

### WebSocket

Connect to `ws://localhost:8546` for subscription-based interactions.

## Pre-deployed Contracts

| Address                                    | Contract              |
|--------------------------------------------|-----------------------|
| `0x000000000022d473030f116ddee9f6b43ac78ba3` | Permit2 (Uniswap)     |
| `0xca11bde05977b3631167028862be2a173976ca11` | Multicall3            |
| `0x4e59b44847b379578588920cA78FbF26c0B4956c` | CREATE2 Deployer      |
| `0xba5ed099633D3B313e4d5F7bdc1305d3c28BA5Ed` | Proxy Factory         |
| `0xcccccccc00000000000000000000000000000001` | Validator Registry    |
| `0x20c0000000000000000000000000000000000000` | Token Config          |
| `0xfeec000000000000000000000000000000000000` | Fee Config            |

## Troubleshooting

### Node won't start

- Ensure all required ports are free
- Check Docker logs: `docker compose logs`
- Verify `config/genesis.json` is present

### Can't connect to peers

- Ensure ports `3000` and `3001` are accessible from the internet
- Check firewall rules

### Disk space

Monitor the `./data` directory size. You may need to prune old data periodically.

## License

This project is licensed under the MIT License.

## Links

- [Tempo Client](https://github.com/tempoxyz/tempo)
- [ThaiChain GitHub](https://github.com/thaichain)
