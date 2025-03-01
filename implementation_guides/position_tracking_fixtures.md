# Position Tracking Test Fixtures Implementation Guide

This guide provides detailed implementation specifics for `/tests/interfaces/position_tracking/conftest.py`, including concrete examples, edge cases, and Indian market-specific details.

## Core Fixtures Overview

### `sample_position()`

```python
@pytest.fixture
def sample_position(request):
    """
    Creates a sample Position instance for testing.
    
    Args:
        symbol (str, optional): Trading symbol. Defaults to "RELIANCE".
        quantity (int, optional): Position quantity. Defaults to 100.
        average_price (float, optional): Average entry price. Defaults to 2500.0.
        current_price (float, optional): Current market price. Defaults to 2550.0.
        realized_pnl (float, optional): Realized P&L. Defaults to 0.0.
        
    Returns:
        Position: A configured Position instance
    """
    # Default parameters
    params = {
        "symbol": "RELIANCE", 
        "quantity": 100,
        "average_price": 2500.0, 
        "current_price": 2550.0,
        "realized_pnl": 0.0,
        "last_updated": datetime.now(timezone(timedelta(hours=5, minutes=30)))  # IST timezone
    }
    
    # Override defaults with any provided parameters
    if hasattr(request, 'param'):
        params.update(request.param)
    
    # Calculate unrealized P&L
    unrealized_pnl = (params["current_price"] - params["average_price"]) * params["quantity"]
    
    # Create and return Position instance
    position = Position(
        symbol=params["symbol"],
        quantity=params["quantity"],
        average_price=params["average_price"],
        current_price=params["current_price"],
        unrealized_pnl=unrealized_pnl,
        realized_pnl=params["realized_pnl"],
        last_updated=params["last_updated"]
    )
    return position
```

### `sample_trade_history()`

```python
@pytest.fixture
def sample_trade_history(request):
    """
    Creates a list of Trade objects representing a position's execution history.
    
    Args:
        symbol (str, optional): Trading symbol. Defaults to "RELIANCE".
        trade_count (int, optional): Number of trades to generate. Defaults to 3.
        price_range (tuple, optional): Min/max price range. Defaults to (2450.0, 2550.0).
        quantity_range (tuple, optional): Min/max quantity range. Defaults to (10, 50).
        trade_type (str, optional): Type of trades - "BUY", "SELL", or "MIXED". Defaults to "MIXED".
        
    Returns:
        List[Trade]: A list of Trade objects
    """
    # Default parameters
    params = {
        "symbol": "RELIANCE",
        "trade_count": 3,
        "price_range": (2450.0, 2550.0),
        "quantity_range": (10, 50),
        "trade_type": "MIXED",  # "BUY", "SELL", or "MIXED"
    }
    
    # Override defaults with any provided parameters
    if hasattr(request, 'param'):
        params.update(request.param)
    
    trades = []
    now = datetime.now(timezone(timedelta(hours=5, minutes=30)))  # IST timezone
    
    for i in range(params["trade_count"]):
        # Determine trade direction
        if params["trade_type"] == "BUY":
            side = "BUY"
        elif params["trade_type"] == "SELL":
            side = "SELL"
        else:  # MIXED
            side = "BUY" if i % 2 == 0 else "SELL"
        
        # Generate random price and quantity within specified ranges
        price = random.uniform(*params["price_range"])
        quantity = random.randint(*params["quantity_range"])
        
        # Create trade with timestamp slightly in the past (each 5 minutes earlier)
        trade_time = now - timedelta(minutes=5 * (params["trade_count"] - i))
        
        trade = Trade(
            trade_id=f"TRD{i+1:03d}",
            symbol=params["symbol"],
            quantity=quantity,
            price=price,
            side=side,
            timestamp=trade_time,
            exchange="NSE",
            order_id=f"ORD{i+1:03d}"
        )
        trades.append(trade)
    
    return trades
```

### `sample_position_with_trades()`

