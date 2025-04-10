# Solana Escrow Protocol 🔄🔒  
**Secure Token Exchange System with PDA-Managed Vaults**  
*An Anchor-based implementation of cross-chain-like token swaps on Solana*

[![Anchor v0.29.0](https://img.shields.io/badge/Anchor-0.29.0-3cf)](https://anchor-lang.com)
[![SPL Token](https://img.shields.io/badge/SPL%20Token-3.5.0-blue)](https://spl.solana.com/token)
[![Solana 1.16](https://img.shields.io/badge/Solana-1.16.18-purple)](https://solana.com)

## Overview 📜
A decentralized escrow system enabling atomic swaps between SPL tokens using program-derived addresses (PDAs) for secure asset custody. Implements maker-taker pattern with automated settlement and refund capabilities.

![Escrow State Diagram](https://mermaid.ink/svg/pako:eNqNkkFPwzAMhf9KlNdN2gVKQ0hI3EBC4rKJg7m0cUjTtYk6rRP_Dl3HkBCXHZ7t7_-e7ShWkFqD3sJbC5uJ0Cv0rq0bUYh1s7U7cG1fB1G6zqOoGk9R6gP4FmIh7Kz4n6Lg6xqjDvUaU4dGdWZqHX2bKz8DqH0A5QY1Q5DgHcY6o5q7bYQ1Qh1JX4jBGGUzQr8M4qM7mQ2wD9B7qHqoGigqKEooN9C00HdQ9lD1UPRQd1C1kDdQtFA2UNRQVlBUsBqgHqEZoR2hG6EboB2gG6EfoR-hG6AboBuhH6EfoRugG6AboR-hH6EboBugG6EfoR-hG6AboBuhH6EboBmhHaEdYj7AeYT3CeoT1COsR1iOsR1iPsB5hPcJ6hPUI6xHWI6xHWI-wHmE9wnqE9QjrEdYjrEdYj7AeYT3CeoT1COsR1iOsR1iPsB5hPcJ6hPUI6xHWI6xHWI-wHmE9wnqE9QjrEdYjrEdYj7AeYT3CeoT1COsR1iOsR1iPsB5hPcJ6hPUI6xHWI6xHWI_wD8N6S3Y)

## Key Features 🛡️
- **Atomic Swap Mechanism**: Trustless token exchanges between two parties
- **PDA Vault System**: Secure custody of escrowed assets
- **Multi-Token Support**: SPL token standard compliance (Fungible & NFT-ready)
- **Automated Settlement**: Pre-encoded business logic for swap execution
- **Safety Guarantees**:
  - Maker refund capability
  - Taker deposit verification
  - Anti-theft protections through PDA ownership

## Technical Architecture 🧠
```mermaid
sequenceDiagram
    participant Maker
    participant Taker
    participant EscrowProgram
    participant TokenProgram
    
    Maker->>EscrowProgram: Initialize Escrow (make)
    EscrowProgram->>TokenProgram: Create Vault PDA
    EscrowProgram->>TokenProgram: Transfer Token A to Vault
    
    Taker->>EscrowProgram: Execute Swap (take)
    EscrowProgram->>TokenProgram: Verify Taker Deposit (Token B)
    EscrowProgram->>TokenProgram: Transfer Vault (Token A) to Taker
    EscrowProgram->>TokenProgram: Transfer Token B to Maker
    
    alt Refund Scenario
    Maker->>EscrowProgram: Initiate Refund
    EscrowProgram->>TokenProgram: Return Token A to Maker
    end
```

## Core Components 🔩

### 1. Escrow State Account
```rust
#[account]
#[derive(InitSpace)]
pub struct Escrow {
    pub seed: u64,      // 8
    pub maker: Pubkey,  // 32
    pub mint_a: Pubkey, // 32
    pub mint_b: Pubkey, // 32
    pub receive: u64,   // 8
    pub bump: u8        // 1
}                       // Total: 113 bytes
```

### 2. Instruction Handlers
| Instruction | Parameters | Description |
|-------------|------------|-------------|
| `make`      | seed, receive | Initialize escrow & deposit Token A |
| `take`      | -          | Execute swap by depositing Token B |
| `refund`    | -          | Cancel escrow and reclaim Token A |

### 3. Security Model
```rust
// PDA Validation in Take instruction
#[account(
    mut,
    close = maker,
    has_one = maker,
    has_one = mint_a,
    has_one = mint_b,
    seeds = [b"escrow", maker.key().as_ref(), escrow.seed.to_le_bytes().as_ref()],
    bump = escrow.bump
)]
escrow: Account<'info, Escrow>,
```

## Development Setup ⚙️

### Requirements
- Rust 1.65+
- Solana CLI 1.16.18+
- Anchor 0.29.0

### Installation
```bash
git clone https://github.com/your-org/solana-atomic-swap.git
cd solana-atomic-swap
anchor build
solana-test-validator --reset
anchor deploy
```

## Usage Examples 💼

### 1. Initialize Escrow (Maker)
```typescript
const [escrowPDA] = await PublicKey.findProgramAddressSync(
  [
    Buffer.from("escrow"),
    maker.publicKey.toBuffer(),
    new BN(seed).toArrayLike(Buffer, "le", 8)
  ],
  program.programId
);

await program.methods.make(seed, receiveAmount)
  .accounts({
    maker: maker.publicKey,
    mintA: tokenAMint,
    mintB: tokenBMint,
    escrow: escrowPDA,
    vault: vaultATA,
    systemProgram: SystemProgram.programId,
    tokenProgram: TOKEN_PROGRAM_ID,
  })
  .signers([maker])
  .rpc();
```

### 2. Execute Swap (Taker)
```typescript
await program.methods.take()
  .accounts({
    taker: taker.publicKey,
    maker: makerAddress,
    escrow: escrowPDA,
    vault: vaultATA,
    takerAtaA: takerTokenAATA,
    takerAtaB: takerTokenBATA,
    makerAtaB: makerTokenBATA,
    tokenProgram: TOKEN_PROGRAM_ID,
  })
  .signers([taker])
  .rpc();
```

## Security Model 🔐
1. **PDA Authority Enforcement**
   ```rust
   let signer_seeds = [&[
       b"escrow",
       self.maker.to_account_info().key.as_ref(),
       &self.escrow.seed.to_le_bytes()[..],
       &[self.escrow.bump],
   ]];
   ```
2. **Cross-Program Invocation Safety**
3. **State Validation Guards**:
   - `has_one` constraints for mint consistency
   - Close-on-completion patterns
   - Associated token account verification


## License 📄
Apache 2.0 - See [LICENSE](LICENSE) for full terms

