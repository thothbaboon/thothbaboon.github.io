---
title: "Authentication with Ethereum and Metamask in a dApp"
date: 2022-12-18
description: "How to allow your users to log into your app using Metamask instead of creating an account and filling their login and password."
---

## Quick Reminder About How Login/Password Authentication Works

We are used to log into web applications using a unique identifier and
a password. We fill the login form with our unique identifier and
password, and click on the login button.

The **unique identifier** can be an email, a phone number, a
username, … whatever. The goal is to uniquely identify the user. It’s
a public information.

The **password** is there to prove that it’s not someone else
trying to use our identity. That’s why a password is a private
information.

The server then search into the database for the account with this
unique identifier, and check if the password is the correct one. If
yes, we are authenticated.

The server can send back a JWT token to the web app that will be sent
to the server with each request.

![simplified login password authentication](/authentication-with-ethereum-and-metamask-in-a-dapp/login_password.png)

## Using an Ethereum Account

When creating an account on the Ethereum blockchain, we get a
**private key**. The **public key** is generated from the
private key. And **addresses** are derived from the public key.

To do authentication, the principle is the same: we need a public and
unique identifier, and a secret information that only the owner has.

The identifier is the address, and the private key is the secret
information.

But unlike a password, **we never share a private key** ! We share
our password with the web app when creating an account or log into the
app. But the private key of our Ethereum account must never be shared.

Instead, we are gonna use **digital signatures**.

The server gives us some text: **the challenge**

- We sign this challenge using our private key (it can be done with Metamask)
- From the original challenge and the signed challenge, the server is able to find back the
  signatory address
- If the signatory address is our address, we are authenticated as it shows we are owner of the
  private key, which means we are the owner of the Ethereum account.
- The server can send back a JWT token to the web app that will be sent to the server with each
  request.

![ethereum account authentication using Metamask](/authentication-with-ethereum-and-metamask-in-a-dapp/login_metamask.png)

## Ethereum Account Authentication Using Metamask

Let’s see some code

### Web APP

Here is a complete login flow on the font end side.

```js
// login with Metamask (pick the address)
const accounts = await window.ethereum.request({
  method: "eth_requestAccounts",
});

// this is the address with which we wants to authenticate
const publicAddress = accounts[0];

// call the server to get the challenge
// (getAuthChallenge is a HTTP GET to the server)
const challenge = await getAuthChallenge(publicAddress);

// sign the challenge sent by the server with Metamask
const signChallenge = (id: string, challenge: string): Promise<string> => {
  return new Promise((resolve, reject) => {
    window.ethereum.sendAsync(
      {
        method: "personal_sign",
        params: [challenge, id],
        from: id,
      },
      function (err: any, result: any) {
        if (err) {
          reject(err);
        }
        if (result.error) {
          reject(result.error);
        }

        resolve(result.result);
      }
    );
  });
};
const signature = await signChallenge(publicAddress, challenge);

// send the signature to the server
// this is a HTTP POST to the server, responding with the JWT if success
const jwt = await getJwt(publicAddress, signature);

// now we can use this JWT in our HTTP request
```

### Server

On the back end side, it’s really simple. Here are the main parts of
the authentication flow.

```js
// /////
// SERVER

// to generate a challenge, wecan use uuid
import { v4 as uuidv4 } from "uuid";
const generateChallenge = (): string => {
  return uuidv4();
};

// ..........

// to verify a signature, we can use a library like web3js or etherjs
// here is an example with web3js

import Web3 from "web3";
const verifySignature = async (
  publicAddress: string,
  challenge: string,
  signature: string
): Promise<boolean> => {
  const address = await web3.eth.accounts.recover(challenge, signature);
  return address.toLowerCase() === publicAddress.toLowerCase();
};
```

## Conclusion

Doing authentication with an Ethereum account is pretty easy ! Instead
of comparing the hash of the password sent and the hash of the
password in the database, we ask the user to sign a message we his
private key. We can then verify the signature and authenticate the
user.

🎉🎉
