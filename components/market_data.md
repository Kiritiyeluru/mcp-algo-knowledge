# Market Data Knowledge Map

This document outlines the market data component of the MCP Algorithmic Trading System, responsible for fetching, processing, and providing market data to other components.

## 1️⃣ Interfaces

### MarketDataProviderBase
- **File Path**: `/src/interfaces/market_data/base.py`
- **Core Methods**:
  - `subscribe(symbol: str, data_type: DataType) -> bool`: Subscribe to market data
  - `unsubscribe(symbol: str, data_type: DataType) -> bool`: Unsubscribe from market data
  - `get_market_data(symbol: str, data_type: DataType) -> MarketData`: Get latest data
  - `get_historical_data(symbol: str, data_type: DataType, start_time: datetime, end_time: datetime) -> List[MarketData]`: Get historical data

### LiveMarketDataProviderBase
- **File Path**: `/src/interfaces/market_data/base.py`
- **Extends**: `MarketDataProviderBase`
- **Additional Methods**:
  - `connect() -> bool`: Connect to data feed
  - `disconnect() -> bool`: Disconnect from data feed
  - `is_connected() -> bool`: Check connection status

## 2️⃣ Models

### DataType
- **File Path**: `/src/interfaces/market_data/models.py`
- **Values**: `TICK`, `OHLC`, `DEPTH`, `FULL_QUOTE`

### MarketData
- **File Path**: `/src/interfaces/market_data/models.py`
- **Properties**:
  - `symbol: str`: Trading symbol
  - `timestamp: datetime`: Data timestamp
  - `data_type: DataType`: Type of market data
  - `data: Dict[str, Any]`: Actual data (format depends on data_type)

### TickData
- **File Path**: `/src/interfaces/market_data/models.py`
- **Extends**: `MarketData`
- **Additional Properties**:
  - `last_price: float`: Last traded price
  - `last_quantity: int`: Last traded quantity
  - `volume: int`: Total volume
  - `open_interest: int`: Open interest (for derivatives)

### OHLCData
- **File Path**: `/src/interfaces/market_data/models.py`
- **Extends**: `MarketData`
- **Additional Properties**:
  - `open: float`: Opening price
  - `high: float`: Highest price
  - `low: float`: Lowest price
  - `close: float`: Closing price
  - `volume: int`: Total volume
  - `interval: str`: Timeframe interval (e.g., "1m", "5m", "1h", "1d")

## 3️⃣ Validation Rules

- **Symbol Format**: Valid alphanumeric with optional exchange prefix (e.g., "NSE:RELIANCE")
- **F&O Symbol Format**: Must follow Indian market format (e.g., "NIFTY25MAR18000CE")
- **Data Type**: Must be one of the defined enum values
- **Timestamp**: Must be timezone-aware and in IST timezone
- **Price Values**: Must be positive and within circuit limits
- **Volume/Quantity**: Must be positive integers

## 4️⃣ Test Patterns

### Test Classes
- `TestMarketDataProvider`: Tests for the provider interface
- `TestMarketDataModels`: Tests for data models
- `TestMarketDataValidation`: Tests for validation logic

### Standard Test Methods
- `test_initialization()`: Test object initialization with valid parameters
- `test_validation()`: Test validation logic for inputs
- `test_serialization()`: Test to/from dict methods for models
- `test_subscription()`: Test subscription management
- `test_data_retrieval()`: Test data fetching methods
- `test_connection()`: Test connection management

## 5️⃣ Common Fixtures

### `sample_tick_data()`
- **Returns**: A valid TickData instance
- **Parameters**: 
  - `symbol: str = "RELIANCE"`: Trading symbol
  - `last_price: float = 2500.0`: Last traded price
  - `timestamp: Optional[datetime] = None`: Data timestamp

### `sample_ohlc_data()`
- **Returns**: A valid OHLCData instance
- **Parameters**:
  - `symbol: str = "NIFTY"`: Trading symbol
  - `interval: str = "5m"`: Timeframe interval
  - `timestamp: Optional[datetime] = None`: Data timestamp

### `mock_market_data_provider()`
- **Returns**: A mock implementation of MarketDataProviderBase

## 6️⃣ Indian Market-Specific Features

- **Circuit Limits**: ±20% for most stocks, variable for indices
- **Trading Hours**: 9:15 AM - 3:30 PM IST, with pre/post market data
- **F&O Symbol Format**: `{SYMBOL}{EXPIRY_DATE}{STRIKE_PRICE}{OPTION_TYPE}`
- **Exchange-Specific Data**: Support for NSE, BSE data formats
- **Market Depth**: Special handling for Indian market depth data

## 7️⃣ Implementation Priorities

1. Base market data interface implementation
2. Core data models (Tick, OHLC)
3. Subscription management
4. Historical data retrieval
5. Real-time data handling