```python
@pytest.fixture
def sample_position_with_trades(sample_position, sample_trade_history):
    """
    Creates a Position with associated trade history.
    
    This fixture links a position with its trade history and ensures consistency
    between the position's average price and the weighted average of the trades.
    
    Returns:
        Tuple[Position, List[Trade]]: Position and its trade history
    """
    position = sample_position
    trades = sample_trade_history(param={"symbol": position.symbol})
    
    # Calculate position metrics based on the trades
    buy_quantity = sum(trade.quantity for trade in trades if trade.side == "BUY")
    sell_quantity = sum(trade.quantity for trade in trades if trade.side == "SELL")
    
    # Ensure net quantity matches position's quantity
    position.quantity = buy_quantity - sell_quantity
    
    # Calculate average price from buy trades
    if buy_quantity > 0:
        position.average_price = sum(trade.price * trade.quantity for trade in trades if trade.side == "BUY") / buy_quantity
    
    # Calculate realized PnL from sell trades
    if sell_quantity > 0:
        avg_sell_price = sum(trade.price * trade.quantity for trade in trades if trade.side == "SELL") / sell_quantity
        position.realized_pnl = (avg_sell_price - position.average_price) * sell_quantity
    
    # Update unrealized PnL
    position.unrealized_pnl = (position.current_price - position.average_price) * position.quantity
    
    return position, trades
```

### `sample_pnl_data()`

```python
@pytest.fixture
def sample_pnl_data(request):
    """
    Creates a PnlData instance for testing.
    
    Args:
        symbol (str, optional): Trading symbol. Defaults to "RELIANCE".
        realized (float, optional): Realized P&L. Defaults to 5000.0.
        unrealized (float, optional): Unrealized P&L. Defaults to 2500.0.
        product_type (str, optional): "MIS" (intraday) or "CNC" (delivery). Defaults to "MIS".
        
    Returns:
        PnlData: A configured PnlData instance
    """
    # Default parameters
    params = {
        "symbol": "RELIANCE",
        "realized": 5000.0,
        "unrealized": 2500.0,
        "product_type": "MIS",  # "MIS" (intraday) or "CNC" (delivery)
        "closing_price": 2550.0,
    }
    
    # Override defaults with any provided parameters
    if hasattr(request, 'param'):
        params.update(request.param)
    
    # Create and return PnlData instance
    return PnlData(
        symbol=params["symbol"],
        realized_pnl=params["realized"],
        unrealized_pnl=params["unrealized"],
        product_type=params["product_type"],
        closing_price=params["closing_price"],
        timestamp=datetime.now(timezone(timedelta(hours=5, minutes=30)))  # IST timezone
    )
```

### `mock_position_tracker()`

```python
@pytest.fixture
def mock_position_tracker(sample_position, sample_pnl_data):
    """
    Creates a mock implementation of PositionTrackerBase for testing.
    
    This mock records method calls and provides controlled responses.
    
    Returns:
        MockPositionTracker: A mock implementation of PositionTrackerBase
    """
    class MockPositionTracker(PositionTrackerBase):
        def __init__(self):
            self.positions = {sample_position.symbol: sample_position}
            self.update_calls = []
            self.get_position_calls = []
            self.calculate_pnl_calls = []
            self.close_position_calls = []
        
        def update_position(self, trade):
            """Mock implementation that records the call and updates the position."""
            self.update_calls.append(trade)
            
            position = self.positions.get(trade.symbol)
            if position is None:
                # Create new position if it doesn't exist
                if trade.side == "BUY":
                    position = Position(
                        symbol=trade.symbol,
                        quantity=trade.quantity,
                        average_price=trade.price,
                        current_price=trade.price,
                        unrealized_pnl=0.0,
                        realized_pnl=0.0,
                        last_updated=trade.timestamp
                    )
                else:
                    # Can't sell what we don't have
                    raise ValidationError(
                        "Cannot sell non-existent position", 
                        "PositionTracker", 
                        {"trade": trade}
                    )
            else:
                # Update existing position
                if trade.side == "BUY":
                    # Calculate new average price
                    total_quantity = position.quantity + trade.quantity
                    position.average_price = (
                        (position.average_price * position.quantity) + 
                        (trade.price * trade.quantity)
                    ) / total_quantity
                    position.quantity = total_quantity
                else:  # SELL
                    if trade.quantity > position.quantity:
                        raise ValidationError(
                            "Insufficient quantity to sell", 
                            "PositionTracker", 
                            {"trade": trade, "position": position}
                        )
                    # Calculate realized P&L
                    realized_pnl = (trade.price - position.average_price) * trade.quantity
                    position.realized_pnl += realized_pnl
                    position.quantity -= trade.quantity
                
                # Update position timestamp
                position.last_updated = trade.timestamp
            
            # Store updated position
            self.positions[trade.symbol] = position
            return None
        
        def get_position(self, symbol):
            """Mock implementation that records the call and returns a position."""
            self.get_position_calls.append(symbol)
            return self.positions.get(symbol)
        
        def get_all_positions(self):
            """Mock implementation that returns all positions."""
            return list(self.positions.values())
        
        def calculate_pnl(self, position):
            """Mock implementation that records the call and calculates P&L."""
            self.calculate_pnl_calls.append(position)
            
            # For testing, return a predefined PnlData
            return sample_pnl_data(param={"symbol": position.symbol})
        
        def close_position(self, symbol):
            """Mock implementation that records the call and closes the position."""
            self.close_position_calls.append(symbol)
            
            if symbol in self.positions:
                del self.positions[symbol]
                return True
            return False
    
    return MockPositionTracker()
```

