# Frequently Asked Questions (FAQ)

This document answers common questions about implementing the MCP Algorithmic Trading System to avoid repeating explanations across multiple sessions.

## General Implementation Questions

### How should I structure my implementation requests?

Always structure your implementation requests to be file-specific and reference existing knowledge rather than repeating it:

```
Please implement /path/to/file.py
Reference knowledge map: /components/component_name.md
Focus on: [specific aspects]
```

See the [Token Efficiency Guide](token_efficiency_guide.md) for more detailed templates.

### What's the difference between knowledge maps and implementation guides?

- **Knowledge Maps**: Concise reference documents that outline interfaces, models, validation rules, and test patterns
- **Implementation Guides**: Detailed documents with concrete code examples for specific implementation patterns

Knowledge maps tell you "what" to implement, while implementation guides show you "how" to implement it.

### How should I handle Indian market-specific requirements?

All Indian market-specific validation (symbol formats, timing, circuit limits, lot sizes) should be implemented following the patterns in the [Indian Market Validation Guide](implementation_guides/indian_market_validation_part1.md). This ensures consistent handling across all components.

## Technical Implementation Questions

### How should timezone handling be implemented?

Always use the Indian Standard Time (IST) timezone for all datetime operations:

```python
from datetime import datetime
from zoneinfo import ZoneInfo

IST = ZoneInfo("Asia/Kolkata")
current_time = datetime.now(IST)
```

### What's the proper way to implement validation?

Follow these guidelines for validation:
1. Always validate inputs at the method/function level
2. Raise `ValidationError` with component name and clear message
3. Use dedicated validator classes for complex validation logic
4. Add Indian market-specific validation where applicable

### How should I handle F&O symbol parsing?

F&O symbols follow a specific format (e.g., `NIFTY23JUN18000CE`). Use the parsing functions from the [Indian Market Validation Guide](implementation_guides/indian_market_validation_part1.md#symbol-format-validation) to extract:
- Underlying symbol
- Expiry date
- Strike price
- Option type

### How should error handling be implemented?

Follow this pattern:
1. Define exception hierarchies in `exceptions.py`
2. Use specific exception types for different error categories
3. Include component name, error message, and context in exceptions
4. Document all possible exceptions in method docstrings

## Testing Questions

### What's the recommended approach for mocking dependencies?

Use the patterns described in the [Mock Implementation Patterns Guide](implementation_guides/mock_implementation_patterns.md). In general:
1. Create mock classes that inherit from the base interfaces
2. Track method calls for verification
3. Implement basic validation and business logic
4. Make mocks configurable for different test scenarios

### How should test fixtures be organized?

1. Common fixtures go in `conftest.py`
2. Component-specific fixtures go in the component's test directory
3. Create sample instances of all models being tested
4. Provide configurable fixtures for different test scenarios

See the [Position Tracking Fixtures Guide](implementation_guides/position_tracking_fixtures.md) for examples.

### What test coverage is expected?

Aim for comprehensive coverage:
1. Test all public methods and classes
2. Include both positive and negative test cases
3. Test edge cases and boundary conditions
4. Test Indian market-specific validation
5. Verify exception handling

### How should I test Indian market-specific functionality?

Create specific test cases for:
1. Symbol format validation (NSE/BSE symbols, F&O symbols)
2. Lot size validation for F&O instruments
3. Circuit limit validation
4. Trading hour restrictions
5. Product type compatibility

## Component-Specific Questions

### How should order management handle bracket orders?

Bracket orders should:
1. Be created with entry, stop-loss, and target parameters
2. Only be allowed with MIS product type
3. Validate price levels (entry > stop-loss, target > entry for buy orders)
4. Track parent-child relationships between orders

### How should position tracking handle F&O instruments?

F&O position tracking should:
1. Validate lot sizes according to standard lot sizes
2. Track expiry dates based on symbol parsing
3. Calculate P&L considering contract multipliers
4. Properly account for premium values for options

### How should risk management handle margin requirements?

Risk management should:
1. Calculate required margin based on SPAN+Exposure formula (simplified)
2. Track margin utilization across all positions
3. Validate new orders against available margin
4. Implement different margin requirements for different product types

### How should market data handle tick data vs. OHLC data?

Market data should:
1. Use different model classes for different data types
2. Implement type-specific validation
3. Handle subscription management separately for each data type
4. Process and store historical data appropriately for each type

## Project Structure Questions

### Where should validation logic be placed?

1. Basic model validation: Inside model classes
2. Cross-field validation: In dedicated validator classes
3. Business rule validation: In service/manager classes
4. Common validation utilities: In `validation.py`
5. Indian market-specific validation: Follow patterns in the Indian Market Validation Guide

### How should interfaces be organized?

1. Define base interfaces in `interfaces/<component>/base.py`
2. Define model classes in `interfaces/<component>/models.py`
3. Define exceptions in `interfaces/exceptions.py`
4. Define common validation in `interfaces/validation.py`

### Where should mock implementations be placed?

1. For tests: In `tests/<component>/conftest.py` or dedicated mock files
2. For examples: In `examples/<component>/`
3. For documentation: In implementation guides

## Integration Questions

### How do components communicate with each other?

Components communicate through well-defined interfaces:
1. Market Data → Strategy: Provides market updates
2. Strategy → Order Management: Generates order signals
3. Order Management → Position Tracking: Updates positions
4. Position Tracking → Risk Management: Provides exposure data

### How should broker integration be implemented?

Broker integration should:
1. Implement the broker-specific interfaces in `interfaces/integration/broker_integration/`
2. Handle authentication, order submission, and market data retrieval
3. Transform broker-specific responses to internal models
4. Implement proper error handling for API failures

### How should websocket connections be managed?

WebSocket connections should:
1. Implement the base interfaces in `interfaces/websocket/`
2. Handle reconnection logic with exponential backoff
3. Process messages according to their type
4. Maintain connection state and health checks

## Implementation Process Questions

### What's the recommended order for implementing components?

Follow this general order:
1. Start with Market Data (foundation for other components)
2. Implement Strategy components next
3. Implement Order Management
4. Implement Position Tracking
5. Implement Risk Management
6. Implement Integration components

### How should I handle incrementally developing a component?

1. Start with interfaces and models
2. Implement basic validation
3. Add core business logic
4. Implement tests for what's completed
5. Add advanced features and edge cases
6. Complete comprehensive tests

For more details, see the [Component Implementation Checklist](component_implementation_checklist.md).
