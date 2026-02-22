<div align="center">

# erc8004-search

**Rust SDK for the [ERC-8004 Semantic Search](https://github.com/qntx/8004)**

[![Crates.io](https://img.shields.io/crates/v/erc8004-search.svg)](https://crates.io/crates/erc8004-search)
[![Documentation](https://docs.rs/erc8004-search/badge.svg)](https://docs.rs/erc8004-search)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg)](#license)

Discover on-chain AI agents through semantic search — with zero configuration and built-in [x402](https://www.x402.org/) micropayment support.

</div>

---

## Highlights

- **Zero-config default** — Ships with a built-in hosted endpoint (`https://search.qntx.fun`); start querying in two lines of code.
- **x402 payment middleware** — Automatic EVM (EIP-155) and Solana payment signing when the service requires micropayments.
- **Typed & ergonomic API** — Strongly-typed request/response models matching the [ERC-8004 v1 spec](https://github.com/qntx/erc8004-search-service/blob/main/docs/SEMANTIC_SEARCH_STANDARD_V1.md), with builder patterns and cursor pagination.
- **Production-ready** — Connection pooling, configurable timeouts, structured error handling, and `tracing` instrumentation.

## Installation

Add to your `Cargo.toml`:

```sh
cargo add erc8004-search
```

## Quick Start

```rust
use erc8004_search::SearchClient;
use alloy_signer_local::PrivateKeySigner;

#[tokio::main]
async fn main() -> erc8004_search::Result<()> {
    let signer: PrivateKeySigner = std::env::var("PRIVATE_KEY")?.parse()?;
    let client = SearchClient::builder()
        .evm_signer(signer)
        .build()?;

    let response = client.search("DeFi lending agent").await?;
    for item in &response.results {
        println!("#{} {} — score {:.3}", item.rank, item.name, item.score);
    }
    Ok(())
}
```

For custom endpoints, chain `.base_url("https://...")` on the builder.

## Advanced Usage

### Filtered Search

Use the structured `Filters` builder to narrow results by on-chain metadata:

```rust
use erc8004_search::{SearchClient, SearchRequest, Filters, Protocol};

let client = SearchClient::new();

let request = SearchRequest::new("MCP tool server")
    .limit(5)
    .min_score(0.5)
    .filters(
        Filters::new()
            .chain_id(8453)
            .active(true)
            .protocols([Protocol::Mcp, Protocol::A2a])
    );

let resp = client.execute(request).await?;
```

### Cursor Pagination

Automatically walk through all pages of results:

```rust
// Collect up to 10 pages of results in a single call.
let all_results = client.search_all("blockchain agent", 10).await?;
println!("Total results: {}", all_results.len());
```

### Health & Capabilities

```rust
let health = client.health().await?;
println!("Status: {}", health.status);

let caps = client.capabilities().await?;
println!("Max query length: {}", caps.limits.max_query_length);
```

### Custom HTTP Settings

```rust
use std::time::Duration;

let client = SearchClient::builder()
    .timeout(Duration::from_secs(30))
    .user_agent("my-app/1.0")
    .build()?;
```

## Feature Flags

| Feature  | Default | Description                                 |
|----------|---------|---------------------------------------------|
| `evm`    | **yes** | EVM chain payment support via `r402-evm`    |
| `solana` | no      | Solana chain payment support via `r402-svm` |

Enable Solana support:

```toml
[dependencies]
erc8004-search = { version = "0.2", features = ["evm", "solana"] }
```

## Examples

The `examples/` directory contains runnable demos:

```sh
# Basic search (uses built-in endpoint by default)
PRIVATE_KEY="0x..." cargo run --example search

# Filtered search with pagination
PRIVATE_KEY="0x..." cargo run --example search_filters

# Override with a custom endpoint
PRIVATE_KEY="0x..." SEARCH_URL="https://your-server.com" cargo run --example search
```

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or <https://www.apache.org/licenses/LICENSE-2.0>)
- MIT License ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/licenses/MIT>)

at your option.

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in this project shall be dual-licensed as above, without any additional terms or conditions.
