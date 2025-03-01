# Example: Position Tracking Test Implementation

This example demonstrates how to implement a test file for the Position Tracking component using the streamlined knowledge structure to minimize token usage while ensuring complete implementation.

## Implementation Request

Here's the implementation request that would be used to create this file:

```
Please implement /tests/interfaces/position_tracking/test_models.py

Reference:
- Knowledge map: /components/position_tracking.md
- Implementation guide: /implementation_guides/position_tracking_fixtures.md

Focus on:
- Position model initialization tests
- Position validation tests
- P&L calculation tests

These tests should verify proper handling of Indian market symbols and lot sizes.
```

## Resulting Implementation

The AI would generate the following implementation:

```python
"""
Tests for the Position model in the Position Tracking component.

This module contains unit tests for the Position model including:
- Initialization
- Validation
- P&L calculation

Tests include Indian market-specific validation including symbol formats and lot sizes.
"""

import pytest
from datetime import datetime, timedelta
from decimal import Decimal
from zoneinfo import ZoneInfo

from src.interfaces.position_tracking.models import Position
from src.interfaces.exceptions import ValidationError

# Constants
IST = ZoneInfo("Asia/Kolkata")


# Fixtures
@pytest.fixture
def sample_position():
    """
    Return a sample valid Position instance.
    """
    return Position(
        symbol="RELIANCE",
        quantity=100,
        average_price=2500.0,
        current_price=2550.0,
        unrealized_pnl=5000.0,
        realized_pnl=0.0,
        last_updated=datetime.now(IST)
    )


@pytest.fixture
def sample_fno_position():
    """
    Return a sample Position instance for F&O.
    """
    return Position(
        symbol="NIFTY23JUN18000CE",
        quantity=50,  # One lot of NIFTY
        average_price=250.0,
        current_price=300.0,
        unrealized_pnl=2500.0,
        realized_pnl=0.0,
        last_updated=datetime.now(IST)
    )


# Test classes
class TestPositionInitialization:
    """Tests for Position initialization."""

    def test_basic_initialization(self, sample_position):
        """Test that position initializes with valid parameters."""
        assert sample_position.symbol == "RELIANCE"
        assert sample_position.quantity == 100
        assert sample_position.average_price == 2500.0
        assert sample_position.current_price == 2550.0
        assert sample_position.unrealized_pnl == 5000.0
        assert sample_position.realized_pnl == 0.0
        assert isinstance(sample_position.last_updated, datetime)
        assert sample_position.last_updated.tzinfo == IST

    def test_fno_initialization(self, sample_fno_position):
        """Test that F&O position initializes with valid parameters."""
        assert sample_fno_position.symbol == "NIFTY23JUN18000CE"
        assert sample_fno_position.quantity == 50
        assert sample_fno_position.average_price == 250.0
        assert sample_fno_position.current_price == 300.0
        assert sample_fno_position.unrealized_pnl == 2500.0

    def test_initialization_with_minimal_parameters(self):
        """Test position initialization with minimal required parameters."""
        position = Position(
            symbol="INFY",
            quantity=10,
            average_price=1800.0,
            current_price=1800.0
        )
        
        assert position.symbol == "INFY"
        assert position.quantity == 10
        assert position.average_price == 1800.0
        assert position.current_price == 1800.0
        assert position.unrealized_pnl == 0.0
        assert position.realized_pnl == 0.0
        assert isinstance(position.last_updated, datetime)
        assert position.last_updated.tzinfo == IST


class TestPositionValidation:
    """Tests for Position validation."""

    def test_invalid_symbol(self):
        """Test position with invalid symbol fails validation."""
        with pytest.raises(ValidationError) as excinfo:
            Position(
                symbol="",  # Empty symbol
                quantity=100,
                average_price=2500.0,
                current_price=2500.0
            )
        assert "Symbol cannot be empty" in str(excinfo.value)

    def test_invalid_quantity(self):
        """Test position with invalid quantity fails validation."""
        with pytest.raises(ValidationError) as excinfo:
            Position(
                symbol="RELIANCE",
                quantity=-10,  # Negative quantity
                average_price=2500.0,
                current_price=2500.0
            )
        assert "Quantity must be positive" in str(excinfo.value)

    def test_invalid_price(self):
        """Test position with invalid price fails validation."""
        with pytest.raises(ValidationError) as excinfo:
            Position(
                symbol="RELIANCE",
                quantity=100,
                average_price=-2500.0,  # Negative price
                current_price=2500.0
            )
        assert "Price must be positive" in str(excinfo.value)

    def test_lot_size_validation_for_fno(self):
        """Test lot size validation for F&O positions."""
        # Valid lot size for NIFTY (50)
        position = Position(
            symbol="NIFTY23JUN18000CE",
            quantity=50,
            average_price=250.0,
            current_price=250.0
        )
        assert position.quantity == 50
        
        # Invalid lot size for NIFTY
        with pytest.raises(ValidationError) as excinfo:
            Position(
                symbol="NIFTY23JUN18000CE",
                quantity=10,  # Not a multiple of 50
                average_price=250.0,
                current_price=250.0
            )
        assert "multiple of lot size" in str(excinfo.value)


class TestPositionPnLCalculation:
    """Tests for Position P&L calculation."""

    def test_calculate_unrealized_pnl(self):
        """Test unrealized P&L calculation."""
        position = Position(
            symbol="RELIANCE",
            quantity=100,
            average_price=2500.0,
            current_price=2550.0,
            unrealized_pnl=0.0  # Will be recalculated
        )
        
        # Call the method to recalculate
        position.calculate_unrealized_pnl()
        
        # Verify the calculation: (current_price - average_price) * quantity
        expected_pnl = (2550.0 - 2500.0) * 100
        assert position.unrealized_pnl == expected_pnl

    def test_calculate_pnl_with_price_update(self):
        """Test P&L recalculation after price update."""
        position = Position(
            symbol="RELIANCE",
            quantity=100,
            average_price=2500.0,
            current_price=2550.0,
            unrealized_pnl=5000.0
        )
        
        # Update the price and recalculate
        position.update_current_price(2600.0)
        
        # Verify the new price and P&L
        assert position.current_price == 2600.0
        expected_pnl = (2600.0 - 2500.0) * 100
        assert position.unrealized_pnl == expected_pnl

    def test_realized_pnl_on_partial_close(self):
        """Test realized P&L calculation on partial position close."""
        position = Position(
            symbol="RELIANCE",
            quantity=100,
            average_price=2500.0,
            current_price=2600.0,
            unrealized_pnl=10000.0,
            realized_pnl=0.0
        )
        
        # Close half the position
        position.close_partial(50, 2600.0)
        
        # Verify the updated position
        assert position.quantity == 50
        assert position.average_price == 2500.0  # Unchanged
        
        # Verify realized P&L: (close_price - average_price) * closed_quantity
        expected_realized_pnl = (2600.0 - 2500.0) * 50
        assert position.realized_pnl == expected_realized_pnl
        
        # Verify updated unrealized P&L: (current_price - average_price) * remaining_quantity
        expected_unrealized_pnl = (2600.0 - 2500.0) * 50
        assert position.unrealized_pnl == expected_unrealized_pnl


# Run the tests
if __name__ == "__main__":
    pytest.main(["-v"])
```

