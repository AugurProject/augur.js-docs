General API
===========
<aside class="notice">The General API section is still under construction and may be missing some information. Don't worry! We plan to update the entire documentation prior to Augur launching. Thank you for your patience as we make these updates.</aside>

The Augur API is split up into three sections, the [Simplified API](#simplified-api), the [Call API](#call-api), and the [Transaction API](#transaction-api). The [Simplified API](#simplified-api) is provided to get information about markets on the Augur network. The [Call](#call-api) and [Transaction](#transaction-api) API's are direct contract and method mapping to the `augur.api` object. The [Call API](#call-api) uses `eth_call` to make "get" requests for information stored on the blockchain. The [Transaction API](#transaction-api) uses `eth_sendTransaction` to make "set" requests to the blockchain in order to modify the blockchain's information in some way, like creating a new order on the order book. The [Call](#call-api) and [Transaction](#transaction-api) APIs will be covered in greater detail in their respective sections of these docs.

In this section we will go over how to import `augur.js` into your project and connect it to an Ethereum Node. This section will also cover market creation, initial market loading, and review the Simplified API methods.

Connect to an Ethereum Node
---------------------------
```javascript
// After installing, just require augur.js to use it.
import Augur from 'augur.js';
const augur = new Augur();

const options = {
  httpAddresses: [
    'http://127.0.0.1:8545', // local http address for Geth Node
    'https://eth9000.augur.net' // hosted http address for Augur Node
  ],
  wsAddresses: [
    'ws://127.0.0.1:8545', // local websocket address for Geth Node
    'wss://ws9000.augur.net', // hosted websocket address for Augur Node
  ],
};

// Attempt to connect to a local Ethereum node
// if that fails, fall back to our hosted node
augur.connect(options, function (vitals) { /* ... */ });
// example vitals object:
vitals = {
  networkID: '9000',
  blockNumber: '0xf69b5',
  coinbase: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
  gasPrice: 18000000000,
  api: {
    events: { /*...*/},
    functions: { /* ... */ },
  },
  contracts: { /* ... */ },
  rpc: { /* ... */ },
};
```
The Augur API is a set of JavaScript bindings for the methods encoded in Augur's [smart contracts](https://github.com/AugurProject/augur-core). The API method name, as well as its parameters as keys in the `params` object, are generally identical to those of the underlying smart contract method.

Augur's [core contracts](https://github.com/AugurProject/augur-core) exist on Ethereum's decentralized network. The various serialization, networking, and formatting tasks required to communicate with the Augur contracts from a web application are carried out by Augur's [middleware](#architecture).

[augur.js](https://github.com/AugurProject/augur.js) is the Augur JavaScript SDK, and is the user-facing component of the middleware. If you want to interact with the Augur contracts from a custom application, augur.js is the recommended way to do so. The easiest way to install augur.js is using [npm](https://www.npmjs.com/package/augur.js):

`$ npm install augur.js`

or if you prefer [yarn](https://yarnpkg.com/en/):

`$ yarn add augur.js`

To use the Augur API, `augur.js` must connect to an Ethereum node, which can be either remote (hosted) or local. To specify the connection endpoint, pass your RPC connection info to `augur.connect`. Augur will go through the list of potential connections provided by the `options` argument and attempt to connect to each in turn until one of the connections is successful or all attempts fail.

In the example we have set our first connection to try as `http://127.0.0.1:8545` which is our local geth node. If Augur is unable to connect to the local geth node, then Augur will go to the next provided address. In this case we have provided a single hosted node (`https://eth9000.augur.net`) as the only other attempt to make outside of the local geth node. If a connection is successfully established then a `vitals` object will be returned, otherwise an error message will be returned.

Accounts
--------
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

// Augur.js does not store any account data. Augur.js simply returns the important information. You can use the privateKey Buffer returned to sign your transactions.
```
Augur.js includes a trustless account management system. The purpose of the accounts system is to allow people to use Augur without needing to run an Ethereum node themselves, as running a full Ethereum node can be resource-intensive.

To use the account system, the user specifies a password. Everything else is done automatically for the user. The only requirement for the password is that it be at least 6 characters long.

A private key (+ derived public key and address) is automatically generated for the user.  A secret key derived from the password using PBKDF2, along with a random 128-bit initialization vector, is used to encrypt the private key (using AES-256). Nothing is stored by Augur.js, the account object will be returned to the callback provided or simply returned if no callback is provided.

The Augur UI will handle your account information for you, but if you are using Augur.js on it's own you will need to manage the account yourself. Augur.js doesn't sign any transactions for you if you aren't using the Augur UI.

If you want to use the Augur.js API directly, you will need to sign any transaction that will modify information on the blockchain (non-call transactions). All transactions take a `_signer` parameter which should be set to the sending account's `privateKey` Buffer or a function that will sign a transaction (hardware wallet).

<aside class="notice">Since the user's account key is an ordinary Ethereum private key, the user's key (and therefore their funds and Reputation) can be used with any Ethereum node. Therefore, although the accounts system is managed using an ordinary web server, since the user's funds are neither tied to nor controlled by our server, the accounts are still decentralized in the ways that (in our opinion) matter.</aside>

Numbers
-------

Before we move on, lets quickly talk about Numbers. There are three acceptable ways to pass numerical inputs to the Augur API:

- primitive JS numbers (e.g., `1010101`): ok for integers, but use strings for floating point numbers (see below)

- stringified numbers (e.g., `"1010101"`)

- hexadecimal strings (e.g., `"0xf69b5"`)

Floating-point (decimal) values should be passed to augur.js as strings (e.g., instead of `0.07`, use `"0.07"`), for reasons described in [enormous detail](http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html) elsewhere.

<aside class="notice"><b>All numerical parameters passed to augur.js must be either base 10 (decimal) or base 16 (hexadecimal).</b> Do <b>not</b> use the base 10<sup>18</sup> fixed-point representation that Augur uses internally for decimals!  augur.js handles all fixed-point conversions for you.  Do <b>not</b> send the Loch Ness monster 3.50 &times; 10<sup>18</sup> CASH.  (Probably don't even give him 3.50, but that's a debate for another time.)</aside>


Market creation
---------------
```javascript
var marketID = "0x34e104a15ab3eb2a0c26f1138c278416424ef7f8425799a76127e9b427fac74d";
var params = {
  market: marketID,
  liquidity: 9012,
  startingQuantity: 501,
  bestStartingQuantity: 500,
  priceWidth: "0.08697536855694898",
  initialFairPrices: [ "0.661837082683109", "0.5132635487227519" ]
};
augur.create.generateOrderBook(params, {
  onBuyCompleteSets: function (res) { /* ... */ },
  onSetupOutcome: function (outcome) { /* ... */ },
  onSetupOrder: function (order) { /* ... */ },
  onSuccess: function (orderBook) { /* ... */ },
  onFailed: function (err) { /* ... */ }
});

params.isSimulationOnly = true;
augur.create.generateOrderBook(params, {
  onSimulate: function (simulation) { /* ... */ }
})
// example:
simulation = {
  shares: "4007",
  numBuyOrders: [ 10, 7 ],
  numSellOrders: [ 4, 7 ],
  buyPrices:
   [ [ "0.61834939840463451",
       "0.55853753243153362626",
       "0.49872566645843274252",
       "0.43891380048533185878",
       "0.37910193451223097504",
       "0.3192900685391300913",
       "0.25947820256602920756",
       "0.19966633659292832382",
       "0.13985447061982744008",
       "0.08004260464672655634" ],
     [ "0.46977586444427741",
       "0.40996399847117652626",
       "0.35015213249807564252",
       "0.29034026652497475878",
       "0.23052840055187387504",
       "0.1707165345787729913",
       "0.11090466860567210756" ] ],
  sellPrices:
   [ [ "0.70532476696158349",
       "0.76513663293468437374",
       "0.82494849890778525748",
       "0.88476036488088614122" ],
     [ "0.55675123300122639",
       "0.61656309897432727374",
       "0.67637496494742815748",
       "0.73618683092052904122",
       "0.79599869689362992496",
       "0.8558105628667308087",
       "0.91562242883983169244" ] ],
  numTransactions: 34,
  priceDepth: "0.05981186597310088374"
}
```
`augur.create.generateOrderBook` is a convenience method for generating an initial order book for a newly created market. `generateOrderBook` calculates the number of orders to create, as well as the spacing between orders, using the `calculatePriceDepth` method.

#### generateOrderBook(params, callbacks)

`params` is an object containing the input parameters (see example code).  `liquidity` is the total amount of liquidity to add to the market.  `initialFairPrice` is the center of the bid-ask spread. `startingQuantity` is the number of shares available at each price point. `bestStartingQuantity` can optionally be specified separately, and is the number of shares available at the best price point (those closest to the spread). `priceWidth` is the price difference between the best bid and best ask orders.

`calculatePriceDepth` accepts arguments `liquidity`, `startingQuantity`, `bestStartingQuantity`, `halfPriceWidth`, `minValue`, and `maxValue`.  All arguments to `calculatePriceDepth` are expected to be in `BigNumber` format.  The order book generated by `augur.create.generateOrderBook` is "flat", in the sense that there is the same size order created at each price point (except for the best bid/offer).  A typical generated order book (for a single outcome) might look like:

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="1172px" version="1.1" viewBox="0 0 1172 793" style="max-width:100%;max-height:793px;"><defs><linearGradient x1="0%" y1="0%" x2="100%" y2="0%" id="mx-gradient-dae8fc-1-7ea6e0-1-e-0"><stop offset="0%" style="stop-color:#DAE8FC"/><stop offset="100%" style="stop-color:#7EA6E0"/></linearGradient><linearGradient x1="0%" y1="0%" x2="100%" y2="0%" id="mx-gradient-f8cecc-1-ea6b66-1-e-0"><stop offset="0%" style="stop-color:#F8CECC"/><stop offset="100%" style="stop-color:#EA6B66"/></linearGradient></defs><g transform="translate(0.5,0.5)"><g transform="translate(533.5,508.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="95" height="19" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><span style="font-size: 18px ; line-height: 21.6px"><font color="#9673a6">price width</font></span></div></div></foreignObject><text x="48" y="16" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><g transform="translate(1073.5,731.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="84" height="40" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><div><font style="font-size: 18px" color="#0066cc">maximum</font></div><div><font style="font-size: 18px" color="#0066cc">price</font></div></div></div></foreignObject><text x="42" y="26" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><rect x="1053" y="154" width="83" height="570" fill="url(#mx-gradient-dae8fc-1-7ea6e0-1-e-0)" stroke="#6c8ebf" pointer-events="none"/><rect x="201" y="154" width="80" height="570" fill="url(#mx-gradient-f8cecc-1-ea6b66-1-e-0)" stroke="#b85450" pointer-events="none"/><rect x="121" y="154" width="80" height="570" fill="url(#mx-gradient-f8cecc-1-ea6b66-1-e-0)" stroke="#b85450" pointer-events="none"/><rect x="281" y="154" width="80" height="570" fill="url(#mx-gradient-f8cecc-1-ea6b66-1-e-0)" stroke="#b85450" pointer-events="none"/><rect x="361" y="154" width="80" height="570" fill="url(#mx-gradient-f8cecc-1-ea6b66-1-e-0)" stroke="#b85450" pointer-events="none"/><rect x="441" y="354" width="80" height="370" fill="url(#mx-gradient-f8cecc-1-ea6b66-1-e-0)" stroke="#b85450" pointer-events="none"/><rect x="970" y="154" width="83" height="570" fill="url(#mx-gradient-dae8fc-1-7ea6e0-1-e-0)" stroke="#6c8ebf" pointer-events="none"/><rect x="887" y="154" width="83" height="570" fill="url(#mx-gradient-dae8fc-1-7ea6e0-1-e-0)" stroke="#6c8ebf" pointer-events="none"/><rect x="804" y="154" width="83" height="570" fill="url(#mx-gradient-dae8fc-1-7ea6e0-1-e-0)" stroke="#6c8ebf" pointer-events="none"/><rect x="721" y="154" width="83" height="570" fill="url(#mx-gradient-dae8fc-1-7ea6e0-1-e-0)" stroke="#6c8ebf" pointer-events="none"/><path d="M 101 724 L 101 62.24" fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" pointer-events="none"/><path d="M 101 56.24 L 105 64.24 L 101 62.24 L 97 64.24 Z" fill="#000000" stroke="#000000" stroke-width="2" stroke-miterlimit="10" pointer-events="none"/><rect x="641" y="354" width="80" height="370" fill="url(#mx-gradient-dae8fc-1-7ea6e0-1-e-0)" stroke="#6c8ebf" pointer-events="none"/><path d="M 101 724 L 1152.76 724" fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" pointer-events="none"/><path d="M 1158.76 724 L 1150.76 728 L 1152.76 724 L 1150.76 720 Z" fill="#000000" stroke="#000000" stroke-width="2" stroke-miterlimit="10" pointer-events="none"/><g transform="translate(861.5,741.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="57" height="26" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><font style="font-size: 24px"><b>price</b></font></div></div></foreignObject><text x="29" y="19" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><g transform="translate(22.5,2.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="113" height="47" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><div><font style="font-size: 24px"><b>order size</b></font></div><font style="font-size: 18px"><b>(# shares)</b></font></div></div></foreignObject><text x="57" y="30" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><g transform="translate(20.5,330.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="70" height="61" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><font style="font-size: 18px" color="#009999">best</font><div><font style="font-size: 18px" color="#009999">starting</font><div><font style="font-size: 18px" color="#009999">quantity</font></div></div></div></div></foreignObject><text x="35" y="37" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><g transform="translate(19.5,131.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="70" height="40" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 102); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><font style="font-size: 18px" color="#000099">starting</font><div><span style="font-size: 18px ; line-height: 1.2"><font color="#000099">quantity</font></span></div></div></div></foreignObject><text x="35" y="26" fill="#000066" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><g transform="translate(540.5,739.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="80" height="40" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><font color="#00cc00" style="font-size: 18px">initial fair</font><div><font color="#00cc00" style="font-size: 18px">price</font></div></div></div></foreignObject><text x="40" y="26" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><path d="M 580 724 L 580 51" fill="none" stroke="#00cc00" stroke-width="5" stroke-miterlimit="10" stroke-dasharray="15 15" pointer-events="none"/><path d="M 537.92 538.67 L 624.74 538.67" fill="none" stroke="#9673a6" stroke-width="5" stroke-miterlimit="10" pointer-events="none"/><path d="M 526.92 538.67 L 537.92 533.17 L 537.92 544.17 Z" fill="#9673a6" stroke="#9673a6" stroke-width="5" stroke-miterlimit="10" pointer-events="none"/><path d="M 635.74 538.67 L 624.74 544.17 L 624.74 533.17 Z" fill="#9673a6" stroke="#9673a6" stroke-width="5" stroke-miterlimit="10" pointer-events="none"/><g transform="translate(64.5,728.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="80" height="40" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><font style="font-size: 18px" color="#cc0000">minimum</font><div><font style="font-size: 18px" color="#cc0000">price</font></div></div></div></foreignObject><text x="40" y="26" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><path d="M 337.92 751.26 L 384.41 751.07" fill="none" stroke="#994c00" stroke-width="5" stroke-miterlimit="10" pointer-events="none"/><path d="M 326.92 751.31 L 337.9 745.76 L 337.95 756.76 Z" fill="#994c00" stroke="#994c00" stroke-width="5" stroke-miterlimit="10" pointer-events="none"/><path d="M 395.41 751.02 L 384.43 756.57 L 384.39 745.57 Z" fill="#994c00" stroke="#994c00" stroke-width="5" stroke-miterlimit="10" pointer-events="none"/><g transform="translate(312.5,763.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="97" height="19" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><span style="font-size: 18px ; line-height: 21.6px"><font color="#994c00">price depth</font></span></div></div></foreignObject><text x="49" y="16" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><path d="M 401 729 L 401 734 Q 401 739 401 749 L 401 759" fill="none" stroke="#994c00" stroke-width="2" stroke-miterlimit="10" pointer-events="none"/><path d="M 321 759 L 321 729" fill="none" stroke="#994c00" stroke-width="2" stroke-miterlimit="10" pointer-events="none"/><path d="M 1151 155 L 91 155" fill="none" stroke="#000099" stroke-width="3" stroke-miterlimit="10" stroke-dasharray="9 9" pointer-events="none"/><path d="M 1151 354 L 91 354" fill="none" stroke="#009999" stroke-width="3" stroke-miterlimit="10" stroke-dasharray="9 9" pointer-events="none"/><path d="M 346 -97 L 323.5 -97 L 323.5 104 L 301 104 L 323.5 104 L 323.5 305 L 346 305" fill="none" stroke="#b85450" stroke-width="3" stroke-miterlimit="10" transform="rotate(90,323.5,104)" pointer-events="none"/><g transform="translate(790.5,49.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="197" height="24" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><span style="line-height: 21.6px"><font style="font-size: 24px" color="#0066cc">asks (sell orders)</font></span></div></div></foreignObject><text x="99" y="18" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g><path d="M 911 -143 L 888.5 -143 L 888.5 104 L 866 104 L 888.5 104 L 888.5 351 L 911 351" fill="none" stroke="#0066cc" stroke-width="3" stroke-miterlimit="10" transform="rotate(90,888.5,104)" pointer-events="none"/><g transform="translate(222.5,49.5)"><switch><foreignObject style="overflow:visible;" pointer-events="all" width="196" height="24" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; white-space: nowrap; font-weight: bold; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><span style="line-height: 21.6px"><font style="font-size: 24px" color="#b85450">bids (buy orders)</font></span></div></div></foreignObject><text x="98" y="18" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica" font-weight="bold">[Not supported by viewer]</text></switch></g></g></svg>

This calculation is done as follows.  Let \\(p_\mathrm{max}\\) and \\(p_\mathrm{min}\\) be the maximum and minimum allowed prices, \\(I\\) be the initial fair price for the outcome under consideration, \\(q\\) be the starting quantity (size of each order), \\(q_0\\) be the size of the best order, and \\(w\\) be the price width (the size of the bid-ask spread).  The liquidity available on the buy and sell order books is given by:

<p>
$$
L_{\mathrm{buy}} = q_0 + \frac{q}{\delta} \left( p_\mathrm{min} + I - \frac{w}{2} \right)
$$
</p>
<p>
$$
L_{\mathrm{sell}} = q_0 + \frac{q}{\delta} \left( p_\mathrm{max} - I - \frac{w}{2} \right)
$$
</p>

where \\(\delta\\) is the "price depth", the distance between orders on the same side of the book.  The total liquidity \\(L = L_{\mathrm{buy}} + L_{\mathrm{sell}}\\) is an input parameter, so the only unknown is the price depth \\(\delta\\), which can now be calculated:

<p>
$$
\delta = \frac{\left( p_\mathrm{min} + p_\mathrm{max} - w \right) q}{L - 2q_0}
$$
</p>

The initial order book is now fully specified.

`augur.create.generateOrderBook` also accepts an optional parameter, `isSimulationOnly`, which if set to `true`, returns information about what the method will do (see code example).  `shares` is the number of complete sets of shares to purchased, `numBuyOrders` is the number of buy orders that will be created, `numSellOrders` is the number of sell orders that will be created, and `numTransactions` is the total number of Ethereum transactions needed to set up the order book.

Initial market loading
----------------------
To load the basic market info for all markets, first call `augur.markets.getMarketsInfo({ [ branch, offset, numMarketsToLoad, volumeMin, volumeMax] }, callback)`. You will likely need to chunk the results so that the request does not time out. More detailed market info (including prices) for each market in each chunk (page) is then loaded using `augur.markets.getMarketInfo({ marketID[, account ] }, callback)`. `getMarketInfo` does not return the full order book; to get the order book for a market, call `augur.trading.orderBook.getOrderBook({ market[, offset, numTradesToLoad, scalarMinMax: { minValue, maxValue } ] }, callback)`.

<aside class="notice">Cache nodes regularly call <code>augur.markets.getMarketsInfo({ branch, offset, numMarketsToLoad } callback)</code>. The first time <code>getMarketsInfo</code> is called, all markets should be loaded. Subsequent <code>getMarketsInfo</code> calls should only load markets created since the previous call.</aside>

Simplified API
--------------
```javascript
const market = '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42';

augur.trading.orderBook.getOrderBook({ market }, function (orderBook) { /* ... */ })
// example output:
{
  buy: {
    '0x1f4f112a9aa99282e306cb58abc95b5b46199f802bd36d68f1619ba98866963a': {
      amount: '10',
      block: 217,
      fullPrecisionAmount: '10',
      fullPercisionPrice: '0.1208333333333333',
      id: '0x1f4f112a9aa99282e306cb58abc95b5b46199f802bd36d68f1619ba98866963a',
      market: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
      outcome: '2',
      owner: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
      price: '0.1208',
      type: 'buy',
    },
    '0x2b054b0c9ca2fcf22ee73ba14ae41da70c7039c1a5b8125a40b9f2b68a20080b': {
      amount: '10',
      block: 219,
      fullPrecisionAmount: '10',
      fullPercisionPrice: '0.0083333333333333',
      id: '0x2b054b0c9ca2fcf22ee73ba14ae41da70c7039c1a5b8125a40b9f2b68a20080b',
      market: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
      outcome: '2',
      owner: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
      price: '0.0083',
      type: 'buy',
    },
    '0x3c3958b3cad3fb693a6fdd013a615485ef42d824aaa3bd57734f5ec21567ebdc': {
      amount: '10',
      block: 220,
      fullPrecisionAmount: '10',
      fullPercisionPrice: '0.1770833333333333',
      id: '0x3c3958b3cad3fb693a6fdd013a615485ef42d824aaa3bd57734f5ec21567ebdc',
      market: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
      outcome: '3',
      owner: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
      price: '0.177',
      type: 'buy',
    }
  },
  sell: {
    '0x0a056c290d73ca11b22531ef0c4ea970bdc3e7ccd64a60f8127fedaabd231f15': {
      amount: '10',
      block: 224,
      fullPrecisionAmount: '10',
      fullPercisionPrice: '0.9958333333333333',
      id: '0x0a056c290d73ca11b22531ef0c4ea970bdc3e7ccd64a60f8127fedaabd231f15',
      market: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
      outcome: '1',
      owner: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
      price: '0.9959',
      type: 'sell',
    },
    '0x0bd9bf9c18ea08f98c70312ccc8deac7b58d88d3c5f2d0cc9a5bed201a90191e': {
      amount: '10',
      block: 220,
      fullPrecisionAmount: '10',
      fullPercisionPrice: '0.6583333333333333',
      id: '0x0bd9bf9c18ea08f98c70312ccc8deac7b58d88d3c5f2d0cc9a5bed201a90191e',
      market: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
      outcome: '3',
      owner: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
      price: '0.6584',
      type: 'sell',
    },
    '0x1afbdc152df5d674a26459b0267a22cb13a3903f7922affcf526485662293269': {
      amount: '10',
      block: 222,
      fullPrecisionAmount: '10',
      fullPercisionPrice: '0.7145833333333333',
      id: '0x1afbdc152df5d674a26459b0267a22cb13a3903f7922affcf526485662293269',
      market: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
      outcome: '1',
      owner: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
      price: '0.7146',
      type: 'sell',
    }
  }
}

augur.markets.getMarketInfo({ marketID: market }, function (market) { /* ... */ });
// example output:
{
  author: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
  branchID: '0xf69b5',
  consensus: null,
  creationBlock: 213,
  creationFee: '9',
  creationTime: 1502248462,
  cumulativeScale: '1',
  description: 'What will be the status of the U.S. electoral college on January 1, 2020?~|>Unchanged from 2016|Undermined but still in existence (e.g., National Popular Vote bill)|Formally abolished',
  endDate: 1577952000,
  eventBond: '4.5',
  extraInfo: 'http://www.nationalpopularvote.com',
  id: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
  makerFee: '0.01',
  maxValue: '3',
  minValue: '1',
  network: '9000',
  numOutcomes: 3,
  outcomes: [{
    id: 1,
    outstandingShares: '310',
    price: '0.6020833333333333',
    sharesPurchased: '0',
  }, {
    id: 2,
    outstandingShares: '310',
    price: '0.4333333333333333',
    sharesPurchased: '0',
  }, {
    id: 3,
    outstandingShares: '310',
    price: '0.4333333333333333',
    sharesPurchased: '0',
  }],
  tags: ['politics', 'elections', 'US politics'],
  takerFee: '0.02',
  topic: 'politics',
  tradingFee: '0.02',
  tradingPeriod: 9131,
  type: 'categorical',
  volume: '1710.399999999999999985'
}

const marketIDs = [ market, '0xe095e00863aecd814003a739da97b54c2b6737bd'];
const account = '0xb3f636cae9e8ad9795d14d3bdda3e382dba47c73';

augur.markets.batchGetMarketInfo({ marketIDs, account }, function (marketsInfo) { /* ... */ })
// example output:
{
  '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42': {
    author: '0x05ae1d0ca6206c6168b42efcd1fbe0ed144e821b',
    branchID: '0xf69b5',
    consensus: null,
    creationBlock: 213,
    creationFee: '9',
    creationTime: 1502248462,
    cumulativeScale: '1',
    description: 'What will be the status of the U.S. electoral college on January 1, 2020?~|>Unchanged from 2016|Undermined but still in existence (e.g., National Popular Vote bill)|Formally abolished',
    endDate: 1577952000,
    eventBond: '4.5',
    extraInfo: 'http://www.nationalpopularvote.com',
    id: '0x9368ff3e9ce1c0459b309fac6dd4e69229b91a42',
    makerFee: '0.01',
    maxValue: '3',
    minValue: '1',
    network: '9000',
    numOutcomes: 3,
    outcomes: [{
      id: 1,
      outstandingShares: '310',
      price: '0.6020833333333333',
      sharesPurchased: '0',
    }, {
      id: 2,
      outstandingShares: '310',
      price: '0.4333333333333333',
      sharesPurchased: '0',
    }, {
      id: 3,
      outstandingShares: '310',
      price: '0.4333333333333333',
      sharesPurchased: '0',
    }],
    tags: ['politics', 'elections', 'US politics'],
    takerFee: '0.02',
    topic: 'politics',
    tradingFee: '0.02',
    tradingPeriod: 9131,
    type: 'categorical',
    volume: '1710.399999999999999985'
  },
  '0xe095e00863aecd814003a739da97b54c2b6737bd': {
    author: '0x9e12c5b6067c61c811add8fb55f5d6fcb88a9e5f',
    branchID: '0xf69b5',
    consensus: null,
    creationBlock: 5156,
    creationFee: '9',
    creationTime: 1502291695,
    cumulativeScale: '1',
    description: 'Will the Bitcoin (BTC) price be equal to or higher than USD 3,400 on 31 August 2017 as shown on www.coinmarketcap.com?',
    endDate: 1504130400,
    eventBond: '4.5',
    extraInfo: 'http://www.nationalpopularvote.com',
    id: '0xe095e00863aecd814003a739da97b54c2b6737bd',
    makerFee: '0.01',
    maxValue: '2',
    minValue: '1',
    network: '9000',
    numOutcomes: 2,
    outcomes: [{
      id: 1,
      outstandingShares: '2101.2',
      price: '0',
      sharesPurchased: '0',
    }, {
      id: 2,
      outstandingShares: '2101.2',
      price: '0.99',
      sharesPurchased: '5',
    }],
    resolutionSource: 'www.coinmarketcap.com',
    tags: ['Bitcoin', 'Cryptocurrency', null],
    takerFee: '0.02',
    topic: 'Bitcoin',
    tradingFee: '0.02',
    tradingPeriod: 8704,
    type: 'binary',
    volume: '8346'
  }
}
// all keys in the options object are optional, if branch is not provided then
// the default of augur.constants.DEFAULT_BRANCH_ID will be used.
const options = {
  branch: '0xf69b5',
  offset: '3',
  numMarketsToLoad: '2',
  volumeMin: '1000',
  volumeMax: '3000'
}

augur.markets.getMarketsInfo(options, function (marketsInfo) { /* ... */ })
// example output:
{
  '0xfb9165a0f492a910082c02bc174d3b3b7f7f979e': {
    author: '0x9e12c5b6067c61c811add8fb55f5d6fcb88a9e5f',
    branchID: '0xf69b5',
    consensus: null,
    creationTime: 1502248542,
    description: 'Who will win the University of Georgia vs. University of Florida football game in 2017?~|>Georgia|Florida|Vanderbilt',
    endDate: 1509346800,
    id: '0xfb9165a0f492a910082c02bc174d3b3b7f7f979e',
    makerFee: '0.01',
    maxValue: '3',
    minValue: '1',
    numOutcomes: 3,
    tags: ['sports', 'college football', 'football'],
    takerFee: '0.02',
    topic: 'sports',
    tradingFee: '0.02',
    tradingPeriod: 8734,
    type: 'categorical',
    volume: '1195.999999999999999997'
  },
  '0x563b377a956c80d77a7c613a9343699ad6123911': {
    author: '0x9e12c5b6067c61c811add8fb55f5d6fcb88a9e5f',
    branchID: '0xf69b5',
    consensus: null,
    creationTime: 1502248522,
    description: "Will Augur's live release happen by the end of August, 2017?",
    endDate: 1504249200,
    id: '0x563b377a956c80d77a7c613a9343699ad6123911',
    makerFee: '0.010000000000000000125',
    maxValue: '2',
    minValue: '1',
    numOutcomes: 2,
    tags: ['Augur', 'release date', 'Ethereum'],
    takerFee: '0.030000000000000000375',
    topic: 'Augur',
    tradingFee: '0.026666666666666667',
    tradingPeriod: 8705,
    type: 'binary',
    volume: '29623.019599999999999969'
  }
}
```
All of the methods in the Simplified API are getter methods that use an `eth_call` RPC request; for transactional requests (`eth_sendTransaction`), see the [Transaction API](#transaction-api) section below. This API is simplified in the sense that single requests to this API can be used to fetch a large amount of data, without the need for complicated RPC batch queries.

### augur.trading.orderBook.getOrderBook({ market[, offset, numTradesToLoad, scalarMinMax: { minValue, maxValue } ] }[, callback])

Retrieves the full order book for `market`. The order book's format is an object with fields `buy` and `sell` containing arrays of buy and sell orders. If your `market` is a scalar `market` then you would want to include the optional object `scalarMinMax`. `scalarMinMax` should have a `minValue` and `maxValue` for the `market` you are querying. There are two optional fields, `offset` and `numTradesToLoad`, which can be used to split up the query into multiple chunks, which can be helpful for larger order books.

### augur.markets.getMarketInfo({ marketID[, account] }[, callback])

Reads all information about a market (`marketID`) that is stored on-contract. It also determines the `type` of the market, which can be `binary` (two outcomes; i.e., Yes or No), `categorical` (more than two outcomes, i.e., Multiple Choice), or `scalar` (answer can be any of a range of values; i.e., Numerical). You can optionally pass an `account` address to show the amount of shares purchased by that `account` for each outcome.

### augur.markets.batchGetMarketInfo({ marketIDs[, account] }[, callback])

Retrieve a `marketsInfo` object for the market IDs in array `marketIDs`. The `marketsInfo` object will contain `marketInfo` objects (see above for example) which are indexed by their respective market IDs. `account` is an optional parameter used to determine shares purchased by that `account` address for each outcome in the markets returned in the `marketsInfo` object.

### augur.markets.getMarketsInfo({ [branch, offset, numMarketsToLoad, volumeMin, volumeMax] }[, callback])

Returns basic information about markets on a specific `branch` or the `augur.constants.DEFAULT_BRANCH_ID` if no `branch` is provided. This function returns a `marketsInfo` object containing `marketInfo` objects indexed by their respective market IDs similar to `batchGetMarketInfo`. The `offset` and `numMarketsToLoad` optional params are also available to this function to break up the market information as the number of markets on a single branch is typically too large for a single RPC request. The final two optional parameters are `volumeMin` and `volumeMax` which are used to filter markets by a minimum and/or maximum trade volume.

<aside class="notice">Markets are indexed in the order created; i.e., the first market created has index 0, the second 1, etc. This ordering allows us to break up a large aggregate request like <code>getMarketsInfo</code> into manageable chunks.

For example, suppose you were displaying markets on separate pages. You might want to retrieve information about all markets, but, to keep your loading time reasonable, only get 5 markets per request. To get the first 5 markets, you would set <code>offset</code> to 0 and <code>numMarketsToLoad</code> to 5: <code>augur.markets.getMarketsInfo({offset: 0, numMarketsToLoad: 5}, cb)</code>.  To get the second 5, <code>offset</code> would be 5: <code>augur.markets.getMarketsInfo({offset: 5, numMarketsToLoad: 5}, cb)</code>.  The third 5, <code>offset</code> would be 10: <code>augur.markets.getMarketsInfo({offset: 10, numMarketsToLoad: 5}, cb)</code>, and so on.</aside>
