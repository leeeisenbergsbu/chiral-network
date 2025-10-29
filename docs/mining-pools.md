# Mining & Pool Mining Guide

Chiral Network implements a comprehensive mining system with support for both solo mining and peer-to-peer pool mining using the Ethash consensus algorithm with DHT-based decentralized pool infrastructure.

## Purpose

- Enable cryptocurrency mining to secure the blockchain and earn rewards
- Provide decentralized P2P pool mining with automatic discovery
- Support both solo and collaborative mining strategies
- Implement fair payout distribution through PPLNS and PPS algorithms
- Facilitate pool coordination without centralized servers

---

## Overview

Chiral Network mining consists of two primary modes:

### Solo Mining
Individual miners compete to find blocks and receive full block rewards. Best for miners with significant hash power or those who prefer complete independence.

### Pool Mining
Miners collaborate to increase block finding frequency and receive proportional rewards. Pools can be:
- **Traditional Pools**: Pre-configured centralized pools with established infrastructure
- **P2P Pools**: Decentralized user-created pools discovered via DHT network

---

## Solo Mining

### Features

- **Real-time Statistics**: Live hashrate monitoring and block count
- **Temperature Monitoring**: CPU thermal monitoring for safe operation
- **Mining History**: Track performance metrics over time
- **Automatic Rewards**: Mining rewards automatically sent to your "wallet"

### How to Mine Solo

#### Prerequisites
1. **Create or Import Wallet**
   - Navigate to Account page
   - Create new account or import existing keystore
   - Ensure wallet is unlocked

#### Starting Solo Mining

1. **Navigate to Mining Page**
2. **Verify wallet address** is displayed
3. **Click "Start Mining" button**
4. **Monitor real-time statistics**:
   - Current hashrate (H/s, KH/s, MH/s)
   - Total blocks mined
   - Shares submitted
   - Estimated earnings
   - Hardware temperature

#### Stopping Mining

1. Click "Stop Mining" button
2. System safely shuts down mining process
3. Final statistics are saved to history

### Mining Statistics

The Mining page displays comprehensive statistics:

**Current Session**:
- Hashrate (updates every few seconds)
- Blocks found
- Total shares submitted
- Uptime

**Performance Metrics**:
- Average hashrate over time
- Acceptance rate
- Estimated earnings per 24 hours

**Hardware Monitoring**:
- CPU temperature
- Memory usage
- Power consumption estimates

---

## Pool Mining

### Pool Types

#### Traditional Pools (Centralized)

**Characteristics**:
- Placeholders for large community pools in each region

#### P2P Pools (Decentralized)

User-created pools discovered via DHT:

- **No Central Server**: Completely peer-to-peer architecture
- **Automatic Discovery**: Found through Kademlia DHT network
- **Peer Coordination**: Share tracking via distributed storage

**Characteristics**:
- User-controlled
- No third-party trust required
- Network-wide visibility
- Community-driven

### Discovering Pools

#### Method 1: Traditional Pool Discovery

Query DHT for pre-configured pools:

```javascript
// Discover all pools
await invoke('discover_mining_pools')

// Filter by region
await invoke('query_dht_for_pools', { 
  regionFilter: "Europe" 
})
```

**Via UI**:
1. Navigate to Mining page
2. Click "Discover Pools (DHT)" button
3. Available pools appear in list
4. View details: hashrate, miners, fees, region

#### Method 2: P2P Pool Discovery

Discover user-created decentralized pools:

```javascript
// Discover all P2P pools
await invoke('p2p_discover_pools', {
  poolId: null
})

// Discover specific pool
await invoke('p2p_discover_pools', {
  poolId: "pool-123"
})
```

**Via UI**:
1. Click "P2P Pools" button
2. System queries DHT pool directory
3. P2P pools appear with special badge
4. Shows creator, creation date, statistics

### Creating a Pool

#### Via UI

1. **Navigate to Mining page**
2. **Click "Create Pool" button**
3. **Fill in pool configuration**:
   - **Name**: Pool display name (e.g., "Community Mining Pool")
   - **Description**: Brief description of pool purpose
   - **Fee Percentage**: 0-10% (your pool fee)
   - **Minimum Payout**: Minimum balance before payout (e.g., 0.1 Chiral)
   - **Payment Method**: PPLNS or PPS (see below)
   - **Region**: Geographic region (Global, Europe, Asia, Americas)

