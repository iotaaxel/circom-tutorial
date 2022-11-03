# circom-tutorial

I am following along with the introduction documentation from [Circom](https://docs.circom.io/).  

## Compile circuit: 
1. Run `circom multiplier2.circom --r1cs --wasm --sym --c`

## Compute witness (with WASM): 
1. Navigate to the `multiplier2_js` directory
2. Add an `input.json` file with `a` and `b` values as string inputs
3. Run `node generate_witness.js multiplier2.wasm input.json witness.wtns`

## Prove circuit with Groth16 ZkSnark Protocol
### Trusted Setup (part 1) - Powers of Tau
- Note that this step is independent of the circuit.
1. Start "Powers of Tau" ceremony: `snarkjs powersoftau new bn128 12 pot12_0000.ptau -v`
2. Contribute to the ceremony: `snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v`
### Trusted Setup (part 2)
- Note that this step is circuit-specific.
1. Setup by running `snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v`
2. Generate a .zkey file with both the proving and verification keys and all phase 2 contributions: `snarkjs groth16 setup multiplier2.r1cs pot12_final.ptau multiplier2_0000.zkey`
    - Troubleshoot `multiplier2.r1cs` not found: use `../multiplier2.r1cs` in the above command
3. Contribute to part 2 of the ceremony: `snarkjs zkey contribute multiplier2_0000.zkey multiplier2_0001.zkey --name="1st Contributor Name" -v`
4. Export the verification key: `snarkjs zkey export verificationkey multiplier2_0001.zkey verification_key.json`

### Generate a Proof
1. Run `snarkjs groth16 prove multiplier2_0001.zkey witness.wtns proof.json public.json`

### Verify a Proof (option 1)
1. Run `snarkjs groth16 verify verification_key.json public.json proof.json`

Outputs: ```[INFO]  snarkJS: OK!```

### Verify a Proof with a Smart Contract (option 2)
1. Generate the Solidity code: `snarkjs zkey export solidityverifier multiplier2_0001.zkey verifier.sol`
2. Compile `verifier.sol` and deploy Verifier contract on [Remix](https://remix.ethereum.org/) (on `Goerli` testnet)
3. Generate call parameters for `verifier.sol`'s view function `verifyProof`: `snarkjs generatecall`
4. Input output of previous call to the parameters field of the verifyProof method in Remix. 
    - Returns `true` if verifiable, otherwise `false`
    - To return `false`, change a single bit in the parameters field

Note that the Goerli testnet was congested, so I might have to wait for a couple days to have sufficient funds to deploy the Verify contract.
