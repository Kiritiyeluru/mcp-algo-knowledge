# Order Management Knowledge Map

This document outlines the order management component of the MCP Algorithmic Trading System, responsible for creating, validating, modifying, and tracking orders.

## 1️⃣ Interfaces

### OrderManagerBase
- **File Path**: `/src/interfaces/order_management/base.py`
- **Core Methods**:
  - `create_order(order: Order) -> str`: Create and submit a new order
  - `cancel_order(order_id: str) -> bool`: Cancel an existing order
  - `modify_order(order_id: str, modifications: Dict[str, Any]) -> bool`: Modify an existing order
  - `get_order_status(order_id: str) -> OrderStatus`: Get the current status of an order
  - `get_orders(filters: Optional[Dict[str, Any]] = None) -> List[Order]`: Get orders matching filters

### OrderValidatorBase
- **File Path**: `/src/interfaces/order_management/base.py`
- **Core Methods**:
  - `validate_order(order: Order) -> bool`: Validate an order before submission
  - `validate_modification(order_id: str, modifications: Dict[str, Any]) -> bool`: Validate order modifications

### OrderExecutionHandlerBase
- **File Path**: `/src/interfaces/order_management/base.py`
- **Core Methods**:
  - `execute_order(order: Order) -> str`: Execute an order
  - `handle_order_update(update: OrderUpdate) -> None`: Handle order status updates

## 2️⃣ Models

### OrderType
- **File Path**: `/src/interfaces/order_management/models.py`
- **Values**: `MARKET`, `LIMIT`, `STOP`, `STOP_LIMIT`, `BRACKET`, `COVER`, `AMO`

### OrderStatus
- **File Path**: `/src/interfaces/order_management/models.py`
- **Values**: `CREATED`, `VALIDATED`, `PENDING`, `OPEN`, `FILLED`, `CANCELLED`, `REJECTED`
- **Valid Transitions**:
  - `CREATED → VALIDATED → PENDING → OPEN → FILLED`
  - `CREATED → REJECTED`
  - `VALIDATED → REJECTED`
  - `PENDING → REJECTED`
  - `OPEN → CANCELLED`

### Order
- **File Path**: `/src/interfaces/order_management/models.py`
- **Properties**:
  - `id: Optional[str]`: Order ID (None until submitted)
  - `symbol: str`: Trading symbol
  - `quantity: int`: Order quantity
  - `price: Optional[float]`: Order price (None for market orders)
  - `trigger_price: Optional[float]`: Trigger price for stop orders
  - `order_type: OrderType`: Type of order
  - `status: OrderStatus`: Current order status
  - `created_at: datetime`: Order creation timestamp
  - `updated_at: Optional[datetime]`: Last update timestamp
  - `exchange: str`: Exchange (NSE, BSE, etc.)
  - `product_type: str`: Product type (MIS, CNC, NRML)
  - `validity: str`: Order validity (DAY, IOC)
  - `parent_order_id: Optional[str]`: Parent order ID for bracket/cover orders
  - `is_amo: bool`: After market order flag

### OrderUpdate
- **File Path**: `/src/interfaces/order_management/models.py`
- **Properties**:
  - `order_id: str`: Order ID
  - `status: OrderStatus`: New order status
  - `filled_quantity: int`: Quantity filled
  - `remaining_quantity: int`: Remaining quantity
  - `average_price: Optional[float]`: Average fill price
  - `timestamp: datetime`: Update timestamp
  - `message: Optional[str]`: Additional information

## 3️⃣ Validation Rules

- **Symbol**: Valid alphanumeric with optional exchange prefix
- **Quantity**: Must be positive and adhere to lot size requirements
- **Price**: Must be positive for LIMIT orders and within circuit limits
- **Trigger Price**: Required and positive for STOP and STOP_LIMIT orders
- **Order Type**: Must be one of the defined enum values
- **Product Type**: Must be valid for the given exchange (MIS, CNC, NRML)
- **AMO Orders**: Must be submitted during allowed hours
- **F&O Orders**: Must have valid expiry date and strike price format

## 4️⃣ Test Patterns

### Test Classes
- `TestOrder`: Tests for the Order model
- `TestOrderManager`: Tests for the order manager interface
- `TestOrderValidator`: Tests for order validation
- `TestOrderExecution`: Tests for order execution

### Standard Test Methods
- `test_order_initialization()`: Test order initialization with valid parameters
- `test_order_validation()`: Test order validation logic
- `test_order_status_transitions()`: Test order status transitions
- `test_order_execution()`: Test order execution flow
- `test_order_cancellation()`: Test order cancellation
- `test_order_modification()`: Test order modification

## 5️⃣ Common Fixtures

### `sample_order()`
- **Returns**: A valid Order instance
- **Parameters**:
  - `symbol: str = "RELIANCE"`: Trading symbol
  - `quantity: int = 10`: Order quantity
  - `price: Optional[float] = 2500.0`: Order price
  - `order_type: OrderType = OrderType.LIMIT`: Order type

### `mock_order_manager()`
- **Returns**: A mock implementation of OrderManagerBase

### `mock_validator()`
- **Returns**: A mock implementation of OrderValidatorBase

## 6️⃣ Indian Market-Specific Features

- **Order Varieties**: MIS (intraday), CNC (delivery), NRML (F&O)
- **Exchange Requirements**: NSE, BSE specific order formats
- **Circuit Limits**: Price bands for order validation
- **Trading Hours**: Orders accepted 9:15 AM - 3:30 PM IST
- **After-Market Orders**: Special handling for AMO orders
- **F&O Order Format**: Support for futures and options with proper syntax
- **Block/Bulk Deals**: Support for large order special handling

## 7️⃣ Implementation Priorities

1. Core order model and status transitions
2. Order validation logic
3. Order submission and tracking
4. Order cancellation and modification
5. Indian market-specific order handling
