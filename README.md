# Databricks Genie API Load Testing

A comprehensive Databricks notebook for load testing the Genie Space API with built-in best practices, including exponential backoff, retry logic, jitter, and MLflow tracking.

## Features

- **Exponential Backoff with Jitter**: Prevents thundering herd problems during rate limiting
- **Automatic Retry Logic**: Handles 429 rate limit errors gracefully with configurable retries
- **MLflow Integration**: Complete tracking of all API calls, metrics, and results
- **Concurrent Request Handling**: Thread-based concurrency for realistic load testing
- **Multiple Test Scenarios**: Pre-configured test cases from single questions to high-load scenarios
- **Detailed Analytics**: Comprehensive reporting with percentiles, success rates, and throughput metrics

## Prerequisites

- Databricks workspace with access to Genie Space API
- MLflow enabled in your workspace
- API token with appropriate permissions
- Python 3.8+

## Installation

1. Clone or download this repository to your Databricks workspace

2. Install dependencies in your Databricks cluster or notebook:

```python
%pip install -r requirements.txt
```

## Configuration

### Step 1: API Connection Configuration (Section 1)

Update the following configuration variables:

```python
# Genie API Configuration
WORKSPACE_URL = "https://your-workspace.cloud.databricks.com"  # Your Databricks workspace URL
GENIE_SPACE_ID = "your-genie-space-id"  # Your Genie Space ID
API_TOKEN = dbutils.secrets.get(scope="your-scope", key="your-key")  # Your API token

# MLflow Configuration
MLFLOW_EXPERIMENT_NAME = "/Shared/genie-load-test"  # Your MLflow experiment path
```

**How to Get Your Configuration Values:**

1. **WORKSPACE_URL**: Your Databricks workspace URL (e.g., `https://your-org.cloud.databricks.com`)
2. **GENIE_SPACE_ID**: Find this in the Genie Space URL or via the API
3. **API_TOKEN**: Store in Databricks secrets for security:
   ```python
   # Create a secret scope if you haven't already
   dbutils.secrets.help()
   ```
4. **MLFLOW_EXPERIMENT_NAME**: Path where MLflow experiments will be logged
   - Examples: `/Shared/genie-load-test`, `/Users/your.email@company.com/experiments`, `/Teams/engineering/genie-tests`

### Step 2: Load Test Scenario Configuration (Section 2)

The notebook includes **7 predefined load test scenarios**. Select one by changing the `ACTIVE_SCENARIO` variable:

```python
ACTIVE_SCENARIO = "high_load"  # Change this to switch scenarios
```

**Available Scenarios:**

| Scenario | Questions | Duration | Workers | Description |
|----------|-----------|----------|---------|-------------|
| `single_test` | 1 | Immediate | 1 | Single question validation |
| `light_load` | 10 | 60s | 3 | Light load test |
| `moderate_load` | 20 | 30s | 5 | Moderate load test |
| `high_load` | 35 | 30s | 10 | **High load - 35 questions in 30 seconds** |
| `stress_test` | 50 | 20s | 15 | Stress test |
| `burst_test` | 50 | Immediate | 20 | Burst test (all at once) |
| `sustained_load` | 100 | 120s | 10 | Sustained load test |

### Step 3: Custom Scenario (Optional)

Create your own custom scenario by setting `USE_CUSTOM_SCENARIO = True` and modifying:

```python
CUSTOM_SCENARIO = {
    "num_questions": 35,        # Total number of questions
    "target_duration": 30,      # Duration in seconds (None = immediate)
    "max_workers": 10,          # Concurrent workers
    "description": "My custom load test"
}
USE_CUSTOM_SCENARIO = True  # Set to True to use custom scenario
```

### Advanced Configuration

You can adjust retry behavior and timeouts:

```python
MAX_RETRIES = 5              # Maximum number of retry attempts
BASE_WAIT_TIME = 1          # Initial wait time in seconds
MAX_WAIT_TIME = 60          # Maximum wait time between retries
JITTER_MAX = 2              # Maximum random jitter in seconds
TIMEOUT = 300               # Maximum time to wait for a response
```

## Usage

### Quick Start

