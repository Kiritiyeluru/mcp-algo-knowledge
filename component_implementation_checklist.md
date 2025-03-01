# Component Implementation Checklist

This standardized checklist helps ensure complete and consistent implementation of components in the MCP Algorithmic Trading System while minimizing token usage. Follow this checklist when implementing any component to ensure you don't miss critical aspects.

## Initial Planning
- [ ] Identify the component to implement
- [ ] Review the relevant knowledge map at `/components/<component_name>.md`
- [ ] Note specific Indian market requirements that apply

## Core Interface Implementation

### Base Interface Classes
- [ ] Implement abstract base classes
- [ ] Include proper type hints for all methods
- [ ] Add comprehensive Google-style docstrings
- [ ] Implement proper validation methods
- [ ] Ensure all required methods are defined

### Interface Method Implementation Checklist
For each method in the interface:
- [ ] Check parameter types
- [ ] Validate input parameters 
- [ ] Implement error handling
- [ ] Return appropriate values/exceptions
- [ ] Document edge cases in comments

## Model Implementation

### For Each Model
- [ ] Define all required properties
- [ ] Implement validation logic
- [ ] Implement to_dict() and from_dict() methods
- [ ] Add property getters/setters if needed
- [ ] Implement equality comparison if needed

### Common Model Requirements
- [ ] Ensure proper inheritance hierarchy
- [ ] Implement proper serialization/deserialization
- [ ] Handle timezone-aware timestamps (IST)
- [ ] Add representation methods (__str__, __repr__)

## Validation Implementation

- [ ] Implement symbol validation for Indian markets
- [ ] Implement price/circuit validation
- [ ] Implement quantity/lot size validation
- [ ] Add F&O-specific validation if applicable
- [ ] Implement product type validation

## Test Implementation

### Test Setup
- [ ] Create test fixtures in `conftest.py`
- [ ] Implement mock objects as needed
- [ ] Define test data constants
- [ ] Set up common test utilities

### Test Categories to Implement
- [ ] Basic initialization tests
- [ ] Validation logic tests
- [ ] Method behavior tests
- [ ] Edge case tests
- [ ] Exception handling tests
- [ ] Integration tests with other components

### Test Structure
- [ ] Organize tests by functionality
- [ ] Use descriptive test names
- [ ] Include assertions that test expected behavior
- [ ] Add test documentation describing the test purpose

## Indian Market-Specific Considerations

- [ ] Implement proper symbol format validation (NSE/BSE)
- [ ] Handle F&O symbol formats correctly
- [ ] Implement circuit limit validation
- [ ] Add lot size validation for F&O
- [ ] Implement IST timezone handling
- [ ] Add market timing validation

## Documentation and Cleanup

- [ ] Ensure all public methods have docstrings
- [ ] Add module-level docstrings
- [ ] Document any non-obvious code with inline comments
- [ ] Ensure consistent code formatting
- [ ] Remove any debug print statements

## Implementation Request Guidelines

When requesting component implementation, use the following format to minimize token usage:

```
Please implement [file_path]

Reference:
- Knowledge map: [component_knowledge_map_path]
- Implementation guide: [relevant_guide_path]

Focus on:
- [specific aspect to focus on]
- [another specific aspect]

This component should [specific requirement or constraint].
```

Examples:

```
Please implement /src/interfaces/position_tracking/base.py

Reference:
- Knowledge map: /components/position_tracking.md
- Implementation guide: /implementation_guides/mock_implementation_patterns.md

Focus on:
- PositionTrackerBase interface methods
- Position lifecycle management

This component should handle Indian F&O position tracking with proper lot size calculations.
```

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

By following this structured approach and referring to specific knowledge resources rather than including their content, you can achieve complete implementations while maintaining optimal token efficiency.
