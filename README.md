# STORZ: Decentralized Secure Cloud Storage System

[![Solidity](https://img.shields.io/badge/Solidity-^0.8.24-363636?logo=solidity)](https://soliditylang.org/)
[![Ethereum](https://img.shields.io/badge/Ethereum-Smart%20Contracts-3C3C3D?logo=ethereum)](https://ethereum.org/)
[![IPFS](https://img.shields.io/badge/IPFS-InterPlanetary%20File%20System-65C2CB?logo=ipfs)](https://ipfs.tech/)
[![Next.js](https://img.shields.io/badge/Next.js-14%20App%20Router-000000?logo=next.js)](https://nextjs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**STORZ** is a decentralized, tamper-proof cloud storage and file-sharing platform leveraging **IPFS**, **Ethereum Smart Contracts**, and **Web3 Authentication**. It eliminates traditional cloud storage vulnerabilities such as centralized Single Points of Failure (SPOF), administrative privacy compromises, data tampering, and vendor lock-in.

Based on the research and system specification by the Department of Computer Science and Engineering, Jalpaiguri Government Engineering College.

---

## 1. System Architecture

```
+-----------------------------------------------------------------------------------------+
|                                    STORZ APPLICATION                                    |
+-----------------------------------------------------------------------------------------+
                                             |
                   +-------------------------+-------------------------+
                   |                                                   |
                   v                                                   v
+------------------------------------+              +-------------------------------------+
|        PRESENTATION LAYER          |              |       CLIENT SECURITY PIPELINE      |
|  - Next.js 14 App Router           |              |  - Web Crypto API (AES-256-GCM)     |
|  - Tailwind CSS + Lucide Icons     |              |  - Random 256-bit key + 96-bit IV   |
|  - Web3 Auth (MetaMask personal_sign)|            |  - Key wrapping via wallet signature|
|  - Real-time Stage Pipeline Dialog |              |  - Local zero-knowledge decryption  |
+------------------------------------+              +-------------------------------------+
                   |                                                   |
                   v                                                   v
+------------------------------------+              +-------------------------------------+
|         BLOCKCHAIN LAYER           |              |         IPFS STORAGE LAYER          |
|  - Ethereum FileRegistry.sol       |              |  - Pinata Cloud IPFS API / Kubo     |
|  - Hardhat Local & Sepolia Testnet |              |  - Deterministic IPFS Gateway       |
|  - Granular Access Control Mappings|              |  - Encrypted Blob Storage (Zero Raw)|
|  - Immutable Audit Events          |              |  - Content Identifier (CIDv1)       |
+------------------------------------+              +-------------------------------------+
```

---

## 2. Core Modules & Implementation

### A. Smart Contract (`contracts/FileRegistry.sol`)
- **`File` Struct**: Contains `cid`, `fileName`, `fileSize`, `fileType`, `owner`, `timestamp`, and `exists`.
- **Granular Access Control**:
  - `uploadFile(...)`: Records file metadata and emits `FileUploaded`.
  - `getFile(uint256 fileId)`: Enforces caller is either owner or explicitly authorized in `filePermissions`. Reverts with `UnauthorizedAccess`.
  - `shareFile(uint256 fileId, address recipient)`: Grants read access, tracks in recipient's active file list, emits `FileShared`.
  - `revokeAccess(uint256 fileId, address recipient)`: Revokes permissions on-chain, emits `AccessRevoked`.
  - `deleteFile(uint256 fileId)`: Soft-deletes the file, preventing any subsequent reads, emits `FileDeleted`.
  - `getUserFiles(address user)`: Returns only existing files owned or authorized for that user.

### B. Client-Side Cryptography Pipeline (`src/lib/crypto.ts`)
- **AES-256-GCM**: Military-grade authenticated encryption standard using browser-native Web Crypto API.
- **Zero-Plaintext Storage**: Plaintext files never leave browser memory.
- **Envelope Encryption**: Generates fresh 256-bit symmetric keys per file, packed with 96-bit random IVs and authentication tags. File keys are wrapped deterministically using master keys derived from MetaMask signatures (`personal_sign`).

### C. IPFS Storage Integration (`src/lib/ipfs.ts`)
- **Dual-Mode IPFS**:
  - **Live Pinata Cloud**: Uses `NEXT_PUBLIC_PINATA_JWT` or API keys for remote pinning to IPFS cluster.
  - **Deterministic Local IPFS Engine**: Embedded fallback calculating SHA-256 multihash CIDv1 identifiers and in-memory/session storage, enabling out-of-the-box local testing without paid third-party accounts.
- **Multi-Gateway Fallback**: Resolves blobs across Pinata, IPFS.io, and Cloudflare gateways.

### D. Web3 Dashboard UI (`src/app/dashboard/page.tsx`)
- **MetaMask Authentication**: Cryptographic challenge signing for wallet authentication without gas fees.
- **Multi-Stage Upload Pipeline**: Real-time progress tracker (Encrypting -> IPFS Pinning -> MetaMask Sign -> On-Chain Mined).
- **File Explorer**:
  - Filter tabs: All Files, My Files, Shared with Me.
  - Quick actions: Download & Decrypt, Share Access, Revoke Access, Soft Delete.
  - Live search and CID copy tools.
- **Audit Feed**: Real-time on-chain activity stream showing transaction hashes and event logs.

---

## 3. Quick Start Guide

### Prerequisites
- [Node.js](https://nodejs.org/) (v18+ or v20+)
- [MetaMask](https://metamask.io/) browser extension

### Installation
```bash
# 1. Navigate to the project directory
cd C:\Users\aveng\.gemini\antigravity\scratch\storz

# 2. Install dependencies
npm install
```

### Running Smart Contract Tests
Run the comprehensive Chai test suite covering 100% of access control rules:
```bash
npm test
# Or: npx hardhat test
```

### Local Blockchain & Contract Deployment
```bash
# Terminal 1: Start local Hardhat Ethereum node
npm run node

# Terminal 2: Deploy FileRegistry contract to local network
npm run deploy:local
```
This automatically updates `src/config/contractInfo.json` with the freshly deployed contract address and ABI!

### Starting Next.js Web3 Dashboard
```bash
npm run dev
```
Open [http://localhost:3000/dashboard](http://localhost:3000/dashboard) in your browser.

---

## 4. Testnet Deployment (Sepolia)
To deploy to the Ethereum Sepolia Testnet:
1. Configure your `.env.local`:
   ```env
   SEPOLIA_RPC_URL=https://rpc.sepolia.org
   PRIVATE_KEY=your_wallet_private_key_without_0x
   NEXT_PUBLIC_CONTRACT_ADDRESS=your_deployed_contract_address
   ```
2. Run:
   ```bash
   npm run deploy:sepolia
   ```

---

## 5. Security & Verification
- **Audit Trails**: All file uploads, sharing delegations, revocations, and deletions emit indexed Solidity events queryable on Etherscan.
- **Cryptographic Independence**: Deleting a file or revoking access prevents unauthorized retrieval on-chain; encrypted blobs remaining on IPFS cannot be decrypted without the private AES key.
