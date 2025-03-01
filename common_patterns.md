# Common Implementation Patterns

This document consolidates patterns, constants, and structures shared across multiple components to avoid redundancy in individual knowledge maps.

## 1. Constants & Boundaries

### Market Timing
```python
# Market schedule constants (Indian market - NSE/BSE)
MARKET_OPEN_TIME = time(9, 15)     # 9:15 AM
MARKET_CLOSE_TIME = time(15, 30)   # 3:30 PM
MARKET_PRE_OPEN_START = time(9, 0) # 9:00 AM
MARKET_PRE_OPEN_END = time(9, 8)   # 9:08 AM
```

### Symbol Patterns
```python
# Symbol pattern for NSE/BSE equities and F&O
NSE_EQUITY_PATTERN = r'^[A-Z]+$'  # Simple equity symbols like RELIANCE, INFY
NSE_FNO_PATTERN = r'^[A-Z]+\d{2}[A-Z]{3}[A-Z0-9]+$'  # F&O symbols like NIFTY22MARFUT
```

### Order Constraints
```python
MAX_ORDER_VALUE = Decimal('10000000.00')  # ₹1 crore max order value
MIN_ORDER_VALUE = Decimal('100.00')        # ₹100 min order value
MAX_ORDER_QUANTITY = 10000                # Maximum quantity per order
MIN_ORDER_QUANTITY = 1                    # Minimum quantity per order
```

## 2. Exception Hierarchy & Error Handling

```
- MCPError
  |- ValidationError
  |- StateError  
  |- ConfigurationError
  |- ProcessingError
     |- MarketDataError
     |- StrategyError
     |- OrderError
     |- PositionError
     |- RiskError
```

### Standard Error Pattern

```python
class ValidationError(MCPError):
    def __init__(self, message: str, component: str, invalid_data: Any = None, 
                 context: Dict[str, Any] = None):
        self.component = component
        self.invalid_data = invalid_data
        self.context = context or {}
        super().__init__(f"{component}: {message}")
```

## 3. Test Patterns

### Test File Structure
Each component test directory follows this standard structure:
```
component_name/
├── __init__.py - Package definition
├── conftest.py - Component-specific fixtures
├── test_models.py - Model initialization and methods tests
├── test_validation.py - Validation logic tests
├── test_interface.py - Interface implementation tests
```

### Test Class Naming Conventions
- Test classes follow the pattern: `Test<ModelName>` (e.g., `TestSignal`, `TestStrategyParameters`)
- Test methods follow the pattern: `test_<functionality>` (e.g., `test_initialization`, `test_to_dict`)
- Test methods for invalid cases: `test_invalid_<case_description>` (e.g., `test_invalid_signal_empty_symbol`)

### Fixture Patterns
- For each model, create a sample fixture: `sample_<model_name>` (e.g., `sample_signal`, `sample_strategy_parameters`)
- Component instances use naming pattern: `mock_<component_name>` (e.g., `mock_strategy`, `mock_validator`)
- Batch/list fixtures use plural naming: `sample_signals`, `sample_trades`

## 4. Common Test Data

### Symbols
- **Equities**: `"RELIANCE"`, `"INFY"`, `"HDFCBANK"`, `"TCS"`, `"ICICIBANK"`
- **Indices**: `"NIFTY"`, `"BANKNIFTY"`, `"FINNIFTY"`
- **F&O**: `"RELIANCE25FEB23CE"`, `"NIFTY25FEB18000PE"`, `"NIFTY22MARFUT"`

### Price Points
- `"RELIANCE"`: 2500.0
- `"INFY"`: 1800.0
- `"TCS"`: 3000.0
- `"NIFTY"`: 18000.0
- `"BANKNIFTY"`: 40000.0

### Product Types
- `"MIS"`: Intraday (Margin Intraday Square-off)
- `"CNC"`: Delivery (Cash and Carry)
- `"NRML"`: Normal for F&O

## 5. Mock Implementation Pattern

```python
class MockComponentValidator:
    """Mock implementation of ComponentValidator for testing."""
    
    def __init__(self):
        """Initialize with call tracking."""
        self.validate_calls = []
    
    def validate_data(self, data: Any) -> bool:
        """Mock validation with call recording."""
        self.validate_calls.append(data)
        
        # Basic validation logic
        if data is None:
            raise ValidationError("Data cannot be None", "MockValidator")
            
        return True
```

## 6. State Transition Patterns

### Order Status Transitions
```python
VALID_STATUS_TRANSITIONS = {
    OrderStatus.PENDING: {OrderStatus.ACTIVE, OrderStatus.REJECTED, OrderStatus.CANCELLED},
    OrderStatus.ACTIVE: {OrderStatus.FILLED, OrderStatus.PARTIALLY_FILLED, OrderStatus.CANCELLED, OrderStatus.EXPIRED},
    OrderStatus.PARTIALLY_FILLED: {OrderStatus.FILLED, OrderStatus.CANCELLED, OrderStatus.EXPIRED},
    OrderStatus.FILLED: set(),  # Terminal state
    OrderStatus.CANCELLED: set(),  # Terminal state
    OrderStatus.REJECTED: set(),  # Terminal state
    OrderStatus.EXPIRED: set(),  # Terminal state
}
```

## 7. Standard Modules to Import

```python
# Standard libraries
import pytest
import logging
from datetime import datetime, time, timedelta
from decimal import Decimal
from typing import List, Dict, Optional, Any, Tuple, Union, Protocol
from enum import Enum, auto

# Project imports
from src.interfaces.exceptions import MCPError, ValidationError
from src.interfaces.validation import BaseValidator
```