4. **Click "Create"**
   - Pool is created locally
   - Automatically announced to DHT network
   - Becomes discoverable by other peers

#### Via API

```javascript
const pool = await invoke('create_mining_pool', {
  address: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  name: "My Mining Pool",
  description: "High-performance community pool",
  feePercentage: 1.0,
  minPayout: 0.1,
  paymentMethod: "PPLNS",
  region: "Global"
})

// Announce to P2P network
await invoke('p2p_announce_pool', { pool })
```

### Joining a Pool

#### Via UI

1. **Discover or select pool** from list
2. **Click pool card** to view details
3. **Review pool information**:
   - Current miners count
   - Total pool hashrate
   - Fee structure
   - Payment method
   - Acceptance rate
4. **Click "Join Pool" button**
5. **Confirm** your wallet address
6. **System connects** to pool
7. **Start mining** - shares automatically submitted to pool

#### Via API

```javascript
await invoke('join_mining_pool', {
  poolId: "pool-123",
  address: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
})
```

### Leaving a Pool

To leave a pool and return to solo mining:

**Via UI**:
1. Click "Leave Pool" button on Mining page
2. Confirm action
3. Mining automatically switches to solo mode

**Via API**:
```javascript
await invoke('leave_mining_pool')
```

### Pool Statistics

When connected to a pool, monitor your performance:

**Your Statistics**:
- Your hashrate contribution
- Your share percentage
- Shares submitted/accepted
- Estimated payout (24h)
- Last share submission time

**Pool Statistics**:
- Total pool hashrate
- Connected miners
- Pool acceptance rate
- Blocks found (24h)
- Total shares in current window

#### Getting Pool Info

```javascript
// Get current pool info
const poolInfo = await invoke('get_current_pool_info')
// Returns:
// {
//   pool: { id, name, url, ... },
//   stats: { your_hashrate, shares_submitted, ... },
//   joined_at: timestamp
// }

// Get detailed pool statistics
const stats = await invoke('get_detailed_pool_stats', {
  poolId: "pool-123"
})
// Returns:
// {
//   total_miners: 42,
//   total_hashrate: "1.5 GH/s",
//   total_shares: 10000,
//   acceptance_rate: 98.5,
//   shares_in_window: 500
// }
```

---

## P2P Pool System Architecture

### Overview

The P2P pool system is built on **libp2p** networking with **Kademlia DHT** for decentralized pool discovery and coordination.

### Key Components

#### 1. DHT Record Storage

Pools are stored as distributed hash table records:

- **Pool Directory**: `chiral:pool:directory`
  - Contains array of all pool IDs
  - Enables discovery of all available pools
  - Updated when pools are created

- **Individual Pool Data**: `chiral:pool:{pool_id}`
  - Complete pool configuration
  - Creator address
  - Statistics and metadata
  - Discovery timestamp

- **Share Submissions**: `chiral:share:{pool_id}`
  - Miner share submissions
  - Difficulty and timestamp
  - Used for payout calculations

#### 2. P2P Pool Manager

Located in `src-tauri/src/pool_p2p.rs`, manages:

- Pool lifecycle (create, announce, discover)
- DHT operations (put/get records)
- Share tracking and validation
- Pool directory maintenance

#### 3. DHT Integration

Kademlia DHT configuration:

- **Replication Factor**: 3 (records stored on 3 peers)
- **Quorum**: One (fast writes, eventually consistent)
- **Record Expiration**: Periodic republishing required
- **Peer Discovery**: Automatic via libp2p

### Pool Discovery Flow

```
1. Miner A Creates Pool
   ↓
2. Pool Announced to DHT
   - Store pool data: chiral:pool:{id}
   - Update directory: chiral:pool:directory
   ↓
3. DHT Replicates Records
   - Stored on 3 closest peers
   - Periodically republished
   ↓
4. Miner B Discovers Pools
   - Query directory: chiral:pool:directory
   - Receive pool IDs: ["pool-1", "pool-2", ...]
   - Fetch each pool: chiral:pool:{id}
   ↓
5. Display in UI
   - Show pool details
   - Mark as "P2P" pool
   - Enable joining
```

### Share Submission Flow

```
1. Miner finds valid share
   ↓
2. Create share record
   {
     pool_id: "pool-123",
     miner_address: "0x...",
     nonce: 0x1234,
     hash: "0xabcd...",
     difficulty: 1000000,
     timestamp: 1698624000
   }
   ↓
3. Submit to pool
   - Via p2p_submit_share command
   - Stored locally and in DHT
   ↓
4. Share validated
   - Verify hash difficulty
   - Check timestamp
   - Update miner statistics
   ↓
5. Include in payout calculation
```

