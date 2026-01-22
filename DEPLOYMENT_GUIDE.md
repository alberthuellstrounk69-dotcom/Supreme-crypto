# VelocityToken Deployment Guide

## ⚠️ IMPORTANT LICENSE NOTICE

**This deployment guide is exclusively for Albert Peter Huellstrounk.**
**All others must obtain a paid license before deployment.**

---

## PRE-DEPLOYMENT CHECKLIST

### 1. License Verification
- [ ] Confirm you are Albert Peter Huellstrounk
- [ ] If not, obtain commercial license
- [ ] Have written permission to deploy

### 2. Technical Requirements
- [ ] Node.js v16+ installed
- [ ] npm or yarn installed
- [ ] Wallet with ETH for gas fees
- [ ] RPC endpoint for target network
- [ ] Etherscan API key (for verification)

### 3. Configuration
- [ ] `.env` file created
- [ ] Private key added to `.env`
- [ ] RPC URLs configured
- [ ] Etherscan API key added

---

## DEPLOYMENT STEPS

### Step 1: Install Dependencies

```bash
npm install
```

### Step 2: Configure Environment

Create `.env` file:

```bash
cp .env.example .env
```

Edit `.env` with your credentials:

```env
# Your wallet private key (without 0x prefix)
PRIVATE_KEY=your_private_key_here

# Sepolia Testnet RPC
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/YOUR_INFURA_PROJECT_ID

# Mainnet RPC (for mainnet deployment)
MAINNET_RPC_URL=https://mainnet.infura.io/v3/YOUR_INFURA_PROJECT_ID

# Etherscan API Key (for contract verification)
ETHERSCAN_API_KEY=your_etherscan_api_key_here
```

### Step 3: Compile Contract

```bash
npm run compile
```

Expected output:
```
Compiled 1 Solidity file successfully
```

### Step 4: Choose Deployment Network

#### Option A: Local Hardhat Network (Testing Only)

```bash
npm run deploy
```

**Network Details:**
- Chain ID: 1337
- Gas: Free
- Purpose: Testing and development

#### Option B: Sepolia Testnet (Recommended First)

```bash
npm run deploy:sepolia
```

**Network Details:**
- Chain ID: 11155111
- Gas: Sepolia testnet ETH (free from faucets)
- Purpose: Testing before mainnet deployment

**Get Sepolia ETH:**
- https://sepoliafaucet.com/
- https://faucet.quicknode.com/ethereum/sepolia

#### Option C: Ethereum Mainnet (Production)

```bash
npm run deploy:mainnet
```

**Network Details:**
- Chain ID: 1
- Gas: Real ETH required
- Purpose: Production deployment

**⚠️ WARNING:** Mainnet deployment is irreversible. Ensure:
- Thorough testing on testnet
- Sufficient ETH for deployment (~0.01-0.05 ETH)
- Contract code reviewed
- All parameters verified

---

## DEPLOYMENT OUTPUT

After successful deployment, you'll see:

```
Deploying VelocityToken with account: 0x...
Account balance: ...
VelocityToken deployed to: 0x...

Deployment Summary:
- Token Name: Velocity Token
- Token Symbol: VELO
- Initial Supply: 10,000,000 VELO
- Owner: 0x...
- Treasury Wallet: 0x...
- Contract Address: 0x...

Deployment info saved to deployment.json
```

**Save the contract address** - you'll need it for:
- Frontend integration
- Token listing on exchanges
- Contract verification
- Future upgrades or interactions

---

## POST-DEPLOYMENT VERIFICATION

### Verify on Etherscan

1. Visit Etherscan for your network:
   - Mainnet: https://etherscan.io
   - Sepolia: https://sepolia.etherscan.io

2. Enter your contract address

3. Click "Contract" tab → "Verify and Publish"

4. Or use Hardhat verification (add to hardhat.config.js):

```javascript
etherscan: {
  apiKey: process.env.ETHERSCAN_API_KEY
}
```

Then run:
```bash
npx hardhat verify --network <network> <contract-address> <treasury-address>
```

### Verify Token Details

Check that:
- [ ] Token name is "Velocity Token"
- [ ] Symbol is "VELO"
- [ ] Total supply is 10,000,000 VELO
- [ ] Owner is set correctly
- [ ] Treasury wallet is set correctly
- [ ] Max wallet limit is 2% (200,000 VELO)

---

## POST-DEPLOYMENT CONFIGURATION

### 1. Set Fee Exemptions

Add addresses that should bypass fees:

```javascript
// Example using Hardhat console
npx hardhat console --network <network>

const token = await ethers.getContractAt("VelocityToken", "CONTRACT_ADDRESS");

// Exempt DEX router
await token.setFeeExempt("UNISWAP_ROUTER_ADDRESS", true);

// Exempt another address
await token.setFeeExempt("SOME_ADDRESS", true);
```

### 2. Add Liquidity

To make the token tradable:

