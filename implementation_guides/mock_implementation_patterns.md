# Mock Implementation Patterns

This guide provides best practices and concrete examples for implementing effective mock classes for testing the MCP Algorithmic Trading System.

## Core Principles for Effective Mocks

1. **Record All Method Calls**: Track method invocations for verification
2. **Implement Business Logic**: Don't just return canned responses
3. **Handle Edge Cases**: Include validation and error handling
4. **Maintain State**: Keep internal state consistent across method calls
5. **Simulate Realistic Behavior**: Model real-world component behavior

## Standard Mock Implementation Pattern

```python
class MockComponent(ComponentBase):
    """Mock implementation of ComponentBase for testing."""
    
    def __init__(self):
        """Initialize with tracking structures for method calls."""
        # Record of method calls
        self.method1_calls = []
        self.method2_calls = []
        
        # Internal state
        self.internal_data = {}
        self.initialized = True
    
    def method1(self, param1, param2):
        """
        Mock implementation of method1.
        
        Args:
            param1: First parameter
            param2: Second parameter
            
        Returns:
            Result based on the parameters
            
        Raises:
            ValidationError: If parameters are invalid
        """
        # Record the call
        self.method1_calls.append((param1, param2))
        
        # Validate inputs
        if param1 is None:
            raise ValidationError("param1 cannot be None", "MockComponent")
            
        # Update internal state
        self.internal_data[param1] = param2
        
        # Return a calculated result
        return f"Processed: {param1}"
    
    def method2(self, param):
        """
        Mock implementation of method2.
        
        Args:
            param: A parameter
            
        Returns:
            Result based on internal state
        """
        # Record the call
        self.method2_calls.append(param)
        
        # Return result based on internal state
        return self.internal_data.get(param, "Not found")
```

## Advanced Mock Implementation Examples

### Market Data Provider Mock

```python
class MockMarketDataProvider(MarketDataProviderBase):
    """Mock implementation of MarketDataProvider for testing."""
    
    def __init__(self):
        """Initialize with subscription tracking and data storage."""
        self.subscriptions = {}  # {symbol: {data_type: True/False}}
        self.market_data = {}    # {symbol: {data_type: MarketData}}
        
        # Call tracking
        self.subscribe_calls = []
        self.unsubscribe_calls = []
        self.get_market_data_calls = []
        self.get_historical_data_calls = []
    
    def subscribe(self, symbol, data_type):
        """
        Mock implementation of subscribe.
        
        Args:
            symbol (str): Trading symbol
            data_type (DataType): Type of market data
            
        Returns:
            bool: Success flag
            
        Raises:
            ValidationError: If symbol or data_type is invalid
        """
        # Record the call
        self.subscribe_calls.append((symbol, data_type))
        
        # Validate inputs
        if not symbol:
            raise ValidationError("Symbol cannot be empty", "MarketDataProvider")
        
        if not isinstance(data_type, DataType):
            raise ValidationError("Invalid data type", "MarketDataProvider")
        
        # Update subscriptions
        if symbol not in self.subscriptions:
            self.subscriptions[symbol] = {}
        
        self.subscriptions[symbol][data_type] = True
        return True
    
    def unsubscribe(self, symbol, data_type):
        """
        Mock implementation of unsubscribe.
        
        Args:
            symbol (str): Trading symbol
            data_type (DataType): Type of market data
            
        Returns:
            bool: Success flag
        """
        # Record the call
        self.unsubscribe_calls.append((symbol, data_type))
        
        # Update subscriptions
        if symbol in self.subscriptions and data_type in self.subscriptions[symbol]:
            self.subscriptions[symbol][data_type] = False
            return True
        
        return False
    
    def get_market_data(self, symbol, data_type):
        """
        Mock implementation of get_market_data.
        
        Args:
            symbol (str): Trading symbol
            data_type (DataType): Type of market data
            
        Returns:
            MarketData: The requested market data
            
        Raises:
            ValidationError: If not subscribed to the requested data
        """
        # Record the call
        self.get_market_data_calls.append((symbol, data_type))
        
        # Check subscription
        if not (symbol in self.subscriptions and 
                data_type in self.subscriptions[symbol] and 
                self.subscriptions[symbol][data_type]):
            raise ValidationError(
                f"Not subscribed to {data_type} for {symbol}", 
                "MarketDataProvider"
            )
        
        # Return data if available, or generate default data
        if symbol in self.market_data and data_type in self.market_data[symbol]:
            return self.market_data[symbol][data_type]
        
        # Generate default data
        if data_type == DataType.TICK:
            data = TickData(
                symbol=symbol,
                timestamp=datetime.now(IST),
                data_type=DataType.TICK,
                last_price=100.0,
                last_quantity=10,
                volume=1000,
                open_interest=500
            )
        elif data_type == DataType.OHLC:
            data = OHLCData(
                symbol=symbol,
                timestamp=datetime.now(IST),
                data_type=DataType.OHLC,
                open=100.0,
                high=105.0,
                low=95.0,
                close=102.0,
                volume=1000,
                interval="1m"
            )
        else:
            data = MarketData(
                symbol=symbol,
                timestamp=datetime.now(IST),
                data_type=data_type,
                data={}
            )
        
        # Store and return the data
        if symbol not in self.market_data:
            self.market_data[symbol] = {}
        
        self.market_data[symbol][data_type] = data
        return data
    
    def get_historical_data(self, symbol, data_type, start_time, end_time):
        """
        Mock implementation of get_historical_data.
        
        Args:
            symbol (str): Trading symbol
            data_type (DataType): Type of market data
            start_time (datetime): Start time
            end_time (datetime): End time
            
        Returns:
            List[MarketData]: Historical market data
            
        Raises:
            ValidationError: If time range is invalid
        """
        # Record the call
        self.get_historical_data_calls.append((symbol, data_type, start_time, end_time))
        
        # Validate inputs
        if start_time >= end_time:
            raise ValidationError(
                "Start time must be before end time",
                "MarketDataProvider"
            )
        
        # Generate historical data
        result = []
        current_time = start_time
        
        while current_time < end_time:
            if data_type == DataType.TICK:
                data = TickData(
                    symbol=symbol,
                    timestamp=current_time,
                    data_type=DataType.TICK,
                    last_price=100.0 + (random.random() * 10 - 5),
                    last_quantity=random.randint(1, 100),
                    volume=random.randint(100, 10000),
                    open_interest=random.randint(100, 1000)
                )
            elif data_type == DataType.OHLC:
                base_price = 100.0 + (random.random() * 10 - 5)
                data = OHLCData(
                    symbol=symbol,
                    timestamp=current_time,
                    data_type=DataType.OHLC,
                    open=base_price,
                    high=base_price * (1 + random.random() * 0.02),
                    low=base_price * (1 - random.random() * 0.02),
                    close=base_price * (1 + random.random() * 0.01 - 0.005),
                    volume=random.randint(100, 10000),
                    interval="1m"
                )
            else:
                data = MarketData(
                    symbol=symbol,
                    timestamp=current_time,
                    data_type=data_type,
                    data={}
                )
            
            result.append(data)
            current_time += timedelta(minutes=1)
        
        return result
```

