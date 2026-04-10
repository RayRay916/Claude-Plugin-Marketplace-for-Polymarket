---
description: "Authentication guide (L1, L2, Builder credentials)"
---

# Polymarket CLOB API - Authentication

## Authentication Methods

### 1. API Key Authentication (Read-Only)
For public data access (orderbook, pricing, trades):

**Python**:
```python
from py_clob_client.client import ClobClient

client = ClobClient(
    host="https://clob.polymarket.com",
    key="YOUR_API_KEY",  # Optional for public endpoints
    chain_id=137
)
```

**TypeScript**:
```typescript
import { ClobClient } from '@polymarket/clob-client';

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  key: 'YOUR_API_KEY',  // Optional for public endpoints
  chainId: 137,
});
```

### 2. L2 Signature-Based Auth (Trading)
For order creation, cancellation, and account management:

**Python**:
```python
from py_clob_client.client import ClobClient
from py_clob_client.signer import Signer

# Create signer with private key
signer = Signer(private_key="YOUR_PRIVATE_KEY", chain_id=137)

# Initialize authenticated client
client = ClobClient(
    host="https://clob.polymarket.com",
    key="YOUR_API_KEY",
    chain_id=137,
    signer=signer
)

# Derive API credentials from wallet
api_creds = client.create_or_derive_api_key()
```

**TypeScript**:
```typescript
import { ClobClient } from '@polymarket/clob-client';
import { Wallet } from 'ethers';

// Create wallet signer
const wallet = new Wallet('YOUR_PRIVATE_KEY');

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  chainId: 137,
  signer: wallet,
});

// Derive API credentials
const apiCreds = await client.createOrDeriveApiKey();
```

### 3. Builder Credentials
For production applications requiring dedicated API access:

**Python**:
```python
client = ClobClient(
    host="https://clob.polymarket.com",
    key="BUILDER_API_KEY",
    secret="BUILDER_SECRET",
    chain_id=137
)
```

**TypeScript**:
```typescript
const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  key: 'BUILDER_API_KEY',
  secret: 'BUILDER_SECRET',
  chainId: 137,
});
```

## Wallet Setup

### Generate New Wallet

**Python**:
```python
from py_clob_client.signer import Signer
from eth_account import Account

# Generate new account
account = Account.create()
print(f"Address: {account.address}")
print(f"Private Key: {account.key.hex()}")

# Create signer
signer = Signer(private_key=account.key.hex(), chain_id=137)
```

**TypeScript**:
```typescript
import { Wallet } from 'ethers';

// Generate new wallet
const wallet = Wallet.createRandom();
console.log('Address:', wallet.address);
console.log('Private Key:', wallet.privateKey);
```

## API Key Management

### Create API Key
```python
# Python
api_key = client.create_api_key()
print(f"API Key: {api_key['apiKey']}")
print(f"Secret: {api_key['secret']}")
print(f"Passphrase: {api_key['passphrase']}")
```

```typescript
// TypeScript
const apiKey = await client.createApiKey();
console.log('API Key:', apiKey.apiKey);
console.log('Secret:', apiKey.secret);
console.log('Passphrase:', apiKey.passphrase);
```

### Derive API Key from Wallet
```python
# Python - derives consistent credentials from wallet signature
api_creds = client.create_or_derive_api_key()
```

```typescript
// TypeScript
const apiCreds = await client.createOrDeriveApiKey();
```

## Security Best Practices
1. **Never hardcode private keys** - Use environment variables
2. **Store API secrets securely** - Use secret management tools
3. **Rotate credentials regularly** - Especially for production apps
4. **Use read-only keys** when possible - Limit scope of credentials
5. **Test on testnet first** - Validate auth flow before mainnet

## Environment Variables Example

**Python (.env)**:
```bash
POLYGON_PRIVATE_KEY=0x...
POLYMARKET_API_KEY=your_api_key
POLYMARKET_SECRET=your_secret
POLYMARKET_PASSPHRASE=your_passphrase
```

```python
import os
from dotenv import load_dotenv

load_dotenv()

client = ClobClient(
    host="https://clob.polymarket.com",
    key=os.getenv("POLYMARKET_API_KEY"),
    secret=os.getenv("POLYMARKET_SECRET"),
    chain_id=137
)
```

**TypeScript (.env)**:
```bash
POLYGON_PRIVATE_KEY=0x...
POLYMARKET_API_KEY=your_api_key
POLYMARKET_SECRET=your_secret
POLYMARKET_PASSPHRASE=your_passphrase
```

```typescript
import * as dotenv from 'dotenv';
dotenv.config();

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  key: process.env.POLYMARKET_API_KEY,
  secret: process.env.POLYMARKET_SECRET,
  chainId: 137,
});
```

## Resources
- [Authentication Docs](https://docs.polymarket.com/developers/CLOB/authentication)
- [API Key Management](https://docs.polymarket.com/developers/CLOB/api-keys)
