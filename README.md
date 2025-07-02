# Drosera Hoodi Guide Setup

## Overview

This repository provides a step-by-step guide to set up your Drosera Trap and Operator node on the Hoodi testnet.

---

## Trap Configuration (`drosera.toml`)

Initially, **do not** include the `address` field in your `drosera.toml`. The address will be auto-generated after applying your trap config.

```toml
ethereum_rpc = "https://ethereum-hoodi-rpc.publicnode.com"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps]

[traps.helloworld]
path = "out/HelloWorldTrap.sol/HelloWorldTrap.json"
response_contract = "0x183D78491555cb69B68d2354F7373cc2632508C7"
response_function = "helloworld(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["YOUR_OPERATOR_WALLET_ADDRESS"]

# address = "DELETE THIS LINE WHEN APPLYING"
# For existing users, add your trap address here before applying.