### Order Manager Mock

```python
class MockOrderManager(OrderManagerBase):
    """Mock implementation of OrderManager for testing."""
    
    def __init__(self):
        """Initialize with order tracking."""
        self.orders = {}  # {order_id: Order}
        self.next_order_id = 1
        
        # Call tracking
        self.create_order_calls = []
        self.cancel_order_calls = []
        self.modify_order_calls = []
        self.get_order_status_calls = []
        self.get_orders_calls = []
    
    def create_order(self, order):
        """
        Mock implementation of create_order.
        
        Args:
            order (Order): Order to create
            
        Returns:
            str: Order ID
            
        Raises:
            ValidationError: If order is invalid
        """
        # Record the call
        self.create_order_calls.append(order)
        
        # Validate inputs
        if not order.symbol:
            raise ValidationError("Symbol cannot be empty", "OrderManager")
        
        if order.quantity <= 0:
            raise ValidationError("Quantity must be positive", "OrderManager")
        
        if order.order_type == OrderType.LIMIT and (order.price is None or order.price <= 0):
            raise ValidationError("Limit order must have a positive price", "OrderManager")
        
        # Generate order ID
        order_id = f"ORD{self.next_order_id:06d}"
        self.next_order_id += 1
        
        # Create a copy of the order with the ID and initial status
        new_order = copy.deepcopy(order)
        new_order.id = order_id
        new_order.status = OrderStatus.VALIDATED
        new_order.created_at = datetime.now(IST)
        new_order.updated_at = datetime.now(IST)
        
        # Store the order
        self.orders[order_id] = new_order
        
        # Simulate order progression
        # In a real implementation, this would happen asynchronously
        # For testing, we can simulate immediate acceptance
        new_order.status = OrderStatus.PENDING
        new_order.updated_at = datetime.now(IST)
        
        # For market orders, simulate immediate fill
        if order.order_type == OrderType.MARKET:
            new_order.status = OrderStatus.OPEN
            new_order.updated_at = datetime.now(IST)
            
            # Simulate fill
            new_order.status = OrderStatus.FILLED
            new_order.updated_at = datetime.now(IST)
        
        return order_id
    
    def cancel_order(self, order_id):
        """
        Mock implementation of cancel_order.
        
        Args:
            order_id (str): Order ID to cancel
            
        Returns:
            bool: Success flag
            
        Raises:
            ValidationError: If order cannot be cancelled
        """
        # Record the call
        self.cancel_order_calls.append(order_id)
        
        # Validate inputs
        if order_id not in self.orders:
            raise ValidationError(f"Order {order_id} not found", "OrderManager")
        
        order = self.orders[order_id]
        
        # Check if order can be cancelled
        if order.status in [OrderStatus.FILLED, OrderStatus.CANCELLED, OrderStatus.REJECTED]:
            raise ValidationError(
                f"Cannot cancel order in {order.status} state", 
                "OrderManager"
            )
        
        # Update order status
        order.status = OrderStatus.CANCELLED
        order.updated_at = datetime.now(IST)
        
        return True
    
    def modify_order(self, order_id, modifications):
        """
        Mock implementation of modify_order.
        
        Args:
            order_id (str): Order ID to modify
            modifications (Dict[str, Any]): Modifications to apply
            
        Returns:
            bool: Success flag
            
        Raises:
            ValidationError: If order cannot be modified
        """
        # Record the call
        self.modify_order_calls.append((order_id, modifications))
        
        # Validate inputs
        if order_id not in self.orders:
            raise ValidationError(f"Order {order_id} not found", "OrderManager")
        
        order = self.orders[order_id]
        
        # Check if order can be modified
        if order.status not in [OrderStatus.VALIDATED, OrderStatus.PENDING, OrderStatus.OPEN]:
            raise ValidationError(
                f"Cannot modify order in {order.status} state", 
                "OrderManager"
            )
        
        # Validate modifications
        if "price" in modifications and modifications["price"] <= 0:
            raise ValidationError("Price must be positive", "OrderManager")
        
        if "quantity" in modifications and modifications["quantity"] <= 0:
            raise ValidationError("Quantity must be positive", "OrderManager")
        
        # Apply modifications
        for key, value in modifications.items():
            if hasattr(order, key):
                setattr(order, key, value)
        
        # Update timestamp
        order.updated_at = datetime.now(IST)
        
        return True
    
    def get_order_status(self, order_id):
        """
        Mock implementation of get_order_status.
        
        Args:
            order_id (str): Order ID
            
        Returns:
            OrderStatus: Current order status
            
        Raises:
            ValidationError: If order not found
        """
        # Record the call
        self.get_order_status_calls.append(order_id)
        
        # Validate inputs
        if order_id not in self.orders:
            raise ValidationError(f"Order {order_id} not found", "OrderManager")
        
        return self.orders[order_id].status
    
    def get_orders(self, filters=None):
        """
        Mock implementation of get_orders.
        
        Args:
            filters (Dict[str, Any], optional): Filters to apply
            
        Returns:
            List[Order]: Orders matching the filters
        """
        # Record the call
        self.get_orders_calls.append(filters)
        
        # Default to all orders
        if filters is None:
            return list(self.orders.values())
        
        # Apply filters
        result = []
        for order in self.orders.values():
            match = True
            
            for key, value in filters.items():
                if hasattr(order, key) and getattr(order, key) != value:
                    match = False
                    break
            
            if match:
                result.append(order)
        
        return result
```

