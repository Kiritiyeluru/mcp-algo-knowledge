# Indian Market Validation Guide - Part 1

This guide provides comprehensive validation rules, code examples, and implementation patterns specific to Indian markets (NSE/BSE) for the MCP Algorithmic Trading System.

## Table of Contents for Complete Guide

1. [Market Timing Validation](#market-timing-validation) (Part 1)
2. [Symbol Format Validation](#symbol-format-validation) (Part 1)
3. [Circuit Limit Implementation](#circuit-limit-implementation) (Part 1)
4. [Lot Size Validation](#lot-size-validation) (Part 2)
5. [Product Type Validation](#product-type-validation) (Part 2)
6. [Order Value Constraints](#order-value-constraints) (Part 2)
7. [Market-Specific Order Types](#market-specific-order-types) (Part 3)
8. [F&O-Specific Validation](#fo-specific-validation) (Part 3)
9. [Comprehensive Validation Implementation](#comprehensive-validation-implementation) (Part 3)

## Market Timing Validation

Indian markets operate with specific trading hours and sessions that must be validated to ensure orders are only placed during valid trading periods.

### Constants and Types

```python
from datetime import time, datetime, timedelta
from enum import Enum
from zoneinfo import ZoneInfo

# Indian Standard Time timezone
IST = ZoneInfo("Asia/Kolkata")

# Market schedule constants
class MarketSession(Enum):
    """Market session types."""
    PRE_OPEN = "pre_open"
    NORMAL = "normal"
    POST_CLOSE = "post_close"
    CLOSED = "closed"

# NSE/BSE market timings
MARKET_PRE_OPEN_START = time(9, 0)  # 9:00 AM
MARKET_PRE_OPEN_END = time(9, 8)    # 9:08 AM
MARKET_OPEN_TIME = time(9, 15)      # 9:15 AM
MARKET_CLOSE_TIME = time(15, 30)    # 3:30 PM
```

### Market Timing Validation Functions

```python
def is_market_open(check_time=None):
    """
    Check if the market is currently open for normal trading.
    
    Args:
        check_time (datetime, optional): Time to check, defaults to current time
        
    Returns:
        bool: True if market is open, False otherwise
    """
    if check_time is None:
        check_time = datetime.now(IST)
    
    # Check if weekend (5=Saturday, 6=Sunday)
    if check_time.weekday() >= 5:
        return False
    
    # Check time of day
    time_of_day = check_time.time()
    return MARKET_OPEN_TIME <= time_of_day < MARKET_CLOSE_TIME

def get_market_session(check_time=None):
    """
    Determine the current market session.
    
    Args:
        check_time (datetime, optional): Time to check, defaults to current time
        
    Returns:
        MarketSession: Current market session
    """
    if check_time is None:
        check_time = datetime.now(IST)
    
    # Check if weekend (5=Saturday, 6=Sunday)
    if check_time.weekday() >= 5:
        return MarketSession.CLOSED
    
    # Check time of day
    time_of_day = check_time.time()
    
    if MARKET_PRE_OPEN_START <= time_of_day < MARKET_PRE_OPEN_END:
        return MarketSession.PRE_OPEN
    elif MARKET_OPEN_TIME <= time_of_day < MARKET_CLOSE_TIME:
        return MarketSession.NORMAL
    else:
        return MarketSession.CLOSED

def validate_order_timing(order_type, product_type, check_time=None):
    """
    Validate if an order can be placed at the current time.
    
    Args:
        order_type (OrderType): Type of order
        product_type (str): Product type (MIS, CNC, NRML)
        check_time (datetime, optional): Time to check, defaults to current time
        
    Returns:
        bool: True if order timing is valid
        
    Raises:
        ValidationError: If order timing is invalid
    """
    if check_time is None:
        check_time = datetime.now(IST)
    
    session = get_market_session(check_time)
    
    # AMO orders can be placed after market hours
    if order_type == OrderType.AMO:
        # AMO orders are accepted between 3:45 PM to 9:00 AM next day
        if session == MarketSession.NORMAL:
            raise ValidationError(
                "AMO orders cannot be placed during market hours",
                "OrderValidator"
            )
        return True
    
    # Regular orders can only be placed during normal trading hours
    if session != MarketSession.NORMAL:
        if session == MarketSession.PRE_OPEN:
            message = "Orders cannot be placed during pre-open session (9:00-9:08 AM)"
        elif session == MarketSession.CLOSED:
            message = "Orders cannot be placed outside market hours (9:15 AM - 3:30 PM)"
        else:
            message = "Invalid market session for order placement"
            
        raise ValidationError(message, "OrderValidator")
    
    # MIS orders cannot be placed in the last 15 minutes
    if product_type == "MIS" and check_time.time() >= time(15, 15):
        raise ValidationError(
            "MIS orders cannot be placed in the last 15 minutes of trading",
            "OrderValidator"
        )
    
    return True
```

### Usage Examples

```python
# Check if market is currently open
if is_market_open():
    print("Market is open for trading")
else:
    print("Market is closed")

# Validate a regular order during market hours
try:
    validate_order_timing(OrderType.LIMIT, "CNC")
    print("Order timing is valid")
except ValidationError as e:
    print(f"Order timing is invalid: {e}")

# Validate an AMO order after market hours
after_market = datetime.now(IST).replace(hour=18, minute=0)
try:
    validate_order_timing(OrderType.AMO, "CNC", after_market)
    print("AMO order timing is valid")
except ValidationError as e:
    print(f"AMO order timing is invalid: {e}")
```

## Symbol Format Validation

Indian markets have specific symbol formats for equities, F&O, and indices that must be strictly validated.

### Symbol Pattern Constants

```python
import re

# Symbol patterns
NSE_EQUITY_PATTERN = r'^[A-Z]+$'  # Simple equity symbols like RELIANCE, INFY
BSE_EQUITY_PATTERN = r'^[0-9]{6}$'  # BSE symbol format (numeric code)

# F&O symbol pattern components
INDEX_NAMES = ['NIFTY', 'BANKNIFTY', 'FINNIFTY']
MONTH_CODES = ['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 
               'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC']
OPTION_TYPES = ['CE', 'PE']

# Complete F&O pattern for NSE
# Format: SYMBOL + YY + MONTH + STRIKE + OPTIONTYPE
# Example: NIFTY23JUN18000CE, RELIANCE23JUN2800PE
NSE_FNO_PATTERN = r'^([A-Z]+)(\d{2})(' + '|'.join(MONTH_CODES) + r')(\d+)(' + '|'.join(OPTION_TYPES) + r')$'

# Futures pattern
# Example: NIFTY23JUNFUT
NSE_FUTURES_PATTERN = r'^([A-Z]+)(\d{2})(' + '|'.join(MONTH_CODES) + r')FUT$'

# Full exchange-prefixed symbol pattern
# Example: NSE:RELIANCE, BSE:500325
EXCHANGE_SYMBOL_PATTERN = r'^(NSE|BSE):(.+)$'
```

### Symbol Validation Functions

```python
def is_valid_symbol(symbol):
    """
    Validate if the symbol follows any of the valid Indian market formats.
    
    Args:
        symbol (str): Trading symbol to validate
        
    Returns:
        bool: True if symbol is valid
    """
    # Check for exchange prefix
    exchange_match = re.match(EXCHANGE_SYMBOL_PATTERN, symbol)
    if exchange_match:
        exchange, base_symbol = exchange_match.groups()
        if exchange == 'NSE':
            return (re.match(NSE_EQUITY_PATTERN, base_symbol) is not None or
                    re.match(NSE_FNO_PATTERN, base_symbol) is not None or
                    re.match(NSE_FUTURES_PATTERN, base_symbol) is not None)
        elif exchange == 'BSE':
            return re.match(BSE_EQUITY_PATTERN, base_symbol) is not None
        return False
    
    # Without exchange prefix
    return (re.match(NSE_EQUITY_PATTERN, symbol) is not None or
            re.match(BSE_EQUITY_PATTERN, symbol) is not None or
            re.match(NSE_FNO_PATTERN, symbol) is not None or
            re.match(NSE_FUTURES_PATTERN, symbol) is not None)

def parse_fno_symbol(symbol):
    """
    Parse an F&O symbol into its components.
    
    Args:
        symbol (str): F&O symbol to parse
        
    Returns:
        dict: Components of the F&O symbol
        
    Raises:
        ValidationError: If symbol is not a valid F&O symbol
    """
    # Remove exchange prefix if present
    base_symbol = symbol
    exchange_match = re.match(EXCHANGE_SYMBOL_PATTERN, symbol)
    if exchange_match:
        exchange, base_symbol = exchange_match.groups()
    
    # Parse options symbol
    option_match = re.match(NSE_FNO_PATTERN, base_symbol)
    if option_match:
        underlying, year, month, strike, option_type = option_match.groups()
        return {
            'type': 'option',
            'underlying': underlying,
            'expiry_year': f'20{year}',  # Convert to full year
            'expiry_month': month,
            'strike_price': float(strike),
            'option_type': option_type  # CE or PE
        }
    
    # Parse futures symbol
    futures_match = re.match(NSE_FUTURES_PATTERN, base_symbol)
    if futures_match:
        underlying, year, month = futures_match.groups()
        return {
            'type': 'future',
            'underlying': underlying,
            'expiry_year': f'20{year}',  # Convert to full year
            'expiry_month': month,
        }
    
    raise ValidationError(
        f"Invalid F&O symbol format: {symbol}",
        "SymbolValidator"
    )

def validate_symbol(symbol, allowed_types=None):
    """
    Validate a trading symbol and check if it's allowed for trading.
    
    Args:
        symbol (str): Trading symbol
        allowed_types (list, optional): List of allowed instrument types
        
    Returns:
        bool: True if symbol is valid and allowed
        
    Raises:
        ValidationError: If symbol is invalid or not allowed
    """
    if not symbol:
        raise ValidationError("Symbol cannot be empty", "SymbolValidator")
    
    if not is_valid_symbol(symbol):
        raise ValidationError(
            f"Invalid symbol format: {symbol}",
            "SymbolValidator"
        )
    
    # Check allowed types if specified
    if allowed_types:
        # Determine symbol type
        symbol_type = None
        if re.match(NSE_FNO_PATTERN, symbol):
            symbol_type = 'option'
        elif re.match(NSE_FUTURES_PATTERN, symbol):
            symbol_type = 'future'
        elif re.match(NSE_EQUITY_PATTERN, symbol) or re.match(BSE_EQUITY_PATTERN, symbol):
            symbol_type = 'equity'
        
        if symbol_type not in allowed_types:
            raise ValidationError(
                f"Symbol type '{symbol_type}' not allowed. Allowed types: {allowed_types}",
                "SymbolValidator"
            )
    
    return True
```

### Usage Examples

```python
# Validate equity symbol
try:
    validate_symbol("RELIANCE")
    print("Valid equity symbol")
except ValidationError as e:
    print(f"Invalid symbol: {e}")

# Validate F&O symbol
try:
    validate_symbol("NIFTY23JUN18000CE")
    print("Valid F&O symbol")
except ValidationError as e:
    print(f"Invalid symbol: {e}")

# Parse F&O symbol
try:
    components = parse_fno_symbol("NIFTY23JUN18000CE")
    print(f"Underlying: {components['underlying']}")
    print(f"Expiry: {components['expiry_month']} {components['expiry_year']}")
    print(f"Strike: {components['strike_price']}")
    print(f"Option Type: {components['option_type']}")
except ValidationError as e:
    print(f"Parsing failed: {e}")
```

## Circuit Limit Implementation

Indian markets implement circuit limits (price bands) to control excessive volatility. These limits must be checked when validating order prices.

### Circuit Limit Constants

```python
# Default circuit limits (percentage)
DEFAULT_CIRCUIT_LIMIT = 0.20  # 20% for most stocks

# Special circuit limits for specific indices/stocks
SPECIAL_CIRCUIT_LIMITS = {
    "NIFTY": 0.10,      # 10% for NIFTY index options
    "BANKNIFTY": 0.10,  # 10% for Bank NIFTY options
    "FINNIFTY": 0.10,   # 10% for Fin NIFTY options
    "SENSEX": 0.10,     # 10% for SENSEX
}

# Circuit freezes
# When prices hit circuit limits, trading may be halted temporarily
CIRCUIT_FREEZE_DURATION = 15  # 15 minutes freeze
```

### Circuit Limit Functions

```python
def calculate_circuit_limits(symbol, reference_price):
    """
    Calculate upper and lower circuit limits for a symbol.
    
    Args:
        symbol (str): Trading symbol
        reference_price (float): Reference price (usually previous close)
        
    Returns:
        tuple: (lower_limit, upper_limit)
    """
    # Determine appropriate circuit limit
    base_symbol = symbol
    
    # Extract underlying for F&O symbols
    if re.match(NSE_FNO_PATTERN, symbol) or re.match(NSE_FUTURES_PATTERN, symbol):
        match = re.match(r'^([A-Z]+)', symbol)
        if match:
            base_symbol = match.group(1)
    
    # Get percentage circuit limit
    percentage = SPECIAL_CIRCUIT_LIMITS.get(base_symbol, DEFAULT_CIRCUIT_LIMIT)
    
    lower_limit = round(reference_price * (1 - percentage), 2)
    upper_limit = round(reference_price * (1 + percentage), 2)
    
    return lower_limit, upper_limit

def validate_price_circuit_limits(symbol, price, reference_price):
    """
    Validate if a price is within circuit limits.
    
    Args:
        symbol (str): Trading symbol
        price (float): Price to validate
        reference_price (float): Reference price
        
    Returns:
        bool: True if price is within circuit limits
        
    Raises:
        ValidationError: If price is outside circuit limits
    """
    lower_limit, upper_limit = calculate_circuit_limits(symbol, reference_price)
    
    if price < lower_limit:
        raise ValidationError(
            f"Price {price} below lower circuit limit {lower_limit}",
            "PriceValidator"
        )
    
    if price > upper_limit:
        raise ValidationError(
            f"Price {price} above upper circuit limit {upper_limit}",
            "PriceValidator"
        )
    
    return True
```

### Usage Examples

```python
# Calculate circuit limits for stock
lower, upper = calculate_circuit_limits("RELIANCE", 2500.0)
print(f"Circuit limits for RELIANCE at reference price 2500: {lower} - {upper}")

# Validate price within circuit limits
try:
    validate_price_circuit_limits("RELIANCE", 2600.0, 2500.0)
    print("Price is within circuit limits")
except ValidationError as e:
    print(f"Price validation failed: {e}")

# Validate price for index derivatives
try:
    validate_price_circuit_limits("NIFTY23JUN18000CE", 200.0, 180.0)
    print("Option price is within circuit limits")
except ValidationError as e:
    print(f"Option price validation failed: {e}")
```

## See Also

- [Indian Market Validation - Part 2](indian_market_validation_part2.md) for Lot Size Validation, Product Type Validation, and Order Value Constraints
- [Indian Market Validation - Part 3](indian_market_validation_part3.md) for Market-Specific Order Types, F&O-Specific Validation, and Comprehensive Validation Implementation