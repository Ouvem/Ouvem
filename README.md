PulseChain Testnet

The PulseChain Testnet is up and running. This document will guide you through connecting Metamask to the network and bootstrapping a PulseChain node of your own.

Disclaimer: This is a Testnet, and issues may arise as the network or certain front-ends see increased load. The team will work diligently to address any issues as they come.
Disclaimer 2: The state of this network may be reset on occasion, and nothing should be considered permanent after the fork block. We will communicate ahead of time when these resets are planned.
Version 2

The second version of the PulseChain Testnet brings the following enhancements:

Enables validator rotation, registration, and staking
Simplifies client setup when running a node (genesis and config files no longer required!)
Connecting Metamask

If you had metamask connected to Testnet Version 1, you will need to reset your accounts through the metamask options, or remove and re-add the network below
Follow these instructions to manually add the Testnet to your Metamask plugin. A button will be available in the future to do this automatically.

1. Click the Networks dropdown and select "Custom Network"

2. Enter the following information:

Network Name: PulseChain Testnet
New RPC URL: https://rpc.v2.testnet.pulsechain.com
Chain ID: 940
Currency Symbol: tPLS
Block Explorer URL: https://scan.v2.testnet.pulsechain.com
Step 1	Step 2
Metamask step1	Metamask step2
Congratulations! You are now connected to the PulseChain Testnet. Existing ethereum accounts that had balances as of block 13,224,745 (Sep-14-2021 03:48:51 PM +UTC) will have the equivalent on balance on the PulseChain Testnet.

Getting tPLS to use on the PulseChain Testnet

To get tPLS you can use the tPLS faucet.

Navigate to the tPLS faucet https://faucet.v2.testnet.pulsechain.com/
Connect your Metamask wallet by clicking on the button.
Enter the address you want to send tPLS to and click the Request button.
Wait up to 60 seconds to receive your tPLS.
Connecting a PulseChain Node

If you were previously running a Testnet V1 node, you can re-use your existing blockchain database by rolling back the blockchain. See Using An Existing Blockchain DB below.

Warning: The PulseChain Testnet includes all of the Ethereum mainnet state up to block 13,224,745. This means that the system requirements for running a node will be high, particularly the storage requirements. You should only run your own testnet node if needed for development purposes, etc...
HARDWARE

You will need at least 750 GB of free storage to store the synchronized chain.
At least 4 cores and 8GB of RAM are recommended.
SOFTWARE

Docker is recommended, and the commands below will tailored to running a dockerized node. By building and running the node in docker, we eliminate any environmental differences like the local golang version or the host OS.
If you prefer, you can compile and run the executable directly, but you will need to tweak the commands below.
NOTE: All commands below assume that you want to store all chain data in a local /blockchain directory (must have at least 750GB free space).

If needed, you can modify the commands to mount a different directory in the docker container. To do so, you will change the absolute path on the left side of the colon :, e.g., docker run -v /path/to/my/dir:/blockchain ...

For more information see the Docker run command reference.
1. Prepare the Blockchain Directory

First, ensure that the intended blockchain datadir has at least 750GB of free space. The directory should be empty.

2. Start the Pulse Node

Once your blockchain directory is ready, you can start the node and connect to the network by providing the --pulsechain-testnet flag.

docker run -v /blockchain:/blockchain -P registry.gitlab.com/pulsechaincom/go-pulse --datadir=/blockchain --pulsechain-testnet

Using An Existing Blockchain DB

Warning: This is only valid for nodes that were sync'd with a previous version of the PulseChain Testnet or the Ethereum Mainnet past block 13,224,745.
Performing the rollback will require geth to re-generate its internal snapshot.
1. Stop the Existing Blockchain

Stop any existing Go-Pulse or Go-Ethereum processes and let the blockchain gracefully shut down.

2. Dump New Genesis File

Dump the genesis.json file from the latest Go-Pulse release. It is recommended you dump this file into your existing --datadir used by the previously running node. Assuming the /blockchain directory was being used, we can dump the updated genesis.json file with the command below.

docker run registry.gitlab.com/pulsechaincom/go-pulse --pulsechain-testnet dumpgenesis > /blockchain/genesis.json

Confirm that the genesis.json file has been written to your blockchain datadir. Double check the modified date to ensure this file was just created/updated.

3. Perform Rollback

In order to rollback the chain, we need to launch the geth console to issue the debug.setHead("0xC9CB29") command. It's important to run with the --nodiscover flag to prevent the node from syncing any new blocks during this process.

docker run -it registry.gitlab.com/pulsechaincom/go-pulse --pulsechain-testnet --nodiscover console

From the interactive console, verify chain state and perform the rollback:

Verify the current block number is above 13,224,745

> eth.blockNumber

