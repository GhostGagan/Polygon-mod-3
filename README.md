# Polygon-mod-3

## NFT Collection Project with Circom Circuit Verification

This project involves generating an NFT collection using DALL-E 2 or Midjourney, storing items on IPFS using Pinata.cloud, deploying an ERC721 or ERC1155 contract to the Goerli Ethereum Testnet, and performing circuit verification using Circom.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
3. [NFT Generation](#nft-generation)
4. [IPFS Storage](#ipfs-storage)
5. [Smart Contract Deployment](#smart-contract-deployment)
6. [Token Mapping](#token-mapping)
7. [Minting and Transferring NFTs](#minting-and-transferring-nfts)
8. [Circom Circuit Implementation](#circom-circuit-implementation)
9. [Proof Generation and Verification](#proof-generation-and-verification)
10. [Testing the Contract](#testing-the-contract)

## Prerequisites

- Node.js and npm
- Hardhat
- Circom
- SnarkJS
- Ethereum wallet (e.g., MetaMask)
- Goerli Testnet ETH
- Sepolia or Mumbai Testnet ETH
- Polygon Mumbai Testnet MATIC

## Project Setup

1. Clone the repository:
    ```bash
    git clone https://github.com/your-repo/nft-circom-project.git
    cd nft-circom-project
    ```

2. Install dependencies:
    ```bash
    npm install
    ```

## NFT Generation

Generate your NFT collection using DALL-E 2 or Midjourney and save the images in the `images` directory.

## IPFS Storage

1. Create an account on [Pinata.cloud](https://pinata.cloud/).
2. Upload the images to Pinata and obtain the IPFS hashes.
3. Update the `metadata` files in the `metadata` directory with the corresponding IPFS hashes.

## Smart Contract Deployment

1. Update the contract configuration in `hardhat.config.js`.
2. Deploy the ERC721 or ERC1155 contract to the Goerli Testnet:
    ```bash
    npx hardhat run scripts/deploy.js --network goerli
    ```

## Token Mapping

Map the NFT collection using the Polygon network token mapper.

## Minting and Transferring NFTs

1. Write Hardhat scripts to batch mint and transfer NFTs between Ethereum and Polygon Mumbai using FxPortal Bridge.
2. Execute the minting and transfer scripts:
    ```bash
    npx hardhat run scripts/mintAndTransfer.js --network goerli
    ```

## Circom Circuit Implementation

1. Create the `circuit.circom` file with the following content:
    ```circom
    pragma circom 2.0.0;

    template IsEqual() {
        signal input A;
        signal input B;
        signal output out;

        out <== (A === B) ? 1 : 0;
    }

    component main = IsEqual();
    ```

2. Compile the circuit:
    ```bash
    circom circuit.circom --r1cs --wasm --sym --c
    ```

3. Generate the trusted setup and zkey:
    ```bash
    snarkjs powersoftau new bn128 12 pot12_0000.ptau
    snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
    snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau
    snarkjs groth16 setup circuit.r1cs pot12_final.ptau circuit_0000.zkey
    snarkjs zkey contribute circuit_0000.zkey circuit_final.zkey --name="Final contribution" -v
    snarkjs zkey export verificationkey circuit_final.zkey verification_key.json
    ```

4. Generate the proof using inputs `A=0` and `B=1`:
    ```bash
    echo '{"A":"0","B":"1"}' > input.json
    snarkjs groth16 prove circuit_final.zkey input.json proof.json public.json
    ```

## Proof Generation and Verification

1. Deploy the verifier contract to Sepolia or Mumbai Testnet:
    ```bash
    npx hardhat run scripts/deployVerifier.js --network sepolia
    ```

2. Call the `verifyProof` method on the verifier contract and assert the output is true:
    ```js
    const verifier = await ethers.getContractAt("Verifier", verifierAddress);
    const proof = JSON.parse(fs.readFileSync('proof.json'));
    const publicSignals = JSON.parse(fs.readFileSync('public.json'));

    const isValid = await verifier.verifyProof(proof.a, proof.b, proof.c, publicSignals);
    console.log("Proof valid:", isValid);
    ```

## Testing the Contract

1. Implement tests to check the `balanceOf` on Mumbai and ensure the correct functionality of the NFT minting and transferring process.

---

This readme file covers all the essential steps to implement, deploy, and verify an NFT collection project with Circom circuit verification. Make sure to customize the paths, network names, and addresses according to your specific project setup.
