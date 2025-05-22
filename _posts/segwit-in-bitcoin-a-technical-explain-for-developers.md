# Segregated Witness (SegWit) in Bitcoin: A Technical Explain for Developers

Segregated Witness (SegWit) is a Bitcoin protocol upgrade that fundamentally changes how transactions are structured and validated. Activated in August 2017 as a soft fork, SegWit addresses transaction malleability and increases block capacity by separating signature data (witness data) from transaction data. This guide dives into the technical details of SegWit, focusing on transaction formats, TXID calculation, and implications for developers.

---

## Key Changes in SegWit

1. **Transaction Malleability Fix**: Signature data is removed from the transaction ID (TXID) calculation, preventing third parties from altering TXIDs.
2. **Block Capacity Increase**: Witness data is stored separately, allowing more transactions per block.
3. **New Address Format**: Bech32 addresses (`bc1`) are introduced for SegWit transactions.

---

## Transaction Format: Pre-SegWit vs. SegWit

### Pre-SegWit Transaction Format (YAML)

```yaml
version: 1
inputs:
  - prev_tx_hash: "abc123..."
    prev_tx_index: 0
    script_sig: "3045022100... 03abc..."
    sequence: 0xffffffff
outputs:
  - value: 100000000
    script_pub_key: "76a914...88ac"
lock_time: 0

```

### SegWit Transaction Format (YAML)

```yaml
version: 1
marker: 0x00
flag: 0x01
inputs:
  - prev_tx_hash: "abc123..."
    prev_tx_index: 0
    script_sig: ""  # Empty or minimal
    sequence: 0xffffffff
outputs:
  - value: 100000000
    script_pub_key: "0014..."
witness_data:
  - input_index: 0
    witness_items:
      - "3045022100..."  # Signature
      - "03abc..."       # Public key
lock_time: 0

```

Key differences:

- **Marker and Flag**: Indicate a SegWit transaction.
- **Empty `script_sig`**: Signature data is moved to `witness_data`.
- **Witness Data**: Contains signatures and public keys for each input.

---

## TXID Calculation: Pre-SegWit vs. SegWit

### Pre-SegWit TXID Calculation

The TXID is computed by hashing the entire transaction data, including the `script_sig` (signature). This makes TXIDs malleable, as signatures can be modified without invalidating the transaction.

Example:

```python
tx_data = serialize(version, inputs, outputs, lock_time)
txid = sha256(sha256(tx_data))

```

### SegWit TXID Calculation

In SegWit, the TXID is computed by hashing only the non-witness data (excluding the `witness_data` field). This ensures TXIDs are immutable.

Example:

```python
tx_data = serialize(version, marker, flag, inputs, outputs, lock_time)
txid = sha256(sha256(tx_data))

```

### WTXID: Witness Transaction ID

SegWit introduces a new identifier, **WTXID**, which includes the witness data. It is used for transaction propagation in the P2P network.

Example:

```python
wtxid_data = serialize(version, marker, flag, inputs, outputs, witness_data, lock_time)
wtxid = sha256(sha256(wtxid_data))

```

---

## Block Size and Weight Units

SegWit introduces **weight units** to measure block size. Each byte of data is assigned a weight:

- Non-witness data: 4 weight units per byte.
- Witness data: 1 weight unit per byte.

The maximum block weight is **4,000,000 weight units**, equivalent to:

- 1MB of non-witness data.
- Up to 4MB of total data (including witness data).

The **virtual size** of a transaction is calculated as:

```
virtual_size = (weight_units) / 4

```

This reduces the effective size of witness data, lowering transaction fees.

---

## Bech32 Address Format

SegWit uses Bech32 addresses (starting with `bc1`) for native SegWit transactions. Bech32 addresses:

- Are case-insensitive.
- Include error detection and correction.
- Are more compact than legacy addresses.

Example:

```
bc1qar0srrr7xfkvy5l643lydnw9re59gtzzwf5mdq

```

---

## Implications for Developers

### 1. **Transaction Construction**

- For SegWit transactions, the `script_sig` field is empty or minimal.
- Signature data is placed in the `witness_data` field.

### 2. **Signature Verification**

- Extract signatures and public keys from the `witness_data` field.
- Follow SegWit-specific validation rules (e.g., for P2WPKH or P2WSH).

### 3. **Fee Calculation**

- Fees are based on the **virtual size** of the transaction:

  ```
  virtual_size = (weight_units) / 4
  
  ```

- Witness data is discounted, making SegWit transactions cheaper.

### 4. **Backward Compatibility**

- SegWit is a soft fork, so non-upgraded nodes can still validate SegWit transactions (but won't process witness data).
- Ensure your application handles both SegWit and non-SegWit transactions.

---

## Example: Building a SegWit Transaction

1. **Select Inputs**: Choose UTXOs to spend.
2. **Create Outputs**: Specify the recipient address and amount.
3. **Generate Witness Data**: Sign the transaction and place the signature and public key in the `witness_data` field.
4. **Calculate TXID**: Hash the non-witness data to compute the TXID.
5. **Broadcast the Transaction**: Send the transaction to the Bitcoin network.

---

## Conclusion

SegWit is a critical upgrade for Bitcoin, solving transaction malleability and increasing block capacity. For developers, understanding its technical details—such as the new transaction format, TXID calculation, and weight units—is essential for building modern Bitcoin applications. By adopting SegWit, you can improve transaction efficiency and reduce costs for your users.
