# Smart Contract Deployment Guide

This guide will walk you through deploying and verifying your lottery smart contracts. Don't worry if you're not technical - we'll explain everything step by step!

## Prerequisites

Before we start, you'll need:

- A computer with internet connection
- Your wallet's private key
- Some ETH for gas fees (deployment costs)
- Some LINK tokens for VRF and Automation services
- About 30-45 minutes

## Step 1: Install Node.js

Node.js is a program that lets us run the deployment scripts.

### For Windows:

1. Go to [https://nodejs.org](https://nodejs.org)
2. Click the green "LTS" button to download
3. Run the downloaded file and follow the installation wizard
4. Keep clicking "Next" and then "Install"

### For Mac:

1. Go to [https://nodejs.org](https://nodejs.org)
2. Click the green "LTS" button to download
3. Open the downloaded `.pkg` file
4. Follow the installation steps

### Verify Installation:

1. Open Terminal (Mac) or Command Prompt (Windows)
2. Type: `node --version`
3. You should see a version number like `v20.x.x`

## Step 2: Download the Project

1. Download the project files (you should have received a ZIP file)
2. Extract the ZIP file to your Desktop
3. You should see a folder called `lottery-contracts-deployment`

## Step 3: Open Terminal in Project Folder

### For Windows:

1. Open the `lottery-contracts-deployment` folder
2. Hold `Shift` and right-click in the empty space
3. Select "Open PowerShell window here" or "Open command window here"

### For Mac:

1. Open Terminal application
2. Type: `cd ` (with a space after cd)
3. Drag the `lottery-contracts-deployment` folder into Terminal
4. Press Enter

## Step 4: Install Dependencies

Dependencies are additional programs our deployment needs to work.

1. In your terminal, type: `npm install`
2. Press Enter and wait (this might take 2-3 minutes)
3. You'll see lots of text scrolling - this is normal!
4. When it's done, you'll see your cursor ready for the next command

## Step 5: Set Up Chainlink VRF (Randomness Service)

Before deploying your contracts, you need to set up Chainlink VRF which provides secure randomness for your lottery.

### Create VRF Subscription

1. **Go to the VRF Dashboard**

   - Visit [https://vrf.chain.link/](https://vrf.chain.link/)
   - Connect your wallet when prompted

2. **Select Your Target Network**

   - At the top of the page, select the network you plan to deploy to:
     - **"Base Mainnet"** for production deployment
     - **"Ethereum Sepolia"** for testing
   - Make sure the correct network is selected before proceeding

3. **Copy Important Information**

   - **VRF Coordinator**: Look for "VRF Coordinator" section and copy the address (should be `0x9Ddf...8B1B`)
   - **Key Hash**: Look for "Key hash" section. You'll see one or more key hashes available:
     - Choose the key hash based on your gas price preference
     - Lower gas price = slower response, Higher gas price = faster response(preferred)
     - Copy the full key hash (starts with `0x787d...`)

   **Important**: Write down both the VRF Coordinator address and Key Hash - you'll need these during deployment!

4. **Create New Subscription**

   - Click the "Create Subscription" button
   - A form will appear:
     - **Email** (optional): Enter your email for notifications
     - **Project Name** (optional): Enter a name like "My Lottery Project"
   - Click "Submit"
   - Approve the transaction in your wallet

5. **Get Your Subscription ID**
   - After the transaction completes, you'll see a "My Subscriptions" section appear
   - Click on your newly created subscription
   - On the subscription details page, you'll see a shortened Subscription ID
   - Click the copy button next to it to copy the full subscription ID
   - **Important**: Save this Subscription ID - you'll need it during deployment!

## Step 6: Deploy Your Contracts

Now we'll deploy your smart contracts to the blockchain!

### Gather Your Information

Before starting, have these ready from the previous step:

- **Private Key**: Your wallet's private key (64 characters, starts with or without 0x)
- **VRF Coordinator**: The address you copied from the VRF dashboard (e.g., `0x9Ddf...8B1B`)
- **VRF Key Hash**: The key hash you copied from the VRF dashboard (e.g., `0x787d...`)
- **Subscription ID**: The subscription ID you copied from your VRF subscription
- **Treasury Address**: Where fees go (optional - leave empty to keep fees in lottery contract)

### Choose Your Network and Start Deployment

Choose which network you want to deploy to:

**For Base Mainnet (production):**

1. In your terminal, type: `npm run deploy:base`
2. Press Enter

**For Sepolia Testnet (testing):**

1. In your terminal, type: `npm run deploy:sepolia`
2. Press Enter

### Follow the Prompts

The system will ask you several questions. Answer them carefully:

#### Check for Existing Contracts

If you've deployed before, you'll see:

```
⚠️  Existing deployments found for chain 8453:
   - HourlyLotteryToken: 0x1234...
   - HourlyLottery: 0x5678...
Do you want to overwrite existing deployments and deploy new contracts?
```

- Choose **No** if you want to keep existing contracts
- Choose **Yes** if you want to deploy new ones (this will replace the old ones)

#### Private Key

```
Enter your private key:
```

**⚠️ IMPORTANT**: Your private key will be hidden as you type (you won't see the characters). This is for security.
Paste your private key and press Enter.

#### VRF Coordinator

```
Enter VRF Coordinator address:
```

Paste the VRF coordinator address you copied from the VRF dashboard (e.g., `0x9Ddf...8B1B`).

#### VRF Key Hash

```
Enter VRF Key Hash:
```

Paste the VRF key hash you copied from the VRF dashboard (e.g., `0x787d...`).

#### VRF Subscription ID

```
Enter VRF Subscription ID:
```

Type your subscription ID number that you copied from your VRF subscription.

#### Treasury Address

```
Enter treasury address (press Enter to keep treasury in lottery contract):
```

- Leave empty and press Enter to keep fees in the lottery contract
- OR paste a specific address where you want fees sent

#### Enable Trading

```
Enable trading immediately after deployment? (Y/n)
```

- Press Enter or type `Y` to allow trading immediately
- Type `n` if you want to enable trading later from block explorer

### Wait for Deployment

The deployment will now start. You'll see:

1. **Connecting to network**
2. **Deploying token contract**
3. **Enabling trading** (if you chose yes)
4. **Deploying lottery contract**
5. **Setting up exemptions**
6. **Saving deployment info**

### Deployment Complete!

When successful, you'll see:

```
✅ Deployment completed successfully!
📄 Deployment summary:
   Network: base (8453)
   Token: 0x1234567890abcdef...
   Lottery: 0xabcdef1234567890...

💡 Run the verify script to verify these contracts on the block explorer.
```

**🎉 Congratulations! Your contracts are now deployed!**

**Important**: Save the contract addresses shown - you'll need them for the next steps!

## Step 7: Add Contract as VRF Consumer

Now you need to add your deployed lottery contract as a consumer to your VRF subscription.

1. **Go back to VRF Dashboard**

   - Visit [https://vrf.chain.link/](https://vrf.chain.link/) again
   - Make sure you're on Base Mainnet

2. **Access Your Subscription**

   - Make sure you're on the same network you deployed to (Base Mainnet or Sepolia)
   - In the "My Subscriptions" section, click on your subscription
   - This will open the subscription details page

3. **Add Consumer**

   - Look for the "Add consumer" button (usually at the bottom right)
   - Click "Add consumer"
   - In the form that appears:
     - **Contract address**: Paste your **lottery contract address** (not the token address!)
     - This should be the address shown as "Lottery: 0x..." in your deployment summary
   - Click "Add consumer"
   - Approve the transaction in your wallet

4. **Fund Your Subscription**

   - After adding the consumer, look for the "Fund subscription" button (next to "Add consumer")
   - Click "Fund subscription"
   - Enter the amount of LINK tokens you want to fund (recommended: at least 80-100 LINK)
   - Click "Fund subscription"
   - Approve the transaction in your wallet

   **Note**: These LINK tokens will be used to pay for randomness requests. Each lottery draw will consume some LINK.

## Step 8: Set Up Chainlink Automation

Automation will trigger your lottery draws every hour automatically.

1. **Go to Automation Dashboard**

   - Visit [https://automation.chain.link/](https://automation.chain.link/)
   - Connect your wallet if prompted
   - Make sure you're on the same network you deployed to (Base Mainnet or Sepolia)

2. **Register New Upkeep**

   - Click the "Register new Upkeep" button

3. **Configure Trigger**

   - **Trigger mechanism**: Select "Custom logic"
   - **Target contract address**: Paste your **lottery contract address** (same as used for VRF consumer)
   - Click "Next"

4. **Set Upkeep Details**

   - **Upkeep name**: Enter a descriptive name like "Hourly Lottery Automation" or "My Lottery Draws"
   - **Starting balance**: Enter the amount of LINK tokens to fund this upkeep (recommended: at least 5-10 LINK)
   - **Check Data**: Enter `0x00` (exactly as shown)
   - **Email** (optional): Your email for notifications
   - **Project name** (optional): Same as used for VRF
   - Keep all other settings as default

5. **Submit Registration**

   - Click "Register upkeep"
   - Approve the transaction in your wallet
   - You may be asked to sign an additional message - approve this too

6. **Upkeep Created**
   - After the transaction completes, your upkeep will be created
   - The automation system will now monitor your contract and trigger lottery draws every hour

## Step 9: Verify Your Contracts

Verification makes your contract code publicly viewable on the blockchain explorer, which builds trust with users.

### Start Verification

Choose the same network you deployed to:

**For Base Mainnet:**

1. In your terminal, type: `npm run verify:base`
2. Press Enter

**For Sepolia Testnet:**

1. In your terminal, type: `npm run verify:sepolia`
2. Press Enter

### Wait for Verification

The system will:

1. **Find your deployed contracts** from the previous step
2. **Submit them for verification** to the blockchain explorer
3. **Check verification status** until complete

You'll see messages like:

```
🔍 Verifying HourlyLotteryToken at 0x1234...
✅ HourlyLotteryToken verified successfully

🔍 Verifying HourlyLottery at 0x5678...
✅ HourlyLottery verified successfully
```

### Verification Complete!

When done, you'll see:

```
🎉 Verification process completed!

📊 Final verification status:
   HourlyLotteryToken: ✅ Verified
   HourlyLottery: ✅ Verified
```

## Step 10: Check Your Contracts

Your contracts are now live! You can view them on the blockchain explorer:

### For Base Mainnet:

1. Go to [https://basescan.org](https://basescan.org)
2. Paste your contract address in the search box
3. You should see your verified contract with a green checkmark ✅

### For Sepolia Testnet:

1. Go to [https://sepolia.etherscan.io](https://sepolia.etherscan.io)
2. Paste your contract address in the search box
3. You should see your verified contract with a green checkmark ✅

## How Your Lottery System Works

Now that everything is set up, here's how your hourly lottery operates:

### Automatic Process Flow

1. **Hourly Trigger**: Chainlink Automation monitors your contract and triggers a new lottery draw every hour
2. **Random Number Request**: When triggered, your contract requests a random number from Chainlink VRF
3. **Random Number Delivery**: VRF provides a cryptographically secure random number to your contract
4. **Winner Selection**: Your contract uses the random number to fairly select winners from all entries
5. **Prize Distribution**: Winners automatically receive their prizes in tokens
6. **New Round Begins**: The cycle repeats for the next hour

### What Happens Next

- **Users buy lottery entries** using your token throughout each hour
- **Every hour, winners are automatically selected** and paid
- **No manual intervention needed** - everything runs automatically
- **You can monitor** the draws through the blockchain explorer

### Monitoring Your Services

- **VRF Usage**: Check [https://vrf.chain.link/](https://vrf.chain.link/) to monitor your subscription balance
- **Automation Status**: Check [https://automation.chain.link/](https://automation.chain.link/) to ensure your upkeep is running
- **Refund as needed**: Add more LINK tokens to either service when balances get low

## Troubleshooting

### Common Issues:

**"Command not found" error**

- Make sure Node.js is installed correctly
- Restart your terminal and try again

**"Invalid private key" error**

- Check your private key is exactly 64 or 66 characters
- Make sure there are no extra spaces

**"Network connection failed"**

- Check your internet connection
- Verify the RPC URL is correct

**"Insufficient funds" error**

- Make sure your wallet has enough ETH for gas fees
- Try again when network fees are lower

**"Verification failed"**

- Wait a few minutes and try the verify command again
- Network might be busy

**"Not enough LINK tokens" or VRF/Automation not working**

- Check your VRF subscription balance at [https://vrf.chain.link/](https://vrf.chain.link/)
- Check your Automation upkeep balance at [https://automation.chain.link/](https://automation.chain.link/)
- Fund with more LINK tokens if balances are low
- Make sure you added the correct lottery contract address as a consumer

**"Lottery draws not happening automatically"**

- Check that your Automation upkeep is active at [https://automation.chain.link/](https://automation.chain.link/)
- Ensure the upkeep has sufficient LINK balance
- Verify the lottery contract address is correct in the upkeep
- Make sure there are enough entries in the lottery (minimum required for draws)

**"Can't find my subscription/upkeep"**

- Make sure you're connected with the same wallet used to create them
- Verify you're on the correct network (Base Mainnet or Sepolia, matching your deployment)
- Check that the transactions were confirmed on the blockchain

## Security Reminders

- **Never share your private key** with anyone
- **Keep backups** of your contract addresses, VRF subscription ID, and Automation upkeep ID
- **Double-check all addresses** before submitting (especially when adding consumers)
- **Monitor your LINK balances** regularly to ensure services don't run out of funding
- **Test with small amounts** first if unsure
- **Save your VRF subscription and Automation upkeep URLs** for easy access later

## Need Help?

If you encounter any issues:

1. Take a screenshot of the error message
2. Note which step you were on
3. Contact support with these details

---

**🎉 You've successfully deployed and verified your lottery smart contracts!**

Your contracts are now live on the blockchain and ready for use.
