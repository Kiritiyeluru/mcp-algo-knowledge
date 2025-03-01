# Position Tracking Knowledge Map

This knowledge map covers the components for tracking open positions, historical records, and P&L calculations within the MCP Algorithmic Trading System.

## 1️⃣ Interfaces

### PositionTrackerBase
- **File Path**: `/src/interfaces/position_tracking/base.py`
- **Core Methods**:
  - `update_position(trade: Trade) -> None`: Updates position based on executed trade
  - `get_position(symbol: str) -> Optional[Position]`: Retrieves current position by symbol
  - `get_all_positions() -> List[Position]`: Returns all tracked positions
  - `calculate_pnl(position: Position) -> float`: Computes P&L for position
  - `close_position(symbol: str) -> bool`: Closes position for a specific symbol

## 2️⃣ Models

### Position
- **File Path**: `/src/interfaces/position_tracking/models.py`
- **Properties**:
  - `symbol: str`: Trading symbol
  - `quantity: int`: Number of shares/contracts held
  - `average_price: float`: Average price of acquisition
  - `current_price: float`: Latest market price
  - `unrealized_pnl: float`: `(current_price - average_price) * quantity`
  - `realized_pnl: float`: Accumulated profit/loss from closed trades
  - `last_updated: datetime`: Last update timestamp

## 3️⃣ Validation Rules

- **Real-Time Updates**: Positions must update current price in real-time
- **Position Limits**: Max 2-3 concurrent positions allowed
- **Data Integrity**: Valid alphanumeric symbol and non-negative quantity required
- **P&L Calculation**: Consistent formulas for both realized and unrealized P&L
- **Update Frequency**: Updates on every trade execution or market interval

## 4️⃣ Test Patterns

### Test Classes
- `TestPositionTracker`: Overall position tracker functionality
- `TestPnLCalculations`: P&L computation accuracy

### Standard Test Methods
- `test_update_position()`: Verify position updates with trade data
- `test_get_position()`: Check retrieval by symbol
- `test_get_all_positions()`: Verify complete position list retrieval
- `test_calculate_pnl()`: Confirm P&L calculation accuracy
- `test_close_position()`: Validate position closing logic

## 5️⃣ Common Fixtures

### `sample_position()`
- **Returns**: Position instance for testing
- **Parameters**:
  - `symbol: str = "RELIANCE"`: Trading symbol
  - `quantity: int = 100`: Share count
  - `average_price: float = 2500.0`: Entry price
  
### `mock_position_tracker()`
- **Returns**: Mock implementation of PositionTrackerBase

### `sample_trade_history()`
- **Returns**: List of Trade objects representing execution history

### `sample_pnl_data()`
- **Returns**: PnlData object with realized and unrealized P&L

## 6️⃣ Market-Specific Features

- **Intraday Focus**: Positions reset at market close (no overnight carry)
- **Trading Hours**: Factor in market hours (9:15 AM – 3:30 PM IST)
- **Circuit Limits**: Consider market-imposed limits on price movements
- **Product Types**: Support for MIS (intraday), CNC (delivery), NRML (F&O) positions

## 7️⃣ Implementation Priorities

1. Base interface with core position tracking methods
2. Position model with complete P&L tracking
3. Test fixtures for various position scenarios
4. Validation rules for position integrity
5. Real-time update mechanism for prices
