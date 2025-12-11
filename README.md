# Megabytes Name System (MNS)

Megabytes introduces a fully **on-chain name registration system**, where every name is unique and enforced directly by consensus.
Registered identities are immutable, permanent, and require no external servers or DNS layers.
This provides a cleaner user experience for wallets, exchanges, and merchants.
The system replaces unreadable base58/bech32 addresses, improving usability and helping broader adoption.
It enables sending funds to human-readable names in the future, while offering decentralized identity primitives with guaranteed uniqueness and censorship resistance.

A name such as **walletname** becomes linked to a wallet identity key, enabling user-friendly interactions.
```
mgbrt1qvphkue9tpls6lxza9mwx0fsfgj0gvtwd6hcu5y -> walletname
```

---

## Total cost to create a name
```
Registering an MNS name costs 1 MGB (burned) plus the normal network transaction fee.
```

Consensus validation enforces:

- Name format: `[a-z0-9]`, 3–32 characters  
- Names are **unique forever**  
- The transaction must burn **1 MGB minimum**  
- Once confirmed, the name becomes permanent  

---

## 4. RPC Commands

### 4.1 Register a new name

```
mns_register "walletname" "mgb address"
```

Example:

```
mns_register "walletname" "mgbrt1qvphkue9tpls6lxza9mwx0fsfgj0gvtwd6hcu5y"
```

Result:

```json
{
  "name": "walletname",
  "identity_pubkey": "02a34...",
  "identity_address": "mgbrt1qvph...",
  "burn": 1.00000000,
  "txid": "7ac9c2f3..."
}
```

---

### 4.2 List all registered names on chain

```
mns_list
```

Result:

```json
[
  {
    "name": "walletname",
    "identity_pubkey": "02a34...",
    "identity_address": "mgbrt1qvph...",
    "height": 1452,
    "txid": "7ac9c2f3...",
    "vout": 1
  }
]
```

---

### 4.3 List names owned by your wallet

```
mns_listmine
```

Result:

```json
[
  {
    "name": "walletname",
    "identity_pubkey": "02a34...",
    "identity_address": "mgb1qxyz...",
    "height": 1452,
    "txid": "7ac9c2f3...",
    "vout": 1
  }
]
```

---

### 4.4 Resolve a name

```
mns_resolve "walletname"
```

Result:

```json
{
  "found": true,
  "name": "walletname",
  "identity_pubkey": "02a34...",
  "identity_address": "mgbrt1qvph...",
  "height": 1452,
  "txid": "7ac9c2f3...",
  "vout": 1
}
```

If not found:

```json
{ "found": false }
```

---

## 5. Consensus Guarantees

- Names stored in consensus — **cannot be overwritten**
- Correct handling of chain reorganizations  
- Full index reloaded at startup  
- Duplicate names rejected by mempool and block validation  
- Normalization prevents collisions (`Walletname` → `walletname`)

---

## 7. Example Workflow

1. Register name  
2. Resolve name  
3. View owned names  

Example:

```
getnewaddress
mns_register "nickname" "<mgb address>"
mns_resolve "nickname"
mns_listmine
mns_list
```