1. Open `genie_api_load_test.ipynb` in Databricks
2. Update the API connection configuration in **Section 1**
3. Select your desired load test scenario in **Section 2** by changing `ACTIVE_SCENARIO`
4. Run all cells - the notebook will use your selected scenario automatically

### Example: Running Different Load Tests

**To run 35 questions in 30 seconds (high load):**
```python
ACTIVE_SCENARIO = "high_load"  # In Section 2
```

**To run a burst test (50 questions immediately):**
```python
ACTIVE_SCENARIO = "burst_test"  # In Section 2
```

**To create a custom test:**
```python
CUSTOM_SCENARIO = {
    "num_questions": 100,
    "target_duration": 60,  # 100 questions over 1 minute
    "max_workers": 20,
    "description": "Heavy load test"
}
USE_CUSTOM_SCENARIO = True  # In Section 2
```

### How It Works

1. **Section 2** - Select or define your load test scenario
2. **Section 8** - Run the test (automatically uses your configured scenario)
3. **Section 9** - Analyze results with detailed statistics
4. **Section 10** - (Optional) Run additional ad-hoc tests

The configuration is **centralized and easy to change** - just modify the values in Section 2!

## Architecture

### Components

#### 1. GenieAPIClient
Core client implementing:
- HTTP request handling with authentication
- Exponential backoff with jitter
- Automatic retry for 429 errors (using tenacity library)
- Conversation management
- Query result polling

#### 2. LoadTestOrchestrator
Orchestration layer providing:
- Concurrent test execution
- MLflow experiment tracking
- Results aggregation
- Performance metrics calculation

#### 3. MLflow Integration
Tracks the following for each test:
- **Parameters**: Question text, timestamps, conversation IDs
- **Metrics**: Duration, success/failure, row counts, throughput
- **Artifacts**: Full API responses, results CSV
- **Tags**: Status, test names

### Retry Logic

The client implements sophisticated retry logic:

1. **Detection**: Catches HTTP 429 (rate limit) errors
2. **Wait Calculation**: 
   - Respects `Retry-After` header if present
   - Falls back to exponential backoff: `wait_time = base * (2 ^ attempt)`
   - Adds random jitter (0-2 seconds) to prevent thundering herd
3. **Max Retries**: Fails after 5 attempts (configurable)
4. **Logging**: All retries are logged with timestamps

### Concurrency Model

- Uses Python's `ThreadPoolExecutor` for concurrent requests
- Configurable worker pool size (max_workers)
- Optional request spreading over time (target_duration)
- Thread-safe MLflow logging with nested runs

## MLflow Tracking

### Viewing Results

1. Navigate to your Databricks workspace
2. Go to **Experiments** in the left sidebar
3. Find the experiment with the name you configured in `MLFLOW_EXPERIMENT_NAME` (default: `/Shared/genie-load-test`)
4. Click on runs to view details

### Metrics Available

**Parent Run (Test Level)**:
- `num_questions`: Total questions in test
- `total_duration_seconds`: Total test duration
- `successful_questions`: Count of successful queries
- `failed_questions`: Count of failed queries
- `success_rate`: Percentage of successful queries
- `avg_question_duration`: Average time per question
- `throughput_qps`: Questions per second

**Child Runs (Question Level)**:
- `duration_seconds`: Time taken for this question
- `success`: 1 for success, 0 for failure
- `row_count`: Number of rows returned (if applicable)
- Full API response saved as artifact

## Best Practices

### 1. Rate Limiting
- The client automatically handles 429 errors
- Jitter prevents multiple clients from retrying simultaneously
- Adjust `MAX_RETRIES` and `MAX_WAIT_TIME` based on your SLA

### 2. Load Testing
- Start with small tests (Test Case 1) to validate configuration
- Gradually increase load (Test Case 2 → 3 → 4)
- Monitor MLflow metrics to identify bottlenecks
- Use `target_duration` to spread load realistically

### 3. Concurrency
- Don't set `max_workers` too high initially
- Recommended starting point: 5-10 workers
- Increase based on your API quota and requirements