---

## Payout Methods

### PPLNS (Pay Per Last N Shares)

**Full Name**: Pay Per Last N Shares

**How it Works**:
1. Maintain sliding window of last N shares (e.g., 100 shares)
2. When block found, count each miner's shares in window
3. Calculate percentage: `miner_shares / total_shares`
4. Apply pool fee deduction
5. Distribute: `block_reward × percentage × (1 - fee/100)`

**Example**:
- Block reward: 2.0 Chiral
- Pool fee: 1%
- Last 100 shares:
  - Miner A: 30 shares (30%)
  - Miner B: 50 shares (50%)
  - Miner C: 20 shares (20%)

Payouts:
- Miner A: 2.0 × 0.30 × 0.99 = 0.594 Chiral
- Miner B: 2.0 × 0.50 × 0.99 = 0.99 Chiral
- Miner C: 2.0 × 0.20 × 0.99 = 0.396 Chiral
- Pool fee: 2.0 × 0.01 = 0.02 Chiral

**Characteristics**:
- Rewards recent contributions
- Discourages pool hopping
- Variance depends on luck
- Fair for consistent miners

#### API Usage

```javascript
const payout = await invoke('calculate_pplns_payout', {
  minerAddress: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  blockReward: 2.0,
  nShares: 100 // Window size
})
```

### PPS (Pay Per Share)

**Full Name**: Pay Per Share

**How it Works**:
1. Fixed payout per share submitted
2. Independent of block finding
3. Payment guaranteed for valid shares
4. Pool assumes all risk

**Example**:
- Expected shares per block: 1000
- Block reward: 2.0 Chiral
- Pool fee: 2%
- Per-share value: (2.0 / 1000) × 0.98 = 0.00196 Chiral

If miner submits 50 shares:
- Payout: 50 × 0.00196 = 0.098 Chiral
- Paid regardless of block finding

**Characteristics**:
- Predictable income
- Low variance
- Pool takes financial risk
- Usually higher pool fees

#### API Usage

```javascript
const payout = await invoke('calculate_pps_payout', {
  minerAddress: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  blockReward: 2.0
})
```

---

## API Reference

### Mining Control

#### Start Mining
```typescript
invoke('start_mining', { 
  address: string 
}): Promise<void>
```

**Parameters**:
- `address`: Wallet address to receive rewards

**Example**:
```javascript
await invoke('start_mining', {
  address: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
})
```

#### Stop Mining
```typescript
invoke('stop_mining'): Promise<void>
```

**Example**:
```javascript
await invoke('stop_mining')
```

#### Get Miner Status
```typescript
invoke('get_miner_status'): Promise<MinerStatus>
```

**Returns**:
```typescript
{
  mining: boolean,
  hashrate: string,
  blocks_mined: number,
  shares_submitted: number
}
```

#### Get Mining Statistics
```typescript
invoke('get_mining_stats'): Promise<MiningStats>
```

**Returns**:
```typescript
{
  current_hashrate: string,
  average_hashrate: string,
  blocks_found: number,
  shares_accepted: number,
  shares_rejected: number,
  uptime: number,
  estimated_earnings_24h: number
}
```

### Pool Discovery

#### Discover All Pools
```typescript
invoke('discover_mining_pools'): Promise<MiningPool[]>
```

**Returns**: Array of all available pools (traditional + P2P)

#### Query DHT for Pools
```typescript
invoke('query_dht_for_pools', {
  regionFilter?: string
}): Promise<MiningPool[]>
```

**Parameters**:
- `regionFilter` (optional): Filter by region ("Europe", "Asia", "Americas", "Global")

#### P2P Pool Discovery
```typescript
invoke('p2p_discover_pools', {
  poolId?: string
}): Promise<MiningPool[]>
```

**Parameters**:
- `poolId` (optional): Specific pool ID to discover, or null for all pools

**Example**:
```javascript
// Discover all P2P pools
const pools = await invoke('p2p_discover_pools', { poolId: null })

// Discover specific pool
const pool = await invoke('p2p_discover_pools', { 
  poolId: "pool-123" 
})
```

### Pool Management

