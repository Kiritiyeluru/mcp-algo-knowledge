# MCP Algorithmic Trading System - Project Overview

## Core Project Purpose

The MCP Algorithmic Trading System is an algorithmic trading platform specifically designed for Indian markets, supporting NSE equities, futures, and options. It enables systematic trading through well-defined interfaces for market data handling, strategy implementation, order management, position tracking, and risk management.

## Design Philosophy

- **Modularity**: Clean separation between components via interfaces
- **Testability**: Comprehensive test coverage for all components
- **Reliability**: Robust error handling and recovery mechanisms
- **Performance**: Optimized for low-latency trading operations
- **Flexibility**: Support for various trading strategies and instruments

## Key Components & Relationships

1. **Market Data** → Provides data to → **Strategy**
2. **Strategy** → Generates signals for → **Order Management**
3. **Order Management** → Updates → **Position Tracking**
4. **Position Tracking** → Informs → **Risk Management**
5. **Risk Management** → Constrains → **Strategy** & **Order Management**
6. **Monitoring** → Observes all components for health and performance

## Directory Structure

```
mcp_algo/
├── src/interfaces/
│   ├── __init__.py
│   ├── base.py - Base component interfaces
│   ├── exceptions.py - Exception hierarchy
│   ├── validation.py - Core validation framework
│   ├── market_data/ - FULLY IMPLEMENTED
│   ├── strategy/ - FULLY IMPLEMENTED
│   ├── order_management/ - FULLY IMPLEMENTED
│   ├── position_tracking/ - FULLY IMPLEMENTED
│   ├── risk_management/ - FULLY IMPLEMENTED
│   ├── integration/ - PARTIALLY IMPLEMENTED
│   ├── monitoring/ - FULLY IMPLEMENTED
│   ├── config/ - FULLY IMPLEMENTED
│   ├── recovery/ - FULLY IMPLEMENTED
│   ├── session/ - PARTIALLY IMPLEMENTED
│   └── websocket/ - FULLY IMPLEMENTED
└── tests/interfaces/
    ├── conftest.py - Common test fixtures
    ├── test_base.py - Base tests
    ├── test_integration.py - Global integration tests
    ├── market_data/ - FULLY IMPLEMENTED
    ├── strategy/ - PARTIALLY IMPLEMENTED
    └── other component test dirs - NOT IMPLEMENTED
```

## Indian Market Specifics

- Support for Indian exchanges (NSE, BSE, MCX)
- F&O (Futures & Options) specific functionality
- Support for Indian market order types (MIS, CNC, NRML)
- Circuit limits and market hour constraints
- Market timing (9:15 AM - 3:30 PM IST)

## Implementation Priorities

1. Core interface definitions with proper type hints
2. Market data and strategy interfaces (most foundational)
3. Order management and position tracking interfaces
4. Risk management and monitoring components
5. Integration tests across components
