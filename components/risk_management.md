# Risk Management Knowledge Map

This document outlines the risk management component for the MCP Algorithmic Trading System, covering risk limits, exposure controls, and margin handling.

## 1️⃣ Interfaces

### RiskManagerBase

- **File Path**: `/src/interfaces/risk_management/base.py`
- **Core Methods**:
  - `check_risk(order: Order) -> bool`: Evaluates order against risk limits
  - `update_exposure(trade: Trade) -> None`: Adjusts exposure based on new trade
  - `evaluate_margin(account: Account) -> bool`: Checks if account meets margin requirements
  - `get_current_risk_metrics() -> RiskMetrics`: Retrieves real-time risk metrics
  - `set_risk_limits(new_limits: RiskLimit) -> None`: Updates risk parameters dynamically

## 2️⃣ Models

### RiskLimit
- **File Path**: `/src/interfaces/risk_management/models.py`
- **Properties**:
  - `max_exposure: float`: Maximum allowed total exposure (% of portfolio)
  - `max_position_size: int`: Maximum allowed position size
  - `margin_requirement: float`: Minimum required margin percentage
  - `leverage_limit: float`: Maximum allowable leverage

### RiskMetrics
- **File Path**: `/src/interfaces/risk_management/models.py`
- **Properties**:
  - `total_exposure: float`: Current aggregated exposure
  - `current_margin: float`: Available margin after positions
  - `current_leverage: float`: Calculated leverage ratio
  - `risk_status: RiskStatus`: Risk state (SAFE, WARNING, CRITICAL)

## 3️⃣ Validation Rules

- **Max Exposure**: Total exposure must not exceed 20% of portfolio value
- **Position Size**: No position should exceed the maximum allowed size
- **Margin Requirements**: All trades must meet minimum margin requirements
- **Leverage Limits**: 10x max for equities, 5x max for F&O
- **Stop Loss Requirement**: Each position must have a stop loss defined
- **Real-Time Updates**: Risk metrics must update after every trade

## 4️⃣ Test Patterns

### Test Classes
- `TestRiskManager`: Overall risk management functionality
- `TestMarginHandling`: Margin evaluation and enforcement
- `TestExposureControl`: Exposure tracking and limiting

### Standard Test Methods
- `test_risk_limit_enforcement()`: Verify orders exceeding limits are rejected
- `test_margin_requirement_validation()`: Check margin evaluation logic
- `test_exposure_calculation()`: Ensure accurate exposure updates
- `test_dynamic_risk_update()`: Verify risk limit changes apply immediately
- `test_stop_loss_presence()`: Validate stop loss enforcement

## 5️⃣ Common Fixtures

### `sample_risk_limit()`
- **Returns**: RiskLimit object for testing
- **Parameters**:
  - `max_exposure: float = 0.2`: 20% of portfolio value
  - `max_position_size: int = 1000`: Maximum position size
  - `margin_requirement: float = 0.5`: 50% margin requirement
  - `leverage_limit: float = 10.0`: 10x leverage limit

### `mock_risk_manager()`
- **Returns**: Mock implementation of RiskManagerBase

## 6️⃣ Market-Specific Features

- **Regulatory Compliance**: Alignment with Indian market regulations
- **Circuit Breakers**: Account for market-imposed circuit breakers
- **Dynamic Margin**: Real-time margin adjustment based on volatility
- **Stop Loss Requirements**: Mandatory stop loss for all positions
- **Exposure Calculation**: Special handling for F&O with higher risk weightage

## 7️⃣ Implementation Priorities

1. Core risk limit validation logic
2. Exposure tracking and calculation
3. Margin requirement enforcement
4. Stop loss validation
5. Integration with order management to block risky trades