#### Create Mining Pool
```typescript
invoke('create_mining_pool', {
  address: string,
  name: string,
  description: string,
  feePercentage: number,
  minPayout: number,
  paymentMethod: string,
  region: string
}): Promise<MiningPool>
```

**Parameters**:
- `address`: Pool creator/owner wallet address
- `name`: Pool display name
- `description`: Pool description
- `feePercentage`: Pool fee (0-10)
- `minPayout`: Minimum payout threshold
- `paymentMethod`: "PPLNS" or "PPS"
- `region`: Geographic region

**Example**:
```javascript
const pool = await invoke('create_mining_pool', {
  address: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  name: "Community Pool",
  description: "Low-fee community mining pool",
  feePercentage: 1.0,
  minPayout: 0.1,
  paymentMethod: "PPLNS",
  region: "Global"
})
```

#### Announce Pool to P2P Network
```typescript
invoke('p2p_announce_pool', {
  pool: MiningPool
}): Promise<void>
```

**Parameters**:
- `pool`: Pool object to announce

#### Join Mining Pool
```typescript
invoke('join_mining_pool', {
  poolId: string,
  address: string
}): Promise<JoinedPoolInfo>
```

**Parameters**:
- `poolId`: ID of pool to join
- `address`: Your wallet address

**Returns**:
```typescript
{
  pool: MiningPool,
  stats: PoolStats,
  joined_at: number
}
```

#### Leave Mining Pool
```typescript
invoke('leave_mining_pool'): Promise<void>
```

#### Get Current Pool Info
```typescript
invoke('get_current_pool_info'): Promise<JoinedPoolInfo | null>
```

**Returns**: Current pool info or null if not in pool

#### Refresh Pool Info
```typescript
invoke('refresh_pool_info', {
  poolId: string
}): Promise<MiningPool>
```

**Parameters**:
- `poolId`: Pool to refresh

### Pool Statistics

#### Get Pool Stats
```typescript
invoke('get_pool_stats'): Promise<PoolStats>
```

**Returns**:
```typescript
{
  connected_miners: number,
  pool_hashrate: string,
  your_hashrate: string,
  your_share_percentage: number,
  shares_submitted: number,
  shares_accepted: number,
  estimated_payout_24h: number,
  last_share_time: number
}
```

#### Get Detailed Pool Stats
```typescript
invoke('get_detailed_pool_stats', {
  poolId: string
}): Promise<DetailedPoolStats>
```

**Parameters**:
- `poolId`: Pool to query

**Returns**:
```typescript
{
  total_miners: number,
  total_hashrate: string,
  total_shares: number,
  acceptance_rate: number,
  shares_in_window: number,
  blocks_found_24h: number,
  last_block_time: number
}
```

### Share Submission

#### Submit Mining Share
```typescript
invoke('submit_mining_share', {
  minerAddress: string,
  nonce: number,
  hash: string,
  difficulty: number
}): Promise<boolean>
```

**Parameters**:
- `minerAddress`: Miner's wallet address
- `nonce`: Nonce value
- `hash`: Share hash
- `difficulty`: Share difficulty

**Returns**: `true` if share accepted

#### P2P Share Submission
```typescript
invoke('p2p_submit_share', {
  share: ShareSubmission
}): Promise<void>
```

**Parameters**:
```typescript
{
  pool_id: string,
  miner_address: string,
  nonce: number,
  hash: string,
  difficulty: number,
  timestamp: number
}
```

#### Get Pool Shares
```typescript
invoke('p2p_get_shares_for_pool', {
  poolId: string,
  since?: number
}): Promise<ShareSubmission[]>
```

**Parameters**:
- `poolId`: Pool to query
- `since` (optional): Unix timestamp filter

### Payout Calculations

#### Calculate PPLNS Payout
```typescript
invoke('calculate_pplns_payout', {
  minerAddress: string,
  blockReward: number,
  nShares: number
}): Promise<number>
```

**Parameters**:
- `minerAddress`: Miner to calculate for
- `blockReward`: Block reward amount
- `nShares`: Share window size

**Returns**: Payout amount in Chiral

#### Calculate PPS Payout
```typescript
invoke('calculate_pps_payout', {
  minerAddress: string,
  blockReward: number
}): Promise<number>
```

**Parameters**:
- `minerAddress`: Miner to calculate for
- `blockReward`: Block reward amount

**Returns**: Payout amount in Chiral

#### Update Pool Hashrate
```typescript
invoke('update_pool_hashrate', {
  minerAddress: string,
  hashrate: number
}): Promise<void>
```

