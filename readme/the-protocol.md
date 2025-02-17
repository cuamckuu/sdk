# The Monaco Protocol

The Monaco Protocol is a liquidity matching program to be deployed on the Solana blockchain. The protocol launched on mainnet in Q4 2022.

# Fully Audited

Every release of The Monaco Protocol is fully audited by [sec3](https://www.sec3.dev/).

- [Initial Audit](../media/pdf/sec3_audit_the_monaco_protocol_nov_2022.pdf)
- [All Audit Reports](https://github.com/MonacoProtocol/protocol/tree/develop/audit/sec3)

# What The Protocol Does

The Monaco Protocol offers the ability to wager on binary-outcome event, it is agnostic to the nature of these events. Orders can be placed either `FOR` the outcome (it will happen) or to `AGAINST` it (it won't happen) at a specific price (odds) and stake (wager).

# Architecture

![Diagram of the market account and associated accounts](../media/images/architecture_overview_1.png)

The concept of a market is broken down into three primary accounts:

- A market account
  - Core market information including title, type, status, lock time, outcome and linked event.
- Outcome account
  - One for each possible outcome on the market.
  - Each containing a price ladder.
- Outcome matching pools
  - One for each possible resolution and price of a market (for example: price of 10 `FOR`, price of 10 `AGAINST`, price of 2 `FOR`).
  - Contains matching queues to facilitate matching of orders.

## Markets

- Outcomes
  - Markets can be created on anything: from the outcome of a sporting event, the weather in Nantucket on a specific day, to the winner of the next election.
  - Outcomes on the protocol are represented by an index, in a market with outcomes of `TEAM_A, DRAW, TEAM_B` a draw result will be a winning index of `1`.
- Price Ladder
  - These markets can have their own price ladder. A sporting event may wish to offer prices ranging from `1.001` to `1000` whilst another market may just offer prices from `1.2` to `5` - these are set by the market operator at time of creation.
- Token for Entry
  - Markets can be created specifying any [spl-token](https://spl.solana.com/token) as the entry currency, from USDT to an NFT projects native token.
- Market Lock
  - Markets are created with a lock time set by a unix timestamp - to the second - in `UTC`. Once this time has elapsed, orders can no longer be placed on the market.
- Market Title/Type
  - Each market has a human-readable title and a machine-readable type.
- Market Lock Order Behaviour
  - (Not yet implemented) what should occur to unmatched orders when a market locks (either cancelled or nothing).
- Event
  - Markets can be linked with an event, this event is an on-chain record of the real-world event the market is associated with. The event will act as the source of truth for outcomes. **Note:** the service supporting events is yet to be released, but markets can still be created and settled independently of their existence. 

### In-Play Markets

All markets contain in-play focused flags that allow for orders to be placed during an event. In-play markets are facilitated by off-chain cranking.

- In-play Enabled
  - Flag to showing if the market is an in-play market.
- In-play
  - Flag showing the the market is currently in play.
- In-play Order Delay
  - How many seconds must elapse before an order on an in play market can be matched.
- Event Start Order Behaviour
  - What should occur to unmatched orders when an in-play market moved to in play (either cancelled or nothing).
- Event Start Timestamp
  - Timestamp a market will go in-play.

## Orders

- Wallets are able to place orders on a market.
- The order contains:
  - The market the order is to be placed against.
  - Price of the order.
  - Stake.
  - Index of the predicted winning outcome.
  - Whether or not the order is for the outcome happening or against it.
- Once an order is placed it is recorded in a market matching pool to allow for matching of orders.
- Fund are recorded in escrow.
- Orders can be cancelled until they are fully matched.
- Due to market position (see below) wallets can trade in and out of market positions on unmatched orders.

## Market Position

- Once a wallet places an order on a market, it will have a market position.
- This market position is used to track potential win/loss, maximum exposure, and leverage offset.
- This allows a wallet to trade in and out of positions where they have unmatched order funds.
- It also enables a wallet to see its current risk profile on the market.

## Matching

- The protocol will match orders on a market.
- Orders can be partially matched.
- Matching is designed to match at the best price possible for those orders.
- Matching is facilitated by off-chain cranking.

## Trades

- When orders are matched, a trade account is created.
- This trade account indicates the matching price and the orders involved in the trade.
- Trades can be viewed on a market level or an order level.

## Voiding

- When a market is marked to be voided all stakes/risks will be returned to wallets.
- Voiding is facilitated by off-chain cranking.

## Settlement

- When an outcome is passed to a market, the protocol will then process orders for settlement.
- Winnings are sent directly to the purchasing wallet for that order.
- Settlement is facilitated by off-chain cranking.

## Off-Chain Cranking

- Due to the nature of the blockchain, it is necessary to facilitate on-chain actions with a crank.
- Cranks send regular commands to an on-chain program in order to make it perform an action.
- To ensure decentralization, multiple cranks will power the protocol. The tooling will be open-source and the process of cranking rewarded.

## Operators

- There are 3 different operator types on the protocol.
  - ADMIN: this operator role allows for the administration of the other roles.
  - MARKET: this operator role allows to the creation and administration (status change, marking for settlement) of markets.
  - CRANK: this operator role allows for off-chain cranking.
- Operators are identified by a wallet publicKey.
- All admin functions, apart from querying operators and roles, require the correct operator role.
- Operator roles are assigned to wallets by the Monaco Protocol Foundation.

## Monaco Protocol Commission

- Currently set at 0%.
- Allows for commission to be charged on unique wallets.
- Commission is a % of net winnings on a market.
- Commission rate and implementation will be determined by the Monaco Protocol community as it affects all traders in the ecosystem.

## Market Closure

- Operators can chose to close markets at the end of their lifecycle.
- Closing a market will trigger an off-chain crank that will close all markets associated with a market (order, matching pools, trades etc).
- The closure process returns SOL used for rent to the wallet that initialized the account.

## Events

As previously stated, events are not yet an available on-chain account, however when they are they will contain data relating to the event the market is for. Such as:

- Participants
- Categorization
- Event name
- Start time
- Event state
- Results

These event accounts will eventually act as the source of truth for settlement of markets. With the ideal scenario of having event accounts updated by oracle services.