Perform the chain rollback to block 13,224,745

> debug.setHead("0xC9CB29")

Verify the current block number is now 13,224,745

> eth.blockNumber
13224745

Exit the geth console

> exit

4. Re-Initialize Genesis w/ Updated Chain Config

With the blockchain rolled back to the last ethereum mainnet block, you can now reinitialize the genesis for Testnet V2, using the genesis.json file you dumped in step 2.

docker run -v /blockchain:/blockchain registry.gitlab.com/pulsechaincom/go-pulse --datadir=/blockchain init /blockchain/genesis.json

After the init command has completed, you can follow the normal steps above to Start the Pulse Node and begin syncing from the fork block.

Validator Registration, Rotation, and Staking

PulseChain validator registration, staking, rotation, and revenue share are managed via system contracts that can be interacted with through the PulseChain validator & staking ui.

The staking UI will require that you have Metamask configured and connected to the PulseChain Testnet. See steps above for connecting Metamask.
From the staking UI, users can:

View information about the network validators: their total rewards, % revenue share, number of misdemeanors & felonies.
Delegate stake to registered validators.
Remove stake from registered validators.
Register new validators.
Validator Rotation

PulseChain validators are rotated at the end of each era, on a targeted ~24-hour basis, every 28,800 blocks. On these blocks, several operations will occur:

Validators already in rotation for the past era will be rewarded transaction fees for blocks they produced throughout the era.
Earned fees are paid proportionally to the validator's stakers, based on the validator's revenue share percentage.
For example, if the validator has 50% revenue share, and two stakers exist with 2 and 3 tPLS delegated, then the stakers will receive 20% and 30% of the validator's rewards respectively, with the remaining 50% going directly to the validator.
Registered validators will be ranked by their total delegated stake.
The top 33 validators will be selected as the authorized set of validators for the next era.
Validator Staking

Users can delegate their tPLS to registered validators, earning a portion of the validator's rewards if the validator is selected for rotation. Stakers have a responsibility to delegate stake to high-performing and reliable nodes that will service the network well, and users should make informed decisions when staking.

Considerations when choosing validators for staking:

The total stake: only the top 33 validators by total stake will be brought into rotation, and only validators in rotation will earn rewards for themselves and their stakers. Additionally, staker revenue is paid out proportionally based on your share of the total stake.
The number of misdemeanors & felonies represents the historical offenses/slashes that have been given to the validator for lack of uptime or bad behavior. Misdemeanor and felonies result in loss of rewards (but not principal) for the validator and its stakers.
The total rewards is the total amount of revenue earned by the validator over time, including revenue shared with its stakers.
Pay attention to the user interface when adding stakes, it will show your projected stake and projected revenue share. Your projected revenue share is based on the validator's revenue share * your portion of the total stake delegated to that validator.
Add Stake	Remove Stake
Add Stake	Remove Stake
Staking rules:

Stakes added to validators already in rotation will be considered "Pending" until the end of the current era (validator rotation).
This means that for any given era the validator is in rotation, only the stakes that contributed to the validator being selected (staked before rotation) will be considered during revenue share.
Users can only remove stake from a validator after 24 hours.
You can add to or remove from existing stakes, and it is also possible to remove just a portion of the total stake.
Validator Registration

Running a validator node performs an important role for the PulseChain network, and the performance of the network depends on the performance of its underlying validators.

Users should not consider running a validator node unless you have experience running a full blockchain node with high availability!

Before Registering: You should have the validator node running and fully synchronized with the network so that the moment the validator is brought into rotation it is ready to sign blocks for the network.

Once again: only register a validator if you are fully committed to maintaining its availability and uptime - THIS IS NOT FOR MOST USERS!
With the above caveats understood, you can register the validator through the PulseChain validator & staking ui. Steps for doing so include:

Making a non-refundable deposit of 500,000 tPLS to the staking contract from the validator address. This serves to prevent spam registrations.
About the Deposit: This is intentionally high during the Testnet phase to limit the number of 3rd party validators and ensure network availability and performance. If you have serious intentions of running a high-availability validator for testnet, please reach out to the PulseChain team via Telegram, and we can assist with funding via the Testnet treasury account.
Register the new validator with the following options:
Fee Address (Optional): If provided, rewards earned by the validator will be sent to the fee address. If not provided, the fees are paid directly to the validator address.
Revenue Share: The percent of revenue that will be paid out to users with stake delegated to this validator.
Considerations: higher revenue share encourages more stakers, lower revenue share captures a higher share of profits for the validator.
Unregister a Validator

Validators looking to cease operation can un-register themselves via the same PulseChain validator & staking ui. An un-registered validator will be removed from the validator pool at the next validator rotation.

A validator can be re-registered without requiring a new deposit.
