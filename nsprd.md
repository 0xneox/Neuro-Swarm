
# Project Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Core Components](#core-components)
4. [Frontend Application](#frontend-application)
5. [Blockchain Integration](#blockchain-integration)
6. [Database & State Management](#database--state-management)
7. [Compute Infrastructure](#compute-infrastructure)
8. [Security Model](#security-model)
9. [Performance Optimization](#performance-optimization)
10. [Deployment & Environment](#deployment--environment)
11. [Developer Guide](#developer-guide)
12. [Appendix: API Reference](#appendix-api-reference)

## Project Overview

NeuroSwarm is a decentralized AI compute network built on the Solana blockchain, enabling users to contribute GPU computational resources to process AI tasks and earn token rewards. The platform leverages browser-based WebGPU computing with multi-level fallbacks for broad compatibility and uses Solana for fast, low-cost transactions.

### Key Features
- **Decentralized Compute Network**: Harnesses idle GPU resources across user devices.
- **WebGPU Integration**: Primary execution with WebGL, WASM, and CPU fallbacks.
- **Solana Blockchain Integration**: Fast, low-cost token rewards and verification.
- **Task Scheduling System**: Automated task assignment based on node capabilities.
- **Token Economics**: Staking, rewards, and reputation mechanisms via NLOV tokens.
- **Interactive Dashboard**: Real-time monitoring and node management interface.

### Technology Stack
- **Frontend**: React, TypeScript, Vite, Tailwind CSS
- **Blockchain**: Solana, Anchor Framework, Rust
- **Compute**: WebGPU, WebGL, WASM, CPU
- **Database**: PostgreSQL via Supabase
- **State Management**: Zustand, React Context, Custom Hooks
- **Build Tools**: Vite, ESBuild

## System Architecture

NeuroSwarm's architecture comprises interconnected layers for compute, blockchain, application, and data management.

### High-Level Architecture
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│               │      │               │      │               │
│  Browser      │◄────►│  NeuroSwarm   │◄────►│  Solana       │
│  Client       │      │  Frontend     │      │  Blockchain   │
│  (React App)  │      │               │      │               │
└─────────┬─────┘      └───────┬───────┘      └───────────────┘
          │                    │
          ▼                    ▼
┌───────────────┐      ┌───────────────┐
│               │      │               │
│  Supabase     │      │  WebGPU       │
│  (PostgreSQL) │      │  Compute      │
│               │      │  Engine       │
└───────────────┘      └───────────────┘

### Component Interactions
1. Browser client initializes WebGPU compute capabilities.
2. Client registers node with Solana blockchain.
3. Task service distributes tasks to nodes based on capabilities.
4. Nodes process tasks, submit proofs to blockchain.
5. Blockchain validates proofs, distributes rewards.
6. Dashboard displays real-time metrics and node status.

### Key System Flows

**Node Registration Flow**:
- User connects wallet.
- Device capabilities detected.
- Node registered on blockchain.
- Node begins polling for tasks.

**Task Execution Flow**:
- Task submitted to network.
- Scheduler assigns to suitable node.
- Node computes result.
- Proof submitted to blockchain.
- Task validated, rewards distributed.

## Core Components

### ComputeNode (`src/core/ComputeNode.ts`)

Manages a participant's lifecycle in the network.

```typescript
export class ComputeNode {
  private id: string;
  private device: Device;
  private status: 'online' | 'offline' | 'maintenance';
  private currentTask: AITask | null;
  private metrics: NodeMetrics;
  private stake: number;
  private ownerKey: PublicKey | null;

  // Core methods
  async initialize(): Promise<void>;
  async startNode(): Promise<void>;
  async stopNode(): Promise<void>;
  async executeTask(task: AITask): Promise<TaskResult>;
  reportMetrics(): NodeMetrics;
  async submitTaskProof(taskId: string, result: TaskResult): Promise<void>;
  async stakeTokens(): Promise<void>;
}
```
State Management (using Zustand):

```typescript
export const useNodeStore = create<NodeState>((set) => ({
  isActive: false,
  metrics: { taskCount: 0, successRate: 100, averageExecutionTime: 0, totalEarnings: 0 },
}));
```
WebGPUCompute (src/core/WebGPUCompute.ts)
Handles GPU computation with fallbacks.

```typescript
export class WebGPUCompute {
  private device: GPUDevice | null;
  private fallback: ComputeFallback | null;
  private capabilities: ComputeCapabilities = { webgpu: false, webgl2: false, wasm: false, cpu: true };

  // Core methods
  async compute(data: Float32Array, shader: string): Promise<Float32Array>;
  private async webGPUCompute(data: Float32Array, shader: string): Promise<Float32Array>;
  private async setupWebGL2Fallback(): Promise<ComputeFallback>;
  private async setupWasmFallback(): Promise<ComputeFallback>;
  private async setupCPUFallback(): Promise<ComputeFallback>;
}
```
Fallback Hierarchy:
- WebGPU (primary)
- WebGL2 (fragment shaders)
- WASM (specialized compute)
- CPU (web workers)

- DeviceManager (src/core/DeviceManager.ts)
- Manages device registration and monitoring.

```typescript
export class DeviceManager {
  // Core methods
  async registerDevice(specs: DeviceSpecs): Promise<string>;
  async getDeviceStatus(deviceId: string): Promise<DeviceStatus>;
  async updateDeviceStatus(deviceId: string, status: DeviceStatus): Promise<void>;
  async getAvailableDevices(): Promise<Device[]>;
}
```
TaskManager (src/core/TaskManager.ts)
Handles task lifecycle.

```typescript
export class TaskManager {
  // Core methods
  async submitTask(taskType: string, requirements: TaskRequirements): Promise<string>;
  async assignTask(taskId: string, nodeId: string): Promise<boolean>;
  async processTaskResult(taskId: string, result: TaskResult): Promise<boolean>;
  async validateTask(taskId: string, result: TaskResult): Promise<boolean>;
}
```
TaskScheduler (src/core/TaskScheduler.ts)
Optimizes task allocation.

```typescript
export class TaskScheduler {
  private taskQueue: PriorityQueue<AITask>;
  private nodeRegistry: Map<string, ComputeNode>;

  // Core methods
  scheduleTask(task: AITask): Promise<string | null>;
  registerNode(node: ComputeNode): void;
  findSuitableNode(task: AITask): ComputeNode | null;
}
```

Frontend A
- Built with React and TypeScript, using Vite.
Key UI Components

- Dashboard (src/components/Dashboard.tsx): Displays network stats, active nodes, tasks, and earnings.
- DevicePanel (src/components/DevicePanel.tsx): Manages device registration and status.
- AITasksPanel (src/components/AITasksPanel.tsx): Shows task listings, creation, and results.
- EarningsPanel (src/components/EarningsPanel.tsx): Tracks earnings and payouts.
- NetworkStats (src/components/NetworkStats.tsx): Real-time network metrics.
- ReferralPanel (src/components/ReferralPanel.tsx): Manages referral program.


State Management
- Zustand: Core node state.
- React Context: Global app state.
- Supabase Subscriptions: Real-time updates.
- Local/Session Storage: Persistent state.

Blockchain Integration
- SolanaService (src/services/SolanaService.ts)
- Handles blockchain interactions.

```typescript
export class SolanaService implements ISolanaService {
  private connection: Connection;
  private program: Program<Idl>;

  // Core methods
  async registerDevice(deviceSpecs: DeviceSpecs): Promise<string>;
  async submitTask(owner: PublicKey, taskType: string, requirements: TaskRequirements, payer: Keypair): Promise<TransactionSignature>;
  async submitTaskProof(proofData: TaskProofData, payer: Keypair): Promise<TransactionSignature>;
  async distributeReward(recipient: PublicKey, amount: number, payer: Keypair, tokenAccount: PublicKey): Promise<TransactionSignature>;
  async stakeTokens(user: PublicKey, amount: number, payer: Keypair, tokenAccount: PublicKey): Promise<string>;
}
```
ComputeToken (src/contracts/ComputeToken.ts)
Manages token economy.

```typescript
export class ComputeToken {
  private mintLimits = {
    hourly: 1000,
    daily: 10000,
    minStake: 1000,
    maxStake: 100000,
    slashingPenalty: 0.1,
    cooldownPeriod: 24 * 60 * 60 * 1000
  };

  // Core methods
  async mintTokens(recipient: PublicKey, amount: number, proof?: Buffer): Promise<string>;
  async stakeTokens(user: PublicKey, amount: number): Promise<boolean>;
  async unstakeTokens(user: PublicKey, amount: number): Promise<boolean>;
  async distributeRewards(tasks: Task[]): Promise<void>;
}
```
Solana Program (swarm_network/programs/swarm_network/src)
Implemented in Rust using Anchor.
rust
```typescript
#[program]
pub mod swarm_network {
  use super::*;

  pub fn initialize(ctx: Context<Initialize>, bump: u8, decimals: u8) -> Result<()> {}
  pub fn register_device(ctx: Context<RegisterDevice>, gpu_model: String, vram: u64, hash_rate: u64, referrer: Option<Pubkey>) -> Result<()> {}
  pub fn create_task(ctx: Context<CreateTask>, task_id: String, requirements: TaskRequirements) -> Result<()> {}
  pub fn complete_task(ctx: Context<CompleteTask>, task_id: String, result: TaskResult) -> Result<()> {}
  pub fn claim_reward(ctx: Context<ClaimReward>, amount: u64) -> Result<()> {}
}
```
State Accounts:
- DeviceAccount
- TaskAccount
- UserAccount
- NetworkState
- Database & State Management

SupabaseService (src/services/SupabaseService.ts)
- Manages database operations.

```typescript
export class SupabaseService {
  private client: SupabaseClient;

  // Core methods
  async getNetworkStats(): Promise<NetworkStats>;
  async subscribeToTasks(callback: (task: AITask) => void): () => void;
  async logTaskProof(proofData: TaskProofData): Promise<void>;
  async getEarningHistory(days: number, walletAddress?: string): Promise<EarningHistory[]>;
}
```
Database Schema
nodes:
sql
```typescript
CREATE TABLE nodes (
  id UUID PRIMARY KEY,
  owner_wallet TEXT NOT NULL,
  gpu_model TEXT,
  vram INTEGER,
  hash_rate INTEGER,
  status TEXT,
  last_seen TIMESTAMP WITH TIME ZONE,
  reputation FLOAT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
tasks:
sql
CREATE TABLE tasks (
  id UUID PRIMARY KEY,
  type TEXT NOT NULL,
  status TEXT NOT NULL,
  requirements JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  completed_at TIMESTAMP WITH TIME ZONE,
  assigned_node UUID REFERENCES nodes(id),
  result JSONB,
  user_id TEXT,
  priority INTEGER DEFAULT 1
);
earnings:
sql
CREATE TABLE earnings (
  id UUID PRIMARY KEY,
  node_id UUID REFERENCES nodes(id),
  wallet_address TEXT NOT NULL,
  amount FLOAT NOT NULL,
  tasks_completed INTEGER,
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  transaction_hash TEXT
);
device_stats:
sql
CREATE TABLE device_stats (
  id UUID PRIMARY KEY,
  node_id UUID REFERENCES nodes(id),
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  cpu_usage FLOAT,
  memory_usage FLOAT,
  temperature FLOAT,
  available_vram FLOAT,
  current_load FLOAT
);
referrals:
sql
CREATE TABLE referrals (
  id UUID PRIMARY KEY,
  referrer_wallet TEXT NOT NULL,
  referred_wallet TEXT NOT NULL,
  status TEXT NOT NULL,
  reward_amount FLOAT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  claimed_at TIMESTAMP WITH TIME ZONE
);
```
Compute Infrastructure
- WebGPU Implementation
- Uses modern web standards for GPU computing.

```typescript
private async webGPUCompute(data: Float32Array, shader: string): Promise<Float32Array> {
  if (!this.device) throw new Error('WebGPU not initialized');

  const device = this.device;
  const inputBuffer = device.createBuffer({
    size: data.byteLength,
    usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_DST
  });
  // Compute pipeline setup and execution
}
```
Fallback Mechanisms
WebGL2:

```typescript
private async setupWebGL2Fallback(): Promise<ComputeFallback> {
  const canvas = document.createElement('canvas');
  const gl = canvas.getContext('webgl2');
  // WebGL2 compute implementation
}
WASM: Specialized compute modules.
CPU: Web workers for parallel processing.
Security Model
Rate Limiting
typescript
export class RateLimiter {
  private requestCounts: Map<string, number> = new Map();

  checkLimit(identifier: string, limit: number): boolean;
  trackRequest(identifier: string): void;
}
```
Limits:
- Requests: 60/min (standard), 300/min (verified)
- Tasks: 100/hour (standard), 500/hour (verified)

Task Validation
```typescript
export class TaskValidator {
  validate(task: AITask, result: any): boolean;
  addValidationRule(taskType: TaskType, validator: (result: any) => boolean): void;
}
```
Node Reputation
```typescript
interface NodeReputation {
  score: number;
  taskCount: number;
  successCount: number;
  failureCount: number;
  lastActive: number;
  isBanned: boolean;
}
```
Performance Optimization
- WebGPU: Optimized shader pipelines.
- Task Scheduling: Priority-based queuing.
- Database: Indexed queries, caching.
- Frontend: Code splitting, lazy loading.

Deployment & Environment


git clone <repo-url>
cd NeuroSwarm
npm install
cp .env.example .env
npm run dev


env
# Solana
VITE_SOLANA_NETWORK=devnet
VITE_PROGRAM_ID=your_program_id

# Supabase
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_KEY=your_supabase_key

# Features
VITE_ENABLE_WEBGPU=true
VITE_ENABLE_REFERRALS=true

# Security
VITE_RATE_LIMIT_TASKS=100
VITE_NODE_MIN_STAKE=1000

- Build Configuration (vite.config.ts)
- Node polyfills for Solana
- WebGPU/WebGL optimization
- Bundling for core libraries

Deployment Process

- Build: npm run build
- Deploy frontend to hosting
- Deploy Solana program
- Configure Supabase

Structure
- NeuroSwarm/
- ├── src/
- │   ├── components/        # React UI components
- │   ├── config/            # App configuration
- │   ├── contracts/         # Blockchain interfaces
- │   ├── core/              # Compute engine
- │   ├── hooks/             # React hooks
- │   ├── idl/               # Solana program interfaces
- │   ├── providers/         # Context providers
- │   ├── services/          # Backend services
- │   ├── types/             # TypeScript definitions
│-    └── utils/             # Utilities
- ├── public/                # Static assets
- ├── server/                # Backend code
- ├── tests/                 # Test suite
- └── swarm_network/         # Solana program


Dev Tasks
- New Task Type:
- Update TaskType in src/types/index.ts
- Add handler in TaskService
- Update UI in AITasksPanel
- Add validation in SecurityService
- New Compute Method:
- Update WebGPUCompute
- Implement fallback
- Update capability detection
- Blockchain Feature:
- Update Anchor program
- Deploy program
- Update IDL
- Implement in SolanaService
