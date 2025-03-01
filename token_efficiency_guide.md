# Token Efficiency Guide

This guide explains how to optimize token usage when implementing the MCP Algorithmic Trading System, ensuring you get the most out of AI assistance without wasting tokens on redundant information.

## Understanding Token Usage

Tokens are the units of text processing used by AI assistants. Each token is roughly 4 characters or 3/4 of a word in English. When implementing complex systems, token efficiency is crucial because:

1. **Limited Context Window**: There's a maximum number of tokens that can be processed in a single conversation
2. **Processing Time**: More tokens means longer processing time
3. **Cost Efficiency**: Token usage directly impacts usage costs

## How to Formulate Implementation Requests Efficiently

### Do's:
- **Be specific about the file you need implemented**
  ```
  Please implement /tests/interfaces/position_tracking/test_models.py
  ```

- **Reference knowledge maps instead of pasting their content**
  ```
  Please refer to the Position Tracking Knowledge Map at /components/position_tracking.md
  ```

- **Focus on unique requirements only**
  ```
  This implementation should focus specifically on P&L calculation methods
  ```

### Don'ts:
- **Don't paste entire files for reference**
- **Don't include boilerplate code that's already documented elsewhere**
- **Don't repeat implementation patterns in each request**

## When to Use Knowledge Maps vs. Implementation Guides

### Knowledge Maps
Use knowledge maps when you need:
- **Structure information**: Interface definitions, method signatures
- **Quick reference**: Validation rules, component relationships
- **Overview**: Understanding how components fit together

Example request:
```
Please implement the PositionTracker interface based on the knowledge map at /components/position_tracking.md
```

### Implementation Guides
Use implementation guides when you need:
- **Concrete examples**: Specific implementation patterns with code
- **Edge cases**: Handling of special scenarios
- **Complex logic**: Detailed algorithmic implementations

Example request:
```
Please implement the P&L calculation method following the pattern in /implementation_guides/position_tracking_fixtures.md
```

## Patterns for Incremental Implementation

1. **Implement Core Interfaces First**
   ```
   Please implement the base PositionTrackerBase interface
   ```

2. **Add Model Implementations Next**
   ```
   Now implement the Position model referenced in the interface
   ```

3. **Add Validation Logic After**
   ```
   Now implement the validation methods for the Position model
   ```

4. **Implement Tests Last**
   ```
   Finally, implement tests for the Position model
   ```

## Referencing Specific Sections Without Pasting Large Blocks

### Using Section Anchors
When you need to refer to a specific section of a knowledge map or guide, use its heading as an anchor:

```
Please implement the validation logic as described in the Position Tracking Knowledge Map under "Validation Rules"
```

### Using Example References
When you need to reference a specific example:

```
Please follow the pattern shown in the Mock Implementation Patterns guide under "Advanced Mock Implementation Examples"
```

## Template Implementation Requests

### For Interface Implementation
```
Please implement /src/interfaces/[component]/[file].py

Reference knowledge map: /components/[component].md

Focus on these methods:
- method1
- method2
```

### For Test Implementation
```
Please implement /tests/interfaces/[component]/[file].py

Reference knowledge map: /components/[component].md
Reference implementation guide: /implementation_guides/[relevant_guide].md

Focus on these test cases:
- test_case_1
- test_case_2
```

## Conclusion

By following these token efficiency guidelines, you can maximize the effectiveness of AI assistance while implementing the MCP Algorithmic Trading System. The key principle is to reference rather than repeat, and to focus each implementation request on a specific task with clear references to the relevant knowledge sources.