**Parameters**:
- `minerAddress`: Miner address
- `hashrate`: Current hashrate value

### DHT Utilities

#### Get DHT Peer ID
```typescript
invoke('get_dht_peer_id'): Promise<string>
```

**Returns**: Your DHT peer ID

#### Get Peer Count
```typescript
invoke('get_dht_peer_count'): Promise<number>
```

**Returns**: Number of connected DHT peers

#### Get Connected Peers
```typescript
invoke('get_dht_connected_peers'): Promise<PeerInfo[]>
```

**Returns**: Array of connected peer information

### P2P Pool List

#### List Local P2P Pools
```typescript
invoke('p2p_list_local_pools'): Promise<MiningPool[]>
```

**Returns**: Pools stored locally in P2P manager

---

## Data Structures

### MiningPool

```typescript
interface MiningPool {
  id: string                    // Unique pool identifier
  name: string                  // Display name
  url: string                   // Pool URL/endpoint
  description: string           // Pool description
  fee_percentage: number        // Pool fee (0-10)
  miners_count: number          // Active miners
  total_hashrate: string        // Total pool hashrate
  last_block_time: number       // Unix timestamp
  blocks_found_24h: number      // Blocks in last 24h
  region: string                // Geographic region
  status: PoolStatus            // Current status
  min_payout: number            // Minimum payout
  payment_method: PaymentMethod // PPLNS or PPS
  created_by?: string           // Creator address (P2P)
  discovered_via_p2p?: boolean  // Discovery source
}
```

### PoolStatus

```typescript
type PoolStatus = 
  | "Active"      // Pool accepting miners
  | "Maintenance" // Temporarily offline
  | "Full"        // At capacity
  | "Offline"     // Not responding
```

### PaymentMethod

```typescript
type PaymentMethod = 
  | "PPLNS"  // Pay Per Last N Shares
  | "PPS"    // Pay Per Share
```

### PoolStats

```typescript
interface PoolStats {
  connected_miners: number       // Total miners in pool
  pool_hashrate: string          // Total pool hashrate
  your_hashrate: string          // Your contribution
  your_share_percentage: number  // Your share (0-100)
  shares_submitted: number       // Your shares submitted
  shares_accepted: number        // Your accepted shares
  estimated_payout_24h: number   // Estimated 24h earnings
  last_share_time: number        // Last share timestamp
}
```

### ShareSubmission

```typescript
interface ShareSubmission {
  miner_address: string  // Wallet address
  pool_id: string        // Pool identifier
  nonce: number          // Nonce value
  hash: string           // Share hash
  difficulty: number     // Share difficulty
  timestamp: number      // Submission time
}
```

### JoinedPoolInfo

```typescript
interface JoinedPoolInfo {
  pool: MiningPool       // Pool details
  stats: PoolStats       // Current statistics
  joined_at: number      // Join timestamp
}
```

---

## Developer Tools

### Browser Console API

The `window.poolDebug` object provides testing and debugging functions:

```javascript
// Check DHT connectivity
await window.poolDebug.getPeerId()
// Returns: "12D3KooWAbc..."

await window.poolDebug.getPeerCount()
// Returns: 5

await window.poolDebug.getConnectedPeers()
// Returns: [{ id: "...", addresses: [...] }, ...]

// Pool operations
const testPool = {
  id: "test-pool-" + Date.now(),
  name: "Test Pool",
  url: "localhost:3333",
  description: "Testing pool",
  fee_percentage: 1.0,
  miners_count: 0,
  total_hashrate: "0 H/s",
  last_block_time: Date.now(),
  blocks_found_24h: 0,
  region: "Global",
  status: "Active",
  min_payout: 0.1,
  payment_method: "PPLNS"
}

// Announce pool
await window.poolDebug.announcePool(testPool)

// Discover pools
const pools = await window.poolDebug.discoverPoolsP2P()
const specificPool = await window.poolDebug.discoverPoolsP2P("pool-123")

// Share submission
const share = {
  pool_id: "pool-123",
  miner_address: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  nonce: 123456,
  hash: "0xabcdef1234567890",
  difficulty: 1000000,
  timestamp: Date.now()
}
await window.poolDebug.submitShareP2P(share)

// Get shares
const shares = await window.poolDebug.getPoolShares("pool-123")

// Direct Tauri invoke
await window.poolDebug.invoke('get_mining_stats')
```

### Testing Pool Discovery

