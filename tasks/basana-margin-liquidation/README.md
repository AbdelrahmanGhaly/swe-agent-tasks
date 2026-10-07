# Add margin calls and liquidations to the backtesting exchange

| | |
|---|---|
| **Project** | [gbeced/basana](https://github.com/gbeced/basana) |
| **Base commit** | [`8e722becd8`](https://github.com/gbeced/basana/tree/8e722becd8255db70cce1f139e0f4cebfe604b48) |
| **Language** | Python |
| **Reference solution** | +432 / -6 lines across 8 files |
| **New tests** | 51 test functions, +1,298 lines in 1 test file |

## Problem

`MarginLoans` should warn about, and then close out, an account whose margin level falls too low. It gets four new optional arguments: `margin_call_level` and `liquidation_level`, both on the same scale as `margin_level` and refused unless below 100, `liquidation_fee`, a percentage that defaults to zero, and `margin_call_timeout`, a timedelta. Without the two levels the exchange behaves exactly as it does today. The margin level is checked, with the latest prices, once the exchange has processed all the bars that share a timestamp, before those bars reach anyone else.

A check that finds the level below `margin_call_level` sends a `MarginCallEvent`, stamped with the time of the check and carrying that `margin_level`, to handlers registered through `Exchange.subscribe_to_margin_call_events`. The call then stays open, and no other call goes out, until a check finds the level back at or above `margin_call_level`.

A liquidation starts at a check that finds the level below `liquidation_level`, or that comes `margin_call_timeout` or more after the call that is still open went out. Every open order is canceled, and until the liquidation ends, creating an order or a loan raises the exchange's Error.

On each later bar whose pair is quoted in the lending strategy's quote symbol, the exchange trades the pair's base symbol while a loan is open. What it owes in a symbol is the principal of its open loans in it plus the accrued interest charged in it. When it owes more of the base symbol than it has available, it buys the smallest amount that covers the difference once any fee charged in the base symbol is taken out; otherwise it sells what it has beyond what it owes, any fee charged in the base symbol coming out of that. Trades follow the pair's base precision and never exceed what the bar's liquidity allows, and a trade the account cannot fully pay for, slippage and fees included, shrinks to the largest amount it can pay for.

Each trade is a market order of its own, listed with the exchange's orders and reported through order events. It fills straight away against that bar the way a market order fills at a bar's open, with the usual slippage and fees plus the liquidation fee on its quote amount, charged in the quote symbol and rounded up together with the other fees. Then, on every bar during the liquidation, each open loan the account can repay in full is repaid, in the order the loans were taken out. The liquidation ends at the first check that finds no loan open or the margin level at 100 or more.

The exchange's `get_liquidations()` coroutine lists the liquidations in the order they started. Each has `started_at` and `ended_at` (None while in progress), the times of the checks that started and ended it, the `margin_level` that started it, the `canceled_order_ids`, the `order_ids` of its trades and the `repaid_loan_ids` of the loans it repaid, the last two in the order they happened.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/gbeced/basana repo && cd repo
git checkout 8e722becd8255db70cce1f139e0f4cebfe604b48
cp ../Dockerfile . && git apply ../test.patch
docker build -t basana-margin-liquidation .
docker run --rm basana-margin-liquidation ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t basana-margin-liquidation .
docker run --rm basana-margin-liquidation ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm basana-margin-liquidation ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
