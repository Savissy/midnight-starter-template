# Midnight Starter Template — Student Step-by-Step Guide

This guide is tailored to this exact repository and shows how to build privacy-preserving dApps using Compact contracts and the included React frontend.

## A) Repo Understanding

### 1) What each major folder does

- `counter-contract/`: Compact contract source, generated managed artifacts, and contract-side tests.
- `counter-cli/`: Node/TypeScript scripts to run standalone setup, proof server/indexer/node flows, and environment-oriented tests.
- `frontend-vite-react/`: Vite + React app with wallet integration and contract interaction logic.
- `educational-material/`: learning/session notes and links.
- Root `package.json` + `turbo.json`: monorepo/workspace orchestration (build/compact/lint across packages).

### 2) Where contracts are, where compilation happens, and where frontend is

- Compact contract source:
  - `counter-contract/src/counter.compact`
- Compact compile command:
  - `counter-contract/package.json` script `compact`
  - command: `compact compile +0.28.0 src/counter.compact src/managed/counter`
- Compiled artifacts target:
  - `counter-contract/src/managed/counter/`
  - contains `contract/`, `keys/`, `zkir/`, and compiler metadata
- Frontend location:
  - `frontend-vite-react/`
  - counter page: `frontend-vite-react/src/pages/counter/index.tsx`
  - contract API wiring: `frontend-vite-react/src/modules/midnight/counter-sdk/api/contractController.ts`

---

## B) Environment Setup

## 1) Prerequisites by OS

The repo README requires Node, npm, Docker, Git LFS, Compact tools, and Lace wallet.

### macOS

```bash
# Node/npm (example with nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install --lts
nvm use --lts

# Git LFS
brew install git-lfs
git lfs install

# Docker Desktop
# Install from https://www.docker.com/products/docker-desktop/

# Compact tools
curl --proto '=https' --tlsv1.2 -LsSf \
  https://github.com/midnightntwrk/compact/releases/latest/download/compact-installer.sh | sh
compact update +0.28.0
```

### Linux (Ubuntu/Debian)

```bash
# Node/npm (example with nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install --lts
nvm use --lts

# Git LFS
sudo apt-get update
sudo apt-get install -y git-lfs
git lfs install

# Docker
sudo apt-get install -y docker.io docker-compose-plugin
sudo usermod -aG docker $USER

# Compact tools
curl --proto '=https' --tlsv1.2 -LsSf \
  https://github.com/midnightntwrk/compact/releases/latest/download/compact-installer.sh | sh
compact update +0.28.0
```

### Windows

- Use **WSL2** (recommended for this repo’s Docker + shell scripts).
- Install Node in WSL (nvm), Docker Desktop (with WSL integration), Git LFS in WSL.
- Install Lace in your Windows browser for wallet flows.

## 2) Node version + package manager + Nix/Docker expectations

- Package manager expected in this repo: **npm** (root has `package-lock.json` and `packageManager: npm@10.9.2`).
- Node:
  - README says `v23+`.
  - root engines say `>=18`.
  - safest practical choice: use latest LTS that works with npm 10+, and verify by running the project scripts.
- Nix:
  - no `flake.nix`, `default.nix`, or `shell.nix` found.
- Docker:
  - expected for standalone/proof/indexer/node flows through `counter-cli/*.yml` compose files.

## 3) Install repo dependencies and validate tools

From repo root:

```bash
npm install
node -v
npm -v
docker -v
git lfs version
compact check
```

## 4) Build/compile in this monorepo

```bash
# Runs turbo build across workspaces (includes compact in task graph)
npm run build

# Explicit compact pipeline
npm run compact
```

## 5) Configure environment files

```bash
cp counter-cli/.env_template counter-cli/.env
cp frontend-vite-react/.env_template frontend-vite-react/.env
```

Populate values based on your target network/wallet/prover settings.

## 6) Run local standalone/devnet-like environment supported by this repo

This repo supports an undeployed local stack through `counter-cli` using Docker services and setup scripts.

Terminal 1:

```bash
npm run setup-standalone
```

Terminal 2:

```bash
npm run dev:frontend
```

Optional: run proof server directly via compose script in `counter-cli`:

```bash
cd counter-cli
npm run ps-undeployed
```

If a command is unclear in your environment, discover it from scripts instead of guessing:

```bash
cat package.json
cat counter-cli/package.json
cat counter-contract/package.json
cat frontend-vite-react/package.json
cat turbo.json
```

---

## C) Smart Contract Workflow (Compact)

## 1) Create a new Compact contract module in this repo

### Step 1: Add contract file

Create, for example:

```text
counter-contract/src/private-counter.compact
```

Start from the existing pattern in `counter-contract/src/counter.compact`.

### Step 2: Add compile output folder

Use a dedicated managed folder, for example:

```text
counter-contract/src/managed/private-counter
```

### Step 3: Add/adjust npm script in `counter-contract/package.json`

```json
{
  "scripts": {
    "compact:private-counter": "compact compile +0.28.0 src/private-counter.compact src/managed/private-counter"
  }
}
```

### Step 4: Re-export generated contract API

Update `counter-contract/src/index.ts` to export your new generated module path.

## 2) Core privacy concepts (high-level)

- **Public ledger state**: values visible on-chain (example: `round` in current counter).
- **Private state**: user/app local encrypted or private context (handled via `PrivateStateProvider` and project private-state helpers).
- **Proof-backed transitions**: contract calls are validated using proving/verifying artifacts (`keys/`, `zkir/`) produced at compile time.
- **Provider separation**:
  - public data provider (ledger reads)
  - private state provider (local/private data)
  - proof provider (proof generation/validation workflow)

## 3) How to structure contract + app pieces

- **Datum/state (ledger)**: define minimal public state in Compact (`ledger ...`).
- **Redeemers/inputs (actions)**: expose `export circuit ...` functions for transitions.
- **Validation logic**: guard state transitions in circuits; keep invariants explicit.
- **Events/logging**:
  - Contract-level event semantics are represented via transaction data and app logs in this repo.
  - Frontend/CLI logs use `pino`; contract effects are observed via state subscription and tx metadata.

## 4) Compile/build using this repo scripts

From repo root:

```bash
# compile contract workspace via turbo graph
npm run compact

# or only contract compile
cd counter-contract
npm run compact
```

## 5) Artifacts produced

For current counter contract, compile outputs under:

- `counter-contract/src/managed/counter/contract/*` (generated JS/TS contract bindings)
- `counter-contract/src/managed/counter/keys/*` (prover/verifier keys)
- `counter-contract/src/managed/counter/zkir/*` (intermediate artifacts)
- `counter-contract/src/managed/counter/compiler/contract-info.json`

Frontend copies key material at build time into:

- `frontend-vite-react/public/midnight/counter/keys/*`
- `frontend-vite-react/public/midnight/counter/zkir/*`

via script `frontend-vite-react` → `copy-contract-keys`.

## 6) Basic tests using existing setup

Contract tests (logic/contract-side):

```bash
cd counter-contract
npm run test
# or compile+test
npm run test:compile
```

CLI/provider integration-focused tests:

```bash
cd counter-cli
npm run private-provider-testing
npm run public-provider-testing
npm run zk-provider-testing
npm run indexer-client-testing
npm run wallet-testing
```

Environment tests (when configured):

```bash
cd counter-cli
npm run test-undeployed
npm run test-preview
npm run test-preprod
```

## 7) Common errors and fixes

- **`compact: command not found`**
  - Install Compact tools and verify with `compact check`.
- **Compiler version mismatch**
  - Contract script pins `+0.28.0`; run `compact update +0.28.0`.
- **Missing keys/zkir in frontend runtime**
  - Run frontend build script (`npm run build` in frontend or root build pipeline) so `copy-contract-keys` executes.
- **Docker services not reachable**
  - Ensure Docker daemon is running and ports `6300`, `8088`, `9944` are free.
- **Wallet not detected in browser**
  - Confirm Lace extension installed/enabled and network matches app config.
- **Wrong command guess**
  - Always inspect package scripts/turbo config first:

```bash
cat package.json
cat counter-cli/package.json
cat counter-contract/package.json
cat frontend-vite-react/package.json
cat turbo.json
```

---

## D) dApp Workflow

