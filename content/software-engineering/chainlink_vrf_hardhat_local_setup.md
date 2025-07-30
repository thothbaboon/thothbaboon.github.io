+++
title = "How to Use Chainlink VRF on Hardhat for dApp Local Development"
description = "Learn how to set up Chainlink VRF mock locally to develop and test a dApp with an experience close to a Node.js backend"
date = 2025-06-24
+++

Chainlink VRF is excellent for generating verifiably random numbers in smart contracts. However, setting up a complete dApp with VRF for local development can be challenging at first.

While Chainlink provides [documentation for using a VRF Coordinator mock with Remix](https://docs.chain.link/vrf/v2-5/subscription/test-locally), this approach is limited to simple contract testing. What if you want to build and test a full dApp locally with a smooth development experience?

This article takes it to the next level, showing you how to integrate the VRF mock into a complete local development environment using Hardhat. You'll learn how to set up a dApp that feels as responsive as working with a traditional Node.js backend, while still leveraging the power of blockchain and Chainlink's VRF.

You can find the complete project [here on my GitHub](https://github.com/antoineprdhmm/vrf-hardhat-local-setup). This article focuses on the most important concepts, so feel free to clone the project locally to fully understand the implementation.

## The Project

The dApp is a "Flip a coin" app that demonstrates how to integrate Chainlink VRF into a complete application. Here's how it works:

1. User clicks on a Flip button in the UI
2. The app calls the smart contract `flip` method
3. The contract requests a random number from Chainlink VRF
4. When the random number is received, the contract determines the result:
   - If the number is even → "Heads"
   - If the number is odd → "Tails"
5. The result is emitted as an event and displayed in the UI

### Tech Stack

- **Hardhat** - Blockchain development environment and testing framework
- **Ethers v6** - Library for interacting with the blockchain (both frontend and backend)
- **Chainlink VRF v2.5** - For generating verifiable random numbers
- **Next.js** - React framework for the frontend (production-ready setup)

## Smart Contracts and Deployment

### FlipCoin - Main Smart Contract

The contract is located at [hardhat/contracts/FlipCoin.sol](https://github.com/antoineprdhmm/vrf-hardhat-local-setup/blob/main/hardhat/contracts/FlipCoin.sol).

Since it needs to request random numbers from Chainlink VRF, it must inherit from `VRFConsumerBaseV2Plus` (V2.5 is sometimes called V2Plus).

Because it inherits this interface, it must implement `fulfillRandomWords` which is the callback function that receives the random words requested.

When the user clicks on the "Flip" button in the app, it's going to call the `flip` method of the smart contract.

```solidity
function flip() public {
    // Request random number from Chainlink VRF
    // Will revert if subscription is not set and funded.
    uint256 requestId = s_vrfCoordinator.requestRandomWords(
        VRFV2PlusClient.RandomWordsRequest({
            keyHash: s_keyHash,
            subId: s_subscriptionId,
            requestConfirmations: REQUEST_CONFIRMATIONS,
            callbackGasLimit: CALLBACK_GAS_LIMIT,
            numWords: NUM_WORDS,
            extraArgs: VRFV2PlusClient._argsToBytes(
                VRFV2PlusClient.ExtraArgsV1({nativePayment: false})
            )
        })
    );

    // Store pending request to track which user made the request
    s_requests[requestId] = msg.sender;

    // Emit event with request ID for debugging and manual fulfillment
    emit Flipping(msg.sender, requestId);
}
```

The method triggers a request to get the random words, which is fulfilled asynchronously.

**⚠️ Important Note:** When using the Mock, the request must be fulfilled manually as [explained in the documentation](https://docs.chain.link/vrf/v2-5/subscription/test-locally#fulfill-the-vrf-request). In production, this happens automatically.

Once `VRFCoordinatorV2Plus` receives the random words, it's going to call the callback method `fulfillRandomWords` on the FlipCoin contract.

This is why it's important to keep track of which sender is associated with the request ID:

```solidity
s_requests[requestId] = msg.sender; 
```

When receiving the random words, the contract computes the result (Tails or Heads), finds which address is associated with this request, and emits an event with the result:

```solidity
function fulfillRandomWords(
    uint256 requestId,
    uint256[] calldata randomWords
) internal override {
    address player = s_requests[requestId];
    require(player != address(0), "Request not found");

    // Clean up
    delete s_requests[requestId];

    string memory result;

    // Use the random value to determine the result of the coin flip
    if (randomWords[0] % 2 == 0) {
        result = "Heads";
    } else {
        result = "Tails";
    }

    emit Flipped(player, result);
}
```

The app simply listens to the events, filtering by player address to only receive the events of the connected player.

### Application Flow

Here is a schema describing the flow of the application:

![flow](/chainlink_vrf_hardhat_local_setup/flow.png)

### VRFCoordinatorV2_5Mock

The remaining parts to explore for the FlipCoin smart contract are the constructors and attributes. These will be used to request the random words.

```solidity
constructor(
    uint256 subscriptionId,
    bytes32 keyHash,
    address vrfCoordinator
) VRFConsumerBaseV2Plus(vrfCoordinator) {
    s_subscriptionId = subscriptionId;
    s_keyHash = keyHash;
}
```

The VRF Coordinator is the contract that we call to request the random words (see the `flip` method above), and that will call our callback function with the random words when they are ready.

This means before deploying FlipCoin, we need to deploy a VRF Coordinator.

Chainlink provides a mock that is very convenient to use. The only thing to do is to create a file called `VRFCoordinatorV2_5Mock.sol` in the hardhat contracts folder next to the `FlipCoin.sol` contract, and import the mock inside.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;
import "@chainlink/contracts/src/v0.8/vrf/mocks/VRFCoordinatorV2_5Mock.sol";
```

The VRF Coordinator constructor requires some parameters, which are fees and prices. Any values work, but lower values are more convenient for testing:

```tsx
const vrfCoordinatorFactory = await hre.ethers.getContractFactory(
    "VRFCoordinatorV2_5Mock"
);
const vrfCoordinator = (await vrfCoordinatorFactory.deploy(
    1000, // set low fee (more convenient for testing)
    10, // set low gas price (more convenient for testing),
    5200000000000000 // WEI for 1 LINK
)) as unknown as VRFCoordinatorV2_5Mock;
```

### VRF Subscription

To request random numbers, a subscription should be created, funded, and provided to the coordinator. The subscription will be debited automatically by the VRF Coordinator.

[See the documentation explaining how it works in detail](https://docs.chain.link/vrf/v2-5/overview/subscription#request-and-receive-data).

```tsx
// Create a new subscription
const subscriptionTx = await vrfCoordinator.createSubscription();
const subscriptionReceipt = await subscriptionTx.wait();

if (!subscriptionReceipt) {
    throw new Error("Failed to create subscription");
}

if (!(subscriptionReceipt.logs[0] instanceof hre.ethers.EventLog)) {
    throw new Error("Unexpected receipt log type");
}

const subscriptionId = subscriptionReceipt.logs[0].args.subId;

// Fund the subscription with LINK tokens
await vrfCoordinator.fundSubscription(subscriptionId, 1000000000000000);
```

### Deploying the FlipCoin contract

Now that the VRF Coordinator is deployed and the subscription is funded, FlipCoin can be deployed:

```tsx
const FlipCoinFactory = await hre.ethers.getContractFactory("FlipCoin");
const flipCoin = await FlipCoinFactory.deploy(
    subscriptionId,
    "0xd89b2bf150e3b9e13446986e571fb9cab24b13cea0a43ea20a6049a85cc807cc", // arbitrary bytes32 for testing
    vrfCoordinatorAddress
);
await flipCoin.waitForDeployment();
```

The second argument is the `keyHash`, which is the maximum gas willing to pay for a request. In our situation, it doesn't matter as it's only for testing.

The last thing to do is to tell the VRF Coordinator that FlipCoin can use the subscription:

```tsx
const flipCoinAddress = await flipCoin.getAddress();
await vrfCoordinator.addConsumer(subscriptionId, flipCoinAddress);
```

## Unit Testing

Now that the VRF Coordinator and FlipCoin contracts are deployed, the subscription funded, and FlipCoin allowed to request random words using this subscription, let's flip some coins.

The first thing is to call flip with one signer and get the request ID from the event emitted at the end of the flip method:

```tsx
const flipCoinWithPlayer = flipCoin.connect(player);

const flipTx = await flipCoinWithPlayer.flip();
const flipReceipt = await flipTx.wait();

if (!flipReceipt) {
    throw new Error("Flip transaction failed");
}

// Get the requestId from the RandomWordsRequested event
const flippingEvent = vrfCoordinator.interface.parseLog(
    flipReceipt.logs[0]
);

if (!flippingEvent) {
    throw new Error("Request event not found");
}

const { requestId } = flippingEvent.args;
```

Then, we need to fulfill the VRF request manually. Obviously in production it's done automatically. But the mock requires us to do it.

We could do it by calling this method: `vrfCoordinator.fulfillRandomWords`.

But the mock provides a more convenient method - `fulfillRandomWordsWithOverride` - which allows us to choose which words to return. This way we can test specific cases, and even use a real random words generator if we want random words:

```tsx
let fulfillRandomWordsTx =
  await vrfCoordinator.fulfillRandomWordsWithOverride(
    requestId,
    await flipCoin.getAddress(),
    [9] // choose which values to return
  );
let fulfillRandomWordsReceipt = await fulfillRandomWordsTx.wait();
```

FlipCoin's `fulfillRandomWords` will be called with [9], and it's going to emit an event with the result.

This result can be tested based on the value provided to `fulfillRandomWordsWithOverride`.

## Testing with the App

Unit tests are nice, but what if you want to play with your app locally?

Well, it's exactly the same thing, except that:

- `flip` method will be called from the app, when clicking on the Flip button
- We need to figure out a way to call `fulfillRandomWordsWithOverride` for local testing. Again, in production there will be no need to call this method, it's just to fulfill the VRF request on the mock.

Remember, when calling the `flip` method, it emits a `Flipping` event with the request ID inside.

In the app, we can listen for this event, and `console.log` the request ID.

To fulfill the VRF request, I made a little script: [fulfillVrfRequest.ts](https://github.com/antoineprdhmm/vrf-hardhat-local-setup/blob/main/hardhat/scripts/fulfillVrfRequest.ts)

This script takes 2 parameters:

- The ID of the request to fulfill
- The number to send to `fulfillRandomWords`

So when clicking on the Flip button:

- A Flipping event with the request ID is emitted
- The request is pending

When executing the script:

- The request is fulfilled with the given number
- A Flipped event is emitted with the result: Tails or Heads

## Conclusion

This is how you can use VRF to write unit tests or play with your app locally, without relying on any test network.

This approach provides several benefits:

- **Full Control**: Complete control over the VRF contract, especially with the override method
- **Fast Development**: No network delays or gas costs during development
- **Predictable Testing**: You can test specific scenarios by controlling the random values
- **Developer Experience**: The development experience is close to working with a Node.js backend

This setup allows you to develop and test your VRF-powered dApp efficiently while maintaining the same patterns you'll use in production.
