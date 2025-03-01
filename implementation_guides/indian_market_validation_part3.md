# Indian Market Validation Guide - Part 3

This guide provides comprehensive validation rules, code examples, and implementation patterns specific to Indian markets (NSE/BSE) for the MCP Algorithmic Trading System.

## Table of Contents for Complete Guide

1. [Market Timing Validation](indian_market_validation_part1.md#market-timing-validation) (Part 1)
2. [Symbol Format Validation](indian_market_validation_part1.md#symbol-format-validation) (Part 1)
3. [Circuit Limit Implementation](indian_market_validation_part1.md#circuit-limit-implementation) (Part 1)
4. [Lot Size Validation](indian_market_validation_part2.md#lot-size-validation) (Part 2)
5. [Product Type Validation](indian_market_validation_part2.md#product-type-validation) (Part 2)
6. [Order Value Constraints](indian_market_validation_part2.md#order-value-constraints) (Part 2)
7. [Market-Specific Order Types](#market-specific-order-types) (Part 3)
8. [F&O-Specific Validation](#fo-specific-validation) (Part 3)
9. [Comprehensive Validation Implementation](#comprehensive-validation-implementation) (Part 3)

## Market-Specific Order Types

Indian markets support several specialized order types with specific validation rules and behaviors.

### Order Type Constants

```python
from enum import Enum

class OrderType(Enum):
    """Order types for Indian markets."""
    MARKET = "MARKET"       # Market order
    LIMIT = "LIMIT"         # Limit order
    SL = "SL"               # Stop-loss order (Stop-loss market)
    SL_LIMIT = "SL_LIMIT"   # Stop-loss limit order
    BRACKET = "BRACKET"     # Bracket order
    COVER = "COVER"         # Cover order
    AMO = "AMO"             # After-market order
    GTT = "GTT"             # Good-till-triggered order


class OrderValidity(Enum):
    """Order validity types for Indian markets."""
    DAY = "DAY"             # Valid for the day
    IOC = "IOC"             # Immediate or cancel
    GTC = "GTC"             # Good till cancelled (for GTT orders)
```

### Bracket Order Implementation

Bracket orders are a special type in Indian markets that include a main order with automatic stop-loss and target orders.

```python
class BracketOrderParameters:
    """Parameters for bracket orders."""
    
    def __init__(self, 
                 entry_price, 
                 stop_loss_price, 
                 target_price, 
                 trailing_stop_loss=None):
        """
        Initialize bracket order parameters.
        
        Args:
            entry_price (float): Entry price for the main order
            stop_loss_price (float): Stop-loss price
            target_price (float): Target profit price
            trailing_stop_loss (float, optional): Trailing stop-loss points
        """
        self.entry_price = entry_price
        self.stop_loss_price = stop_loss_price
        self.target_price = target_price
        self.trailing_stop_loss = trailing_stop_loss

def validate_bracket_order(symbol, order_type, quantity, entry_price, 
                           stop_loss_price, target_price, product_type):
    """
    Validate parameters for a bracket order.
    
    Args:
        symbol (str): Trading symbol
        order_type (OrderType): Order type
        quantity (int): Order quantity
        entry_price (float): Entry price
        stop_loss_price (float): Stop-loss price
        target_price (float): Target price
        product_type (ProductType): Product type
        
    Returns:
        bool: True if bracket order is valid
        
    Raises:
        ValidationError: If bracket order parameters are invalid
    """
    # Only LIMIT entry orders are allowed for bracket orders
    if order_type != OrderType.LIMIT:
        raise ValidationError(
            "Bracket orders can only be placed with LIMIT entry orders",
            "BracketOrderValidator"
        )
    
    # Only MIS product type is allowed for bracket orders
    if product_type != ProductType.MIS:
        raise ValidationError(
            "Bracket orders can only be placed with MIS product type",
            "BracketOrderValidator"
        )
    
    # Validate basic order parameters
    validate_symbol(symbol)
    validate_order_quantity(quantity, symbol)
    validate_order_value(entry_price, quantity)
    
    # For buy orders
    is_buy = True  # Assuming it's a buy order for this example
    
    if is_buy:
        # Stop-loss must be below entry price
        if stop_loss_price >= entry_price:
            raise ValidationError(
                "For buy orders, stop-loss price must be below entry price",
                "BracketOrderValidator"
            )
        
        # Target must be above entry price
        if target_price <= entry_price:
            raise ValidationError(
                "For buy orders, target price must be above entry price",
                "BracketOrderValidator"
            )
    else:  # Sell order
        # Stop-loss must be above entry price
        if stop_loss_price <= entry_price:
            raise ValidationError(
                "For sell orders, stop-loss price must be above entry price",
                "BracketOrderValidator"
            )
        
        # Target must be below entry price
        if target_price >= entry_price:
            raise ValidationError(
                "For sell orders, target price must be below entry price",
                "BracketOrderValidator"
            )
    
    return True
```

### Cover Order Implementation

Cover orders are simplified bracket orders with only a stop-loss, popular in Indian markets.

```python
def validate_cover_order(symbol, order_type, quantity, entry_price, 
                         stop_loss_price, product_type):
    """
    Validate parameters for a cover order.
    
    Args:
        symbol (str): Trading symbol
        order_type (OrderType): Order type
        quantity (int): Order quantity
        entry_price (float): Entry price
        stop_loss_price (float): Stop-loss price
        product_type (ProductType): Product type
        
    Returns:
        bool: True if cover order is valid
        
    Raises:
        ValidationError: If cover order parameters are invalid
    """
    # Only MARKET and LIMIT entry orders are allowed for cover orders
    if order_type not in [OrderType.MARKET, OrderType.LIMIT]:
        raise ValidationError(
            "Cover orders can only be placed with MARKET or LIMIT entry orders",
            "CoverOrderValidator"
        )
    
    # Only MIS product type is allowed for cover orders
    if product_type != ProductType.MIS:
        raise ValidationError(
            "Cover orders can only be placed with MIS product type",
            "CoverOrderValidator"
        )
    
    # Validate basic order parameters
    validate_symbol(symbol)
    validate_order_quantity(quantity, symbol)
    
    if order_type == OrderType.LIMIT:
        validate_order_value(entry_price, quantity)
    
    # For buy orders
    is_buy = True  # Assuming it's a buy order for this example
    
    if is_buy:
        # Stop-loss must be below entry price
        if stop_loss_price >= entry_price:
            raise ValidationError(
                "For buy orders, stop-loss price must be below entry price",
                "CoverOrderValidator"
            )
    else:  # Sell order
        # Stop-loss must be above entry price
        if stop_loss_price <= entry_price:
            raise ValidationError(
                "For sell orders, stop-loss price must be above entry price",
                "CoverOrderValidator"
            )
    
    return True
```

### After-Market Order (AMO) Implementation

AMO orders allow traders to place orders outside normal market hours.

```python
def validate_amo_order(symbol, order_type, quantity, price, product_type, check_time=None):
    """
    Validate parameters for an after-market order (AMO).
    
    Args:
        symbol (str): Trading symbol
        order_type (OrderType): Order type (must be AMO)
        quantity (int): Order quantity
        price (float): Order price (for limit orders)
        product_type (ProductType): Product type
        check_time (datetime, optional): Time to check, defaults to current time
        
    Returns:
        bool: True if AMO order is valid
        
    Raises:
        ValidationError: If AMO order parameters are invalid
    """
    if check_time is None:
        check_time = datetime.now(IST)
    
    # Validate market session
    session = get_market_session(check_time)
    
    # AMO orders can only be placed outside market hours
    if session == MarketSession.NORMAL:
        raise ValidationError(
            "AMO orders cannot be placed during market hours (9:15 AM - 3:30 PM)",
            "AMOValidator"
        )
    
    # Validate basic order parameters
    validate_symbol(symbol)
    validate_order_quantity(quantity, symbol)
    validate_product_type(symbol, product_type)
    
    if order_type in [OrderType.LIMIT, OrderType.SL, OrderType.SL_LIMIT]:
        validate_order_value(price, quantity)
    
    # AMO orders will be processed at market open (9:15 AM)
    next_market_open = get_next_market_open(check_time)
    print(f"AMO order will be processed at {next_market_open}")
    
    return True

def get_next_market_open(check_time):
    """
    Get the next market open time.
    
    Args:
        check_time (datetime): Current time
        
    Returns:
        datetime: Next market open time
    """
    current_day = check_time.date()
    market_open = datetime.combine(current_day, MARKET_OPEN_TIME, tzinfo=IST)
    
    # If current time is after market open, get next trading day
    if check_time >= market_open:
        next_day = current_day + timedelta(days=1)
        
        # Skip weekends
        while next_day.weekday() >= 5:  # 5=Saturday, 6=Sunday
            next_day += timedelta(days=1)
        
        market_open = datetime.combine(next_day, MARKET_OPEN_TIME, tzinfo=IST)
    
    return market_open
```

### Usage Examples

```python
# Validate a bracket order
try:
    validate_bracket_order(
        symbol="RELIANCE",
        order_type=OrderType.LIMIT,
        quantity=10,
        entry_price=2500.0,
        stop_loss_price=2450.0,
        target_price=2550.0,
        product_type=ProductType.MIS
    )
    print("Bracket order is valid")
except ValidationError as e:
    print(f"Bracket order validation failed: {e}")

# Validate a cover order
try:
    validate_cover_order(
        symbol="INFY",
        order_type=OrderType.LIMIT,
        quantity=10,
        entry_price=1800.0,
        stop_loss_price=1780.0,
        product_type=ProductType.MIS
    )
    print("Cover order is valid")
except ValidationError as e:
    print(f"Cover order validation failed: {e}")

# Validate an AMO order
try:
    # Simulate after-hours time
    after_hours = datetime.now(IST).replace(hour=18, minute=0)
    
    validate_amo_order(
        symbol="TCS",
        order_type=OrderType.LIMIT,
        quantity=5,
        price=3000.0,
        product_type=ProductType.CNC,
        check_time=after_hours
    )
    print("AMO order is valid")
except ValidationError as e:
    print(f"AMO order validation failed: {e}")
```

## F&O-Specific Validation

F&O (Futures & Options) instruments in Indian markets have specific validation requirements related to expiry dates, strike prices, and premium calculations.

### Expiry Date Validation

```python
from datetime import date, timedelta

def get_last_thursday_of_month(year, month):
    """
    Get the last Thursday of a given month.
    
    Args:
        year (int): Year
        month (int): Month
        
    Returns:
        date: Last Thursday of the month
    """
    # Start with the last day of the month
    if month == 12:
        last_day = date(year + 1, 1, 1) - timedelta(days=1)
    else:
        last_day = date(year, month + 1, 1) - timedelta(days=1)
    
    # Calculate days until the previous Thursday (3 is Thursday)
    days_until_thursday = (last_day.weekday() - 3) % 7
    
    # Get the last Thursday
    last_thursday = last_day - timedelta(days=days_until_thursday)
    
    return last_thursday

def get_expiry_date(expiry_year, expiry_month):
    """
    Get the expiry date for a given month.
    
    Args:
        expiry_year (int): Expiry year
        expiry_month (str): Expiry month code (e.g., 'JAN', 'FEB')
        
    Returns:
        date: Expiry date
    """
    # Convert month code to month number
    month_dict = {
        'JAN': 1, 'FEB': 2, 'MAR': 3, 'APR': 4, 'MAY': 5, 'JUN': 6,
        'JUL': 7, 'AUG': 8, 'SEP': 9, 'OCT': 10, 'NOV': 11, 'DEC': 12
    }
    
    month_num = month_dict[expiry_month]
    
    # Get the last Thursday of the month
    expiry_date = get_last_thursday_of_month(int(expiry_year), month_num)
    
    return expiry_date

def validate_option_expiry(symbol):
    """
    Validate if an option's expiry date is valid.
    
    Args:
        symbol (str): F&O symbol
        
    Returns:
        bool: True if expiry is valid
        
    Raises:
        ValidationError: If expiry is invalid
    """
    try:
        # Parse the symbol to extract expiry information
        components = parse_fno_symbol(symbol)
        
        if components['type'] != 'option':
            raise ValidationError(
                f"Symbol {symbol} is not an option",
                "ExpiryValidator"
            )
        
        # Get the expiry date
        expiry_date = get_expiry_date(
            components['expiry_year'],
            components['expiry_month']
        )
        
        # Check if expiry date is in the past
        today = date.today()
        if expiry_date < today:
            raise ValidationError(
                f"Option has expired on {expiry_date}",
                "ExpiryValidator"
            )
        
        return True
    except Exception as e:
        if isinstance(e, ValidationError):
            raise
        else:
            raise ValidationError(
                f"Failed to validate option expiry: {str(e)}",
                "ExpiryValidator"
            )
```

### Strike Price Validation

```python
def validate_option_strike_price(symbol, current_price):
    """
    Validate if an option's strike price is valid relative to current price.
    
    Args:
        symbol (str): F&O symbol
        current_price (float): Current price of the underlying
        
    Returns:
        bool: True if strike price is valid
        
    Raises:
        ValidationError: If strike price is invalid
    """
    # Parse the symbol to extract strike price
    components = parse_fno_symbol(symbol)
    
    if components['type'] != 'option':
        raise ValidationError(
            f"Symbol {symbol} is not an option",
            "StrikeValidator"
        )
    
    strike_price = components['strike_price']
    option_type = components['option_type']
    
    # For index options, strikes must be multiples of specific intervals
    if components['underlying'] in ['NIFTY', 'BANKNIFTY', 'FINNIFTY']:
        # NIFTY typically uses 50-point intervals
        if components['underlying'] == 'NIFTY':
            if strike_price % 50 != 0:
                raise ValidationError(
                    f"NIFTY option strike price must be a multiple of 50, got {strike_price}",
                    "StrikeValidator"
                )
        
        # BANKNIFTY typically uses 100-point intervals
        if components['underlying'] == 'BANKNIFTY':
            if strike_price % 100 != 0:
                raise ValidationError(
                    f"BANKNIFTY option strike price must be a multiple of 100, got {strike_price}",
                    "StrikeValidator"
                )
    
    # For deep OTM options, make sure they're not too far from current price
    # This is a simple heuristic, in practice more sophisticated checks might be used
    distance_percentage = abs(strike_price - current_price) / current_price
    
    if distance_percentage > 0.30:  # 30% away from current price
        print(f"Warning: Strike price {strike_price} is {distance_percentage*100:.2f}% away from current price {current_price}")
    
    return True
```

### Option Premium Validation

```python
def validate_option_premium(symbol, premium, underlying_price):
    """
    Validate if an option's premium is reasonable.
    
    Args:
        symbol (str): F&O symbol
        premium (float): Option premium
        underlying_price (float): Price of the underlying
        
    Returns:
        bool: True if premium is valid
        
    Raises:
        ValidationError: If premium is unreasonable
    """
    # Parse the symbol to extract components
    components = parse_fno_symbol(symbol)
    
    if components['type'] != 'option':
        raise ValidationError(
            f"Symbol {symbol} is not an option",
            "PremiumValidator"
        )
    
    strike_price = components['strike_price']
    option_type = components['option_type']
    
    # Basic reasonability checks
    if premium < 0:
        raise ValidationError(
            f"Option premium cannot be negative: {premium}",
            "PremiumValidator"
        )
    
    # For deep ITM options, intrinsic value check
    if option_type == 'CE' and underlying_price > strike_price:
        intrinsic_value = underlying_price - strike_price
        if premium < intrinsic_value * 0.95:  # Allow for some margin of error
            raise ValidationError(
                f"Premium {premium} is below intrinsic value {intrinsic_value} for ITM call option",
                "PremiumValidator"
            )
    
    if option_type == 'PE' and underlying_price < strike_price:
        intrinsic_value = strike_price - underlying_price
        if premium < intrinsic_value * 0.95:  # Allow for some margin of error
            raise ValidationError(
                f"Premium {premium} is below intrinsic value {intrinsic_value} for ITM put option",
                "PremiumValidator"
            )
    
    # For deep OTM options, check if premium is unreasonably high
    if option_type == 'CE' and strike_price > underlying_price * 1.20:  # 20% OTM
        if premium > underlying_price * 0.05:  # More than 5% of underlying price
            print(f"Warning: Premium {premium} may be high for deep OTM call option")
    
    if option_type == 'PE' and strike_price < underlying_price * 0.80:  # 20% OTM
        if premium > underlying_price * 0.05:  # More than 5% of underlying price
            print(f"Warning: Premium {premium} may be high for deep OTM put option")
    
    return True
```

### Usage Examples

```python
# Validate option expiry
try:
    validate_option_expiry("NIFTY23JUN18000CE")
    print("Option expiry is valid")
except ValidationError as e:
    print(f"Option expiry validation failed: {e}")

# Validate option strike price
try:
    validate_option_strike_price("NIFTY23JUN18000CE", 17800.0)
    print("Option strike price is valid")
except ValidationError as e:
    print(f"Option strike price validation failed: {e}")

# Validate option premium
try:
    validate_option_premium("NIFTY23JUN18000CE", 150.0, 17800.0)
    print("Option premium is valid")
except ValidationError as e:
    print(f"Option premium validation failed: {e}")
```

## Comprehensive Validation Implementation

This section shows how to combine all the validation rules into a comprehensive validator for Indian market orders.

### Unified Order Validator

```python
class IndianMarketOrderValidator:
    """Comprehensive validator for Indian market orders."""
    
    def __init__(self):
        """Initialize the validator with market data."""
        self.current_market_data = {}  # {symbol: current_price}
    
    def update_market_data(self, symbol, price):
        """Update market data for a symbol."""
        self.current_market_data[symbol] = price
    
    def validate_order(self, order):
        """
        Validate a complete order for Indian markets.
        
        Args:
            order (Order): Order to validate
            
        Returns:
            bool: True if order is valid
            
        Raises:
            ValidationError: If order is invalid
        """
        # Extract order parameters
        symbol = order.symbol
        order_type = order.order_type
        product_type = order.product_type
        quantity = order.quantity
        price = order.price
        
        # Check if market is open (unless it's an AMO order)
        if order_type != OrderType.AMO:
            self._validate_market_open()
        
        # Validate symbol format
        self._validate_symbol(symbol)
        
        # Validate product type compatibility
        self._validate_product_type(symbol, product_type)
        
        # Validate quantity
        self._validate_quantity(symbol, quantity)
        
        # Validate price (for limit orders)
        if order_type in [OrderType.LIMIT, OrderType.SL_LIMIT] and price is not None:
            self._validate_price(symbol, price)
        
        # Validate order value
        if price is not None:
            self._validate_order_value(price, quantity)
        
        # Special validation for F&O
        if self._is_fno_symbol(symbol):
            self._validate_fno_order(symbol, order_type, product_type)
        
        # Special validation for bracket orders
        if order_type == OrderType.BRACKET:
            self._validate_bracket_order(order)
        
        # Special validation for cover orders
        if order_type == OrderType.COVER:
            self._validate_cover_order(order)
        
        # Special validation for AMO orders
        if order_type == OrderType.AMO:
            self._validate_amo_order(order)
        
        return True
    
    def _validate_market_open(self):
        """Validate that the market is open for trading."""
        if not is_market_open():
            raise ValidationError(
                "Market is closed. Trading hours: 9:15 AM - 3:30 PM IST, weekdays only",
                "MarketValidator"
            )
    
    def _validate_symbol(self, symbol):
        """Validate symbol format."""
        validate_symbol(symbol)
    
    def _validate_product_type(self, symbol, product_type):
        """Validate product type compatibility."""
        validate_product_type(symbol, product_type)
    
    def _validate_quantity(self, symbol, quantity):
        """Validate order quantity."""
        validate_order_quantity(quantity, symbol)
        validate_quantity_lot_size(symbol, quantity)
    
    def _validate_price(self, symbol, price):
        """Validate order price."""
        if symbol in self.current_market_data:
            reference_price = self.current_market_data[symbol]
            validate_price_circuit_limits(symbol, price, reference_price)
    
    def _validate_order_value(self, price, quantity):
        """Validate order value."""
        validate_order_value(price, quantity)
    
    def _is_fno_symbol(self, symbol):
        """Check if symbol is an F&O instrument."""
        return (re.match(NSE_FNO_PATTERN, symbol) is not None or
                re.match(NSE_FUTURES_PATTERN, symbol) is not None)
    
    def _validate_fno_order(self, symbol, order_type, product_type):
        """Validate F&O order specifics."""
        if re.match(NSE_FNO_PATTERN, symbol):
            # Validate option expiry
            validate_option_expiry(symbol)
            
            # Validate strike price if we have market data
            if symbol in self.current_market_data:
                components = parse_fno_symbol(symbol)
                underlying = components['underlying']
                
                if underlying in self.current_market_data:
                    underlying_price = self.current_market_data[underlying]
                    validate_option_strike_price(symbol, underlying_price)
    
    def _validate_bracket_order(self, order):
        """Validate bracket order specifics."""
        # Implementation depends on your Order class structure
        # This is a placeholder for the complete implementation
        if order.product_type != ProductType.MIS:
            raise ValidationError(
                "Bracket orders can only be placed with MIS product type",
                "BracketOrderValidator"
            )
    
    def _validate_cover_order(self, order):
        """Validate cover order specifics."""
        # Implementation depends on your Order class structure
        # This is a placeholder for the complete implementation
        if order.product_type != ProductType.MIS:
            raise ValidationError(
                "Cover orders can only be placed with MIS product type",
                "CoverOrderValidator"
            )
    
    def _validate_amo_order(self, order):
        """Validate AMO order specifics."""
        # Validate that market is closed for AMO orders
        if is_market_open():
            raise ValidationError(
                "AMO orders can only be placed when market is closed",
                "AMOValidator"
            )
```

### Usage Example

```python
# Create a validator instance
validator = IndianMarketOrderValidator()

# Update market data
validator.update_market_data("RELIANCE", 2500.0)
validator.update_market_data("NIFTY", 17800.0)

# Create an order (assume Order class implementation)
order = Order(
    symbol="RELIANCE",
    order_type=OrderType.LIMIT,
    product_type=ProductType.MIS,
    quantity=10,
    price=2480.0
)

# Validate the order
try:
    validator.validate_order(order)
    print("Order is valid and ready to submit")
except ValidationError as e:
    print(f"Order validation failed: {e}")
```

## See Also

- [Indian Market Validation - Part 1](indian_market_validation_part1.md) for Market Timing Validation, Symbol Format Validation, and Circuit Limit Implementation
- [Indian Market Validation - Part 2](indian_market_validation_part2.md) for Lot Size Validation, Product Type Validation, and Order Value Constraints