## Indian Market Specific Implementation Details

### F&O Position Tracking

When implementing position tracking for F&O instruments, consider these Indian market specifics:

```python
@pytest.fixture
def sample_fno_position(request):
    """
    Creates a sample F&O Position instance for testing.
    
    Args:
        symbol (str, optional): F&O symbol. Defaults to "NIFTY25FEB18000CE".
        quantity (int, optional): Position quantity. Defaults to 50 (1 lot).
        average_price (float, optional): Average entry price. Defaults to 150.0.
        current_price (float, optional): Current market price. Defaults to 175.0.
        
    Returns:
        Position: A configured F&O Position instance
    """
    # Default parameters
    params = {
        "symbol": "NIFTY25FEB18000CE", 
        "quantity": 50,  # Standard lot size for NIFTY options
        "average_price": 150.0, 
        "current_price": 175.0,
        "realized_pnl": 0.0
    }
    
    # Override defaults with any provided parameters
    if hasattr(request, 'param'):
        params.update(request.param)
    
    # Parse F&O symbol to extract instrument details
    symbol_parts = parse_fno_symbol(params["symbol"])
    
    # Calculate unrealized P&L
    unrealized_pnl = (params["current_price"] - params["average_price"]) * params["quantity"]
    
    # Create and return Position instance with F&O metadata
    position = Position(
        symbol=params["symbol"],
        quantity=params["quantity"],
        average_price=params["average_price"],
        current_price=params["current_price"],
        unrealized_pnl=unrealized_pnl,
        realized_pnl=params["realized_pnl"],
        last_updated=datetime.now(timezone(timedelta(hours=5, minutes=30))),  # IST timezone
        instrument_type=symbol_parts["instrument_type"],
        expiry=symbol_parts["expiry"],
        strike=symbol_parts["strike"] if "strike" in symbol_parts else None,
        option_type=symbol_parts["option_type"] if "option_type" in symbol_parts else None
    )
    return position

def parse_fno_symbol(symbol):
    """
    Parse Indian market F&O symbol format.
    
    Examples:
        "NIFTY25FEB18000CE" -> 
            {
                "underlying": "NIFTY",
                "expiry": "25FEB",
                "strike": 18000,
                "option_type": "CE",
                "instrument_type": "OPTION"
            }
        "NIFTY25FEBFUT" -> 
            {
                "underlying": "NIFTY",
                "expiry": "25FEB",
                "instrument_type": "FUTURE"
            }
    
    Args:
        symbol (str): F&O symbol in Indian market format
    
    Returns:
        dict: Parsed symbol components
    """
    # Format validation
    if not re.match(NSE_FNO_PATTERN, symbol):
        raise ValidationError(
            f"Invalid F&O symbol format: {symbol}", 
            "PositionTracker"
        )
    
    result = {}
    
    # Extract underlying (everything before the first digit)
    underlying_match = re.match(r'^([A-Z]+)', symbol)
    if underlying_match:
        result["underlying"] = underlying_match.group(1)
    
    # Extract expiry (date part after underlying)
    expiry_match = re.search(r'([0-9]{2}[A-Z]{3})', symbol)
    if expiry_match:
        result["expiry"] = expiry_match.group(1)
    
    # Check if it's a future or option
    if symbol.endswith("FUT"):
        result["instrument_type"] = "FUTURE"
    else:
        result["instrument_type"] = "OPTION"
        
        # For options, extract strike price and option type
        option_match = re.search(r'([0-9]+)([CP]E)', symbol)
        if option_match:
            result["strike"] = int(option_match.group(1))
            result["option_type"] = option_match.group(2)
    
    return result
```

## Timezone Handling for Indian Market

Use this helper function for consistent timezone handling:

```python
def get_ist_now():
    """
    Get current datetime in Indian Standard Time (IST).
    
    Returns:
        datetime: Current datetime with IST timezone (UTC+5:30)
    """
    return datetime.now(timezone(timedelta(hours=5, minutes=30)))
```

## Edge Case Handling

### Zero Quantity Positions