## Mock Parameter Injection

Effective mocks allow for behavior customization through parameter injection:

```python
class ConfigurableMockStrategy(StrategyBase):
    """Mock strategy with configurable behavior."""
    
    def __init__(self, signals_to_generate=None, errors_to_raise=None):
        """
        Initialize with configurable behavior.
        
        Args:
            signals_to_generate (Dict[str, List[Signal]], optional): 
                Signals to generate for each symbol
            errors_to_raise (Dict[str, Exception], optional):
                Errors to raise for specific operations
        """
        self.signals_to_generate = signals_to_generate or {}
        self.errors_to_raise = errors_to_raise or {}
        
        # Call tracking
        self.on_market_data_calls = []
        self.generate_signals_calls = []
    
    def on_market_data(self, market_data):
        """
        Mock implementation of on_market_data.
        
        Args:
            market_data (MarketData): Market data update
            
        Raises:
            Exception: If configured to raise for 'on_market_data'
        """
        # Record the call
        self.on_market_data_calls.append(market_data)
        
        # Check if should raise error
        if 'on_market_data' in self.errors_to_raise:
            raise self.errors_to_raise['on_market_data']
    
    def generate_signals(self, symbol):
        """
        Mock implementation of generate_signals.
        
        Args:
            symbol (str): Symbol to generate signals for
            
        Returns:
            List[Signal]: Generated signals
            
        Raises:
            Exception: If configured to raise for 'generate_signals'
        """
        # Record the call
        self.generate_signals_calls.append(symbol)
        
        # Check if should raise error
        if 'generate_signals' in self.errors_to_raise:
            raise self.errors_to_raise['generate_signals']
        
        # Return pre-configured signals
        return self.signals_to_generate.get(symbol, [])
```

