# Multi-Order Type Orderbook

A C++ limit order book implementation that matches buy and sell orders using price-time priority. When a buy order's price meets or exceeds a sell order's price, they are matched and a trade occurs. Orders at the same price level are filled in the order they were received (FIFO).

The orderbook tracks bids (buy orders) sorted from highest to lowest price, and asks (sell orders) sorted from lowest to highest price. This ensures the best available prices are always at the front for matching.

## Order Types

- **GoodTillCancel** — Stays on book until filled or cancelled
- **FillAndKill** — Executes immediately or gets cancelled

## Architecture

```
Orderbook
├── bids_    : map<Price, list<Order>, greater<Price>>
│              └─ Buy orders, highest price first
├── asks_    : map<Price, list<Order>, less<Price>>
│              └─ Sell orders, lowest price first
└── orders_  : unordered_map<OrderId, OrderEntry>
               └─ Fast lookup for cancel/modify
```

- **Price levels** are stored in `std::map` with custom comparators to keep best prices at the front
- **Orders at each price** are stored in `std::list` to maintain time priority (FIFO)
- **Order lookup** uses `std::unordered_map` for O(1) access when cancelling or modifying

## Quick Start

```cpp
#include "Orderbook.cpp"

int main() {
    Orderbook orderbook;

    // Add a buy order
    auto buyOrder = std::make_shared<Order>(
        OrderType::GoodTillCancel,
        1,           // Order ID
        Side::Buy,
        100,         // Price
        10           // Quantity
    );
    orderbook.AddOrder(buyOrder);

    // Cancel an order
    orderbook.CancelOrder(1);

    return 0;
}
```

## API

```cpp
// Add order, returns any trades that occurred
Trades AddOrder(OrderPointer order);

// Cancel order by ID
void CancelOrder(OrderId orderId);

// Modify existing order (cancel + replace)
Trades MatchOrder(OrderModify order);

// Get number of active orders
std::size_t Size();

// Get bid/ask depth
OrderBookLevelInfos GetLevelInfos();
```

## Build

```bash
g++ -std=c++20 -o orderbook Orderbook.cpp
```
