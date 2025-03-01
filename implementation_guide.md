# MCP Algorithmic Trading System - Implementation Guide

## Knowledge Map Approach

The repository uses a knowledge map system to minimize token usage and provide structured references instead of reviewing the entire codebase for each task.

## Implementation Process

1. **Reference knowledge maps** instead of reviewing entire codebase
2. **Implement one file per response** to avoid token limits
3. **Use GitHub functions directly** (do not paste code in chat)
4. **Focus on the specific task** without summarizing already completed work

## Implementation Request Templates

### For Test Files:
```
Please implement /tests/interfaces/[component]/[file].py.
Reference:
- Knowledge map at /components/[component].md
- Common patterns at /common_patterns.md

Focus on these test cases:
- [test_case_1]
- [test_case_2]
- [test_case_3]
```

### For Interface Files:
```
Please implement /src/interfaces/[component]/[file].py.
Reference:
- Knowledge map at /components/[component].md
- Common patterns at /common_patterns.md

Focus on these methods:
- [method_1]
- [method_2]
- [method_3]
```

### For Model Files:
```
Please implement /src/interfaces/[component]/models.py.
Reference:
- Knowledge map at /components/[component].md
- Common patterns at /common_patterns.md

Focus on these models:
- [model_1]
- [model_2]
- [model_3]
```

## Universal Implementation Requirements

When implementing any file in the MCP Algorithmic Trading System, adhere to these comprehensive requirements:

### 1. Model & Interface Alignment
- Always compare implementation against ALL relevant model files
- Ensure consistency between model fields and implementation
- Follow established naming conventions across files

### 2. Comprehensive Error Handling
- Implement error cases for EVERY method to test validation failures
- Include edge cases for all validators
- Test both expected and unexpected error paths
- Use appropriate exception types with detailed context

### 3. Indian Market Specifics
- Include fixtures/test cases for:
  - F&O instruments with their special formats
  - Market timing constraints (9:15 AM - 3:30 PM IST)
  - Circuit limits and price boundaries
  - Product types specific to Indian brokers (MIS, CNC, NRML)
  - Exchange-specific rules (NSE, BSE)

### 4. Realistic Test Data
- Use timezone-aware datetime objects in IST timezone
- Implement realistic price movements and trading scenarios
- Include both intraday and delivery scenarios
- Simulate real market conditions with appropriate price increments

### 5. Thorough Mock Implementations
- All mocks should record method calls for verification
- Implement realistic business logic in mocks
- Handle edge cases (zero division, None values, empty collections)
- Include both success and failure paths

### 6. Type Safety & Documentation
- Use complete type annotations for all parameters and return values
- Provide Google-style docstrings for all fixtures and methods
- Document parameters, return values, and raised exceptions
- Add implementation notes for complex logic

## Code Quality Standards

- **File Length**: Split files exceeding 500 lines into multiple modules
- **Docstrings**: Use Google-style docstrings with complete parameter descriptions
- **Type Hints**: Include full type hints for all parameters and return values
- **Error Handling**: Implement proper exception handling with specific error types
- **Naming**: Follow existing naming conventions from implemented components