```javascript
// Approve DEX router
await token.approve("UNISWAP_ROUTER_ADDRESS", LIQUIDITY_AMOUNT);

// Add liquidity via DEX interface
// (This would be done through the DEX frontend or contract)
```

### 3. Verify Fee Distribution

Test a transfer to ensure fees work correctly:

```javascript
// Transfer from owner to another address
await token.transfer("RECIPIENT_ADDRESS", ethers.parseEther("1000"));

// Check balances
await token.balanceOf("RECIPIENT_ADDRESS"); // Should be ~950 VELO (5% fees)
await token.balanceOf("TREASURY_ADDRESS"); // Should have increased by ~10 VELO
```

---

## CONTRACT INTERACTIONS

### Using Ethers.js

```javascript
// Connect to contract
const token = new ethers.Contract(
  CONTRACT_ADDRESS,
  VELOCITY_TOKEN_ABI,
  wallet
);

// Get balance
const balance = await token.balanceOf(wallet.address);

// Transfer tokens
await token.transfer(recipient, amount);

// Set fees (owner only)
await token.setFees(2, 2, 1); // burn, liquidity, treasury

// Check fee exemptions
const isExempt = await token.isFeeExempt(address);
```

### Using Web3.js

```javascript
const token = new web3.eth.Contract(ABI, CONTRACT_ADDRESS);

// Get balance
const balance = await token.methods.balanceOf(address).call();

// Transfer
await token.methods.transfer(to, amount).send({ from: account });

// Owner functions
await token.methods.setFees(2, 2, 1).send({ from: owner });
```

---

## MONITORING & MAINTENANCE

### 1. Monitor Token Transfers

Set up monitoring for:
- Large transfers
- Fee collection
- Treasury accumulation
- Burn events

### 2. Regular Checks

- [ ] Review fee distribution
- [ ] Check treasury balance
- [ ] Monitor contract activity
- [ ] Verify exemption list
- [ ] Check wallet limit settings

### 3. Security

- [ ] Monitor for unusual activity
- [ ] Keep owner key secure
- [ ] Use hardware wallet for mainnet
- [ ] Implement multi-sig if needed
- [ ] Regular security audits

---

## TROUBLESHOOTING

### Common Issues

**Issue: "Insufficient funds"**
- Solution: Add ETH to wallet for gas fees

**Issue: "Nonce too low"**
- Solution: Reset wallet nonce or wait for pending transaction

**Issue: "Gas required exceeds allowance"**
- Solution: Increase gas limit in transaction

**Issue: "Contract verification failed"**
- Solution: Ensure compiler version matches exactly (0.8.20)

**Issue: "Transaction reverted"**
- Solution: Check error message and verify inputs

### Debug Mode

Enable detailed logging in Hardhat:

```javascript
// hardhat.config.js
module.exports = {
  mocha: {
    timeout: 100000
  }
};
```

---

## SECURITY CONSIDERATIONS

### Critical Security Points

1. **Owner Key Security**
   - Use hardware wallet
   - Never share private key
   - Implement multi-sig if possible

2. **Initial Distribution**
   - Test thoroughly before mainnet
   - Verify all settings
   - Document configuration

3. **Fee Management**
   - Monitor fee impact
   - Adjust if necessary
   - Keep under 10% total

4. **Exemption Management**
   - Only exempt trusted addresses
   - Document all exemptions
   - Regularly review

5. **Treasury Management**
   - Secure treasury wallet
   - Regular withdrawals
   - Transparent reporting

---

## EMERGENCY PROCEDURES

### If Compromise Detected

1. **Immediate Actions:**
   - Transfer ownership to secure address
   - Suspend token transfers (if possible)
   - Notify community

2. **Recovery Steps:**
   - Deploy new contract
   - Migrate token holders
   - Update frontend
   - Announce migration

3. **Post-Incident:**
   - Analyze breach
   - Implement new security measures
   - Consider security audit

---

## SUPPORT

### For Albert Peter Huellstrounk
Direct access to all features and unlimited support.

### For Licensed Users
Contact for deployment support:
- Email: [Insert Email]
- Documentation: [Insert Docs URL]
- Community: [Insert Community Link]

---

## DEPLOYMENT CHECKLIST

Before deploying to mainnet:

- [ ] Tested on local network
- [ ] Tested on Sepolia testnet
- [ ] All tests passing
- [ ] Code reviewed
- [ ] Security audit completed (if applicable)
- [ ] Gas costs estimated
- [ ] Wallet funded with sufficient ETH
- [ ] RPC endpoints configured
- [ ] Etherscan API key ready
- [ ] Deployment plan documented
- [ ] Rollback plan prepared
- [ ] Team notified
- [ ] Community announcement ready

---

**© 2025 Albert Peter Huellstrounk. All Rights Reserved.**

**PROPRIETARY SOFTWARE - EXCLUSIVELY LICENSED TO ALBERT PETER HUELLSTROUNK**

**UNAUTHORIZED DEPLOYMENT IS ILLEGAL**