Testing P2P pool discovery requires multiple peers:

#### Setup (3+ Peers Required)

**Terminal 1 - Bootstrap Node:**
```bash
npm run tauri dev
```

**Terminal 2 - Client A (Announcer):**
```bash
npm run tauri dev
```

**Terminal 3 - Client B (Discoverer):**
```bash
npm run tauri dev
```

#### Test Procedure

**1. Client A - Announce Pool:**
```javascript
// Open browser console
const pool = {
  id: "test-pool-" + Date.now(),
  name: "Test P2P Pool",
  url: "p2p://discovery",
  description: "Testing DHT discovery",
  fee_percentage: 1.0,
  miners_count: 0,
  total_hashrate: "0 H/s",
  last_block_time: Date.now(),
  blocks_found_24h: 0,
  region: "Global",
  status: "Active",
  min_payout: 0.1,
  payment_method: "PPLNS",
  created_by: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
}

await window.poolDebug.announcePool(pool)
console.log("Pool announced:", pool.id)
```

**2. Wait for DHT Propagation:**
- Minimum 30-60 seconds
- DHT replication factor is 3
- Records propagate to closest peers

**3. Client B - Discover Pool:**
```javascript
// Check connectivity first
console.log("Peer count:", await window.poolDebug.getPeerCount())
// Should be > 0

console.log("Connected peers:", await window.poolDebug.getConnectedPeers())

// Discover pools
const pools = await window.poolDebug.discoverPoolsP2P()
console.log("Discovered pools:", pools)

// Should include pool from Client A
```

**4. Verify Discovery:**
```javascript
const found = pools.find(p => p.id === pool.id)
if (found) {
  console.log("✅ Pool discovered successfully!")
  console.log("Pool details:", found)
} else {
  console.log("❌ Pool not found in discovery")
}
```

### Network Requirements

For successful P2P pool operation:

- **Minimum Peers**: 3 peers (1 bootstrap + 2 clients)
- **DHT Connectivity**: Peers must be connected via DHT
- **Peer Connections**: Clients need direct connections (not just to bootstrap)
- **Propagation Time**: 30-60 seconds for DHT record replication
- **Network Topology**: Mesh network preferred over hub-and-spoke

### Troubleshooting

#### Pool Not Discovered

**Diagnosis:**
```javascript
// 1. Check DHT connectivity
const peerCount = await window.poolDebug.getPeerCount()
console.log("DHT peers:", peerCount) // Should be > 0

// 2. Check peer connections
const peers = await window.poolDebug.getConnectedPeers()
console.log("Connected peers:", peers)

// 3. Try discovering specific pool
const pool = await window.poolDebug.discoverPoolsP2P("pool-id")

// 4. Check local storage
const localPools = await window.poolDebug.invoke('p2p_list_local_pools')
console.log("Local pools:", localPools)
```

**Common Issues**:
- **Low Peer Count**: Ensure bootstrap node is running
- **No Peer Connections**: Check firewall/NAT settings
- **Record Not Replicated**: Wait longer (60+ seconds)
- **DHT Errors**: Check terminal logs for "GetRecord failed"

#### Pool Announced But Not Found

**Solutions**:
1. **Wait Longer**: DHT propagation takes 30-60 seconds
2. **Verify Announcer Connectivity**: Check announcer's peer count
3. **Check Record Storage**:
   ```javascript
   // On announcer
   const local = await window.poolDebug.invoke('p2p_list_local_pools')
   console.log("Locally stored:", local)
   ```
4. **Review Terminal Logs**: Look for DHT error messages

#### "Pool Not Found" When Joining

**Cause**: Pool exists in P2P network but not in static pool lists

**Solution**: Update to latest code - join_mining_pool now checks three sources:
1. AVAILABLE_POOLS (static pools)
2. USER_CREATED_POOLS (user pools)
3. P2P_POOL_MANAGER (P2P discovered pools)

**Verification**:
```javascript
// Check all pool sources
const discovered = await window.poolDebug.invoke('discover_mining_pools')
const p2p = await window.poolDebug.discoverPoolsP2P()
console.log("All pools:", [...discovered, ...p2p])
```

#### Small Network Issues

With only 2-3 peers:
- DHT replication may fail (needs 3 peers minimum)
- Records might not propagate properly
- Discovery becomes unreliable

**Workaround**: Test with 5+ peers for production-like behavior

#### Localhost Testing

