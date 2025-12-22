# Electrum Setup (CLI)

## I. Create/Load/Restore Electrum Wallet

### 1. Create a New Wallet
To create a new standard wallet using the command line:

```bash
# Create a new default wallet file
electrum create
```

By default, this generates a new seed. The seed will be displayed in the output. **Copy this seed immediately**, as it won't be shown again.

### 2. Restore from Seed
To restore a wallet from an existing seed phrase without revealing seedphrase in bash history
`:` is used to invoke prompt

```bash
# Restore wallet from seed with prompts for password and seed
electrum restore --password : :
```

If you want to set a password for this new wallet file immediately (recommended):
```bash
electrum password
```

### 3. Load/Daemon Usage
To use the wallet, you often need the Electrum daemon running:

```bash
# Start daemon
electrum daemon -d

# Load the wallet
electrum load_wallet
```

---

## II. Save seedphrase to keepassxc-cli

To utilize the **PROTECTED** password field for sensitive data, we will create two separate entries: one for the Seed Phrase and one for the Wallet Password.

### 1. Open/Create Database
```bash
# Create if needed
keepassxc-cli db-create secrets
```

### 2. Store Seed Phrase
We store the seed phrase in the *Password* field of this entry to ensure it is protected/hidden by default.

```bash
keepassxc-cli add -p secrets "Crypto/ElectrumSeed"
```
*   **Username:** (Optional)
*   **Password:** <Paste your 12-word seed phrase here>
*   **URL:** (Empty)
*   **Notes:** "Electrum Seed Phrase"

### 3. Store Wallet Password
Store the actual encryption password for the wallet file in a separate entry.

```bash
keepassxc-cli add -p secrets "Crypto/ElectrumWalletPass"
```
*   **Username:** (Optional)
*   **Password:** <Enter your strong wallet password>
*   **URL:** (Empty)
*   **Notes:** "Password to open the local wallet file"

Generate password

```bash
keepassxc-cli generate
```

### 4. Retrieval
To securely retrieve the seed (e.g., to copy to clipboard):

```bash
# Show the protected seed phrase (requires database password)
keepassxc-cli show -s secrets "Crypto/ElectrumSeed"
```
