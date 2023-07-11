# Multi-Party ECDSA

This project is a Rust implementation of {t, n}-threshold ECDSA (Elliptic Curve Digital Signature Algorithm) by the Era team. Era retains all rights to the Era-MPC scheme.

Threshold ECDSA consists of two protocols:
- Distributed Key Generation (DKG) for generating secret key shares.
- Signing using the GG20 algorithm with secret key shares.

ECDSA is widely used in cryptocurrencies such as Bitcoin, Ethereum (secp256k1 curve), NEO (NIST P-256 curve), and more. This library can be used to create MultiSig and ThresholdSig encrypted wallets.

## Running GG20 Demo

In the following steps, the Era team will demonstrate how to generate a 2-of-3 threshold signature private key and sign a message with 2 parties. This demo instance is meant to illustrate how users can recover their MPC accounts.

### Setup

1. You need to install [Rust](https://rustup.rs/) and [GMP library](https://gmplib.org) (optional) on your computer.
2. - Run `cargo build --release --examples`.
   - Don't have GMP installed? Use this command instead:
     ```bash
     cargo build --release --examples --no-default-features --features curv-kzen/num-bigint
     ```
     However, please note that it may result in decreased efficiency.

   Either command will generate the binary files in the `./target/release/examples/` folder.
3. `cd ./target/release/examples/`

### Start SM Server

Run `./gg20_sm_manager`.

This will start an HTTP server at `http://127.0.0.1:8000`. Other parties will use this server for communication. Please note that the communication channel is neither encrypted nor authenticated. In production, you must encrypt and authenticate messages between parties.

### Run Key Generation

> Parameter Explanation:
> - -r Room (optional, default: default-keygen, use the same room for the same batch of operations)
> - -t Threshold (required)
> - -n Number of Participants (required)
> - -i Index (required)
> - -o Output Share File (required)

Open 3 terminal tabs and run:

1. `./gg20_keygen -t 1 -n 3 -i 1 -o share1.json`
2. `./gg20_keygen -t 1 -n 3 -i 2 -o share2.json`
3. `./gg20_keygen -t 1 -n 3 -i 3 -o share3.json`

Each command corresponds to one party. After key generation, you will have 3 new files: `share1.json`, `share2.json`, `share3.json`, which correspond to the local secret key shares of each party.

### Run Signing

> Parameter Explanation:
> - -r Room (optional, default: default-signing, use the same room for the same batch of operations)
> - -p Participants (required, 1,2 means using share files from index 1 and 2 for signing)
> - -d Signature Data (required, RLP encoded transaction message hashed with Keccak256)
> - -l Share File (required)

Since Era-MPC adopts a 2-of-3 scheme (t=1, n=3), any two parties can sign a message. Run:

1. `./gg20_signing -p 1,2 -d '1c8aff950685c2ed4bc3174f3472287b56d9517b9c948127319a09a7a36deac8' -l share1.json`
2. `./gg20_signing -p 1,2 -d '1c8aff950685c2ed4bc3174f3472287b56d9517b9c948127319a09a7a36deac8' -l share2.json`

share1 will generate a signature with one secret key share, and share2 will generate another signature with a different secret key share. The two secret key share signatures can be combined to create the final signature.