When running multiple clients on same machine:
- Use different ports for each instance
- Consider using separate machines or VMs
- Docker containers may have connectivity issues with host

---

## Recent Changes

### DHT-Based Pool Discovery (October 2025)

#### Features Added

**1. Pool Directory System**
- Central registry of all P2P pool IDs
- Stored at DHT key: `chiral:pool:directory`
- Enables discovery of all pools without knowing pool IDs
- Automatically updated when pools are created

**2. Enhanced Pool Discovery**
- `discover_pools()` now queries pool directory
- Returns all network pools when pool_id is null
- Fetches specific pool when pool_id is provided
- Supports both local and DHT pool sources

**3. Improved Pool Joining**
- `join_mining_pool()` now checks three sources:
  1. AVAILABLE_POOLS (traditional pools)
  2. USER_CREATED_POOLS (user-created pools)
  3. P2P_POOL_MANAGER (P2P discovered pools)
- Fixes "pool not found" error for P2P pools
- Seamless integration with DHT-discovered pools

**4. Complete P2P Pool System**
- Pool announcement to DHT network
- Share submission and tracking
- PPLNS and PPS payout calculations
- Detailed pool statistics
- Local and network-wide pool storage

**5. Developer Tools**
- `window.poolDebug` console API
- DHT connectivity testing functions
- Pool operation debugging
- Share submission testing

#### Files Modified

**Backend**:
- `src-tauri/src/dht.rs` - Added `put_record` and `get_record` for DHT storage
- `src-tauri/src/pool_p2p.rs` - **NEW FILE** - Complete P2P pool manager
- `src-tauri/src/pool.rs` - Enhanced share tracking, payout calculations, P2P integration
- `src-tauri/src/main.rs` - Integrated P2P pool manager initialization

**Frontend**:
- `src/pages/Mining.svelte` - Added P2P discovery UI, P2P pool badges, `window.poolDebug` API

#### Bug Fixes

- **Fixed**: Pool discovery returning empty when no pool_id specified
  - **Solution**: Implemented pool directory query
  - **Impact**: Can now discover all pools in network

- **Fixed**: "Pool not found" error when joining P2P pools
  - **Solution**: Added P2P_POOL_MANAGER lookup in join_mining_pool
  - **Impact**: P2P discovered pools can now be joined successfully

- **Fixed**: DHT connection errors with unreachable Docker IPs
  - **Solution**: Filter out unreachable addresses before connections
  - **Impact**: Improved DHT connectivity reliability

#### Code Cleanup

- **Removed**: Stratum protocol implementation (unused)
  - Deleted `relay/src/stratum.rs` (400+ lines)
  - Removed Stratum commands from `pool.rs`
  - Removed Stratum UI code from `Mining.svelte`
  - Removed command registrations from `main.rs`
  - **Reason**: Stratum was scaffolding code, never integrated into system

---

## Future Enhancements

### Planned Features

- [ ] Pool performance metrics and ranking system
- [ ] Automatic pool failover on disconnect
- [ ] Pool reputation system based on payout history
- [ ] Cross-pool statistics aggregation
- [ ] Mining profitability calculator
- [ ] Pool health monitoring dashboard
- [ ] Automatic optimal pool selection based on latency/fees
- [ ] Mobile mining support
- [ ] Alternative payout methods (PROP, SOLO, etc.)
- [ ] Mining rig management interface
- [ ] Historical earnings tracking and charts

### Community Contributions Welcome

We welcome contributions in the following areas:

- Pool discovery optimization algorithms
- Alternative payout method implementations
- Pool analytics dashboard
- Mining profitability tracking
- Hardware monitoring improvements
- Mobile app mining support
- Pool performance benchmarking tools

---

## Support & Resources

### Documentation
- **System Overview**: `docs/system-overview.md`
- **Network Protocol**: `docs/network-protocol.md`
- **API Documentation**: `docs/api-documentation.md`
- **Security**: `docs/security-privacy.md`

### Troubleshooting
- **Pool Discovery**: See "Troubleshooting" section above
- **DHT Issues**: Check terminal logs for error messages
- **Mining Problems**: Verify wallet is unlocked and has sufficient balance

### Community
- **GitHub Issues**: Report bugs and request features
- **Discussions**: Ask questions and share experiences
- **Pull Requests**: Contribute improvements

---

**Last Updated**: October 29, 2025  
**Version**: 1.0.0  
**Authors**: Chiral Network Development Team
