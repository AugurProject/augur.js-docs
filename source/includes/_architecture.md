Architecture
============

Local node
----------
<img src="images/architecture_local.svg" onerror="this.src='images/architecture_local.png'">
If you are running a [local Ethereum node](https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum), sending RPC requests to your local node is the best (fastest) way to use Augur.  To do this, you can either download and run the Augur app locally, or simply visit [http://local.augur.net](http://local.augur.net), which is configured to automatically talk to your local node, if present.

<aside class="notice">To use "sendTransaction" RPC commands (i.e., anything that requires Ether / is actually sent to the network) you will need to unlock your Ethereum node.  If you are running geth, the easiest way to do this is to start it with the <code>--unlock</code> option:

<code class="block">geth --unlock 0 --testnet --rpc --ws --rpccorsdomain "http://localhost:8080" --wsorigins "http://localhost:8080"</code>

The <code>--ws</code> flag allows websocket connections, which are recommended for speed.  The <code>--rpccorsdomain</code> and <code>--wsorigins</code> flags specify what domains are allowed to send RPC requests to your geth node.  Here <code>http://localhost:8080</code> is whitelisted; this is the default address where the <a href="https://github.com/AugurProject/augur">Augur client</a> is <a href="<a href="https://github.com/AugurProject/augur-ui-webserver">">served</a>, if you build Augur locally.  To use your local node with a remote host, you will need to set <code>--rpccorsdomain</code> and/or <code>--wsorigins</code> to your host's address; for example, if the remote host is at <code>http://local.augur.net</code>, then you would use <code>http://local.augur.net</code> for <code>--rpccorsdomain</code> and <code>--wsorigins</code>.</aside>

Because geth's RPC server uses regular (unencrypted) HTTP, in order for the Augur app to communicate with your local geth node, it must be served over HTTP.  The static file served when you visit <a href="https://augur.net">augur.net</a> is presently hosted on Firebase, which is HTTPS-only, and therefore you can't interact with your local geth node from this URL!  As a short-term workaround (until we set up on a more flexible CDN), we set up <a href="http://local.augur.net">local.augur.net</a> on one of our Linode servers.  It's configured for both HTTP and HTTPS, so it will work with your local node.

<aside class="notice">These notes are specifically for <a href="https://github.com/ethereum/go-ethereum">geth</a>, the Go Ethereum implementation, which is the most widely-used implementation.  <a href="https://github.com/ethcore/parity">Parity</a> is another commonly-used implementation, for which we are working on adding official support (and how-to notes)!</aside>

Hosted node
-----------
<img src="images/architecture_hosted.svg" onerror="this.src='images/architecture_hosted.png'">
Understandably, many users will not want to go to the trouble of running a full Ethereum node solely for the purpose of using Augur.  Augur maintains a public Ethereum node at [https://eth3.augur.net](https://eth3.augur.net), which is used automatically by [augur.net](https://augur.net) (and by [local.augur.net](http://local.augur.net) if a local Ethereum node is not detected).  Augur's node is on the Morden testnet.

<aside class="notice">We also maintain a public Ethereum node on the Homestead network at <a href="https://eth2.augur.net">https://eth2.augur.net</a>, which is used for the <a href="https://sale.augur.net">sale site</a>.  However, the Augur contracts are not currently maintained on Homestead.</aside>
