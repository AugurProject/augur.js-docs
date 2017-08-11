Accounts
========
```javascript
/**
 * Create an account using the augur.accounts.register method.
 */
 const password = "thisismysupersecurepassword";

 augur.accounts.register(password, account => console.log("Account:", account));

// output
account = {
  address: "0x0ab5f1a15f2462eba143ecc8e1733f44dfe019bf",
  derivedKey: <Buffer ...>,
  keystore: { /* ... */ },
  privateKey: <Buffer ...>
};

const keystore = account.keystore;

augur.accounts.login(keystore, password, account => console.log("Account:", account));

// output
account = {
  address: "0x0ab5f1a15f2462eba143ecc8e1733f44dfe019bf",
  derivedKey: <Buffer ...>,
  keystore: { /* ... */ },
  privateKey: <Buffer ...>
};

// Augur.js does not store any account data. Augur.js simply returns the important information.
```
augur.js includes a trustless account management system. The purpose of the accounts system is to allow people to use Augur without needing to run an Ethereum node themselves, as running a full Ethereum node can be resource-intensive.

To use the account system, the user specifies a password. Everything else is done automatically for the user. The only requirement for the password is that it be at least 6 characters long.

A private key (+ derived public key and address) is automatically generated for the user.  A secret key derived from the password using PBKDF2, along with a random 128-bit initialization vector, is used to encrypt the private key (using AES-256). Nothing is stored by augur.js, the account object will be returned to the callback provided or simply returned if no callback is provided.

The Augur UI will handle your account information for you, but if you are using augur.js on it's own you will need to manage the account yourself. If you are sending a non-call transaction then you will be required to sign the transaction with your privateKey.

<aside class="notice">Since the user's account key is an ordinary Ethereum private key, the user's key (and therefore their funds and Reputation) can be used with any Ethereum node. Therefore, although the accounts system is managed using an ordinary web server, since the user's funds are neither tied to nor controlled by our server, the accounts are still decentralized in the ways that (in our opinion) matter.</aside>

Legacy Accounts
---------------

```javascript
/**
 * Create an account using the augur.web.register method.
 */

var username = "tinybike";
var password = "tinypassword";

augur.web.register(username, password, function (account) {
    console.log("Account:", account);
});

// output
Account: {
    handle: "tinybike",
    privateKey: <Buffer ...>,
    address: "0x09129dcde8e0f510ffb2a715709b69f5c4a42361",
    nonce: 0
}

/**
 * The user is automatically logged in after registering.  When logged in,
 * the user's data is kept in the the augur.web.account object.  Logging out
 * unsets the augur.web.account object.
 */

augur.web.logout();
console.log("Account:", augur.web.account);

// output
{}

/* Logging in both sets and returns the augur.web.account object: */

augur.web.login(username, password, function (account) {
    console.log(augur.web.account);
});

// output
{
    handle: "tinybike",
    privateKey: <Buffer ...>,
    address: "0x09129dcde8e0f510ffb2a715709b69f5c4a42361",
    nonce: 0
}
```

augur.js includes a trustless account management system.  The purpose of the accounts system is to allow people to use Augur without needing to run an Ethereum node themselves, as running a full Ethereum node can be resource-intensive.

To use the account system, the user specifies a username and password.  Everything else is done automatically for the user.  The only requirement for the password is that it be at least 6 characters long.

A private key (+ derived public key and address) is automatically generated for the user.  A secret key derived from the password using PBKDF2, along with a random 128-bit initialization vector, is used to encrypt the private key (using AES-256).  The username, encrypted private key, and initialization vector are stored in the browser's `localStorage`.

Transaction signing and serialization is entirely carried out in the browser using [ethereumjs-tx](https://github.com/ethereum/ethereumjs-tx) and the `eth_sendRawTransaction` RPC; the plaintext private key never touches our servers at any time.

Invoking contract methods is unchanged from normal use (from the client's perspective).  The Augur API functions can simply be called in the usual way (e.g., `augur.reputationFaucet`), and augur.js will default to client-side transactions if the user is logged in.

<aside class="notice">Since the user's account key is an ordinary Ethereum private key, the user's key (and therefore their funds and Reputation) can be used with any Ethereum node.  Therefore, although the accounts system is managed using an ordinary web server, since the user's funds are neither tied to nor controlled by our server, the accounts are still decentralized in the ways that (in our opinion) matter.</aside>