## Testing with Configurable Mocks

```python
def test_strategy_error_handling():
    """Test strategy error handling with configurable mock."""
    # Create mock with configured error
    mock_strategy = ConfigurableMockStrategy(
        errors_to_raise={
            'generate_signals': ValidationError("Test error", "Strategy")
        }
    )
    
    # Test that the error is raised
    with pytest.raises(ValidationError) as exc_info:
        mock_strategy.generate_signals("RELIANCE")
    
    assert "Test error" in str(exc_info.value)
    assert len(mock_strategy.generate_signals_calls) == 1
    assert mock_strategy.generate_signals_calls[0] == "RELIANCE"

def test_strategy_signal_generation():
    """Test strategy signal generation with configurable mock."""
    # Create sample signals
    signals = [
        Signal(symbol="RELIANCE", signal_type="BUY", price=2500.0, quantity=10),
        Signal(symbol="RELIANCE", signal_type="SELL", price=2600.0, quantity=10)
    ]
    
    # Create mock with configured signals
    mock_strategy = ConfigurableMockStrategy(
        signals_to_generate={"RELIANCE": signals}
    )
    
    # Test signal generation
    result = mock_strategy.generate_signals("RELIANCE")
    
    assert len(result) == 2
    assert result[0].signal_type == "BUY"
    assert result[1].signal_type == "SELL"
    assert len(mock_strategy.generate_signals_calls) == 1
```

## Simulating External Dependencies

For mocks that need to simulate broker APIs, network errors, or market conditions:

```python
class BrokerConnectivityMock:
    """Mock for testing connectivity issues with broker."""
    
    def __init__(self, success_rate=1.0, connection_delay=0.0):
        """
        Initialize with configurable behavior.
        
        Args:
            success_rate (float): Rate of successful connections (0.0 to 1.0)
            connection_delay (float): Simulated delay in seconds
        """
        self.success_rate = success_rate
        self.connection_delay = connection_delay
        self.connection_attempts = 0
        self.successful_connections = 0
    
    def connect(self):
        """
        Simulate connection to broker.
        
        Returns:
            bool: Success flag
            
        Raises:
            ConnectionError: If connection fails
        """
        self.connection_attempts += 1
        
        # Simulate connection delay
        if self.connection_delay > 0:
            time.sleep(self.connection_delay)
        
        # Simulate connection success/failure
        if random.random() < self.success_rate:
            self.successful_connections += 1
            return True
        else:
            raise ConnectionError("Failed to connect to broker")
```

## Best Practices for Mock Testing

1. **Test All Error Paths**: Configure mocks to simulate all possible errors
2. **Verify Method Calls**: Check that methods are called with expected parameters
3. **Test State Changes**: Verify that internal state is updated correctly
4. **Test Boundary Conditions**: Set up mocks to test boundary values
5. **Test Realistic Scenarios**: Configure mocks to simulate realistic trading scenarios

```python
def test_order_lifecycle():
    """Test complete order lifecycle with mock order manager."""
    # Create mock order manager
    mock_order_manager = MockOrderManager()
    
    # Create order
    order = Order(
        symbol="RELIANCE",
        quantity=10,
        price=2500.0,
        order_type=OrderType.LIMIT,
        exchange="NSE",
        product_type="MIS"
    )
    
    # Submit order
    order_id = mock_order_manager.create_order(order)
    
    # Verify order was created
    assert order_id in mock_order_manager.orders
    assert mock_order_manager.orders[order_id].status in [OrderStatus.VALIDATED, OrderStatus.PENDING, OrderStatus.OPEN]
    assert len(mock_order_manager.create_order_calls) == 1
    
    # Modify order
    mock_order_manager.modify_order(order_id, {"price": 2550.0})
    
    # Verify order was modified
    assert mock_order_manager.orders[order_id].price == 2550.0
    assert len(mock_order_manager.modify_order_calls) == 1
    
    # Cancel order
    mock_order_manager.cancel_order(order_id)
    
    # Verify order was cancelled
    assert mock_order_manager.orders[order_id].status == OrderStatus.CANCELLED
    assert len(mock_order_manager.cancel_order_calls) == 1
```

