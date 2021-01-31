---
title: A Practical Guide To Building Zero Knowledge dApps
date: 2020-02-03
author: Kendrick Tan
categories: ["ethereum", "crypto"]
---

---

**[Grab the source code for the blogpost here.](https://github.com/kendricktan/hello-world-zk-dapp)**

Recommended prereading:

- [public-key cryptography](https://stackoverflow.com/questions/2853889/how-does-public-key-cryptography-work)
- [snarkjs tutorial](https://github.com/iden3/circom/blob/master/TUTORIAL.md)
- [truffle](https://www.trufflesuite.com/docs/truffle/quickstart)
- [ethers](https://docs.ethers.io/ethers.js/html/api-contract.html#connecting-to-existing-contracts)

---

## Prelude

For the past few months, I've been building [a couple of](https://github.com/barryWhiteHat/maci) [toy dApp projects](https://github.com/kendricktan/simple-zk-rollups) on Ethereum that ultilize zero knowledge proofs, specifically zk-SNARKs.

As there is little material out there regarding building dApps that ultilizes zero knowledge proofs, I thought I would share my experience building one in a blog post.

The goal of this blog post is to act as a practical guide to help readers build their first zero knowledge dApp (i.e. no maths here sorry).

Note: This blog post assumes that the reader has a _basic_ understanding of [public-key cryptography](https://www.cloudflare.com/learning/ssl/how-does-public-key-encryption-work/), and how to [deploy](https://www.trufflesuite.com/docs/truffle/getting-started/running-migrations) and [interact](https://docs.ethers.io/ethers.js/html/api-contract.html) with smart contracts in JavaScript.

## Overview

We will be building a zk-dApp that proves if a user belongs to a certain group or not, without revealing who that particular user is.

The user flow of said zk-dApp might look like:

![](https://i.imgur.com/ZheTWPV.png)

##### Figure 1: Zk-dApp Identity Verification User Flow

While the development pipeline looks something like:

1. Write zero knowledge circuits.
2. Generate `solidity` library to verify the written zero knowledge circuits.
3. Write out smart contract logic, and integrate the generated `solidity` library from step 2.
4. Deploy contracts.
5. Generate proof locally, and verify it on-chain.

### Environment and Tooling

Just like how you don't need to understand the HTTP procotol to do web development these days, zero knowledge dApp development has sufficient modern tooling that allows developers who don't necessarily have the math background in cryptography (e.g. me), to build applications ultilizing zero knowledge proofs.

My recommended programming languages/tools would be:

- `JavaScript/TypeScript` as it has a very rich ecosystem and support for Ethereum
- `Solidity` for smart contracts due to its maturity and community
- [`Truffle`](https://www.trufflesuite.com/docs/truffle/quickstart) to deploy your smart contracts
- [`Circom`](https://github.com/iden3/circom) for writing zero knowledge proof circuits

## Zero Knowledge Circuits

Our goal here is to create a circuit which when supplied with a `private key`, and an array of `public keys`, constructs a proof if and only if the private key corresponds to one of the public keys (i.e. it will fail if the `private key` does not correspond to one of the `public keys` as the constraint fails and no proof can be generated).

In pseudocode land, it will be something like:

```javascript
// Note that a private key is a scalar value (int)
// whereas a public key is a point in space (Tuple[int, int])
const zk_identity = (private_key, public_keys) => {
  // derive_public_from_private is a function that
  // returns a public key given a private key
  derived_public_key = derive_public_from_private(private_key)
  for (let pk in public_keys):
    if derived_public_key === pk:
      return true
  return false
}
```

We will now start writing the zero knowledge circuits using `circom`. For an overview of the `circom` syntax read the [circom tutorial](https://github.com/iden3/circom/blob/master/TUTORIAL.md).

We will first install the necessary dependencies and create the project folders which will house our zero knowledge circuit logic: `circuits/circuit.circom`.

```bash
npm install circom circomlib snarkjs websnark

mkdir contracts
mkdir circuits
mkdir -p build/circuits

touch circuits/circuit.circom
```

##### <PROJECT_ROOT>

We will start off by including the necessary building blocks:

```javascript
include "../node_modules/circomlib/circuits/bitify.circom";
include "../node_modules/circomlib/circuits/escalarmulfix.circom";
include "../node_modules/circomlib/circuits/comparators.circom";

template PublicKey() {
  // Note: private key needs to be hashed, and then pruned
  // to make sure its compatible with the babyJubJub curve
  signal private input in;
  signal output out[2];

  component privBits = Num2Bits(253);
  privBits.in <== in;

  var BASE8 = [
    5299619240641551281634865583518297030282874472190772894086521144482721001553,
    16950150798460657717958625567821834550301663161624707787222815936182638968203
  ];

  component mulFix = EscalarMulFix(253, BASE8);
  for (var i = 0; i < 253; i++) {
    mulFix.e[i] <== privBits.out[i];
  }

  out[0] <== mulFix.out[0];
  out[1] <== mulFix.out[1];
}
```

##### <PROJECT_ROOT>/circuits/circuit.circom

What the `PublicKey` template does is it derives the public key (`out`) from the supplied private key (`in`) on the [babyJubJub curve](https://iden3-docs.readthedocs.io/en/latest/iden3_repos/research/publications/zkproof-standards-workshop-2/baby-jubjub/baby-jubjub.html) (i.e. it's the `derive_public_from_private` function from the pseudocode above).

Once we have the building blocks, we can now construct the main logic for our zero knowledge circuit: verifying if the user is within a group or not:

```javascript
include ...

template PublicKey() {
  ...
}

template ZkIdentity(groupSize) {
  // Public Keys in the smart contract
  // Note: this assumes that the publicKeys
  // are all unique
  signal input publicKeys[groupSize][2];

  // Prover's private key
  signal private input privateKey;

  // Prover's derived public key
  component publicKey = PublicKey();
  publicKey.in <== privateKey;

  // Make sure that derived public key needs to
  // matche to at least one public key in the
  // smart contract to validate their identity
  var sum = 0;

  // Create a component to check if two values are
  // equal
  component equals[groupSize][2];
  for (var i = 0; i < groupSize; i++) {
    // Helper component to check if two
    // values are equal
    // We don't want to use ===
    // as that will fail immediately if
    // the predicate doesn't hold true
    equals[i][0] = IsEqual();
    equals[i][1] = IsEqual();

    equals[i][0].in[0] <== publicKeys[i][0];
    equals[i][0].in[1] <== publicKey.out[0];

    equals[i][1].in[0] <== publicKeys[i][1];
    equals[i][1].in[1] <== publicKey.out[1];

    sum += equals[i][0].out;
    sum += equals[i][1].out;
  }

  // equals[i][j].out will return 1 if the values are equal
  // and 0 if the values are not equal
  // Therefore, if the derived public key (a point in space)
  // matches a public keys listed in the smart contract, the sum of
  // all the equals[i][j].out should be equal to 2
  sum === 2;
}


// Main entry point
component main = ZkIdentity(2);
```

##### <PROJECT_ROOT>/circuits/circuit.circom

You can now compile, setup, and generate a verifier (`solidity` library) for your circuit:

```bash
$(npm bin)/circom circuits/circuit.circom -o build/circuits/circuit.json

## snarkjs setup might take a few seconds
$(npm bin)/snarkjs setup --protocol groth -c build/circuits/circuit.json --pk build/circuits/provingKey.json --vk build/circuits/verifyingKey.json

## Generate solidity lib to verify proof
$(npm bin)/snarkjs generateverifier --pk build/circuits/provingKey.json --vk build/circuits/verifyingKey.json -v contracts/Verifier.sol

## You should now have a new "Verifier.sol" in your contracts directory
## $ ls contracts
## Migrations.sol Verifier.sol
```

##### <PROJECT_ROOT>

Note that we're generating our `provingKey` and `verifyingKey` with the `groth` protocol as we want to be able [to use websnark to generate the proofs as they are significantly faster than snarkjs](https://github.com/iden3/websnark#important-please-be-sure-you-run-your-setup-with---protocol-groth--websnark-only-generates-groth16-proofs).

Once you've done the above, we have finished a zero knowledge logic. The next section (Smart Contract Verifier), we will be looking at the generated `Verifier.sol` and how we can interact with it nicely.

Note: I've also added some faq below regarding zero knowledge circuits.

#### How Is Supplying The Private Key In The Circuit Safe?

Noticed we specified the `privateKey`'s signal to be a `private` one. And because of that, the generated proof will not contain any information about `private` signals, but it will about `public` ones.

#### Ok, But Can't A User Supply A Wrong List Of Public Keys?

We will talk more about that in the Smart Contract Verifier section below.

## Smart Contract Verifier

After completing the [zero knowledge circuits](/posts/practical_guide_build_zk_dapps/#zero-knowledge-circuits) step, a `solidity` library named `Verifier.sol` should be generated. If you inspect the file's contents, you should be able to see the following function inside:

```javascript
...

  function verifyProof(
            uint[2] memory a,
            uint[2][2] memory b,
            uint[2] memory c,
            uint[4] memory input
        ) public view returns (bool r) {
        Proof memory proof;
        proof.A = Pairing.G1Point(a[0], a[1]);
        proof.B = Pairing.G2Point([b[0][0], b[0][1]], [b[1][0], b[1][1]]);
        proof.C = Pairing.G1Point(c[0], c[1]);
        uint[] memory inputValues = new uint[](input.length);
        for(uint i = 0; i < input.length; i++){
            inputValues[i] = input[i];
        }
        if (verify(inputValues, proof) == 0) {
            return true;
        } else {
            return false;
        }
    }

...
```

##### <PROJECT_ROOT>/contracts/Verifier.sol

That is the helper to verify the validity of the proof. The `verifyProof` function accepts 4 parameters, but we're only interested in the `input` parameter as it represents the public signals (i.e. input signals that are not private inside of template `ZkIdentity` in `<PROJECT_ROOT>/circuits/circuit.circom`).

**Using the `input` parameter, we can validate against the existing set of public keys in the smart contract logic to negate [the prover generating proofs with invalid public keys](/posts/practical_guide_build_zk_dapps/#ok-but-cant-a-user-supply-a-wrong-list-of-public-keys).**

This concept will be more concrete once we write the logic to validate user identity:

```javascript
pragma solidity 0.5.11;

import "./Verifier.sol";

contract ZkIdentity is Verifier {
    address public owner;
    uint256[2][2] public publicKeys;

    constructor() public {
        owner = msg.sender;
        publicKeys = [
            [
                11588997684490517626294634429607198421449322964619894214090255452938985192043,
                15263799208273363060537485776371352256460743310329028590780329826273136298011
            ],
            [
                3554016859368109379302439886604355056694273932204896584100714954675075151666,
                17802713187051641282792755605644920157679664448965917618898436110214540390950
            ]
        ];
    }

    function isInGroup(
        uint256[2] memory a,
        uint256[2][2] memory b,
        uint256[2] memory c,
        uint256[4] memory input // public inputs
    ) public view returns (bool) {
        if (
            input[0] != publicKeys[0][0] &&
            input[1] != publicKeys[0][1] &&
            input[2] != publicKeys[1][0] &&
            input[3] != publicKeys[1][1]
        ) {
            revert("Supplied public keys do not match contracts");
        }

        return verifyProof(a, b, c, input);
    }
}
```

##### <PROJECT_ROOT>/contracts/ZkIdentity.sol

We created a new contract `ZkIdentity.sol` which inherits from the generated `Verifier.sol`, with a initial group of 2 people (`publicKeys`). It contains a function called `isInGroup`, which given a proof, first validates that the public signals matches the specified group in the smart contract. If it doesn't, revert. If it does, then return if the proof passed or not.

And that is pretty much it. The logic above satisfies our goal of proving if a user belong to a certain group or not, without revealing who that particular user is.

Deploy your contracts to your preferred network before moving on.

## Generating Proofs and Interacting With Deployed Smart Contract

Once you have written your zero knowledge circuits and written your smart contract logic, all that is left to do is to generate the proofs and call the smart contract function `isInGroup`.

Since there is a lot of boilerplate code to generate the proofs and instantiate smart contracts in JS, I will be demonstrating the pseudocode for generating the proof, and validating the proof on the smart contract side. If you want the complete version, [you can find it in this file](https://github.com/kendricktan/hello-world-zk-dapp/blob/master/packages/scripts/index.js#L97).

```javascript
// Assuming below already exists
const provingKey // provingKey.json
const circuit // zero-knowledge circuit we wrote
const zkIdentityContract // Zk-Identity contract instance

const privateKey  // Private key that corresponds to one of the public key in the smart contract
const publicKeys = [
  [
      11588997684490517626294634429607198421449322964619894214090255452938985192043n,
      15263799208273363060537485776371352256460743310329028590780329826273136298011n
  ],
  [
      3554016859368109379302439886604355056694273932204896584100714954675075151666n,
      17802713187051641282792755605644920157679664448965917618898436110214540390950n
  ]
]

const circuitInputs = {
  privateKey,
  publicKeys
}

const witness = circuit.calculateWitness(circuitInputs)
const proof = groth16GenProof(witness, provingKey)

const isInGroup = zkIdentityContract.isInGroup(
  proof.a,
  proof.b,
  proof.c,
  witness.publicSignals
)
```

Once you've [converted that into JavaScript](https://github.com/kendricktan/hello-world-zk-dapp/blob/master/packages/scripts/index.js#L97) and executed it, you've just proved that your user belongs in a group, without revealing who that user is!

## Conclusion

Zero knowledge tooling has come very far in the past 3 years. You don't need a phd in cryptography to start building zero knowledge applications these days (tho it'll help while debugging).

**Again, [you can grab the source code for the blogpost here](https://github.com/kendricktan/hello-world-zk-dapp).**