## 1) How frontend connects to Midnight in this repo

- Wallet connection is handled by wallet widget/controller code under:
  - `frontend-vite-react/src/modules/midnight/wallet-widget/api/walletController.ts`
- Contract deployment/join/calls are handled by:
  - `frontend-vite-react/src/modules/midnight/counter-sdk/api/contractController.ts`
- State read pattern:
  - Public state from `publicDataProvider.contractStateObservable(...)`
  - Private state from `privateStateProvider.get(...)`
  - Combined into derived UI state stream (`state$`).

## 2) Minimal end-to-end example using current repo

### Example idea: “Private study streak counter”

- Public: total streak rounds
- Private: personal secret note/count metadata in private state provider

### (1) Contract logic

Use existing `counter.compact` as baseline:

```compact
pragma language_version >= 0.20;

import CompactStandardLibrary;

export ledger round: Counter;

export circuit increment(): [] {
  round.increment(1);
}
```

Then extend in a new module with stricter transition checks as you learn.

### (2) Deploy/initialize flow

From UI, call deploy:

- Hook `useContractSubscription()` provides `onDeploy()`.
- It uses `ContractController.deploy(...)`.
- Deployment creates contract and initializes private state (`getOrCreateInitialPrivateState`).

### (3) Frontend calls

- `deployNew` button calls `onDeploy()`.
- `increment` button calls `deployedContractAPI.increment()`.
- UI renders:
  - `derivedState.round` (public)
  - `derivedState.privateState.privateCounter` (private/local)

## 3) Security notes (practical)

- Never hardcode secrets or mnemonics in frontend code.
- Keep private state in provider-backed storage, not plaintext logs.
- Validate network IDs before connecting wallets.
- Treat copied proving artifacts as build assets; protect deployment pipeline integrity.
- In production, enforce:
  - strict CSP
  - dependency pinning and lockfile integrity
  - minimal logging of sensitive wallet/account info

---

## E) Project Roadmap (Student-Friendly)

## Phase 1 — Hello-world private contract + UI

Difficulty: **Easy/Medium**

Checklist:

- [ ] Run `npm install` and `npm run build` from root.
- [ ] Configure `.env` files in `counter-cli/` and `frontend-vite-react/`.
- [ ] Run standalone setup and frontend.
- [ ] Deploy counter from UI and increment once.
- [ ] Observe public + private state in UI cards.

Learn/skim first:

- `counter-contract/src/counter.compact`
- `contractController.ts` state/deploy flow
- wallet controller connection flow

## Phase 2 — Real features (access control + richer private updates)

Difficulty: **Medium**

Checklist:

- [ ] Add a new Compact module (e.g., private streak with role checks).
- [ ] Add compile script and managed output folder.
- [ ] Export generated bindings in `counter-contract/src/index.ts`.
- [ ] Integrate new module in frontend SDK controller.
- [ ] Add tests in `counter-contract/src/test` and `counter-cli/src/test`.

Learn/skim first:

- Existing provider tests in `counter-cli/src/test/*`
- Contract compile artifacts (`managed/.../keys`, `zkir`, `contract`)
- RxJS state composition in frontend SDK

## Phase 3 — Production hardening

Difficulty: **Medium/Hard**

Checklist:

- [ ] Add CI steps for build + compact + tests.
- [ ] Add lint/typecheck gates per workspace.
- [ ] Add error handling and retry/backoff around provider/network calls.
- [ ] Add deployment runbooks for preview/preprod.
- [ ] Perform security checklist review (secrets, wallet flow, logging, supply chain).

Learn/skim first:

- `DEPLOYMENT_PROCEDURE.md`
- `counter-cli` environment scripts (`setup`, preview/preprod scripts)
- existing logger/test patterns for observability and regressions

---

## Quick Command Reference (root)

```bash
npm install
npm run build
npm run compact
npm run setup-standalone
npm run dev:frontend
npm run lint
```

## If you need to discover the “right” command without guessing

```bash
cat package.json
cat counter-contract/package.json
cat counter-cli/package.json
cat frontend-vite-react/package.json
cat turbo.json
```

This is the fastest and most reliable way to align commands with this specific repo.