### 4. Error Handling
- All errors are logged to MLflow for analysis
- Check failed runs in MLflow to identify patterns
- Common errors: timeouts, authentication, malformed queries

### 5. Security
- Always use Databricks secrets for API tokens
- Never hardcode credentials in notebooks
- Use workspace-level secrets for team access

## Troubleshooting

### Common Issues

**Issue**: Authentication errors (401)
- **Solution**: Verify API token is correct and has proper permissions
- Check token is properly stored in Databricks secrets

**Issue**: Rate limiting (429) even with retries
- **Solution**: Reduce `max_workers` or increase `target_duration`
- Contact Databricks support to increase API quota

**Issue**: Timeouts
- **Solution**: Increase `TIMEOUT` value
- Check if queries are too complex
- Consider breaking down complex queries

**Issue**: MLflow tracking errors
- **Solution**: Ensure MLflow is enabled in your workspace
- Verify experiment path permissions
- Check workspace storage quota

**Issue**: Connection errors
- **Solution**: Verify `WORKSPACE_URL` is correct
- Check network connectivity
- Ensure Genie Space ID is valid

## Switching Between Load Test Scenarios

The notebook makes it **extremely easy** to switch between different load test configurations:

### Method 1: Use Predefined Scenarios

Simply change one line in **Section 2**:

```python
# Change from high_load to any other scenario
ACTIVE_SCENARIO = "stress_test"  # Now running stress test instead
```

Re-run the notebook and it automatically uses the new configuration!

### Method 2: Create Custom Scenario

Define your own parameters:

```python
CUSTOM_SCENARIO = {
    "num_questions": 75,        # Your desired number
    "target_duration": 45,      # Your desired duration
    "max_workers": 12,          # Your desired concurrency
    "description": "Custom test description"
}
USE_CUSTOM_SCENARIO = True
```

### Method 3: Ad-hoc Testing

Use **Section 10** for quick one-off tests without changing the main configuration:

```python
CUSTOM_NUM_QUESTIONS = 20
CUSTOM_TARGET_DURATION = 60
CUSTOM_MAX_WORKERS = 5
# Run without affecting your main test configuration
```

## Performance Tips

1. **Optimize Questions**: Simpler questions return faster
2. **Use Caching**: Repeated questions may be cached by Genie
3. **Monitor Resources**: Check cluster resources during tests
4. **Schedule Tests**: Run during off-peak hours for accurate baselines
5. **Analyze Patterns**: Use MLflow metrics to identify slow queries
6. **Start Small**: Begin with `light_load` scenario, then gradually increase

## Sample Output

```
============================================================
Load Test Summary: high_load_35q_30s
============================================================
Total Questions: 35
Successful: 33
Failed: 2
Success Rate: 94.29%
Total Duration: 45.23s
Avg Question Duration: 8.15s
Throughput: 0.77 QPS
============================================================
```

## Dependencies

See `requirements.txt` for full list:
- `mlflow>=2.9.0`: Experiment tracking
- `requests>=2.31.0`: HTTP client
- `tenacity>=8.2.0`: Retry logic
- `pandas>=2.0.0`: Data analysis
- `numpy>=1.24.0`: Numerical operations

## References

- [Databricks Genie API Documentation](https://docs.databricks.com/en/genie/index.html)
- [MLflow Documentation](https://mlflow.org/docs/latest/index.html)
- [Reference Implementation](https://github.com/sean-zhang-dbx/genie-api-best-practices-pub/)

## Contributing

This is a reference implementation. Feel free to customize based on your needs:
- Add custom metrics
- Implement different load patterns
- Extend error handling
- Add visualization capabilities

## License

MIT License - feel free to use and modify as needed.

## Support

For issues related to:
- **Databricks Genie API**: Contact Databricks support
- **This notebook**: Open an issue in your repository
- **MLflow**: Check MLflow documentation

## Version History

- **v1.0** (Initial Release)
  - Exponential backoff with jitter
  - MLflow integration
  - 4 test scenarios
  - Comprehensive documentation

## Acknowledgments

Based on best practices from:
- Databricks Genie team
- MLflow community
- Reference: https://github.com/sean-zhang-dbx/genie-api-best-practices-pub/