```python
def test_zero_quantity_position(mock_position_tracker):
    """Test that positions with zero quantity are handled properly."""
    # Create a position
    position = Position(
        symbol="INFY",
        quantity=100,
        average_price=1800.0,
        current_price=1800.0,
        unrealized_pnl=0.0,
        realized_pnl=0.0,
        last_updated=get_ist_now()
    )
    
    # Add position to tracker
    mock_position_tracker.positions["INFY"] = position
    
    # Create a trade that closes the position
    trade = Trade(
        trade_id="TRD001",
        symbol="INFY",
        quantity=100,
        price=1850.0,
        side="SELL",
        timestamp=get_ist_now(),
        exchange="NSE",
        order_id="ORD001"
    )
    
    # Update position with the trade
    mock_position_tracker.update_position(trade)
    
    # Position should now have zero quantity
    updated_position = mock_position_tracker.get_position("INFY")
    assert updated_position.quantity == 0
    
    # Verify realized P&L calculation is correct
    assert updated_position.realized_pnl == (1850.0 - 1800.0) * 100
```

### P&L Calculation for Various Scenarios

```python
@pytest.mark.parametrize("scenario", [
    # Simple buy-then-sell
    {
        "trades": [
            {"side": "BUY", "price": 100.0, "quantity": 10},
            {"side": "SELL", "price": 120.0, "quantity": 10}
        ],
        "expected_realized_pnl": 200.0,  # (120 - 100) * 10
        "expected_quantity": 0
    },
    # Partial sell
    {
        "trades": [
            {"side": "BUY", "price": 100.0, "quantity": 10},
            {"side": "SELL", "price": 120.0, "quantity": 5}
        ],
        "expected_realized_pnl": 100.0,  # (120 - 100) * 5
        "expected_quantity": 5
    },
    # Multiple buys at different prices, then sell
    {
        "trades": [
            {"side": "BUY", "price": 100.0, "quantity": 5},
            {"side": "BUY", "price": 110.0, "quantity": 5},
            {"side": "SELL", "price": 120.0, "quantity": 10}
        ],
        "expected_realized_pnl": 150.0,  # (120 - 105) * 10, avg price = 105
        "expected_quantity": 0
    },
])
def test_pnl_calculation_scenarios(mock_position_tracker, scenario):
    """Test P&L calculation for various trading scenarios."""
    symbol = "RELIANCE"
    
    # Process each trade
    for i, trade_data in enumerate(scenario["trades"]):
        trade = Trade(
            trade_id=f"TRD{i+1:03d}",
            symbol=symbol,
            quantity=trade_data["quantity"],
            price=trade_data["price"],
            side=trade_data["side"],
            timestamp=get_ist_now() - timedelta(minutes=5 * (len(scenario["trades"]) - i)),
            exchange="NSE",
            order_id=f"ORD{i+1:03d}"
        )
        mock_position_tracker.update_position(trade)
    
    # Verify results
    position = mock_position_tracker.get_position(symbol)
    assert position.quantity == scenario["expected_quantity"]
    assert position.realized_pnl == pytest.approx(scenario["expected_realized_pnl"])
```

## Implementation Best Practices

### IST Timezone Consistency

Always ensure all timestamps are in IST timezone:

```python
# At the top of conftest.py
IST = timezone(timedelta(hours=5, minutes=30))

# When creating datetimes
timestamp = datetime.now(IST)

# When parsing string dates
date_str = "2023-02-25 09:15:00"
parsed_date = datetime.strptime(date_str, "%Y-%m-%d %H:%M:%S").replace(tzinfo=IST)
```

### Realistic Mock Behavior

Ensure mocks validate inputs and maintain state:

1. Record all method calls for verification
2. Implement business logic in mocks (not just return values)
3. Raise appropriate exceptions for invalid inputs
4. Maintain consistent state across method calls

### Fixture Parameterization

Make fixtures flexible with parameterization:

```python
# Example usage:
@pytest.mark.parametrize(
    "sample_position", 
    [
        {"symbol": "INFY", "quantity": 50, "average_price": 1800.0},
        {"symbol": "RELIANCE", "quantity": 100, "average_price": 2500.0},
        {"symbol": "NIFTY25FEB18000CE", "quantity": 50, "average_price": 150.0}
    ], 
    indirect=True
)
def test_position_update(sample_position, mock_position_tracker):
    # Test with different position configurations
    pass
```

### Testing F&O Specifics

Always test F&O positions with:
1. Correct lot sizes (e.g., 50 for NIFTY options)
2. Proper expiry handling
3. Strike price and option type validation
4. Special P&L calculations for options
