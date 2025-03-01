# Indian Market Validation Guide - Part 2

This guide provides comprehensive validation rules, code examples, and implementation patterns specific to Indian markets (NSE/BSE) for the MCP Algorithmic Trading System.

## Table of Contents for Complete Guide

1. [Market Timing Validation](indian_market_validation_part1.md#market-timing-validation) (Part 1)
2. [Symbol Format Validation](indian_market_validation_part1.md#symbol-format-validation) (Part 1)
3. [Circuit Limit Implementation](indian_market_validation_part1.md#circuit-limit-implementation) (Part 1)
4. [Lot Size Validation](#lot-size-validation) (Part 2)
5. [Product Type Validation](#product-type-validation) (Part 2)
6. [Order Value Constraints](#order-value-constraints) (Part 2)
7. [Market-Specific Order Types](indian_market_validation_part3.md#market-specific-order-types) (Part 3)
8. [F&O-Specific Validation](indian_market_validation_part3.md#fo-specific-validation) (Part 3)
9. [Comprehensive Validation Implementation](indian_market_validation_part3.md#comprehensive-validation-implementation) (Part 3)

## Lot Size Validation

F&O instruments in Indian markets trade in fixed lot sizes, which must be validated for order quantities.

### Lot Size Constants

```python
# Standard lot sizes for major indices and stocks
LOT_SIZES = {
    "NIFTY": 50,
    "BANKNIFTY": 25,
    "FINNIFTY": 40,
    "RELIANCE": 250,
    "TCS": 150,
    "INFY": 300,
    "HDFCBANK": 550,
    "SBIN": 1500,
}

# Default lot size for stocks not in the above list
DEFAULT_LOT_SIZE = 1  # For equities
```

### Lot Size Validation Functions

```python
def get_lot_size(symbol):
    """
    Get the standard lot size for a symbol.
    
    Args:
        symbol (str): Trading symbol
        
    Returns:
        int: Lot size
    """
    # For F&O, extract the underlying
    base_symbol = symbol
    if re.match(NSE_FNO_PATTERN, symbol) or re.match(NSE_FUTURES_PATTERN, symbol):
        match = re.match(r'^([A-Z]+)', symbol)
        if match:
            base_symbol = match.group(1)
    
    return LOT_SIZES.get(base_symbol, DEFAULT_LOT_SIZE)

def validate_quantity_lot_size(symbol, quantity):
    """
    Validate if order quantity is a valid multiple of the lot size.
    
    Args:
        symbol (str): Trading symbol
        quantity (int): Order quantity
        
    Returns:
        bool: True if quantity is valid
        
    Raises:
        ValidationError: If quantity is not a valid multiple of lot size
    """
    if not isinstance(quantity, int) or quantity <= 0:
        raise ValidationError("Quantity must be a positive integer", "QuantityValidator")
    
    lot_size = get_lot_size(symbol)
    
    # For F&O, quantity must be a multiple of lot size
    if re.match(NSE_FNO_PATTERN, symbol) or re.match(NSE_FUTURES_PATTERN, symbol):
        if quantity % lot_size != 0:
            raise ValidationError(
                f"F&O quantity must be a multiple of lot size ({lot_size})",
                "QuantityValidator"
            )
    
    return True
```

### Usage Examples

```python
# Get lot size for index
nifty_lot = get_lot_size("NIFTY")
print(f"NIFTY lot size: {nifty_lot}")

# Get lot size for F&O
option_lot = get_lot_size("NIFTY23JUN18000CE")
print(f"NIFTY option lot size: {option_lot}")

# Validate quantity for F&O
try:
    validate_quantity_lot_size("NIFTY23JUN18000CE", 100)
    print("Quantity is a valid multiple of lot size")
except ValidationError as e:
    print(f"Quantity validation failed: {e}")

# Validate quantity for equity
try:
    validate_quantity_lot_size("RELIANCE", 10)
    print("Quantity is valid for equity")
except ValidationError as e:
    print(f"Equity quantity validation failed: {e}")
```

## Product Type Validation

Indian brokers support specific product types (MIS, CNC, NRML) that have compatibility rules with different instrument types.

### Product Type Constants

```python
from enum import Enum

class ProductType(Enum):
    """Product types for Indian markets."""
    CNC = "CNC"    # Cash and Carry (Delivery)
    MIS = "MIS"    # Margin Intraday Square-off
    NRML = "NRML"  # Normal (for F&O)

# Compatibility matrix for product types
PRODUCT_TYPE_COMPATIBILITY = {
    "equity": [ProductType.CNC, ProductType.MIS],
    "future": [ProductType.NRML, ProductType.MIS],
    "option": [ProductType.NRML, ProductType.MIS],
}
```

### Product Type Validation Functions

```python
def get_instrument_type(symbol):
    """
    Determine the instrument type from a symbol.
    
    Args:
        symbol (str): Trading symbol
        
    Returns:
        str: Instrument type ('equity', 'future', or 'option')
    """
    if re.match(NSE_FNO_PATTERN, symbol):
        return "option"
    elif re.match(NSE_FUTURES_PATTERN, symbol):
        return "future"
    else:
        return "equity"

def validate_product_type(symbol, product_type):
    """
    Validate if a product type is compatible with a symbol.
    
    Args:
        symbol (str): Trading symbol
        product_type (str): Product type to validate
        
    Returns:
        bool: True if product type is valid for the symbol
        
    Raises:
        ValidationError: If product type is invalid for the symbol
    """
    try:
        # Convert string to enum if needed
        if isinstance(product_type, str):
            product_type = ProductType(product_type)
    except ValueError:
        raise ValidationError(
            f"Invalid product type: {product_type}",
            "ProductTypeValidator"
        )
    
    instrument_type = get_instrument_type(symbol)
    allowed_product_types = PRODUCT_TYPE_COMPATIBILITY.get(instrument_type, [])
    
    if product_type not in allowed_product_types:
        raise ValidationError(
            f"Product type {product_type.value} not compatible with {instrument_type} instruments",
            "ProductTypeValidator"
        )
    
    return True

def get_product_type_margin_requirements(product_type):
    """
    Get margin requirements for a specific product type.
    
    Args:
        product_type (ProductType): Product type
        
    Returns:
        float: Margin requirement as a percentage of order value
    """
    margin_requirements = {
        ProductType.CNC: 0.20,  # 20% for delivery
        ProductType.MIS: 0.40,  # 40% for intraday
        ProductType.NRML: 0.60,  # 60% for futures and options
    }
    
    return margin_requirements.get(product_type, 1.0)  # Default to 100% if unknown
```

### Usage Examples

```python
# Check product type compatibility for equity
try:
    validate_product_type("RELIANCE", ProductType.CNC)
    print("CNC is valid for RELIANCE")
except ValidationError as e:
    print(f"Product type validation failed: {e}")

# Check product type compatibility for F&O
try:
    validate_product_type("NIFTY23JUN18000CE", ProductType.NRML)
    print("NRML is valid for NIFTY options")
except ValidationError as e:
    print(f"Product type validation failed: {e}")

# Check invalid combination
try:
    validate_product_type("NIFTY23JUN18000CE", ProductType.CNC)
    print("CNC is valid for options")
except ValidationError as e:
    print(f"Product type validation failed: {e}")

# Get margin requirements for different product types
for product_type in ProductType:
    margin = get_product_type_margin_requirements(product_type)
    print(f"Margin requirement for {product_type.value}: {margin * 100}%")
```

## Order Value Constraints

Indian markets have minimum and maximum order value constraints to ensure market stability and risk management.

### Order Value Constants

```python
from decimal import Decimal

# Order value constraints
MIN_ORDER_VALUE = Decimal('100.00')     # ₹100 min order value
MAX_ORDER_VALUE = Decimal('10000000.00')  # ₹1 crore max order value

# Order quantity constraints
MIN_ORDER_QUANTITY = 1       # Minimum quantity
MAX_ORDER_QUANTITY = 10000   # Maximum quantity per order
```

### Order Value Validation Functions

```python
def validate_order_value(price, quantity):
    """
    Validate if an order's total value is within allowed limits.
    
    Args:
        price (float): Order price
        quantity (int): Order quantity
        
    Returns:
        bool: True if order value is valid
        
    Raises:
        ValidationError: If order value is outside allowed limits
    """
    if price <= 0:
        raise ValidationError("Price must be positive", "OrderValueValidator")
    
    if not isinstance(quantity, int) or quantity <= 0:
        raise ValidationError("Quantity must be a positive integer", "OrderValueValidator")
    
    order_value = Decimal(str(price)) * Decimal(str(quantity))
    
    if order_value < MIN_ORDER_VALUE:
        raise ValidationError(
            f"Order value (₹{order_value}) below minimum (₹{MIN_ORDER_VALUE})",
            "OrderValueValidator"
        )
    
    if order_value > MAX_ORDER_VALUE:
        raise ValidationError(
            f"Order value (₹{order_value}) above maximum (₹{MAX_ORDER_VALUE})",
            "OrderValueValidator"
        )
    
    return True

def validate_order_quantity(quantity, symbol=None):
    """
    Validate if an order quantity is within allowed limits.
    
    Args:
        quantity (int): Order quantity
        symbol (str, optional): Trading symbol for lot size validation
        
    Returns:
        bool: True if quantity is valid
        
    Raises:
        ValidationError: If quantity is outside allowed limits
    """
    if not isinstance(quantity, int) or quantity <= 0:
        raise ValidationError("Quantity must be a positive integer", "QuantityValidator")
    
    if quantity < MIN_ORDER_QUANTITY:
        raise ValidationError(
            f"Quantity ({quantity}) below minimum ({MIN_ORDER_QUANTITY})",
            "QuantityValidator"
        )
    
    if quantity > MAX_ORDER_QUANTITY:
        raise ValidationError(
            f"Quantity ({quantity}) above maximum ({MAX_ORDER_QUANTITY})",
            "QuantityValidator"
        )
    
    # Validate lot size if symbol is provided
    if symbol:
        validate_quantity_lot_size(symbol, quantity)
    
    return True

def calculate_order_margin(symbol, price, quantity, product_type):
    """
    Calculate required margin for an order.
    
    Args:
        symbol (str): Trading symbol
        price (float): Order price
        quantity (int): Order quantity
        product_type (ProductType): Product type
        
    Returns:
        float: Required margin amount
    """
    # Get instrument type
    instrument_type = get_instrument_type(symbol)
    
    # Base order value
    order_value = price * quantity
    
    # Get margin percentage based on product type
    margin_percentage = get_product_type_margin_requirements(product_type)
    
    # Additional margin for F&O
    if instrument_type in ["future", "option"]:
        # SPAN + Exposure margin for F&O (simplified calculation)
        return order_value * margin_percentage
    else:
        # VAR + ELM margin for equity (simplified calculation)
        return order_value * margin_percentage
```

### Usage Examples

```python
# Validate order value
try:
    validate_order_value(100.0, 5)
    print("Order value is valid")
except ValidationError as e:
    print(f"Order value validation failed: {e}")

# Validate order value that's too small
try:
    validate_order_value(10.0, 5)
    print("Order value is valid")
except ValidationError as e:
    print(f"Order value validation failed: {e}")

# Validate order quantity
try:
    validate_order_quantity(100, "NIFTY23JUN18000CE")
    print("Order quantity is valid")
except ValidationError as e:
    print(f"Order quantity validation failed: {e}")

# Calculate margin for equity delivery order
margin = calculate_order_margin("RELIANCE", 2500.0, 10, ProductType.CNC)
print(f"Margin required for RELIANCE delivery order: ₹{margin}")

# Calculate margin for futures order
margin = calculate_order_margin("NIFTY23JUNFUT", 18000.0, 50, ProductType.NRML)
print(f"Margin required for NIFTY futures order: ₹{margin}")
```

## See Also

- [Indian Market Validation - Part 1](indian_market_validation_part1.md) for Market Timing Validation, Symbol Format Validation, and Circuit Limit Implementation
- [Indian Market Validation - Part 3](indian_market_validation_part3.md) for Market-Specific Order Types, F&O-Specific Validation, and Comprehensive Validation Implementation