## Indian Market Specific Mock Behavior

For Indian market-specific behaviors, mocks should include:

1. **Circuit Limits**: Upper/lower circuit limits for price validation
2. **Trading Hours**: Validation for market open/close times
3. **F&O Expiry**: Special handling for expiry days and contract rollovers
4. **Exchange-Specific Errors**: NSE/BSE-specific error messages

```python
class IndianMarketMock:
    """Mock for Indian market-specific behaviors."""
    
    def __init__(self):
        """Initialize with Indian market parameters."""
        self.market_open_time = time(9, 15)  # 9:15 AM
        self.market_close_time = time(15, 30)  # 3:30 PM
        self.circuit_limits = {
            "RELIANCE": (0.10, 0.10),  # (lower_limit_pct, upper_limit_pct)
            "NIFTY": (0.15, 0.15),
            "default": (0.20, 0.20)
        }
        self.lot_sizes = {
            "NIFTY": 50,
            "BANKNIFTY": 25,
            "RELIANCE": 250,
            "default": 1
        }
    
    def is_market_open(self, check_time=None):
        """
        Check if market is open at the given time.
        
        Args:
            check_time (datetime, optional): Time to check
            
        Returns:
            bool: Whether market is open
        """
        if check_time is None:
            check_time = datetime.now(IST)
        
        # Check if weekend
        if check_time.weekday() >= 5:  # 5=Saturday, 6=Sunday
            return False
        
        # Check time of day
        time_of_day = check_time.time()
        return self.market_open_time <= time_of_day < self.market_close_time
    
    def get_circuit_limits(self, symbol, reference_price):
        """
        Get circuit limits for a symbol.
        
        Args:
            symbol (str): Trading symbol
            reference_price (float): Reference price
            
        Returns:
            Tuple[float, float]: (lower_limit, upper_limit)
        """
        limits = self.circuit_limits.get(symbol, self.circuit_limits["default"])
        lower_limit = reference_price * (1 - limits[0])
        upper_limit = reference_price * (1 + limits[1])
        return (lower_limit, upper_limit)
    
    def validate_order_price(self, symbol, price, reference_price):
        """
        Validate order price against circuit limits.
        
        Args:
            symbol (str): Trading symbol
            price (float): Order price
            reference_price (float): Reference price
            
        Returns:
            bool: Whether price is valid
            
        Raises:
            ValidationError: If price is outside circuit limits
        """
        lower_limit, upper_limit = self.get_circuit_limits(symbol, reference_price)
        
        if price < lower_limit:
            raise ValidationError(
                f"Price {price} below lower circuit limit {lower_limit}", 
                "IndianMarket"
            )
        
        if price > upper_limit:
            raise ValidationError(
                f"Price {price} above upper circuit limit {upper_limit}", 
                "IndianMarket"
            )
        
        return True
    
    def get_lot_size(self, symbol):
        """
        Get lot size for a symbol.
        
        Args:
            symbol (str): Trading symbol
            
        Returns:
            int: Lot size
        """
        # For F&O, extract the underlying
        underlying = symbol
        if re.match(NSE_FNO_PATTERN, symbol):
            underlying_match = re.match(r'^([A-Z]+)', symbol)
            if underlying_match:
                underlying = underlying_match.group(1)
        
        return self.lot_sizes.get(underlying, self.lot_sizes["default"])
    
    def validate_quantity(self, symbol, quantity):
        """
        Validate order quantity against lot size.
        
        Args:
            symbol (str): Trading symbol
            quantity (int): Order quantity
            
        Returns:
            bool: Whether quantity is valid
            
        Raises:
            ValidationError: If quantity is not a multiple of lot size
        """
        lot_size = self.get_lot_size(symbol)
        
        if quantity % lot_size != 0:
            raise ValidationError(
                f"Quantity {quantity} not a multiple of lot size {lot_size}", 
                "IndianMarket"
            )
        
        return True
```

## Conclusion

Building effective mocks is critical for thorough testing of the MCP Algorithmic Trading System. By following these patterns, your mocks will:

1. Provide realistic behavior that closely mimics production components
2. Enable testing of both success and failure paths
3. Verify correct interaction between components
4. Simulate Indian market-specific behaviors and constraints
5. Support configurable behavior for different test scenarios

Remember that the goal of mocks is not just to provide test isolation, but to create realistic simulations that ensure your system behaves correctly under all conditions.
