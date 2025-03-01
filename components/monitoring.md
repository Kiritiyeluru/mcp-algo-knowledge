# Monitoring Knowledge Map

This document details the monitoring component of the MCP Algorithmic Trading System, responsible for system health tracking, error detection, and performance monitoring.

## 1️⃣ Interfaces

### MonitoringSystemBase

- **File Path**: `/src/interfaces/monitoring/base.py`
- **Core Methods**:
  - `monitor_system() -> SystemStatus`: Checks system health (latency, errors, resources)
  - `send_alert(alert: Alert) -> None`: Dispatches alerts for threshold breaches
  - `get_metrics() -> Metrics`: Retrieves current performance metrics
  - `log_event(event: MonitoringEvent) -> None`: Records significant system events
  - `trigger_remediation(action: str) -> bool`: Initiates automated recovery

## 2️⃣ Models

### Alert
- **File Path**: `/src/interfaces/monitoring/models.py`
- **Properties**:
  - `alert_id: str`: Unique alert identifier
  - `severity: AlertSeverity`: Urgency level (INFO, WARNING, CRITICAL)
  - `message: str`: Detailed issue description
  - `timestamp: datetime`: Alert generation time
  - `component: str`: Affected system component

### SystemStatus
- **File Path**: `/src/interfaces/monitoring/models.py`
- **Properties**:
  - `is_operational: bool`: Overall system status
  - `uptime: float`: Total uptime in hours
  - `error_count: int`: Error count in monitoring window
  - `latency: float`: Average system latency (ms)
  - `resource_usage: Dict[str, Any]`: CPU/memory/network usage

### Metrics
- **File Path**: `/src/interfaces/monitoring/models.py`
- **Properties**:
  - `cpu_usage: float`: CPU usage percentage
  - `memory_usage: float`: Memory usage percentage
  - `network_latency: float`: Network latency
  - `error_rate: float`: Errors per minute

## 3️⃣ Validation Rules

- **Threshold Alerts**: Trigger alerts when metrics exceed thresholds (CPU > 85%, latency > 500ms)
- **Error Logging**: Log all errors with timestamp and component reference
- **Data Frequency**: Update metrics at regular intervals (every 30 seconds)
- **Alert Severity**: Categorize alerts by severity with appropriate escalation
- **Remediation Verification**: Verify system recovery after automated remediation

## 4️⃣ Test Patterns

### Test Classes
- `TestMonitoringSystem`: System monitoring functionality
- `TestAlertDispatcher`: Alert dispatch mechanism
- `TestMetricCollection`: Performance metrics collection

### Standard Test Methods
- `test_monitor_system_operational()`: Verify operational status reporting
- `test_send_alert_on_threshold_exceedance()`: Test alert triggering
- `test_log_event_integrity()`: Verify event logging accuracy
- `test_trigger_remediation()`: Validate remediation actions
- `test_metric_accuracy()`: Check metric collection accuracy

## 5️⃣ Common Fixtures

### `sample_alert()`
- **Returns**: Alert object for testing
- **Parameters**:
  - `alert_id: str = "ALRT001"`: Example identifier
  - `severity: AlertSeverity = AlertSeverity.CRITICAL`: Severity level
  - `message: str = "Simulated system failure"`: Example message
  - `component: str = "monitoring"`: System component

### `mock_monitoring_system()`
- **Returns**: Mock implementation of MonitoringSystemBase

## 6️⃣ Test Data Sets

### Monitoring Metrics
- **CPU Usage**: 20% to 90% simulated values
- **Memory Usage**: 30% to 95% values
- **Network Latency**: 100ms to 800ms simulated values
- **Error Rates**: 0 to 10 errors per minute

### Scenario Examples
- CPU spike to 92% triggering WARNING/CRITICAL alert
- System downtime simulation for remediation testing
- Consistent metric reporting verification

## 7️⃣ Market-Specific Features

- **Trading Hours Sensitivity**: Adjusted thresholds during market hours (9:15 AM - 3:30 PM IST)
- **Circuit Breaker Awareness**: Recognition of market circuit breakers
- **Alert Enrichment**: Indian market-specific metadata in alerts
- **Automated Remediation**: Service restart capability for critical issues
- **Performance Tracking**: Higher sensitivity during market open/close periods
