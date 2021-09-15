---
title: OVM 2.0 Changeset
lang: en-US
---

# {{ $frontmatter.title }}

During the next regenesis (September 28th on Kovan, October on the main network), we will have a series of breaking changes as a part of an upgrade to the OVM. Fundamentally, this will drastically reduce the differences between the OVM and EVM so that developers can implement their contracts with mostly just one target in mind, instead of managing OVM idiosyncrasies with separate contracts. Thus the changes can be viewed as a reversion to most things listed on this page [https://community.optimism.io/docs/protocol/evm-comparison.html](/docs/protocol/evm-comparison.html). Here is the list of key breaking changes to watch for:

1. Contracts whose source code has not been verified on Etherscan 
   ([Kovan](https://kovan-optimistic.etherscan.io/verifyContract),
   [Optimistic Ethereum](https://optimistic.etherscan.io/verifyContract))
   will be wiped out along with their storage.

1. Contracts whose source code has been verified will be recompiled 
   with the standard Solidity compiler. As a result of this:
   1. The `EXTCODEHASH` and `CODESIZE` of every contract will change.
      Any hardcoded values will break.
   1. The address generated by `CREATE2` may be different (it depends on
      the constructor code).
   1. Code size will no longer be increased during compilation. Any 
      custom OVM work that required reducing code size could now 
      be reverted.
   1. Uniswap pools will be moved to their L1-equivalent 
      addresses corresponding to `CREATE2` with the new bytecode. For more detail, see the [Uniswap](#uniswap) section below

1. ETH will no longer be ERC20 compatible.
   1. Users will **no longer** be able to transfer and interact with ETH 
      as an ERC20 located at `0x4200000000000000000000000000000000000006`.
   1. Please let us know if you rely on this functionality on Optimistic 
      Ethereum mainnet currently as we will have to migrate those balances
      to a standard WETH9 contract which will be deployed to 
      `0x4200000000000000000000000000000000000006`
   1. The `Transfer` event currently emitted on ETH fee payment will be
    removed

1. Our fee scheme will be altered. 
   [See here for details](new-fees.html)    

1. EOAs will no longer be contract wallets
    1. Currently, every new EOA in Optimistic Ethereum deploys a proxy
       contract to that address, making every EOA a contract account.
    1. After the upgrade, every known contract account will be reverted 
       to an EOA with no code.
    1.  To best handle the upgrade, we recommend you write your contracts 
        to handle normal EOAs and to make no assumptions around EOAs
        having code or additional OVM-specific functionality.

1. Gas usage will decrease to be the same as on L1 Ethereum.
   1. Currently, the OVM results in an increase in gas usage because of compiled OVM opcodes, sometimes inflating gas costs by up to 10x the costs on L1 Ethereum.
   1. With this OVM upgrade, gas usage on Optimistic Ethereum should now exactly match the expected gas usage from a deployment to (L1) pre-London Ethereum in almost all cases.

1. Certain opcodes will have updated functionality:
    1. `NUMBER` - `block.number` in L2 will now correspond to the L2 
       block number, not “Last finalized L1 block number” like it used to. Any project currently using `block.number` must check that this will not break their implementation.
    2. `COINBASE` is set by the sequencer. For the near-term future, 
       it will return the `OVM_SequencerFeeVault` address (currently `0x4200000000000000000000000000000000000011`)
    3. `DIFFICULTY` will return 0
    4. `BLOCKHASH` will return the L2 block hash. Note that this value 
       can be manipulated by the sequencer and is not a safe source of randomness.
    5. `SELFDESTRUCT` will now work just as it currently works in the EVM.
    6. `GASPRICE` will now return the l2GasPrice
    7. `BASEFEE` will be unsupported - execution will revert if it is 
       used.
8. Certain OVM system contracts will be wiped from the state. We will remove:
    1.  `OVM_ExecutionManager`
    2.  `OVM_SequencerEntrypoint`
    3. `OVM_StateManager`
    4. `OVM_StateManagerFactory`
    5. `OVM_SafetyChecker`
    6. `OVM_ECDSAContractAccount`
    7. `OVM_ExecutionManagerWrapper`
    8. `Lib_AddressManager`
    
    All other OVM predeploys will remain at the same addresses as before the regenesis


## Applications

### Uniswap    

As bytecode is updated, the codeHash for Uniswap pools will be altered, 
as will be their CREATE2 addresses. This means we will be shifting 
all Uniswap pools to new CREATE2 addresses corresponding to the 
updated pool bytecode.

We will also be migrating the ERC20 balances of each Uniswap pool to 
their new CREATE2'ed addresses.

If your contracts rely on Uniswap pool addresses, ensure that 
you calculate them using the `getPoolAddress` function


::: warning
As with all regenesis-es, all historical transactions and logs will be inaccessible from the new chain, and the new chain will start from Block #0. The old chain data will not be accessible except under extreme circumstances. Applications like Dune Analytics, Tenderly, Etherscan, and the Graph will not support access to transactions or logs from before the regenesis.
:::