## How This Approach Saves Tokens

1. **Focused Reference**: The implementation request references specific knowledge maps and guides but doesn't repeat their content.

2. **Targeted Focus**: By specifying exactly which aspects to focus on, the AI concentrates on implementing those areas without explaining basic patterns that are already documented elsewhere.

3. **Clear Structure**: The implementation follows the standard structure from the knowledge maps, eliminating the need to describe the structure in the prompt.

4. **No Redundant Explanations**: The implementation includes necessary comments but doesn't explain the overall approach, which is already documented in the knowledge maps.

## Implementation Process

1. **Read Knowledge Map**: The AI first consults the Position Tracking Knowledge Map to understand:
   - Position model properties and methods
   - Validation rules
   - Test patterns and fixtures
   - Indian market-specific considerations

2. **Check Implementation Guide**: Next, it reviews the Position Tracking Fixtures guide to understand:
   - Example fixtures for positions
   - Common test scenarios
   - Indian market validation specifics

3. **Implement Core Tests**: Based on the focus areas, the AI implements:
   - Basic initialization tests
   - Validation tests including Indian market validation
   - P&L calculation tests

4. **Add Market-Specific Tests**: The AI adds Indian market-specific tests for:
   - F&O symbol validation
   - Lot size validation
   - IST timezone handling

This structured approach ensures complete implementation while avoiding token waste on repeating information already available in the knowledge structure.
