Flash
=====
Flash is a set of scripts to facilitate the testing of Augur, including the ability to simulate moving time forward.

The flash scripts live in the [Tools](https://github.com/AugurProject/augur-tools) repository.

They can be invoked from the monorepo root or from `packages/augur-tools` with:

`yarn flash --help`

There are two modes: `run` and `interactive`.

Use `run` when you want to execute a single script. However, not all scripts work in this manner.
The default network used is the local node but you can specify others with `--network`.

Use `interactive` when you want to run a local ganache node. You have a lot of control over its state.

Deploying
---------
To deploy to the Kovan test network, run:

`yarn flash run --network kovan deploy --write-artifacts`

To deploy locally, you don't need to specify network since the local node is the default:

`yarn flash run deploy --write-artifacts`

Some local testing does not require overwriting the `addresses.json` etc files but for deploys from the command-line,
you typically want to, so specify `--write-artifacts`.

Running a pre-populated local node
----------------------------------
Enter interactive mode: `yarn flash interactive`
In interactive mode (it will show `augur$` in the terminal):

`ganache`
`deploy`
`create-canned-markets-and-orders`

(Pro Tip: Interactive mode has tab-completion.)

Now you have a local ganache node running in your terminal. You can interact with it using flash commands. To see a
list of commands, type `help`.

Setting time forward 4 hours
----------------------------
Once you have ganache running in your interactive flash session, you can manipulate time.

There's `get-timestamp`, `set-timestamp`, and `push-timestamp`. We want `push-timestamp`.

To see it in action, type these commands in the interactive flash session:

`get-timestamp`
`push-timestamp --count 4h`
`get-timestamp`

Here, `4h` stands for `4 hours`. If you specify `s` or omit the letter entirely then it will set the timestamp some
number of seconds into the future. For example, `push-timestamp --count 45` sets the timestamp 45 seconds into the
future. As with all commands, you can type `--help` after the command to get the details:

`push-timestamp --help`

Saving and loading ganache state
--------------------------------
We store the ganache blockchain state in `Seeds`. They can be saved as files and loaded in later sessions. The chief
benefit to seeds is that you can manually create a scenario for testing, save a seed of that state to a file, and bring
it back later. Aside from saving you a bunch of time reproducing a particular testing state, you can save a "checkpoint"
before you do destructive exploratory testing and reload that state after. Also if you find a strange or buggy
blockchain/contract state then you can upload the seed file along with the bug report so the developers can see the
errant behavior themselves.

The commands are `make-seed`, `save-seed-to-file`, `load-seed-file`, `use-seed`, and `list-seeds`.
There's also `create-basic-seed`, which is used by the automated tests.

The basic idea behind seed management is that you have a list of seeds in-memory that you can swap into ganache at-will.
Use `list-seeds` to see them. Each seed has a name but if you don't specify one then `"default"` is used. Save them to
disk for later and load them as-needed.

Replaying Logs
--------------
In v2, we emit enough information in our logs that it's possible to know the state of Augur on the blockchain just from
the logs. While v1 doesn't quite achieve that just from the logs, the v1 logs provide enough information to know which
contract calls are needed to get a full picture.

Mainnet provides realistic data on how users behave and is therefore an excellent place to go for performance and
usability testing. Except that it's mainnet so the money's real and we can't cheat with time-manipulation and fauceting.
So we capture logs from mainnet then replay them onto a testnet. There are two methods for replaying logs but they both
start with capturing the logs.

Replaying logs is a 2-step process:
1. Gather the logs from the source chain.
2. Replay the logs to the target chain.

### Gather the Logs

Execute to get mainnet logs: `yarn flash run -n mainnet all-logs --from 5926223 --to 5937142 --v1|tee ~/mainnet-log.json`<br>
Then edit the `~/mainnet-log.json` file to remove the non-JSON parts at the top and bottom.<br>
NOTE: The `from` value here is when mainnet was deployed. The `to` value is chosen because it's a small enough range for
alchemy not to fail with a 503 error.

For kovan logs: `KOVAN_PRIVATE_KEY=0x0 yarn flash run -n kovan all-logs|tee ~/kovan-logs.json`<br>
You just need a fake private key. There's no need to specify a log range because alchemy can handle this request on
Kovan.

### Replay the Logs

There are two options for replaying: onto a Ganache chain to create a seed file or onto another chain.

#### Seed File

`yarn flash run -n none create-seed-from-logs --v1 --logs ~/mainnet-logs.json --seed ~/mainnet-seed.json`

You can now load `~/mainnet-seed.json` in an interactive Flash session.

#### Onto a Chain

First you need to run a chain. For local geth:
`yarn workspace @augurproject/tools docker:geth:pop-15`

Then replay onto "environment" aka the local node:
`yarn flash run -n environment replay-logs --logs ~/mainnet-logs.json`